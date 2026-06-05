# REST API Design Playbook

How to design REST endpoints that are predictable, evolvable, and correct. Load this
when defining or changing an HTTP contract.

## Resource modeling

- URLs name **resources** (nouns), not actions: `/orders`, `/orders/{id}`,
  `/orders/{id}/items`. Avoid verbs in paths (`/getOrder` is wrong).
- Use plural collection names. Nest only to express ownership, and keep nesting shallow
  (max ~2 levels). For actions that don't map to CRUD, model a sub-resource or use a
  clearly named action endpoint (`POST /orders/{id}/cancel`).
- Use the HTTP methods for their semantics:
  - `GET` — safe, idempotent, no side effects, cacheable
  - `POST` — create / non-idempotent actions
  - `PUT` — full replace (idempotent)
  - `PATCH` — partial update
  - `DELETE` — remove (idempotent)

## Status codes

Map outcomes precisely; don't return `200` for everything.

- `200 OK` — successful GET/PATCH/PUT with a body
- `201 Created` — successful POST that created a resource; include a `Location` header
- `202 Accepted` — accepted for async processing
- `204 No Content` — success with no body (e.g. DELETE)
- `400 Bad Request` — malformed syntax / invalid input shape
- `401 Unauthorized` — missing/invalid auth
- `403 Forbidden` — authenticated but not allowed
- `404 Not Found` — resource doesn't exist (or hide existence from unauthorized users)
- `409 Conflict` — version/uniqueness conflict
- `422 Unprocessable Entity` — well-formed but semantically invalid (validation)
- `429 Too Many Requests` — rate limited; include `Retry-After`
- `500/502/503/504` — server/dependency failures (never leak internals in the body)

## Request & response bodies

- JSON in, JSON out. Use a consistent field-naming convention (snake_case or camelCase —
  match the repo) and stick to it.
- Separate **wire types** (request/response DTOs) from **domain types**. Map between
  them explicitly; never serialize DB models directly.
- Validate at the edge: required fields, ranges, formats, enum membership. Reject with
  `400/422` and a structured error.
- Use a single, consistent error envelope, e.g.:

```json
{ "error": { "code": "validation_error", "message": "email is invalid",
             "details": [{ "field": "email", "issue": "format" }] } }
```

## Pagination, filtering, sorting

- Prefer **cursor-based** pagination for large/changing datasets (`?limit=50&cursor=...`);
  return the next cursor in the response. Offset pagination is fine for small, stable sets.
- Cap and default `limit`. Document max page size.
- Filtering/sorting via explicit query params (`?status=open&sort=-created_at`);
  whitelist sortable/filterable fields — never interpolate them into SQL.

## Idempotency & concurrency

- Make `PUT`/`DELETE` idempotent. For `POST` creates that may be retried, support an
  `Idempotency-Key` header and dedupe server-side.
- Use optimistic concurrency for updates: an `ETag`/`If-Match` header or a `version`
  field; return `409` on mismatch.

## Versioning & evolution

- Version the API (`/v1/...` or a header). Add fields without breaking; never repurpose
  or remove a field within a version.
- Treat the contract as a promise: additive changes are safe, removals/renames are
  breaking and need a new version + deprecation window.

## Cross-cutting

- **Auth** at the boundary (middleware): authenticate, then authorize the specific
  action/resource before the handler does work.
- **Timeouts**: set server read/write/idle timeouts; set per-request deadlines.
- **Rate limiting** on public endpoints (often Redis-backed — see `database.md`).
- **Observability**: request ID per request, structured access logs, latency + error-rate
  metrics per route, tracing spans across service and DB calls.
- **Content negotiation & compression** where it matters; consistent `Content-Type`.

## Anti-patterns

- Verbs in URLs; tunneling everything through `POST`.
- `200 OK` with an `{"error": ...}` body.
- Returning DB models directly (leaks columns, couples wire to schema).
- Unbounded list endpoints with no pagination.
- Accepting unwhitelisted `sort`/`filter` params straight into a query.
- Breaking changes inside an existing version.
