# Jira · Confluence · Figma Playbook (Web QA)

How QA work lives in the team's tools. Cross-cutting — read alongside the
deliverable-specific reference.

## Jira (test cases, runs, defects)

- **Test cases / runs** tracked in Jira (via the team's test-management approach). Each
  case: preconditions, steps, expected result, priority, mode (manual/automated), and a
  link to the requirement/story it covers.
- **Defects** (current): file in **Multica** for now — a Multica issue or comment with the
  full reproduction (see `bug-reporting.md`), referencing the failing test case, story, and
  PR. (Jira is not the defect destination for now.)
- **Traceability**: requirement/story → test case(s) → result → defect(s). Keep it so
  coverage is visible and nothing ships untested.
- Keep execution status current (pass/fail/blocked) so go/no-go is grounded in data.
- For automated tests, link the test/suite (repo path or CI report) to the case so it's
  clear which cases are covered by automation vs manual.

## Confluence (plans & reports)

- **Test plan/strategy** as a Confluence page (see `test-strategy.md`), linked to the epic,
  stating what's manual vs automated.
- **Execution / release report**: what was tested (manual vs automated), pass/fail counts,
  automated suite status (and any flake), defects by severity, residual risk, untested
  areas, and the go/no-go.

## Figma (UI source of truth)

- Test the UI against the Figma frames: layout, copy, and states (empty/loading/error/
  success). Build vs Figma discrepancies are defects — reference the specific frame.
- Pull edge states and validation messages from Figma into both manual cases and automated
  assertions.
- Where the spec (analyst) and Figma disagree, raise it rather than guessing.

## Readiness checklist

- [ ] Test cases in Jira, each linked to a requirement/story, mode tagged
- [ ] Automated tests linked to the cases they cover; suite green in CI
- [ ] Execution results recorded (pass/fail/blocked) with evidence
- [ ] Defects logged, triaged, linked to cases/stories/PRs
- [ ] Plan + execution report in Confluence with go/no-go and residual risk
- [ ] UI verified against Figma; spec/design conflicts raised
- [ ] Traceability intact: requirement → case → (manual run / automated test) → result → defect
