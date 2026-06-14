# Runbook & Operational Docs Playbook

How to write docs someone follows under pressure — on-call at 2am, mid-incident. Load
this for "runbook", "on-call doc", "operational guide".

## What a runbook is for

A runbook lets a responder who didn't build the service **act correctly and fast**.
Optimize for speed and unambiguity, not completeness. Assume stress, partial context,
and no time to read prose.

## Per-runbook structure

1. **Service summary** — what it does, owning team, criticality, dependencies
   (DBs, Kafka topics, downstream services), dashboards & alert links.
2. **Alerts** — for each alert: what it means, likely causes, and the link to the
   diagnostic step below.
3. **Diagnostics** — how to check health: which Datadog/APM dashboard, which logs (and
   how to filter), key metrics (error rate, latency, consumer lag, pool usage), and
   how to read them.
4. **Common scenarios & fixes** — the recurring problems, each as numbered steps:
   symptom → check → action → verify. Include exact commands.
5. **Mitigations** — restart/scale/rollback/feature-flag/failover steps, and when to use
   each. Note which actions are risky and need approval.
6. **Escalation** — who to page and when; the threshold to escalate; links to the
   incident process.
7. **Rollback** — exact rollback procedure and how to confirm it worked.

## Writing for the responder

- **Numbered, copy-pasteable steps.** Exact commands with placeholders clearly marked.
- State the **expected output** of a diagnostic step so they know if it's normal.
- Put the **most common / highest-impact** scenario first.
- Use **callouts** for danger ("⚠ This restarts the consumer group — events will pause").
- No theory. Link to an explanation doc if they want the why; the runbook is for action.
- Keep it **current** — a stale runbook is dangerous. Date it, owner it, and review after
  each incident that used it.

## Related operational docs

- **Incident postmortem** — blameless: timeline, impact, root cause, what went well/
  badly, action items with owners. Feed fixes back into the runbook.
- **Release notes / deployment notes** — what's changing, migration/flag steps, rollback
  trigger, and verification checklist.
- **Onboarding/ops overview** — how the service runs (env, config, deploy, monitoring) for
  a new team member.

## Quality bar

- A responder unfamiliar with the service could act correctly from it.
- Diagnostics name the exact dashboard/logs/metrics and expected values.
- Scenarios are numbered symptom→check→action→verify with real commands.
- Risky actions flagged; escalation and rollback explicit; dated, owned, kept current.
