# Manual API Testing Playbook

How to test the REST APIs (GoFiber services) by hand with Postman or curl. Load this
for "test the API", "verify the endpoint", "contract testing".

## Before you test

- Get the contract from the system analyst's spec / API doc: method, path, auth,
  request schema, response schema, status codes, error envelope.
- Know the service it hits and any events it should emit (so you can verify the async
  side too).

## What to check per endpoint

- **Happy path** — valid request → correct status (200/201), correct response body
  (every field, correct types/values), and correct side effect (DB state changed,
  expected Kafka event published).
- **Validation** — missing required fields, wrong types, out-of-range/enum-invalid values
  → correct 400/422 with the right error code/message.
- **Auth** — no token / invalid token → 401; valid token but not allowed → 403.
- **Not found / conflict** — unknown id → 404; duplicate/version conflict → 409.
- **Idempotency** — for retryable POST (payments, callbacks): send the same request
  twice → no double effect, consistent response.
- **Boundary** — min/max field lengths and values; pagination edges (page 0, beyond last
  page, limit over cap).
- **Content & headers** — correct Content-Type, the response envelope shape, pagination
  meta.

## Verifying the full effect

A 200 isn't proof of success. Confirm the **end state**:

- The record was created/updated correctly (query the DB / a read endpoint).
- The **event** was published (check the topic/consumer or the downstream read model).
- No unintended side effects on related data.

## Practical tips

- Build a **Postman collection** per service: folder per resource, saved requests for
  each case, environment variables for base URL/token, and example responses.
- Use tests/assertions in Postman to check status + key fields automatically.
- Keep a small set of seed accounts/data and document them.
- For callbacks/webhooks (payment providers), simulate the provider's callback payloads
  including the failure/duplicate variants.

## Reporting an API defect

Include the full request (method, URL, headers minus secrets, body), the actual
response (status + body), the expected response per the contract, and the backend
evidence (log/trace id, DB state, missing event). See `bug-reporting.md`.

## Quality bar

- Every status code in the contract exercised, with correct body and error codes.
- Auth, validation, not-found, conflict, idempotency, and boundary cases covered.
- End state verified in DB and via events — not just the HTTP response.
- Reusable Postman collection with environments and assertions.
