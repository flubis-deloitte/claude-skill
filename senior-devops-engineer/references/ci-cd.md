# CI/CD Pipeline Playbook

How to build, test, ship, and roll back safely. Load this for pipeline, build, release, and
deployment-automation work.

## Pipeline stages (typical)

1. **Build** — compile/package the app (Go services here), produce a container image tagged
   with the immutable commit SHA (not just `latest`).
2. **Test** — unit + integration + lint + `go test -race`; fail fast. Security/deps scan.
3. **Image scan** — scan the image for CVEs before it can be promoted.
4. **Publish** — push the image to the registry (immutable tag).
5. **Deploy** — update the GitOps manifest / trigger the reconciler for the target env.
6. **Verify** — smoke tests / health checks post-deploy; watch metrics (Datadog).

## Safe delivery practices

- **Immutable, SHA-pinned images** — never deploy a moving `latest`. The manifest references
  the exact tag/digest.
- **Environment promotion** — dev → staging → prod; the same image promoted, config per env
  (overlays/values), gates between stages.
- **Progressive delivery** where possible — rolling update by default; canary/blue-green for
  risky changes; feature flags to decouple deploy from release.
- **Automated rollback triggers** — define what auto-rolls-back (error-rate/latency SLO
  breach) and the manual rollback (revert the manifest PR / redeploy prior image).
- **Idempotent, repeatable** — re-running a deploy is safe; no manual snowflake steps.

## Secrets in pipelines

- Pull secrets from a secret store / CI secret vars, never hard-coded. Least-privilege
  deploy credentials. Don't echo secrets in logs. (See `secrets-security.md`.)

## Change safety

- Migrations & feature flags: run DB migrations backward-compatibly; gate risky behavior
  behind flags so deploy ≠ release. Deploy during/adjacent to the agreed window for risky
  changes.
- A green pipeline is necessary, not sufficient — verify the live effect + metrics after
  deploy (see `observability-datadog.md`).

## Rollback

- Fastest safe path: redeploy the previous known-good image / revert the GitOps PR. Know it
  before you ship. Capture what broke for the postmortem (`incident-slo.md`).

## Quality bar
- SHA-pinned images; tests + scans gate promotion; config per env.
- Rollback path defined and fast; secrets never in code/logs; post-deploy verification done.
