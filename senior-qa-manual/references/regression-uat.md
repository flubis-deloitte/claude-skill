# Regression, Smoke, Exploratory & UAT Playbook

How to run the different manual test cycles. Load this for "regression", "smoke test",
"exploratory testing", "UAT".

## Smoke testing

- A quick, shallow pass over the **critical paths** right after a new build/deploy, to
  decide "is this stable enough to test further?".
- Keep a fixed smoke checklist: login, core flows (e.g. create order → pay → confirm),
  key screens load, no obvious breakage. Minutes, not hours.
- If smoke fails, stop and send the build back — don't waste a full cycle.

## Regression testing

- After a change or fix, verify that **existing functionality still works** — bugs love
  to hide in side effects.
- Scope by risk and impact: the changed area, everything that depends on it, and the
  critical paths. You rarely re-run everything manually — maintain a prioritized
  regression suite (P0/P1 cases) and pick the relevant subset.
- Always run regression around a bug fix, not just re-test the single fixed case.
- Candidates for automation: stable, high-value, frequently-run regression cases —
  flag them for the automation/engineering team.

## Exploratory testing

- Time-boxed, unscripted investigation guided by a **charter** ("explore the refund flow
  with focus on partial refunds and concurrency"). You learn, design, and execute at once.
- Use heuristics: boundaries, interruptions (back button, refresh, offline, kill app),
  unusual sequences, double actions, role switching, stale data.
- Take notes as you go; turn findings into defects and new scripted cases.
- Best for new/complex/high-risk areas and for finding what scripted cases miss.

## User Acceptance Testing (UAT)

- Validates the system meets **business needs** from the user's perspective, usually
  with business stakeholders, on staging with realistic data, before go-live.
- Drive it from the **business acceptance criteria** (from the BA): real-world scenarios,
  end to end, in business language — not technical edge cases.
- Provide UAT scripts, capture sign-off per scenario, and log issues with severity so the
  go/no-go is informed.

## Choosing the cycle

| Situation | Cycle |
|-----------|-------|
| New build just deployed | Smoke |
| Code changed / bug fixed | Regression around the change |
| New/complex/risky feature | Exploratory + scripted |
| Pre-release business validation | UAT |

## Quality bar

- Smoke checklist exists and gates deeper testing.
- Regression scoped by impact; run on every fix; automation candidates flagged.
- Exploratory is charter-driven and documented, not aimless clicking.
- UAT runs from business criteria with explicit sign-off and issue severity.
