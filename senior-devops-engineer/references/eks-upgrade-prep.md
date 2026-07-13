# EKS Upgrade Preparation Playbook

Context and checklist for preparing a MyPertamina EKS cluster for a version upgrade (e.g.
1.30 → 1.33) and the node rollout. Load this for upgrade-prep tasks. This file is about
**preparation & safe rollout**; do NOT drain/upgrade unless that is explicitly the task.

## Where PDBs fit

During a node upgrade, nodes are **drained** (pods evicted) and replaced. Without PDBs, a
drain can evict all pods of a service at once → outage. PDBs (`maxUnavailable: 1`, real
selectors — see `pdb-management.md`) ensure at most one pod per workload is disrupted at a
time. **Install PDBs before any drain.**

## Pre-upgrade checklist

- **PDBs in place** for all critical workloads, each with `ALLOWED DISRUPTIONS ≥ 1`.
- **Replica sanity**: every protected workload has ≥ 2 ready replicas so a disruption is
  allowed. If replicas will be scaled down for the window, confirm PDBs use `maxUnavailable`
  (relative), not absolute `minAvailable`, so they don't deadlock the drain.
- **Version skew & deprecations**: check for APIs removed in the target version; confirm
  addons (CNI, CoreDNS, kube-proxy), controllers, and CRDs support the target Kubernetes
  version.
- **Capacity headroom**: enough node capacity (or surge/extra nodes) so evicted pods can
  reschedule during the drain. Check allocatable CPU/mem vs requests.
- **HPA/Cluster-Autoscaler behavior** understood so autoscaling doesn't fight the drain.
- **Maintenance window** agreed (e.g. 00:00–03:00 WIB) and stakeholders notified.
- **Backups / rollback plan** and a health baseline (SLOs, dashboards) captured before start.

## Order of operations (high level — only when the task authorizes it)

1. Upgrade the **control plane** to the target minor version (one minor at a time:
   1.30 → 1.31 → 1.32 → 1.33).
2. Upgrade **addons** to compatible versions.
3. Roll the **node groups**: for each, cordon + drain nodes (respecting PDBs), let pods
   reschedule, then replace/upgrade the node. Proceed node-by-node; watch PDBs and
   readiness.
4. Verify workloads healthy after each node and at the end.

## Guardrails specific to upgrades

- **Never drain a node while a critical workload has `ALLOWED DISRUPTIONS = 0`** — fix the
  PDB/selector or readiness first.
- Drain with `--ignore-daemonsets` and a sane `--timeout`; do not `--force`/`--delete-
  emptydir-data` blindly.
- One node group / one node at a time; verify between steps. Stop on unexpected eviction
  failures and investigate.

## For a preparation-only task
Deliver: PDBs applied + verified, a readiness/replica/capacity report, and a go/no-go note
for the upgrade — but do NOT cordon, drain, or change versions. Draining is a separate,
explicitly-authorized task.
