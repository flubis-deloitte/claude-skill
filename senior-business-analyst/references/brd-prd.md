# BRD / PRD Playbook

How to write a Business Requirements Document (the "why/what" for stakeholders) or a
Product Requirements Document (the "what to build" for the delivery team). Load this
for "BRD", "PRD", "requirements doc".

## BRD vs PRD — pick the right one

- **BRD** — business-facing. The problem, objectives, scope, business requirements,
  success metrics, stakeholders, and high-level solution direction. Audience:
  sponsors, business owners, leadership.
- **PRD** — delivery-facing. The product behavior, user stories, acceptance criteria,
  UX (Figma), non-functional needs, and release plan. Audience: product, design,
  engineering, QA.

A small effort may merge them; a large initiative usually has a BRD that spawns one or
more PRDs.

## Structure (Confluence template)

1. **TL;DR** — 3–5 sentences: problem, who it affects, proposed direction, expected
   outcome.
2. **Problem statement** — the business problem, evidence (data/feedback), and the cost
   of doing nothing.
3. **Goals & non-goals** — what success looks like; what this explicitly will *not* do.
4. **Success metrics / KPIs** — measurable, with baseline and target (link
   `stakeholder-kpi.md`).
5. **Stakeholders** — who's involved and their interest (link `stakeholder-kpi.md`).
6. **Scope** — in scope, out of scope, future/phased.
7. **Requirements** — prioritized (MoSCoW), each testable and traceable (table below).
8. **User stories & acceptance criteria** (PRD) — link `user-stories.md`.
9. **Current vs future process** (where relevant) — link `process-gap-analysis.md`.
10. **UX / design** — Figma links, key screens, states.
11. **Assumptions, constraints, dependencies, risks** — incl. compliance, ops cost.
12. **Release / rollout plan** — phasing, flags, success checkpoints.
13. **Open questions** — owner + decision needed.

## Requirements table

| ID | Requirement | Priority (MoSCoW) | Goal it serves | Metric | Story/AC |
|----|-------------|-------------------|----------------|--------|----------|
| BR-1 | Customer can pay a fuel order via MPS | Must | Increase digital payment share | % digital txns | PTM-101 / AC-1 |

## Writing requirements (business level)

- Format: `BR-<n>: <Who> needs <capability> so that <business value>.`
- Solution-agnostic where possible — describe the need, not the implementation.
- Each is testable: you can point to evidence it's satisfied.
- Prioritize honestly; justify every "Must".

## Quality bar

- Leads with problem and value, not a feature list.
- Goals each have a measurable metric (baseline + target).
- Requirements prioritized, testable, and traceable to goals and stories.
- Scope, assumptions, constraints, dependencies, risks, and open questions explicit.
- Right document type for the audience; Figma linked for UX.
