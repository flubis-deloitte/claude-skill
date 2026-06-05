# Non-Functional & Reliability Playbook

How to set and design for non-functional targets. Load this for "scalability",
"reliability", "security", "observability", "SLO".

Non-functionals are architecture, not an afterthought. Give each a measurable target.

## Scalability & performance

- State the load model: current + projected peak (req/s, events/s, data growth).
- Latency targets per critical path (p95/p99). Throughput targets per service/topic.
- Scale stateless services horizontally; identify and relieve the real bottleneck;
  cache hot reads; partition data when a single store saturates.
- Capacity-plan: rough math on connections, pool sizes, partition counts, storage.

## Reliability & availability

- Availability target (e.g. 99.9%) and what degradation looks like below it.
- **Design for failure**: timeouts everywhere, retries (idempotent only) with backoff +
  jitter, circuit breakers/bulkheads, backpressure, graceful degradation.
- Idempotency on every retryable write and every Kafka consumer.
- Define RPO/RTO for critical data; backups and a tested restore path.
- No single points of failure on critical paths; the system survives one dependency
  being slow or down.

## Security

- AuthN/authZ at every boundary; least privilege for service and DB credentials.
- Data classification; encrypt in transit (TLS) and at rest where required; secrets in a
  vault/env, never in code or logs.
- Validate/sanitize all input; protect against injection, SSRF on outbound calls.
- Never log secrets, tokens, OTPs, or PII. Audit sensitive actions.
- Map to relevant compliance (e.g. payment data handling) — call it out explicitly.

## Observability (design it in)

- **Structured logs** with a correlation/trace ID across services (no PII).
- **Metrics**: RED (Rate, Errors, Duration) per endpoint and dependency; pool
  utilization; Kafka consumer lag and queue depth; cache hit rate.
- **Tracing** (Datadog / Elastic APM) spanning handler → service → DB/cache/Kafka/
  downstream.
- **SLOs** on user-facing symptoms (latency, error rate); alert on SLO burn, not just
  host metrics. Define dashboards and the on-call signals before launch.

## Putting targets in the design

| Dimension | Target | How verified |
|-----------|--------|--------------|
| Latency (GET order) | p95 < 300ms @ 200 rps | load test + Datadog |
| Availability | 99.9% | SLO dashboard |
| Consumer lag | < 30s | Kafka metric + alert |

## Quality bar

- Each NFR has a number and a way to verify it.
- Failure modes designed for (timeout/retry/breaker/degrade); idempotency in place.
- Security at boundaries; secrets/PII handling explicit; compliance noted.
- Logs/metrics/traces/SLOs/alerts defined as part of the design, not later.
