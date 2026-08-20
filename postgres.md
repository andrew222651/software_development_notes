# Postgres

Suzuki, H. _Internals of PostgreSQL_.: https://www.interdb.jp/pg/index.html
* xmax is described as ["first as-yet-unassigned txid"](https://www.interdb.jp/pg/pgsql05.html#:~:text=First%20as%2Dyet%2Dunassigned%20txid)
  when it's actually 1 + the last inactive txid according to later text

Postgres-compatible tools adding sharding, failover, backups, zero-downtime
patching, etc:
* CockroachDB: distributed database
  * ["guarantees serializable SQL transactions"](https://www.cockroachlabs.com/docs/v23.2/frequently-asked-questions#how-is-cockroachdb-strongly-consistent)
    and "this is the only isolation level we offer"
  * [consistency model](https://www.cockroachlabs.com/blog/consistency-model/#cockroachdbs-consistency-model-more-than-serializable-less-than-strict-serializability):
    * more than serializable:
      * if tx1 occurs before tx2, tx2 will see read writes from tx1
    * less than strict serializable:
      * the order of two transactions that do writes may be swapped
  * [nodes](https://www.cockroachlabs.com/docs/stable/architecture/reads-and-writes-overview):
    data is partitioned into ranges, which are stored on replicas. one replica
    per range ("leaseholder") serves all consistent reads.
  * [faster but inconsistent reads may be requested](https://www.cockroachlabs.com/blog/follower-reads-stale-data/)
* [AWS Aurora](https://aws.amazon.com/rds/aurora/)
* [YugabyteDB](https://www.yugabyte.com/): distributed database
* [Citus](https://github.com/citusdata/citus): sharding
  * ["not ACID but eventually consistent"](https://dev.to/yugabyte/citus-is-not-acid-but-eventually-consistent-3711)
* replication for high availability
  * independent of sharding
  * https://github.com/sorintlab/stolon
  * [Patroni](https://github.com/zalando/patroni)


Postgres-backed task queues:
* [PgQue](https://github.com/NikolayS/pgque)
* [Procrastinate](https://procrastinate.readthedocs.io/en/stable/)
* [Hatchet](https://github.com/hatchet-dev/hatchet)
* [apscheduler](https://github.com/agronholm/apscheduler)
* [Oban](https://github.com/oban-bg/oban-py)


In Postgres, the following will cause the error "[27000] ERROR: tuple to be updated was already modified by an operation triggered by the current command"; whereas if the trigger is `after update` the table will have one row with `n=2`:
```
create table t (
  n int
);

create function fn() returns trigger as $$
begin
update t set n = 2;
return new;
end;
$$ language plpgsql;

create trigger trg before update on t
  for each row
  when (NEW.n = 1)
  execute procedure fn();

insert into t (n) values (3);
update t set n = 1;
```

postgres throws an error if you try to put a null character in a text
field ("\x00" in Python). this is a non-printable ascii character.

Connection-scoped text variables:
* `SELECT set_config('session.user_id', '51', false)`
* `SELECT current_setting('session.user_id')`


## SQL

type checker <https://github.com/okbob/plpgsql_check>

compositionality:
* views, "with" clauses https://stackoverflow.com/questions/10674761/difference-between-sql-view-and-with-clause
* table functions https://wiki.postgresql.org/wiki/Inlining_of_SQL_functions#Table_functions
* plpgsql functions https://stackoverflow.com/a/24771561/371334

group by x, within group take row that maximizes y
https://stackoverflow.com/questions/7745609/sql-select-only-rows-with-max-value-on-a-column
or https://dba.stackexchange.com/a/159899/249985

`exclude` constraints generalize `unique` constraints and can e.g. ensure that
ranges don't overlap. indexes are used for the same reason they are with
`unique`

In postgres, `select pg_typeof('foo');` gives `unknown` but
`select 'foo' \gdesc` gives `text`

Postgres has a limit on the total length of a query, but there are other limits
that may be hit first:
* the query `SELECT $1::INTEGER + $2::INTEGER + $1::INTEGER + $2::INTEGER + ...` can cause `stack depth limit exceeded`
* the query `SELECT * FROM USER WHERE id IN ($1::UUID, $2::UUID, $3::UUID, $4::UUID, ...)` can cause an error in clients since [the maximum number of arguments is 32767](https://github.com/MagicStack/asyncpg/issues/127)


## Performance

mysql automatically creates indexes for all primary and foreign keys, and
uniqueness constraints (which it uses to speed up uniqueness enforcement)
<https://dev.mysql.com/doc/refman/8.0/en/constraint-foreign-key.html>.
Postgres doesn't create indexes for foreign keys.

postgres text searching https://dba.stackexchange.com/a/10696/249985
* equality: default index (BST) is fine
* inequalities/prefixes e.g. `LIKE 'blah%'`: if the `text` column has no
  collation set, run `SHOW LC_COLLATE` to see your default collation.
  If the collation is POSIX, you can use the default BST index. Otherwise,
  do
```
create index <name> on <table> (<col> collate "<collation>" text_pattern_ops);
```
* general regexes: https://www.postgresql.org/docs/15/pgtrgm.html
* (basic) natural language search: https://www.postgresql.org/docs/current/textsearch-intro.html
* more full text search features: <https://github.com/paradedb/paradedb>


## Concurrency control features

if clients can acquire multiple locks, they have to be acquired in the same
order. this includes row locks in a single query: https://dba.stackexchange.com/a/257229/249985

`select` with `for update` holds the lock until the end of the _outermost_
transaction https://www.postgresql.org/message-id/1154639749.24639.22.camel%40dogma.v10.wvs

by default, subqueries of an SQL statement may be evaluated against different
snapshots of the db from the parent https://stackoverflow.com/a/64070103/371334

the docs here https://www.postgresql.org/docs/current/transaction-iso.html#XACT-READ-COMMITTED
imply that UPDATE, DELETE, and SELECT ... FOR UPDATE work in two phases, one to
find the rows that match their where clause, then another to operate on each
found row which involves waiting to get a lock, updating/selecting/deleting
the row (if the where clause still matches [what if the where clause
depends on a column from a joined table that's not being locked?]),
then keeping the lock for the rest
of the transaction. if while waiting for the lock a row is added that matches
the `where` clause, you won't update/delete/select the new row.

in postgres, inserting a row in table A with a foreign key reference to a row in table B
involves acquiring a `FOR KEY SHARE` lock on the row in table B. (see <https://www.cybertec-postgresql.com/en/row-locks-in-postgresql/>.)
use `SELECT ... FOR NO KEY UPDATE` if you don't want to conflict with this lock.

In PostgreSQL, if we do these concurrently,
```sql
begin;
insert into table1 (id) values (1);
insert into table1 (id) values (2);
commit;
```
```sql
begin;
select id from table 1;
commit;
```
is it possible that the `select` will get only 1 row? No -- it only sees changes
that were made by transactions that are committed by the time it starts. see
<https://www.interdb.jp/pg/pgsql05.html>. However, if you're inserting/updating,
changes in other in-progress transactions can be visible in a sense:
e.g. a unique constraint may cause you to wait
(<https://rcoh.me/posts/postgres-unique-constraints-deadlock/#why-does-this-happen>).

if transaction A does select for update on a row, then transaction B requests an
exclusive table lock, then transaction C does select for update on a different
row, when transaction A completes, transaction B will get the lock, then when it
completes transaction C will get its lock. Even though when transaction C
requests the lock, there is no lock on that row or the table yet, it waits
because transaction is already waiting for a table lock (try it).
https://dba.stackexchange.com/questions/49596/in-what-order-does-postgresql-handle-queries-which-have-conflicting-locks

I _think_ table locks only conflict with table locks, and row locks only
conflict with row locks (but row operations may do both a table and a row
lock)

Why wouldn't we use `REPEATABLE READ` by default? Well for example,
updating a row in a `REPEATABLE READ` transaction that has been
updated since the transaction started causes an error: <https://www.interdb.jp/pg/pgsql05/08.html#:~:text=(11)-,ABORT,-this%20transaction%20%20/*%20First>

Serializable get-or-create: <https://stackoverflow.com/a/78195304/371334>

An example of SERIALIZABLE isolation level being overzealous follows.
According to [this discussion](https://www.postgresql.org/message-id/1434461532.3040.1%40smtp.gmail.com) this may be a bug.
Removing the foreign key constraint or setting it to `deferrable initially deferred` removes the error.
```sql
-- schema
create table parent (
  id int primary key
);
create table child (
  id int primary key,
  parent_id int references parent(id)
);

-- tx A
begin;
set transaction isolation level serializable;
insert into parent values (1);
insert into child values (1, 1);

-- tx B
begin;
set transaction isolation level serializable;
insert into parent values (2);
insert into child values (2, 2);

-- tx A
-- if we just did `insert into child values (3, 1)` here that would be ok
insert into parent values (3);
insert into child values (3, 3);

-- commit each transaction in either order
-- ERROR! "could not serialize access due to read/write dependencies among
-- transactions"
```
Another example follows. If we use the primary key in the `where` clause instead of the other column, there's no error.
```sql
-- schema
create table t (
  id int primary key,
  n int
);

-- tx A
begin;
set transaction isolation level serializable;
insert into t values (1, 1);

-- tx B
begin;
set transaction isolation level serializable;
insert into t values (2, 2);

-- tx A
select * from t where n = 1;

-- tx B
select * from t where n = 2;

-- commit in either order
-- ERROR
```

acquiring locks in a global order: given a pgsql function, maybe we could
extract the locks acquired using a sequence of `explain (format json)` calls.
we'd still have to inspect application code though. maybe an LLM checker could
work.


tricky deadlock scenario:
```sql
-- setup

CREATE TABLE parent (
    id integer PRIMARY KEY
);

CREATE TABLE child (
    id integer PRIMARY KEY,
    parent_id integer NOT NULL REFERENCES parent(id) ON DELETE CASCADE,
    payload integer NOT NULL DEFAULT 0
);

INSERT INTO parent (id) VALUES (1);
INSERT INTO child (id, parent_id, payload) VALUES (1, 1, 0);


-- tx A
UPDATE child SET payload = payload + 1 WHERE id = 1;

/*
the child row is locked. because `parent_id` is not changed AND
the row was not already inserted/updated in this tx, the fk is
not checked
*/


-- tx B
DELETE FROM parent WHERE id = 1;

/*
This locks/deletes the `parent` row. The `ON DELETE CASCADE` action then tries
to delete the referencing `child` row, but tx A already locked that
row. Transaction B waits for tx A.
*/

-- tx A
UPDATE child SET payload = payload + 1 WHERE id = 1;

/*
Now the fk is checked since the row we're updating
was written by this transaction.
tx A waits for a `key share` lock on the parent row, deadlock!
*/
```

in my production experience, the
best balance of correctness, performance, and
availability is `READ COMMITTED`, using explicit locks that suffice for
serializability given the set of other transactions present in the application(s)
(not all theoretically possible other transactions). 
