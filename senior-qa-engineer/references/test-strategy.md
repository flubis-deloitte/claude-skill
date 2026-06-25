# Test Strategy Playbook — What to Automate vs Test Manually (Web)

How to plan web testing and split work between manual and automation. Load this for
"test plan", "what should we automate", "test strategy".

## The decision: automate vs manual

**Automate when** the case is:
- Stable (the behavior/UI isn't changing every sprint)
- Repetitive / run often (regression, smoke, critical journeys)
- High-value or high-risk (login, payment/checkout, core flows, data integrity)
- Deterministic and verifiable programmatically

**Keep manual when** the case is:
- Exploratory (finding unknown issues), usability, or "does this feel right"
- Visual / pixel / layout vs Figma, and subjective UX
- One-off, or for a feature still rapidly changing (automating now = constant rework)
- Hard/expensive to automate relative to its value

Rule of thumb: **automate the boring, repeatable safety net; spend human time where
judgment and exploration matter.**

## The pyramid (bias to cheap layers)

| Layer | Tool | Share | Use for |
|-------|------|-------|---------|
| Unit / component | Jest + React Testing Library | most | logic, hooks, components in isolation |
| Integration / API | Jest / Playwright APIRequest | some | feature slices + REST calls, contracts |
| End-to-end | Playwright | few | critical user journeys in a real browser |
| Manual / exploratory | — | targeted | usability, visual, cross-browser, the unknown |

Anti-pattern: an inverted pyramid (many slow e2e duplicating what cheap unit/component
tests cover).

## Test plan (Confluence)

1. **Scope** — features, pages, browsers in/out of scope (web only).
2. **Approach** — what's manual, what's automated, at which layer; new vs existing suite.
3. **Environments & data** — dev/staging URL, test accounts, seed data, feature flags,
   API/test backend, third-party sandboxes.
4. **Entry criteria** — build deployed, AC approved, env + data ready.
5. **Exit criteria** — P0/P1 cases executed, automated suite green in CI, no open
   Critical/High defects, regression passed.
6. **Risks** — risk-based focus (payments, auth, transactions first).
7. **Deliverables** — test cases, automated tests, execution report, defect summary,
   go/no-go.

## Cross-browser & responsive scope

- Decide the target browsers (e.g. latest Chrome, Safari, Firefox; mobile web viewports)
  and which get automated (Playwright projects) vs manual spot-checks.
- Include responsive breakpoints and the UI states from Figma (loading/empty/error/success).

## Quality bar

- Each area assigned a mode/layer by stability, value, and risk — justified, not default.
- Pyramid respected (few e2e, many unit/component).
- Scope, environments, entry/exit criteria explicit; risk-prioritized.
- Cross-browser/responsive scope decided.
