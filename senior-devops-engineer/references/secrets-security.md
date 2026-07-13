# Secrets & Platform Security Playbook

Keeping the cluster and pipeline secure. Load this for secrets, RBAC, image, and network
security work.

## Secrets

- **Never commit plaintext secrets** to Git. Use a secret manager (AWS Secrets Manager /
  SSM / Vault) surfaced to pods via External Secrets Operator / CSI, or sealed/encrypted
  secrets (SOPS, Sealed Secrets) if secrets must live in the GitOps repo.
- Kubernetes `Secret` is base64, **not encrypted at rest by default** — enable envelope
  encryption (KMS) on the cluster. Restrict who can read Secrets via RBAC.
- Rotate regularly; scope each credential to the minimum (per-service, read-only where
  possible). Don't print secrets in logs or CI output.

## RBAC & least privilege

- Humans, CI, and agents get **scoped** roles (namespace-limited, verb-limited), never blanket
  cluster-admin. Prefer per-namespace RoleBindings.
- **IRSA** for pods needing AWS access (scoped IAM role via service account), not node creds.
- Automated/agent access (e.g. Datadog MCP, kubectl for the agent) uses a dedicated,
  least-privilege service account.

## Image & supply-chain security

- Scan images for CVEs in CI; block promotion on criticals. Use minimal base images; pin
  digests. Prefer signed images (cosign) if the platform supports it.
- Pull from trusted registries only; keep base images patched.

## Network & runtime

- **Network policies** to restrict pod-to-pod/egress to what's needed (default-deny where
  feasible).
- Pod security: run as non-root, read-only root FS, drop capabilities, no privileged
  containers unless required (Pod Security Standards / admission policy).
- Ingress: TLS everywhere; WAF on public endpoints; sane timeouts.

## Handling findings
- Security scan/audit findings: confirm against the real config, fix at the source (manifest/
  IaC/pipeline), and track to closure — advise, don't apply prod changes unless that's the
  task (mirror the security-findings-triage approach).

## Quality bar
- No plaintext secrets in Git; secrets from a manager, KMS-encrypted, rotated, least-scope.
- Scoped RBAC + IRSA (no blanket admin); images scanned & pinned; network policies + non-root
  pods; TLS on ingress.
