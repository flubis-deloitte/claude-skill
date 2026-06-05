# Backend Architecture Playbook

How to make structural decisions: package layout, service boundaries, caching, async
work, and system design. Load this when designing something new or refactoring shape.

## Project layout (Go)

A conventional, scalable layout — adapt to the repo:

```
cmd/<app>/main.go      # entrypoint: wire config, deps, server, graceful shutdown
internal/              # private application code (not importable externally)
  <domain>/            # e.g. order/, user/ — handler, service, repository, types
  platform/            # db, cache, http client, logging, config helpers
pkg/                   # optional: genuinely reusable, exportable libraries
migrations/            # versioned schema migrations
api/                   # OpenAPI/proto specs if used
```

- Organize by **domain/feature**, not by technical layer across the whole app. Keep a
  domain's handler+service+repository together.
- `internal/` for anything that shouldn't be imported by other modules.
- `main` is the only place that knows how everything is wired (dependency injection by
  construction, not a framework).

## Boundaries & dependencies

- Dependencies point **inward**: transport → service → repository. Inner layers don't
  import outer ones. The service defines small interfaces; the repository implements them.
- Domain types are the lingua franca. Don't leak `http.Request`, SQL rows, or driver
  types across boundaries.
- A package should have one clear responsibility. If a package imports half the codebase,
  it's doing too much.

## Monolith vs. services

- Default to a **well-structured modular monolith** until scale or org boundaries force
  splitting. Most teams reach for microservices too early and pay the distributed-systems
  tax (network failures, eventual consistency, ops overhead) before they need to.
- Split a service out when a domain has a genuinely independent scaling profile,
  deployment cadence, or team ownership — not just for tidiness.
- When you do split: define the contract (REST/gRPC/events) first, own your data (no
  shared DBs between services), and design for partial failure.

## Caching strategy

- Cache to cut latency or load, not to paper over a slow query you could fix.
- Cache-aside with Redis is the default (see `database.md`): read-through on miss, set
  with a TTL, invalidate on write. Define the staleness you can tolerate.
- Know the failure modes: stampede (jittered TTL / single-flight), stale data (explicit
  invalidation), and cache-as-SPOF (the system must still work, slower, if Redis is down).

## Asynchronous work & queues

- Move slow or retryable work (emails, webhooks, image processing, third-party calls)
  off the request path into a worker.
- Use a real queue (Redis streams/lists, or a broker the repo already uses). Design
  consumers to be **idempotent** — messages get delivered at least once.
- Make jobs retryable with backoff; route poison messages to a dead-letter queue;
  make them observable (depth, age, failure rate).
- For "exactly once" effects, dedupe with an idempotency key in the DB.

## Reliability patterns

- **Timeouts** on every outbound call; **retries** with exponential backoff + jitter for
  transient failures only (never retry a non-idempotent write blindly).
- **Circuit breakers** / bulkheads around flaky dependencies so one slow downstream
  doesn't exhaust your pool.
- **Graceful degradation**: decide what a partial outage looks like (serve stale cache,
  shed load, return a clear error) rather than cascading failure.
- **Backpressure**: bounded queues and pools; reject early under overload (`429/503`).

## Data & consistency

- Pick the consistency model deliberately per use case (strong vs. eventual). Document it.
- One service owns each piece of data. Cross-service reads go through APIs/events, not
  shared tables.
- For workflows spanning services, prefer choreography via events or an explicit saga
  with compensating actions; avoid distributed transactions.

## Observability (build it in, not after)

- **Structured logs** (`slog`) with a request/trace ID correlating a request across
  layers. No secrets/PII.
- **Metrics**: RED (Rate, Errors, Duration) per endpoint and key dependency; pool
  utilization; queue depth.
- **Tracing** (OpenTelemetry) spanning handler → service → DB/cache/downstream.
- Define SLOs and alert on symptoms (latency, error rate), not just host metrics.

## Anti-patterns

- Microservices before product-market fit / before you feel monolith pain.
- Shared databases between services (hidden coupling).
- Synchronous chains of calls with no timeouts (one slow hop stalls everything).
- Fire-and-forget goroutines as a "queue" (lost on restart, no retries, no visibility).
- Caching with no invalidation or no TTL.
- Adding observability "later."
