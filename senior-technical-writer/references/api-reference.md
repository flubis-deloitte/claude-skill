# API Reference Documentation Playbook

How to document a GoFiber/REST service so an integrator can use it without reading the
code. Load this for "API documentation", "document the endpoint".

## Page structure for a service

1. **Overview** — what the service does, base URL(s) per environment, the resource model.
2. **Authentication** — scheme (token/header), how to obtain credentials, scopes.
3. **Conventions** — content type, the response envelope, error format, pagination,
   idempotency, versioning, rate limits, timestamps/timezone, money units.
4. **Endpoints** — one section each (below).
5. **Data models / schemas** — shared objects referenced by endpoints.
6. **Events** (if applicable) — Kafka topics this service produces/consumes, with schemas.
7. **Changelog** — versioned changes.

## Per-endpoint template

```
### POST /transaction/v1/orders
Create an order.

Auth: Bearer token (scope: order.write)

Path/query params
| Name | In | Type | Required | Description |
| status | query | string | no | Filter: PENDING|PAID|FAILED |

Request body
| Field | Type | Required | Constraints | Description |
| customerId | string | yes | UUID v4 | Owner of the order |
| amount | integer | yes | > 0, IDR minor units | Total to charge |

Response 201 Created
{ "success": true, "data": { "orderId": "…", "status": "PENDING" }, "code": 201 }

Errors
| Status | Code | When |
| 400 | bad_request | malformed body |
| 422 | validation_error | amount <= 0 |
| 401 | unauthorized | missing/invalid token |
| 409 | conflict | duplicate idempotency key |

Example
curl -X POST https://api.example/transaction/v1/orders \
  -H "Authorization: Bearer <token>" -H "Content-Type: application/json" \
  -d '{"customerId":"…","amount":50000}'
```

## Rules

- Document **every** field: name, type, required?, nullability, constraints/enums, and a
  one-line description. No "etc.".
- Document the **full set of status codes and error codes**, each tied to a condition —
  not just 200.
- Give a **real, runnable example** request and the actual response body for each endpoint
  (or at least each resource).
- Keep request/response shapes as the **DTOs** the service exposes; don't leak internal
  models.
- Note idempotency for retryable POSTs, pagination params + meta for lists, and the
  versioning policy (additive within a version).
- Verify against the actual handler/spec — field names and codes must match the code.
- Single-source where possible: if the service publishes an OpenAPI spec, generate from
  it and document the gaps, rather than hand-maintaining a copy that drifts.

## Quality bar

- Auth, conventions, and error format documented once and applied consistently.
- Every endpoint: all params/fields, full status/error set, a runnable example.
- Schemas match the code; events documented where relevant; changelog kept.
