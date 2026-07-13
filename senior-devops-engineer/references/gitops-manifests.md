# GitOps & Manifest Management Playbook

How to manage Kubernetes manifests as code and deploy via Git. Load this for repo-based
manifest work (e.g. `myptm-service-deployment`), Helm, Kustomize, and GitOps.

## Principle: the repo is the source of truth

In GitOps, the desired cluster state lives in Git; a controller (Argo CD / Flux) or a
pipeline reconciles the cluster to match. So:
- **Change the cluster by changing the repo** (commit + PR), not by `kubectl edit`/`apply`
  by hand. Manual changes drift and get reverted by the reconciler.
- Read the real manifests from the repo to know current desired state (selectors, replicas,
  resources) — see how the PDB task sources selectors from `myptm-service-deployment`.

## Working in the deployment repo

- **Learn the layout first**: per-service folders? environment overlays (dev/staging/prod)?
  Helm charts or Kustomize bases+overlays? Follow the existing convention exactly when adding
  files (e.g. PDBs).
- **Branch + PR**: never commit straight to the default branch; open a PR, let review + the
  pipeline validate, don't merge your own.
- **Keep diffs minimal and reviewable**: one logical change per PR; don't reformat unrelated
  files.

## Helm

- Charts template manifests from `values.yaml`. Change behavior via values/overrides, not by
  editing rendered output. `helm template` to render locally and inspect; `helm diff` (plugin)
  before upgrade. Pin chart + app versions.

## Kustomize

- `base/` + `overlays/<env>/`. Overlays patch the base (replicas, images, resources per env).
  `kustomize build overlays/prod` to render. Add new resources to the base or the right
  overlay + `kustomization.yaml`.

## Validation before merge

- Render (helm template / kustomize build) and lint the output.
- `kubectl apply --dry-run=server` (or `--dry-run=client` if no cluster) / `kubeconform` /
  `kubeval` for schema. Policy checks (OPA/Conftest, Kyverno) if the repo has them.
- Let the pipeline run its checks; fix in the manifest/generator, not by hand-editing rendered
  YAML.

## Rollback

- Revert the PR/commit — the reconciler rolls the cluster back. That's the audit trail.

## Quality bar
- Changes go through the repo (commit/PR), matching the repo's layout/tooling (Helm/Kustomize).
- Rendered + schema-validated before merge; minimal, reviewable diffs; no manual cluster edits.
- Rollback = revert in Git.
