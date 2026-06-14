# Publishing & Maintenance Playbook (Confluence · README · Jira · Figma)

How to publish docs where readers will find them and keep them from rotting.
Cross-cutting — read alongside the document-specific reference.

## Where each doc lives

- **Confluence** — guides, architecture/explanation, runbooks, onboarding, release notes,
  API references for cross-team consumers. Organize by space/team; use a clear page tree.
- **Repo README / `/docs`** — what a developer needs at the code: what the service is,
  quickstart (run, test, env), key conventions, and links to the deeper Confluence docs.
  Keep the README short; link out for depth.
- **Single source of truth** — decide where each fact lives and link to it from
  elsewhere. Don't copy content into two places; it will drift.

## Confluence craft

- Lead with a short intro: what this page is and who it's for. Add a table of contents for
  long pages.
- Use headings (proper hierarchy), tables, code blocks, panels/callouts (info/warning),
  and diagram macros (Mermaid) — keep formatting consistent across the space.
- Put **page metadata** at top or bottom: owner, last-reviewed date, status, and the
  related Jira epic + Figma file.
- Use status macros for Draft/Reviewed/Deprecated. Resolve review comments before publishing.

## Linking & traceability

- Link docs to the **Jira** epic/story they describe, and to the **Figma** frames for any
  UI. Reference the **system analyst's spec / ADRs** for the authoritative behavior, and
  link back so readers can go deeper.
- From code/PRs, link the doc; from the doc, link the code/spec — bidirectional so each
  is discoverable from the other.

## Keeping docs alive

- **Owner + last-reviewed date** on every page. Unowned docs rot.
- Update docs **as part of the change** (doc updates in the same PR/story), not "later".
- Mark superseded docs **Deprecated** and point to the replacement; don't leave two
  conflicting truths.
- Periodically review high-traffic docs; after an incident, update the runbook that was
  used.
- Prefer **generated** docs (e.g. from an OpenAPI spec) over hand-maintained copies where
  possible.

## Quality bar

- Doc is in the right place (Confluence vs README) with one source of truth.
- Consistent formatting; owner, date, status, and Jira/Figma links present.
- Bidirectional links between doc and code/spec; deprecated pages marked and redirected.
- A plan for keeping it current (owned, reviewed, updated with the change).
