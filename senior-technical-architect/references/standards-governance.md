# Technical Standards & Governance Playbook

How to keep the platform coherent as it grows. Cross-cutting — read alongside the
decision-specific reference.

## Standards (consistency lowers cost)

- **Service shape**: new services follow the reference clean-architecture/CQRS pattern
  (GoFiber + `myptm-go-common-library`, `cmd/web` + `service/...`, facade accessors).
  Don't fork a new architecture without an ADR.
- **Shared libraries**: use `myptm-go-common-library`, `myptm-go-model`,
  `myptm-go-client-library`, `myptm-go-producer-library` for logging, errors, config,
  datasource, models, clients, producers. Promote a genuinely reusable pattern into a
  shared lib instead of copy-pasting; version libs and manage upgrades deliberately.
- **Contracts**: REST and event schemas are versioned; additive within a version;
  breaking changes need a version + deprecation window + ADR.
- **Conventions**: naming, error mapping, pagination, auth, observability instrumentation
  are consistent across services. New cross-cutting conventions are documented once and
  reused.

## Governance (decisions that scale)

- **ADRs** capture Type-1 decisions; keep an indexed ADR log in Confluence (see
  `adr.md`). Architecture decisions are visible and revisitable, not tribal knowledge.
- **Design review** for anything that crosses a service boundary, introduces a new
  tech/datastore/topic, or changes a public contract — before build starts. Reviewers
  check boundaries, data ownership, failure handling, NFRs, and reuse.
- **Tech radar / approved stack**: maintain the list of adopted vs trial vs hold
  technologies so teams don't each pick differently. New tech enters via ADR.
- **Tech-debt register**: track known debt with impact and a paydown plan; fold into
  roadmap. Don't let architecture erode silently.

## Working with the rest of the org

- Hand off detailed component specs to the **system analyst**; business requirements
  come from the **business analyst**; implementation to **backend/frontend engineers**.
  The architect owns the cross-cutting "how" and the decisions that bind multiple teams.
- Decisions live in **Confluence** (ADRs + design docs), trace to **Jira** epics, and
  reference **Figma** for client flows. Keep diagrams and written design in sync.

## Quality bar

- New services/contracts conform to platform standards or carry an ADR for deviating.
- Reuse over reinvention; shared libs used and versioned deliberately.
- Boundary-crossing/new-tech/contract changes go through design review + ADR.
- Approved-stack and tech-debt registers maintained; decisions documented and linked.
