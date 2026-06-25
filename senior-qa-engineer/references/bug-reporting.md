# Bug Reporting Playbook (Web)

How to write a defect a developer can reproduce and fix on first read. Load this for
"write a bug report", "log a defect".

> **Where defects go (current):** file bug reports in **Multica** for now — create a
> Multica issue (or comment on the task/issue under test) with the full report below.
> Jira is not the destination for now. (The report structure is the same wherever it's
> filed.)

## Anatomy of a good bug report (Multica issue — for now)

- **Title** — specific: "[Checkout] Promo discount not applied when amount equals minimum".
  Not "checkout bug".
- **Environment** — app version/build, env (staging URL), browser + version + OS, viewport,
  account/role, date-time, and a request/trace id if available.
- **Preconditions** — state, route, and data before the steps.
- **Steps to reproduce** — numbered, exact, from a known start. Anyone follows them and
  hits the same result.
- **Expected result** — what should happen per the spec/AC/Figma (cite it).
- **Actual result** — what happened.
- **Evidence** — screenshot/screen recording, the failing network request/response
  (DevTools), console errors, and backend state where relevant.
- **Severity** and **Priority** (below).
- **Reproducibility** — always / intermittent (how often) / once; browsers affected.

## Severity vs Priority — set both

- **Severity = impact**: Critical (data loss/security/payment wrong/blocker) > High (major
  function broken, no workaround) > Medium (workaround exists) > Low (cosmetic).
- **Priority = urgency**: P0 fix now > P1 this release > P2 soon > P3 backlog.

They're independent: a cosmetic typo on the homepage can be low severity, high priority; a
rare edge crash can be high severity, lower priority. Don't conflate them.

## Make it reproducible

- Trim to the **minimal** repro; remove noise. Note exact data/inputs.
- If intermittent, say how often and any pattern (timing, sequence, specific browser).
- Capture evidence **at the moment of failure** (including the network panel), before
  navigating away.

## Lifecycle (Multica — for now)

`New → Triaged (sev/priority) → In Progress → Fixed → Re-test → Closed` (or `Reopened`).
Track this as the Multica issue's status/comments. On re-test, verify the fix **and** run
regression (automated suite + targeted manual). Reference the failing test case, the
story/task, and the PR in the issue.

## Quality bar

- Specific title; environment (incl. browser), repro steps, expected vs actual, evidence.
- Severity and priority both set and justified; browsers affected noted.
- Minimal reliable reproduction; filed in Multica, referencing the test case, story, and PR.
