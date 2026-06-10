# Test Case Design Playbook

How to derive test cases from requirements with formal techniques so coverage is
reasoned, not random. Load this for "test cases for", "test scenarios".

## Anatomy of a good test case

| Field | Notes |
|-------|-------|
| ID | Stable, e.g. TC-ORDER-001 |
| Title | Clear intent: "Pay order with insufficient MPS balance" |
| Linked requirement | FR/AC/Jira ID (traceability) |
| Preconditions | State + data needed before steps (account, order status, flags) |
| Test data | Exact values used |
| Steps | Numbered, unambiguous, one action each |
| Expected result | Observable outcome — UI **and** backend (state/event) |
| Priority | P0/P1/P2 by risk |
| Type | Positive / negative / boundary / edge |

Write steps so another tester reproduces them exactly. Expected results must be
specific (status, message, screen, DB state, event) — not "it works".

## Design techniques (use them deliberately)

- **Equivalence Partitioning** — group inputs into classes that behave the same; test
  one representative per class (valid + each invalid class). E.g. amount: negative, zero,
  valid, above max.
- **Boundary Value Analysis** — test at and around boundaries (min-1, min, min+1, max-1,
  max, max+1). Most bugs live at edges.
- **Decision Table** — for combinations of conditions (e.g. payment method × balance ×
  promo). Enumerate rules so no combination is missed.
- **State Transition** — for stateful entities (order: PENDING → PAID → REFUNDED).
  Test valid transitions, invalid transitions, and actions in each state.
- **Error Guessing / Exploratory** — apply experience: empty inputs, huge inputs,
  special characters, double-submit, back button, expired session, slow network, offline.

## Coverage checklist per feature

- Positive (happy) path(s).
- Each negative/validation case (invalid, missing, wrong type, out of range).
- Boundaries (min/max/limits, pagination edges, list empty/one/many).
- Permissions/auth (unauthorized, forbidden, expired token, wrong role).
- State-dependent behavior (action not allowed in current state).
- Concurrency/idempotency (double-submit, duplicate callback, retry).
- Cross-platform (web + mobile, key browsers/devices, responsive breakpoints).
- Data/end-state verification across UI + service (DB record, Kafka event).
- UI states from Figma: loading, empty, error, success.

## Quality bar

- Each requirement/AC has ≥1 case; each case traces back to a requirement.
- Techniques applied (not random); boundaries and negatives covered.
- Steps reproducible; expected results specific and cross-layer.
- Priorities set by risk.
