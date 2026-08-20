# Database Query API Generation

Data permissions
* Postgres row-level security
  * Set identity in transaction config
    * https://www.postgresql.org/docs/current/functions-admin.html#FUNCTIONS-ADMIN-SET
    * https://github.com/graphile/crystal/issues/542#issuecomment-820271404
  * Impersonate db user https://stackoverflow.com/a/2998680/371334
* Rewrite query: replace tables with subqueries enforcing permissions in where clause
  * see Zhang, X. "Designing a SQL Query Rewriter to Enforce Database Row Level
Security." (2016)
  * row level security is probably preferable
* use fixed query template with where clause included

Syntax
* Graphql
* SQL (subset)
* rest(ish)
  * Json
  * query string
    * https://github.com/microsoft/api-guidelines/blob/vNext/azure/Guidelines.md#filter

Processing
* graphql + row level security
  * pg_graphql
  * postgraphile
* Rest + row level security
  * Postgrest
* Rest + query template
  * custom tools
    * how is API schema generated?
* SQL + row level security
  * the user should have SELECT-only permission on whitelisted tables
  * Use https://github.com/wseaton/sqloxide to validate query
    * whitelist functions
    * query must be single SELECT statement
    * no explicit row locking (prevent deadlocks)
    * anti-DoS (see below)
  * wrap with json_agg
  * use `REPEATABLE READ` isolation level

Anti-DoS
* max running time for queries can be set with https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-STATEMENT-TIMEOUT
* According to [this](https://stackoverflow.com/a/61348800/371334), RAM usage for a connection is at most temp_buffers + work_mem (both terms are configurable)
* is this sufficient?
  * filling storage is infeasible because it's slow
  * in e.g. 100ms, I was unable to run `select * from generate_series(1, n)` for
  values of `n` large enough to send much data over the network 
  * the query will be using a combination of scarce CPU and I/O time for its duration. the kernel will allocate this \~proportionally among connections (each is a single-threaded process assuming parallelism is disabled).
* if it's somehow not, can we detect resource-hungry queries ahead of time?
  * recursive CTEs could become tight loops
  * same for functions (but they should be whitelisted anyway)
  * ?
* SQL Server has per-user [resource limits](https://learn.microsoft.com/en-us/sql/relational-databases/resource-governor/resource-governor?view=sql-server-ver16)
