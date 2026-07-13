# AWS / EKS Infrastructure & IaC Playbook

The cluster, nodes, networking, identity, and how they're provisioned. Load this for
infra/EKS/Terraform/networking/node work.

## EKS shape

- **Control plane** — AWS-managed; you choose the Kubernetes minor version. Upgrade one minor
  at a time (see `eks-upgrade-prep.md`).
- **Node groups** — managed node groups or self-managed / Karpenter. Nodes (e.g. m5.2xlarge)
  run the workloads; capacity + instance type determine how much fits.
- **Addons** — VPC CNI, CoreDNS, kube-proxy, EBS CSI; keep versions compatible with the
  cluster version.

## Networking

- **VPC/subnets/AZs** — spread nodes across AZs for resilience. Pods get VPC IPs (VPC CNI) —
  watch IP exhaustion on large clusters.
- **Ingress** — ALB via AWS Load Balancer Controller (Ingress) or NLB (Service type
  LoadBalancer). TLS termination, target groups, health checks.
- **Network policies** — restrict pod-to-pod traffic (least privilege) where supported.
- **DNS/service discovery** — CoreDNS; ExternalDNS for records if used.

## Identity & access

- **IRSA** (IAM Roles for Service Accounts) — give pods scoped AWS permissions via a service
  account, not node-wide credentials. Least privilege per workload.
- **RBAC + aws-auth** — map IAM principals to Kubernetes RBAC; scope humans/CI to what they
  need, not cluster-admin.

## Scaling the cluster

- **Cluster Autoscaler / Karpenter** — add/remove nodes based on pending pods. Ensure enough
  capacity (or surge) during drains so evicted pods reschedule. Don't let autoscaling fight a
  maintenance drain.
- Right-size instance types to the workload profile; use Spot for tolerant workloads with
  on-demand base.

## Infrastructure as Code

- Provision cluster, node groups, networking, IAM, and addons via **Terraform** (or the repo's
  IaC). Change infra by changing code + plan/apply through review — never click-ops in prod.
- `terraform plan` first; review the diff; `apply` through the pipeline. State is sacred —
  locked remote state, no manual drift.

## Cost & capacity
- Match node capacity to real requests (Datadog); trim over-provisioned requests/replicas.
  Track allocatable vs requested CPU/mem for headroom (see `scaling-capacity.md`).

## Quality bar
- One-minor-at-a-time control-plane upgrades; addons compatible; nodes spread across AZs.
- IRSA + scoped RBAC (no blanket cluster-admin); network policies where possible.
- Infra via reviewed IaC (plan → apply), no manual prod changes; capacity headroom maintained.
