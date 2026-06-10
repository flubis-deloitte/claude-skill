# Jira · Confluence · Figma Playbook (QA)

How QA work lives in the team's tools. Cross-cutting — read alongside the
deliverable-specific reference.

## Jira (test cases, runs, defects)

- **Test cases / test runs** tracked in Jira (via the team's test-management approach —
  e.g. Xray/Zephyr or a structured issue type). Each case: preconditions, steps,
  expected result, priority, and a link to the requirement/story it covers.
- **Defects** are Jira issues with full reproduction (see `bug-reporting.md`), linked to
  the failing test case and the story/requirement.
- **Traceability**: requirement/story → test case(s) → execution result → defect(s).
  Maintain it so coverage and quality are visible, and nothing ships untested.
- Keep execution status current (pass/fail/blocked) so the team sees real progress and
  the go/no-go is grounded in data.

## Confluence (plans & reports)

- **Test plan/strategy** as a Confluence page (see `test-plan-strategy.md`), linked to
  the epic.
- **Test execution / release report**: what was tested, pass/fail counts, defects by
  severity, residual risks, untested areas, and the go/no-go recommendation.
- Link the plan, the Jira test cases/runs, and the defect summary so reviewers can drill in.

## Figma (the design source of truth for UI)

- Test the UI against the **Figma** frames: layout, copy, states (empty/loading/error/
  success), and interaction. Discrepancies between the build and Figma are defects —
  reference the specific frame.
- Pull edge states and validation messages from Figma into test cases so the UI states
  are actually verified.
- Where the spec (system analyst) and Figma disagree, raise it rather than guessing which
  is correct.

## Handoff / readiness checklist

- [ ] Test cases in Jira, each linked to a requirement/story
- [ ] Execution results recorded (pass/fail/blocked) with evidence
- [ ] Defects logged, triaged (sev/priority), linked to cases and stories
- [ ] Test plan + execution report in Confluence, with go/no-go and residual risk
- [ ] UI verified against Figma; spec/design conflicts raised
- [ ] Traceability intact: requirement → case → result → defect
