# SRS / Technical Spec Playbook

How to write a Software Requirements Specification / technical spec an engineer can
build from. Load this for any "write the spec / SRS" task.

## Structure (use as a Confluence template)

1. **Overview & context** — the problem, the business goal, the trigger (Jira epic).
   One paragraph. Link the source requirement.
2. **Scope** — in scope, out of scope, non-goals. Be explicit; this prevents creep.
3. **Actors & stakeholders** — who/what uses this (end user, merchant, another
   service, a cron, a Kafka topic).
4. **Assumptions & dependencies** — what must be true; which services/teams/data this
   relies on; what's still open (as questions).
5. **Functional requirements (FR)** — numbered, testable. One behavior each.
6. **Non-functional requirements (NFR)** — numbered, measurable (see below).
7. **System design** — affected services (command/query/orchestrator), data flow,
   sequence diagrams, the API contract (link `api-data-model.md`), events
   (link `integration-mapping.md`).
8. **Data model** — entities, fields, ownership, retention.
9. **Error handling & edge cases** — every failure path and the system's response.
10. **Acceptance criteria** — how each FR is verified; traceable to FR IDs.
11. **Rollout & migration** — flags, backfills, sequencing, backward compatibility.
12. **Open questions** — owner + decision needed.

## Writing functional requirements

- Format: `FR-<n>: The system shall <observable behavior> when <condition>.`
- One requirement = one behavior. Split compound sentences.
- Make it testable: include inputs, the rule, and the expected output/state.
- Bad: "The system handles payment." Good: `FR-7: On a successful MPS callback, the
  command service shall mark the order PAID, publish an order.paid event, and return
  200; a duplicate callback for the same transactionId shall be idempotent (no state
  change, 200).`

## Non-functional requirements — make them numbers

Cover the categories that actually bite in production:

- **Performance**: p95/p99 latency target per endpoint; throughput (req/s, events/s).
- **Scalability**: expected peak load; horizontal scaling assumptions.
- **Availability/reliability**: uptime target; degradation behavior; retry/idempotency.
- **Consistency**: strong vs eventual per data item; ordering guarantees for events.
- **Security**: authN/authZ per endpoint; data classification; PII handling; audit.
- **Observability**: required logs (no PII), metrics (RED), traces, alerts.
- **Data**: retention, archival, backup/restore expectations.
- **Compatibility**: API version impact; mobile/web client support.

Each NFR gets an ID (`NFR-<n>`) and a measurable target, e.g. `NFR-2: 95% of
GET /transaction/{id} responses complete within 300 ms at 200 req/s.`

## Traceability matrix

Keep a small table so nothing is orphaned and nothing is unbuilt:

| Req ID | Source (Jira) | Component/Service | API/Event | Acceptance Criteria |
|--------|---------------|-------------------|-----------|---------------------|
| FR-1   | PTM-1234      | direct-order-command | POST /.../payment/create | AC-1, AC-2 |

## Quality bar

- Every FR/NFR is numbered, testable, and traced.
- Happy path + every error/edge path defined.
- Data contract and persistence model complete.
- Diagrams agree with the written contract.
- Assumptions/dependencies/out-of-scope explicit; open questions have owners.
