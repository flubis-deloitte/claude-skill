# Scaling & Capacity Planning Playbook

How to size workloads and the cluster, and how to downsize safely for a maintenance window
(as in the EKS migration plan). Load this for capacity, right-sizing, and downsizing tasks.

## Capacity basics

- **Node allocatable** = instance capacity − system/kubelet reserved. Sum of pod **requests**
  (not limits) must fit allocatable across nodes, with headroom.
- Track: allocatable CPU/mem vs total requested; per-service replicas × requests. Use Datadog
  for real utilization to right-size requests (over-set requests waste nodes; under-set risks
  OOM/eviction).

## Right-sizing requests

- Set CPU/mem **requests** from observed p90–p95 usage; set **limits** to protect neighbors
  (mem limit prevents node OOM; be careful with CPU limits/throttling).
- Recommend via VPA (recommendation mode) + Datadog; apply through the manifest repo.

## Scaling replicas

- HPA for dynamic load (needs requests). Keep a sane **min** so a service always has ≥2 ready
  for resilience + PDB compatibility.
- Match replica floors to the criticality (payment/auth high, batch/worker lower).

## Downsizing for a maintenance window (migration plan pattern)

Example sizing rules from the plan (tune per service):
- current ≥ 40 → 10; current 20 → 6; current 10 → 4; current 3–5 → 2 (floor).

Rules to keep it safe:
- **Never scale a critical service below 2 ready replicas** — 1 replica + a `maxUnavailable: 1`
  PDB = `ALLOWED DISRUPTIONS = 0`, which deadlocks the drain.
- Downsize **before** the window so the reduced footprint fits fewer/soon-to-be-drained nodes,
  but keep enough capacity to serve traffic and to reschedule pods during the drain.
- PDBs use `maxUnavailable: 1` (relative) so they stay valid after the scale-down — no need to
  re-edit them (see `pdb-management.md`).
- Apply the scale-down via the manifest repo (GitOps), verify readiness, then proceed with the
  node roll.
- **Scale back up** after the window; confirm HPA/floors are restored.

## Cluster capacity for a drain
- Ensure enough spare node capacity (or surge/extra nodes / Karpenter) so every evicted pod
  can reschedule. Draining without capacity → pending pods → degraded service.

## Quality bar
- Requests sized from real usage; critical services never below 2 ready replicas.
- Downsize planned so traffic is served and pods can reschedule; PDBs stay valid (relative).
- Enough cluster headroom for the drain; scale back up + verify after the window.
