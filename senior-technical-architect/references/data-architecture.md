# Data Architecture Playbook

How to decide where data lives and how it moves. Load this for "data architecture",
"Mongo vs GORM", "caching strategy", "migration".

## Ownership first

- **One service owns each entity** and is the only writer. Others get it via API or via
  events into their own read model. No shared databases, no cross-service DB reads.
- Decide the **system of record** for every important entity explicitly.

## Choosing the store (polyglot persistence)

- **PostgreSQL / MySQL (via GORM)** — relational, transactional, strong invariants,
  reporting, anything needing joins and constraints. Default for financial/transactional
  state.
- **MongoDB** — flexible/varied document shapes, denormalized read models, aggregates
  read together as one document, rapidly evolving schemas.
- **Redis** — cache, distributed locks (redsync), rate limiting, ephemeral/coordination
  state, queues/streams. Never the sole system of record for important data.
- Match the store to the access pattern and consistency need; don't default to one for
  everything. A service may use more than one.

## Modeling

- Relational: normalize for integrity, index FKs and hot query columns, money as
  decimal/minor-units, timestamps UTC, enforce constraints in the DB. Verify with
  `EXPLAIN`.
- Document: model around access patterns (embed what's read together, reference what's
  shared/large), index every query shape, watch the 16MB doc limit and unbounded arrays.
- Define data classification (PII/sensitive) and retention per entity.

## Caching

- Cache to cut latency/load, not to mask a query you should fix. Cache-aside with Redis,
  TTL + explicit invalidation on write. Plan for stampede (jitter/single-flight) and
  for Redis being down (degrade, don't fail).

## Migrations & evolution

- Versioned, reversible migrations; never edit a shipped migration — add a new one.
- Large tables: add columns safely (nullable/defaults), build indexes `CONCURRENTLY`
  (Postgres), avoid long locks; backfill in batches.
- Schema/data changes that affect events or APIs are additive within a version; breaking
  changes get a version bump + deprecation window + an ADR.
- For data moves across stores/services, plan: dual-write or backfill + replay, a
  reconciliation step, and a cutover with rollback.

## Quality bar

- Every entity has one owning service and a declared system of record.
- Store choice matches access pattern + consistency; polyglot used deliberately.
- Money/time typed correctly; PII classified; retention defined; indexes intentional.
- Caching has TTL + invalidation + failure behavior.
- Migrations reversible, lock-safe, and (cross-store) have backfill + reconciliation +
  rollback.
