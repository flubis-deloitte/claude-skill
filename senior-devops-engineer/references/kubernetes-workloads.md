# Kubernetes Workloads Playbook

Core workload objects and the settings that make them reliable on EKS. Load this for
deployment/service/scaling/config work.

## Deployments

- **Replicas ≥ 2** for anything that must survive a node drain (and to let a PDB with
  `maxUnavailable: 1` allow a disruption). Singletons need a plan (leader election / accept
  downtime).
- **Rolling update strategy**: set `maxUnavailable` / `maxSurge` so updates don't drop
  capacity; keep `maxUnavailable: 0` (or 1) + `maxSurge: 1` for zero/low-downtime rollouts.
- **Resource requests & limits** on every container: requests drive scheduling & bin-packing;
  limits cap noisy neighbors. Set requests from real usage (Datadog), not guesses. Right-size
  to avoid wasting node capacity (see `scaling-capacity.md`).
- **Probes**: `readinessProbe` (gates traffic — a pod not ready is pulled from the Service &
  counts against PDB), `livenessProbe` (restarts a hung pod), `startupProbe` for slow starts.
  Wrong probes cause false disruptions and outages.
- **Graceful shutdown**: handle SIGTERM, set `terminationGracePeriodSeconds`, drain
  in-flight requests; a `preStop` sleep helps LB deregistration.
- **Anti-affinity / topology spread** so replicas don't all land on one node/AZ — otherwise
  one node loss takes the whole service.

## Services & networking

- `Service` (ClusterIP) for in-cluster; `Ingress`/ALB for external. Match selectors to pod
  labels exactly.
- Readiness gates traffic; confirm endpoints (`kubectl get endpoints <svc>`) after changes.

## Config & secrets

- `ConfigMap` for non-secret config; `Secret` for credentials (see `secrets-security.md` —
  don't commit plaintext secrets). Mount as env or files; roll pods on change (checksum
  annotation) since env changes don't auto-restart.

## Autoscaling

- **HPA** scales replicas on CPU/mem/custom metrics; ensure `requests` are set (HPA needs
  them). Don't let HPA fight a drain during upgrades (see `eks-upgrade-prep.md`).
- **VPA** (recommend mode) to right-size requests. **Cluster Autoscaler / Karpenter** scales
  nodes — see `aws-eks-infra.md`.

## Rollouts & rollback

- `kubectl rollout status deploy/<name>` to watch; `kubectl rollout undo` to revert (in
  scope only). Prefer GitOps rollback (revert the manifest/PR) for auditability.

## Quality bar
- ≥2 replicas + anti-affinity for critical services; requests/limits set from real data.
- readiness/liveness/startup probes correct; graceful shutdown handled.
- HPA has requests; selectors/endpoints verified; changes rolled safely.
