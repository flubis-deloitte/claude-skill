# Test Case Design Playbook (Web)

How to derive web test cases from requirements with formal techniques so coverage is
reasoned. Load this for "test cases for", "test scenarios". Cases here feed both manual
execution and automation.

## Anatomy of a good test case

| Field | Notes |
|-------|-------|
| ID | TC-LOGIN-001 |
| Title | "Login fails with wrong password" |
| Linked requirement | AC / Jira ID |
| Preconditions | State + data + which page/route |
| Test data | Exact values |
| Steps | Numbered, one action each |
| Expected result | Observable UI outcome + (where relevant) API/end state |
| Priority | P0/P1/P2 by risk |
| Mode | Manual / automate (and layer) |

Mark whether the case is a candidate for automation and at which layer (unit/component/
e2e), so the suite gets built from the case list.

## Design techniques

- **Equivalence Partitioning** — one representative per input class (valid + each invalid).
- **Boundary Value Analysis** — min-1/min/min+1 … max-1/max/max+1; list edges, field-length
  limits, pagination edges.
- **Decision Table** — combinations of conditions (e.g. role × feature flag × data state).
- **State Transition** — multi-step UI/flows (form wizard steps; cart → checkout → paid).
- **Error guessing / exploratory** — empty inputs, huge inputs, special chars, double-click
  submit, back/refresh, expired session, slow/offline network.

## Web-specific coverage checklist

- Happy path(s).
- Validation: required, format, range, wrong type — with the correct inline error messages
  (pull from Figma).
- UI states: **loading, empty, error, success** for every async surface.
- Auth/permissions: unauthenticated redirect, forbidden, expired token, role differences.
- Boundaries: list empty/one/many, pagination edges, long text/overflow, field limits.
- Forms: client + server validation, submit/disable states, double-submit, unsaved changes.
- Network: slow, failed request, timeout, retry — does the UI handle it gracefully?
- Responsive: key breakpoints (mobile/tablet/desktop), no overflow/layout break.
- Cross-browser: target browsers behave the same.
- End state: the API was called correctly / the data reflects the action.

## Quality bar

- Each requirement/AC → ≥1 case; each case → back to a requirement.
- Techniques applied; boundaries, negatives, and all UI states covered.
- Steps reproducible; expected results specific (UI + end state).
- Mode/layer tagged so automation can be built from the list.
