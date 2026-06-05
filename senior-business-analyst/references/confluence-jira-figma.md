# Confluence · Jira · Figma Playbook (Business Analyst)

How to deliver BA artifacts in the team's tools. Cross-cutting — read alongside the
deliverable-specific reference.

## Confluence (documents live here)

- One BRD/PRD/analysis = one page in the right space, with a status (Draft / In Review
  / Approved) and links to the Jira epic and Figma file at the top.
- Lead with a TL;DR and a scope box. Use tables for the requirements list and the KPI
  table; use Mermaid for process flows; use Gherkin blocks for acceptance criteria.
- Keep a decision log and a sign-off table (sponsor, product, eng lead, QA). Resolve
  review comments before marking Approved.
- Version the page; when scope changes, update the doc and note it in the changelog —
  the doc must stay the single source of truth.

## Jira (work is tracked here)

- **Epic** = the initiative (mirrors the PRD). **Story** = a user-value slice with its
  own acceptance criteria. Tasks/sub-tasks are engineering's breakdown.
- Each story: `As a… I want… so that…` summary, description linking the PRD section,
  acceptance criteria as a checklist, priority (MoSCoW), and dependencies linked.
- Link stories to the epic and "is specified by" the Confluence page; carry requirement
  IDs (BR/AC) into the issues so traceability survives.
- Maintain a groomed, prioritized backlog; mark Definition of Ready before sprint entry.

## Figma (design reference)

- Link the specific frame/flow per story, not just the file. Reference frames by name.
- Behavior/data rules live in the doc; layout/visuals in Figma. If they disagree, flag
  and resolve. Pull UI states (empty/loading/error/success) from Figma into acceptance
  criteria so they're actually specified.

## Handoff checklist

- [ ] Confluence doc complete, reviewed, Approved (sign-off table filled)
- [ ] Jira epic + prioritized stories created, each with acceptance criteria + links
- [ ] Figma frames linked from the relevant sections and stories
- [ ] Traceability intact: goal → requirement → story → AC → metric → design
- [ ] KPIs instrumented/owned so success can actually be measured post-launch
- [ ] Open questions assigned to owners with due dates
