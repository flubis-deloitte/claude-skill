# Confluence · Jira · Figma Playbook (System Analyst)

How to deliver specs in the team's tools. Cross-cutting — read alongside the
deliverable-specific reference.

## Confluence (the spec lives here)

- One spec = one Confluence page, under the right space, with a clear title and status
  (Draft / In Review / Approved). Link the Jira epic and Figma file at the top.
- Use a consistent template (see `srs-spec.md` structure). Lead with a TL;DR and the
  scope box so reviewers orient in seconds.
- Use tables for contracts and the traceability matrix; use Mermaid/diagram macros for
  sequence/flow/ER. Keep the diagram and the written contract in sync.
- Version the page; record decisions in a changelog/decision-log section. Capture
  review comments inline and resolve them before marking Approved.
- Use status macros and a sign-off table (Eng lead, QA, Product) at the bottom.

## Jira (the work is tracked here)

- **Epic** = the initiative (mirrors the Confluence spec). **Story** = one
  user-facing/use-case slice with its own acceptance criteria. **Task/Sub-task** =
  technical steps. **Bug** = defects.
- Each story: clear summary, description linking the spec section, acceptance criteria
  as a checklist, and links to the relevant API/event spec.
- Link types: story → "is specified by" → Confluence; story → "relates to" the events/
  services it touches; dependencies as "blocks/blocked by".
- Keep requirement IDs (FR/NFR/AC) referenced in the issue so traceability survives.

## Figma (the design reference)

- Link the specific frame/flow, not just the file. Reference frames by name in the
  spec so engineers map a screen to its contract.
- The spec is the source of truth for behavior/data; Figma is the source of truth for
  layout/visuals. If they disagree, flag it and resolve — don't let both ship.
- Pull field labels, states (empty/loading/error), and validation hints from Figma
  into the acceptance criteria so the UI states are actually specified.

## Handoff checklist

- [ ] Confluence spec complete, reviewed, and Approved (sign-off table filled)
- [ ] Jira epic + stories created, each with acceptance criteria and spec links
- [ ] Figma frames linked from the relevant spec sections and stories
- [ ] Traceability intact: business need → requirement → story → AC → design
- [ ] Open questions assigned to owners with due dates
