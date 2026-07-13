# Observability with Datadog Playbook

How to use Datadog (MyPertamina is on the EU1 site) to verify state, investigate, and
monitor — including via a connected Datadog MCP. Load this for verification, alerting, and
investigation work.

## The four golden signals (what to look at)

- **Latency** — request duration (p50/p95/p99). Watch the tail, not just the average.
- **Traffic** — throughput (req/s, events/s, Kafka lag).
- **Errors** — error rate / 5xx / failed jobs.
- **Saturation** — CPU/mem vs requests/limits, pool/queue depth, node pressure.
Add **RED** (Rate, Errors, Duration) per service and **USE** (Utilization, Saturation,
Errors) per resource.

## What Datadog gives you (and the MCP can query)

- **Metrics** — infra + custom + APM-derived; timeseries and aggregates.
- **Monitors & alerts** — thresholds/anomaly/forecast; check status and history.
- **Dashboards** — service and platform views.
- **Logs** — search/filter; correlate with traces (trace_id).
- **APM traces** — request paths across services; find the slow/erroring hop.
- **Service Catalog / SLOs** — ownership, dependencies, and SLO burn.
- **Infra/K8s** — pods, nodes, deployments, restarts, readiness.

## Verifying before a change (e.g. the PDB task)

- Confirm a service is **live & monitored** (service catalog / emitting metrics/pods).
- Check **ready replica/pod count** — a `maxUnavailable: 1` PDB needs ≥2 ready to allow a
  disruption; flag <2.
- Look at error rate/throughput to tell high-traffic from idle. Summarize per service.

## Investigating an issue

1. Start from the alert/symptom and the affected service's dashboard.
2. Golden signals → which signal is off (latency? errors? saturation?).
3. Correlate: metrics → logs → traces (same time window / trace_id) to localize the failing
   hop (service, DB, Kafka, downstream).
4. Check recent deploys/changes (deploy markers) as the first suspect.

## Good monitors & alerts

- Alert on **symptoms users feel** (SLO burn, error rate, latency), not just raw CPU.
- Multi-window/multi-burn-rate for SLOs; sane thresholds to avoid alert fatigue; every alert
  is **actionable** with a runbook link (see `incident-slo.md`).

## Safety
- Use a **read-only, least-privilege** Datadog service account for automated/MCP access.
  Don't leak dashboards' sensitive detail beyond need-to-know.

## Quality bar
- Judgments backed by the golden signals + correlated logs/traces, not one metric.
- Pre-change checks (live? #ready? healthy?) done and reported.
- Alerts are symptom-based, actionable, and low-noise.
