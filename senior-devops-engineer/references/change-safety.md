# Production Change Safety Playbook

Guardrails for making changes on a production EKS cluster. Cross-cutting — read alongside
any hands-on task.

## Before

- **Right target**: confirm cluster context + namespace (see `kubectl-safety.md`).
- **Scope in writing**: know exactly which resource types you may touch. Everything else is
  off-limits.
- **Blast radius**: understand what the change affects and what happens if it's wrong.
- **Reversibility**: know how to undo (delete the created objects, re-apply prior manifest).

## During

- **Dry-run server-side, then apply.** Apply from saved manifests (idempotent).
- **Batch sensibly**: validate the whole set with a dry-run; apply together only if safe,
  or in small batches if the change is riskier.
- **Skip & log** anything missing/malformed — never fabricate to fill a gap.
- **Do not** change replicas, HPAs, images, resources, or drain/cordon/delete nodes or pods
  unless that IS the task. Additive protective objects (PDBs, labels) are low-risk;
  disruptive verbs need explicit authorization.
- **Stop on surprise**: if dry-run errors, an object behaves unexpectedly, or verification
  fails, halt and report rather than pushing through.

## After

- **Verify the live effect** against the acceptance criteria (e.g. PDB `ALLOWED
  DISRUPTIONS ≥ 1`), not just "apply succeeded".
- **Report**: what changed, the commands/manifests, verification output, skipped items +
  reasons, anomalies, and any follow-up/residual risk.
- **Leave an audit trail**: keep the manifest folder; note the change so it's reproducible.

## Rollback

- Objects created can be removed: `kubectl -n <ns> delete pdb <name>` (or
  `delete -f manifests/`).
- For edits to existing objects, re-apply the previous manifest / `kubectl rollout undo`
  for deployments (only if in scope).

## Never
- Run a write on an unconfirmed context. Guess field values. Apply to prod without a clean
  server-side dry-run. Exceed the task's scope. Declare success without verification.
