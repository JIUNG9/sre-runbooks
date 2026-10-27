# Connection Pooling with PgBouncer

Each Postgres connection is a backend process with its own memory, so a few hundred
mostly idle application connections cost more than the queries they run. A pooler sits
between the application and the database and multiplexes many client connections onto
a small number of server connections. This note covers pool modes, sizing against
`max_connections`, and what breaks in transaction mode.

## Pool modes

| Mode | Server connection is returned | Use when |
| --- | --- | --- |
| `session` | When the client disconnects | The application relies on session state and you only need to cap connections |
| `transaction` | At the end of each transaction | Many short-lived clients, the usual choice for web services |
| `statement` | After each statement | Autocommit-only workloads. Multi-statement transactions are rejected |

`transaction` gives the multiplexing that makes a pooler worth running. It also removes
the guarantee that two statements from one client reach the same backend, which is the
source of most problems below.

```ini
; pgbouncer.ini
[databases]
app = host=db01.prod.internal port=5432 dbname=app

[pgbouncer]
pool_mode = transaction
max_client_conn = 2000
default_pool_size = 20
reserve_pool_size = 5
reserve_pool_timeout = 3
```

## Sizing against `max_connections`

A pool exists per (database, user) pair, and `default_pool_size` applies to each one.
The worst case on the server is:

```text
sum over pools(default_pool_size + reserve_pool_size)
  + superuser_reserved_connections (default 3)
  + connections that bypass the pooler (replication, monitoring, migrations)
  <= max_connections
```

Two databases and two users behind one PgBouncer can open four pools of 25, which is
100 server connections, and the default `max_connections` is also 100. Check the sum
before raising a pool size.

Start `default_pool_size` near the number of cores on the database host, then adjust
from measurements. Beyond the point where the CPUs are busy, more server connections
add context switching and lock contention without adding throughput. Pool size, not
`max_client_conn`, limits concurrency. `max_client_conn` can be large because an idle
client only costs PgBouncer a few kilobytes.

Running several PgBouncer instances multiplies every pool by the instance count. Divide
`default_pool_size` accordingly.

## What breaks in transaction mode

Anything that stores state in the backend session can land on a different backend for
the next transaction.

| Feature | Symptom | Fix |
| --- | --- | --- |
| `SET` / `SET search_path` | Setting is missing or leaks to another client | `SET LOCAL` inside the transaction, or set it per role or database with `ALTER ROLE ... SET` |
| Session advisory locks (`pg_advisory_lock`) | Lock is held by a backend the client no longer owns, or never released | `pg_advisory_xact_lock`, or use a direct connection for the locking client |
| `LISTEN` / `NOTIFY` | Notifications never arrive | Use a direct connection for listeners |
| Temporary tables, `WITH HOLD` cursors | `relation does not exist` on the next transaction | Create and use them inside one transaction |
| Protocol-level prepared statements | `prepared statement "S_1" does not exist` or `already exists` | See below |

### Prepared statements

Drivers such as pgjdbc, psycopg 3 and asyncpg prepare statements on the connection.
Under transaction pooling the next execution may reach a backend that never saw the
`PREPARE`.

- PgBouncer 1.21 and later tracks protocol-level prepared statements when
  `max_prepared_statements` is set to a non-zero value (the default is 0, which turns
  it off). Start with 100 and watch memory.
- On older versions, turn off server-side preparing in the driver:
  `prepareThreshold=0` for pgjdbc, `prepare_threshold=None` for psycopg 3,
  `statement_cache_size=0` for asyncpg.
- SQL-level `PREPARE` and `EXECUTE` are not tracked by that setting. Avoid them behind
  a transaction pooler.

### Reset query

`server_reset_query` (default `DISCARD ALL`) runs when a server connection is released
in `session` mode only. In `transaction` mode it is skipped unless
`server_reset_query_always = 1`, so state set outside a transaction is not cleaned up
for you.

## Diagnosing a saturated pool

Connect to the admin console (`psql -p 6432 pgbouncer`):

```sql
SHOW POOLS;
SHOW STATS;
SHOW CLIENTS;
```

In `SHOW POOLS`:

- `cl_waiting` above 0 and `maxwait` growing: clients queue for a server connection.
  Either transactions are slow or the pool is too small.
- `sv_active` equals `pool_size` while `sv_idle` is 0: every server connection is in
  use. Look at what they run with `pg_stat_activity`.
- `sv_idle` high and `cl_waiting` 0: the pool is larger than the load needs.

A client that waits longer than `query_wait_timeout` (default 120 seconds) is
disconnected with `query_wait_timeout` in the PgBouncer log. Application timeouts
shorter than that hide the queue behind generic timeout errors.

Before enlarging the pool, check for transactions held open by the application:

```sql
SELECT pid, state, now() - xact_start AS xact_age, left(query, 80) AS last_query
FROM pg_stat_activity
WHERE state = 'idle in transaction'
ORDER BY xact_start;
```

Each one holds a server connection while doing nothing. Set
`idle_in_transaction_session_timeout` on the role and fix the code that leaves the
transaction open.

## Related

- [Query optimization](query-optimization.md)
- [Replication](replication.md)
