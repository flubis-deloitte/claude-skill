# Microservices, CQRS & Eventing Playbook

How to design integration and event flow across the MyPertamina services. Load this
for "event-driven design", "CQRS", "Kafka", "saga", "consistency".

## CQRS in this platform

- **Command** services own writes and state transitions; they emit events.
- **Query / aggregation** services build read models from those events (eventually
  consistent) so reads don't hit the write path.
- **Orchestrators** coordinate a workflow that spans services. Keep orchestration logic
  in one place; keep the participating services autonomous.
- Don't let a query service write, or a command service become a read API. Keep the
  split clean.

## Sync vs async — choose per interaction

- **Synchronous REST** only when the caller needs the result inline (e.g. create a
  payment before confirming an order). Specify timeout, retry (idempotent calls only),
  and the fallback/compensation on failure.
- **Asynchronous Kafka** for fan-out, cross-domain side effects, and decoupled work
  (notifications, read-model updates, downstream effects). Prefer events; avoid long
  synchronous chains where one slow hop stalls the request.

## Event design

For each event: topic name, versioned schema, partition key (ordering unit), producer,
consumer(s), trigger, and idempotency rule.

| Topic | Producer | Consumers | Key | Idempotency |
|-------|----------|-----------|-----|-------------|
| order.paid | order-command | aggregation, notification-worker | orderId | dedupe on (orderId, eventId) |

- **At-least-once delivery** → every consumer must be **idempotent**.
- Version schemas; additive changes only within a version. New required field = new
  version + migration plan.
- Define dead-letter / retry-with-backoff for poison messages.
- Decide ordering needs (per-key ordering via partitioning) explicitly.

## Distributed workflows & consistency

- Avoid distributed transactions. For multi-service workflows use a **saga**:
  - **Choreography** (services react to each other's events) for simple flows.
  - **Orchestration** (an orchestrator drives steps) for complex flows needing central
    control and visibility.
- Define **compensating actions** for each step that can fail after earlier steps
  committed (e.g. refund on downstream failure).
- State the **consistency model** per data item (strong vs eventual) and how the system
  **converges**: idempotent replays, reconciliation cron, outbox pattern for reliable
  publish.
- Use the **transactional outbox** when a state change and its event must both happen
  (write to DB + outbox in one tx; a relay publishes to Kafka).

## Resilience patterns

- Timeouts on every outbound call; retries with backoff + jitter (idempotent only);
  circuit breakers / bulkheads around flaky dependencies; backpressure under overload.
- Graceful degradation: serve stale cache, queue for later, or fail clearly — never
  cascade.

## Quality bar

- Each interaction justified as sync or async; no unbounded sync chains.
- Events fully specified (topic, schema+version, key, producer/consumers, idempotency).
- Multi-service workflows use sagas with compensation; consistency model + convergence
  stated; outbox where atomic write+publish is required.
- Failure handling defined at every integration point.
