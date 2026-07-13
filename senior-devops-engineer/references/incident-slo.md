# Incident Response & SLO / Reliability Playbook

How to run an incident and how to set reliability targets. Load this for on-call, outages,
postmortems, and SLO/error-budget work.

## Incident response

### Triage & severity
- Assess **user impact** and scope → assign severity (SEV1 total/critical outage, SEV2 major
  degradation, SEV3 minor). Severity drives urgency and who's paged.
- Declare the incident; assign an **Incident Commander** (coordinates), Comms, and Ops. Don't
  debug in silence.

### Mitigate first, root-cause later
- Restore service before finding the cause: roll back the recent deploy, scale up, fail over,
  toggle a feature flag, or shed load. The fastest safe mitigation wins.
- Use observability to localize (golden signals → logs → traces; check recent changes first —
  see `observability-datadog.md`).

### Communicate
- Regular, factual status updates (what's impacted, what you're doing, next update time) to
  stakeholders. One source of truth (incident channel/doc).

### Resolve & verify
- Confirm recovery with metrics/SLO, not assumption. Keep monitoring for recurrence before
  closing.

### Blameless postmortem
- Timeline, impact, root cause(s), what went well/badly, and **action items with owners &
  dates**. Blame systems and gaps, not people. Feed fixes back into runbooks/monitors.

## SLO / reliability

- **SLI** — a measured signal of user experience (availability, latency, error rate).
- **SLO** — the target for an SLI over a window (e.g. 99.9% success over 30d).
- **Error budget** — allowed unreliability (100% − SLO). Spend it on velocity; when it's
  exhausted, slow down risky changes and prioritize reliability.
- **Alert on burn rate** (multi-window multi-burn) so you page before the budget is gone.
- Set SLOs on the user-facing critical paths (payment, QR, auth) first.

## Reliability practices
- Design for failure: timeouts, retries (idempotent), circuit breakers, backpressure,
  graceful degradation. No unbounded synchronous dependency chains.
- Capacity headroom + autoscaling so load spikes and drains don't tip over (see
  `scaling-capacity.md`, `aws-eks-infra.md`).
- Redundancy across nodes/AZs; PDBs so maintenance doesn't drop a service.

## Quality bar
- Incidents: severity set, IC assigned, mitigate-first, clear comms, verified recovery,
  blameless postmortem with owned actions.
- SLOs defined on critical paths with error budgets and burn-rate alerts.
