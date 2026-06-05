# Architecture Decision Record (ADR) Playbook

How to capture a significant technical decision so the org can follow it and revisit
it later. Load this for "ADR", "which technology should we use", "document this
decision".

## When to write an ADR

Write one for any **Type-1 (hard to reverse)** or org-wide decision: a service split,
a new datastore or messaging tech, a public API/event contract, an auth approach, a
cross-cutting library, a major pattern change. Skip ADRs for trivial, easily-reversed
choices.

## Format (one Confluence page per ADR)

```
# ADR-<n>: <short decision title>
Status: Proposed | Accepted | Superseded by ADR-<m> | Deprecated
Date: YYYY-MM-DD   Owner: <name>   Deciders: <names>

## Context
The problem, the forces at play, constraints (scale, latency, cost, compliance,
time), and the assumptions. Link the driving Jira/Confluence item.

## Options considered
For each option (always include "do nothing"):
- Summary
- Pros
- Cons
- Cost / risk / effort

## Decision
The option chosen, in one or two sentences.

## Rationale
Why this option beats the others against the decision criteria.

## Consequences
- Positive: what gets better.
- Negative / trade-offs: what we accept or take on (debt, lock-in, ops load).
- Follow-ups: migrations, deprecations, new standards, things to revisit.
```

## Making a sound decision

- Define **decision criteria** before comparing (e.g. latency, operational cost,
  team familiarity, scalability ceiling, lock-in, time-to-market) and weight them.
- Generate **real** alternatives — at least two plus do-nothing. A single option with
  a rubber-stamp is not a decision.
- Decide with **evidence**: benchmarks, prototypes, capacity math, prior incidents —
  not vibes or hype.
- Be honest about **trade-offs and reversibility**. State what would make you revisit
  this (the conditions under which the decision no longer holds).

## Lifecycle

- ADRs are immutable once Accepted; you don't edit history — you supersede with a new
  ADR and mark the old one "Superseded by ADR-n".
- Keep an ADR index in Confluence. Reference the ADR number from affected services'
  docs and from the relevant Jira epics.

## Quality bar

- Clear context and constraints; criteria stated.
- ≥2 options + do-nothing, each with pros/cons/cost.
- Decision, rationale tied to criteria, and consequences (incl. negatives) explicit.
- Status and owner set; revisit conditions noted; indexed and linked.
