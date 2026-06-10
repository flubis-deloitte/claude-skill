# Test Plan & Strategy Playbook

How to plan testing for a feature or release. Load this for "test plan", "test
strategy", "how do we test this".

## Test plan structure (Confluence page)

1. **Overview** — what's being tested and why; link the spec / Jira epic.
2. **Scope** — in scope and out of scope (features, platforms, integrations). Explicit.
3. **Test approach** — which test types and how much of each:
   - Functional, integration (cross-service), API, regression, smoke, exploratory,
     UAT, plus non-functional checks (basic performance, security, accessibility) where
     relevant.
4. **Environments & data** — which env (dev/staging), test accounts, seed data, feature
   flags, third-party sandboxes (payment providers, etc.).
5. **Entry criteria** — what must be true to start (build deployed, spec approved, env
   ready, test data available).
6. **Exit criteria** — what "done" means (e.g. 100% of P0/P1 cases executed, no open
   Critical/High defects, regression passed).
7. **Risks & mitigations** — what could go wrong in testing and the product; risk-based
   focus areas.
8. **Schedule & ownership** — who tests what, by when.
9. **Deliverables** — test cases, execution report, defect summary, go/no-go.

## Risk-based prioritization

Spend effort where failure is most likely and most costly. Rank areas by
**impact × likelihood**:

- High impact: payments, authentication/authorization, transactions, data integrity,
  anything touching money or PII.
- High likelihood: new/complex code, areas with past defects, many integrations,
  tight deadlines.

Test high-risk areas deeply (all techniques, edge cases, negative paths); smoke-test
low-risk stable areas.

## Cross-platform & integration scope

- This platform has **web + mobile** clients — decide which browsers/devices/viewports
  matter and include them.
- A user action often spans **multiple services and Kafka events** — plan to verify the
  end state (DB record, event published, downstream effect), not just the UI.
- Include third-party integration paths (payment providers, adapters) and their failure
  modes (timeout, decline, callback).

## Quality bar

- Scope (in/out), test types, environments, and data all explicit.
- Entry/exit criteria measurable; go/no-go defined.
- Effort prioritized by risk; high-risk areas identified.
- Cross-platform and cross-service/event verification planned, not assumed.
