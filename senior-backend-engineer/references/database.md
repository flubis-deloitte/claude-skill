# Database Playbook — PostgreSQL · MySQL · MongoDB · Redis

How a senior designs schemas, writes queries, and uses each datastore for what it's
good at. Load this for any data-layer work. Always detect which store(s) the repo uses
and follow its existing conventions.

## Universal rules (all stores)

- **Parameterize every query.** Never build SQL/filters by string concatenation. SQL
  injection is non-negotiable.
- **Always pass `context.Context`** to queries so cancellation/timeouts free connections.
- **Close what you open**: `defer rows.Close()`, check `rows.Err()` after iterating.
- **One connection pool per process**, reused. Tune `SetMaxOpenConns`,
  `SetMaxIdleConns`, `SetConnMaxLifetime` (and `SetConnMaxIdleTime`). An undersized or
  unbounded pool is a top cause of latency and outages.
- **No N+1.** Fetch related rows with a join, an `IN (...)` / `= ANY($1)` batch, or a
  dataloader pattern — never a query per item in a loop.
- **Migrations are versioned and reversible.** Every schema change is a migration in
  `migrations/` with up and down. Never edit a shipped migration; add a new one.

## PostgreSQL

- Prefer `pgx` (and `pgxpool`) for performance and Postgres-native types; `sqlc` to
  generate type-safe query code is a strong default.
- Use real types: `uuid`, `timestamptz` (always store UTC, timezone-aware), `numeric`
  for money (never `float`), `jsonb` for flexible blobs, native `enum` or a check
  constraint for closed sets.
- Index intentionally: index foreign keys and frequent `WHERE`/`ORDER BY` columns.
  Use partial and composite indexes where queries warrant. Verify with `EXPLAIN
  (ANALYZE, BUFFERS)`. Don't over-index write-heavy tables.
- Enforce integrity in the DB: `NOT NULL`, `UNIQUE`, foreign keys, `CHECK`. Use
  `ON CONFLICT` for upserts.
- Transactions: keep them short; pick the right isolation (`READ COMMITTED` default,
  `SERIALIZABLE` for invariants that span rows) and handle serialization-failure retries.
- Migrations on big tables: add columns nullable / with defaults carefully, create
  indexes `CONCURRENTLY`, and avoid long-held locks on hot tables.

## MySQL

- Use InnoDB. Choose charset `utf8mb4`. Be explicit about the collation.
- Money as `DECIMAL`, timestamps in UTC (`DATETIME` + app-side UTC, or `TIMESTAMP`
  understanding its range). Beware implicit type coercion in comparisons.
- Default isolation is `REPEATABLE READ` (differs from Postgres) — know the gap-locking
  behavior and how it affects concurrent writes.
- Index FKs and query columns; check `EXPLAIN`. Watch for implicit conversions that
  silently skip indexes (e.g. comparing a string column to an int).
- Upserts via `INSERT ... ON DUPLICATE KEY UPDATE`.

## MongoDB

- Use the official Go driver. Model documents around **access patterns**, not
  normalization: embed data you read together, reference data that's large or shared.
- Create indexes for every query shape (including compound and multikey); confirm with
  `explain()`. An unindexed query is a full collection scan.
- Use schema validation rules on collections to prevent malformed documents.
- Use transactions only when you truly need multi-document atomicity (they cost more
  than in an RDBMS); prefer designs where a single document is the consistency unit.
- Beware unbounded array growth in documents and the 16MB document limit.

## Redis

- Use it for what it's good at: **caching, rate limiting, distributed locks, ephemeral
  state, queues/streams, leaderboards** — not as a primary system of record.
- **Every key has a purpose and (usually) a TTL.** Namespace keys consistently
  (`svc:entity:id`). Avoid unbounded key growth.
- Caching pattern: cache-aside (read: check cache → miss → load DB → set with TTL).
  Have an invalidation story on writes. Beware stampedes (use jittered TTLs / a lock /
  single-flight for hot keys).
- Locks: use Redlock-style or `SET key val NX PX <ttl>`; always set an expiry so a
  crashed holder can't deadlock; release only your own token (check-and-delete via Lua).
- Use pipelining/`MULTI` to batch; pick appropriate structures (hash, sorted set, stream)
  rather than serializing blobs everywhere.

## Choosing the right store

- Relational, transactional, strong invariants, reporting → **Postgres/MySQL**.
- Flexible/varied document shapes, denormalized read models → **MongoDB**.
- Speed, ephemerality, coordination, fan-out → **Redis** (alongside a source of truth).

## Anti-patterns

- String-built SQL/filters; queries without `ctx`.
- N+1 query loops; `SELECT *` into wide structs you don't need.
- `float` for money; naive (non-UTC, non-tz) timestamps.
- Long transactions held across network calls.
- Redis keys with no TTL; using Redis as the only copy of important data.
- Editing an already-applied migration instead of adding a new one.
- Missing indexes on FKs and hot query columns; or indexing everything blindly.
