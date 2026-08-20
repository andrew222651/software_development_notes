pattern for data with history: use two columns, `set_at` (datetime) and
`is_current` (boolean). Exactly one row corresponding to a given datum can have
`is_current=true`. All columns with `is_current=false` are read-only. The
`is_current` column is always read-only. Update a datum by selecting the current
row for update, saving the results, updating the row with the new values
including `set_at`, then inserting a new row equal to the old row with
`is_current=false`. Might be better to have a "current" table and "non-current"
table, then the constraints are easier to realize?

Why might you use a 0/1-to-1 relationship? If you use pessimistic concurrency
control, the smallest entity you can lock is a row, so if you want to lock only
part of a row you can split columns across two tables. Alternatives: Postgres
advisory locks only have a 64-bit key. A somewhat better option is to create an
auxiliary table, where rows contain a row ID from the main table and a lock
type. To modify data from the main table you first lock the corresponding row in
the auxiliary table.

Why might you not use a 0/1-to-1 relationship? If you want a composite index,
its columns have to be in the same table. If you're stuck with two tables and
you want to filter based on one column from each, you'd have to create a new
table-like object with the joined data and a composite index; the table-like
thing can be a normal table kept up to date with triggers, or a materialized
view.

Why not to use `on delete cascade`? A simple method of garbage collection is
using `on delete restrict` or `on delete no action` in the referencing tables,
and then attempting to delete rows in the referenced table. The rows are deleted
without error iff they had no references.

Suppose rows in table B reference rows in table A in a many-to-one relationship,
and we want to impose a constraint on all rows in B that refer to the same row
in A, e.g. enforce a maximum sum. One option is to lock B whenever we insert or
update rows there (_Database Systems: The Complete Book_ sec. 18.6.3). Another
option is for all transactions to lock the row in A referenced by the rows in B
that they're working with. In each case more is locked than necessary (although
see above about 0/1-to-1 relationships). Also in each case we could do the
locking and checking in a `before insert or update` trigger but applications
should probably acquire the locks themselves beforehand to make sure the
transaction gets locks in a certain order to avoid deadlocks. Using a trigger on
B that locks the row in A and performs the validation is a solution that avoids
a table lock but also guarantees correctness even if some applications do not
acquire explicit locks.
