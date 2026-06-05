# User Stories & Acceptance Criteria Playbook

How to break work into epics and user stories with acceptance criteria the team can
build and test. Load this for "user stories", "epics", "acceptance criteria",
"backlog".

## Hierarchy

- **Epic** — a large body of work delivering a business outcome (mirrors a PRD).
- **Story** — one slice of user-facing value, deliverable in a sprint.
- **Task/Sub-task** — technical steps (usually owned by engineering, not the BA).

Split epics into stories by user value (a workflow step, a screen, a rule), not by
technical layer ("the API", "the DB") — those are tasks under a story.

## Writing a user story

- Format: `As a <role>, I want <capability>, so that <benefit>.`
- Roles are real personas (customer, merchant, ops admin, partner), not "user".
- Keep the benefit (the "so that") — it justifies the story and guides trade-offs.

Apply **INVEST**: Independent, Negotiable, Valuable, Estimable, Small, Testable. If a
story fails INVEST (too big, no clear value, untestable), reshape or split it.

## Acceptance criteria (Gherkin)

```
Story: As a customer, I want to pay my fuel order with MPS, so that I can pay cashlessly.

AC-1 (happy path)
  Given an order in PENDING state
  When I confirm payment via MPS with sufficient balance
  Then the order becomes PAID
  And I see a success confirmation

AC-2 (insufficient balance)
  Given an order in PENDING state
  When I confirm payment via MPS with insufficient balance
  Then payment is rejected
  And I see an "insufficient balance" message
  And the order stays PENDING
```

Rules:

- One scenario per criterion. Cover happy path, alternates, and failures.
- Concrete, observable outcomes (state, message, screen) — testable by QA.
- Include negative, boundary, empty, and permission cases.
- Pull UI states (loading/empty/error) from the Figma frames into the AC.

## Splitting big stories (patterns)

- By workflow step, by business rule variation, by data type, by happy-path-then-edge,
  by CRUD operation, by persona, or by phase (basic now, enhancements later).

## Grooming / refinement checklist

- [ ] Clear role + capability + benefit
- [ ] Meets INVEST; small enough for one sprint
- [ ] Acceptance criteria in Gherkin, covering edge/negative cases
- [ ] Priority set; dependencies linked
- [ ] Linked to its epic, the PRD section, and the Figma frame
- [ ] Definition of Ready met before it enters a sprint

## Quality bar

- Stories are vertical slices of value, not technical tasks.
- Every story has testable acceptance criteria including unhappy paths.
- Backlog is prioritized and traceable to goals; dependencies explicit.
