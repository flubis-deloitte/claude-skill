# Bug Reporting Playbook

How to write a defect a developer can reproduce and fix on the first read. Load this
for "write a bug report", "log a defect".

## Anatomy of a good bug report (Jira issue)

- **Title** — concise and specific: "[Order] Duplicate MPS callback marks order PAID
  twice and double-credits points". Not "payment bug".
- **Environment** — app version/build, env (staging), platform (web Chrome 1xx / iOS app
  vx), account/role, date-time, and a trace/correlation id if available.
- **Preconditions** — the state and data before the steps.
- **Steps to reproduce** — numbered, exact, from a known starting point. Anyone should
  follow them and hit the same result.
- **Expected result** — what should happen, per the spec/AC (cite it).
- **Actual result** — what actually happened.
- **Evidence** — screenshot/screen recording, API request/response, console/network log,
  backend log line or trace id, DB state.
- **Severity** and **Priority** (see below).
- **Reproducibility** — always / intermittent (how often) / once.

## Severity vs Priority — set both, deliberately

They are different axes:

- **Severity = impact on the system** if it occurs.
  - Critical: data loss/corruption, security hole, payment wrong, app crash/blocker.
  - High: major function broken, no workaround.
  - Medium: function broken but has a workaround.
  - Low: minor/cosmetic.
- **Priority = how soon it should be fixed** (business urgency).
  - P0/Urgent → fix now; P1/High → this release; P2/Medium → soon; P3/Low → backlog.

A cosmetic typo on the landing page can be low severity but high priority (brand); a
rare edge crash can be high severity but lower priority. Don't conflate them.

## Make it reproducible

- Trim the steps to the **minimal** repro. Remove noise.
- Note the exact data/inputs. If it's intermittent, say how often and any pattern
  (timing, sequence, concurrency).
- Attach evidence at the moment of failure, not after navigating away.

## Defect lifecycle (Jira)

`New → Triaged (sev/priority set) → In Progress → Fixed/Resolved → Re-test → Closed`
(or `Reopened` if it fails re-test). On re-test, verify the fix **and** run regression
around it (see `regression-uat.md`). Link the defect to the failing test case and the
requirement so traceability holds.

## Quality bar

- Title specific; environment + repro steps + expected vs actual + evidence all present.
- Severity and priority both set and justified.
- Minimal, reliable reproduction; intermittency noted if any.
- Linked to the test case and requirement; re-test + regression on fix.
