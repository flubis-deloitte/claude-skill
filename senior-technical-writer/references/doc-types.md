# Documentation Types Playbook (Diátaxis)

Pick the right type before writing — each has a different reader goal and shape. Don't
mix them on one page; link between them instead. Load this for any "what kind of doc /
how should I structure this" question.

## The four modes (Diátaxis)

| Type | Reader's goal | Mindset | Shape |
|------|---------------|---------|-------|
| **Tutorial** | Learn by doing (first time) | Learning | Numbered, guaranteed-to-work lesson; you hold their hand |
| **How-to guide** | Accomplish a specific task | Working | Numbered steps to a goal; assumes some knowledge |
| **Reference** | Look up precise facts | Working | Dry, complete, consistent; API/config/CLI |
| **Explanation** | Understand the why | Learning | Discursive; concepts, trade-offs, background |

If a page is trying to teach, instruct, list facts, and explain all at once, split it.

## Tutorial

- Goal: a beginner succeeds end-to-end and builds confidence. It must **work every time**.
- Concrete, no choices to make, no detours into theory. State prerequisites and the end
  result up front. Each step shows what they'll see.

## How-to guide

- Goal-oriented: "How to add a new endpoint to a command service", "How to configure
  Redis caching". Numbered steps, real commands. Assume the reader knows the basics.
- Title starts with "How to…". Link to reference for the details; don't repeat them.

## Reference

- For looking things up: API endpoints, config/env vars, CLI flags, data models.
- Accurate, exhaustive, consistent structure, minimal prose. Mirror the structure of the
  thing (one section per endpoint/field). See `api-reference.md`.

## Explanation

- The "why": architecture decisions (link the ADR), design rationale, concepts, how
  pieces fit. Discusses alternatives and trade-offs. No step-by-step.

## Common deliverables and which mode they are

- **README** — usually how-to + a bit of reference (what it is, quickstart, run/test,
  key links). Keep it short; link out for depth.
- **Onboarding guide** — tutorial + how-to (set up env, run a service, make a first
  change) plus links to explanation.
- **API docs** — reference (see `api-reference.md`).
- **Runbook** — how-to under pressure (see `runbooks-ops.md`).
- **Architecture/design doc** — explanation (+ diagrams), links to ADRs.
- **Release notes** — reference (what changed, by audience), concise and skimmable.

## Quality bar

- The page is one mode, or clearly sectioned; cross-links instead of mixing.
- Reader goal and prerequisites stated; structure matches the mode.
- Tutorials work end-to-end; how-tos are goal-titled; references are complete and
  consistent; explanations cover the why and trade-offs.
