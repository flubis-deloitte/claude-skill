# PodDisruptionBudget (PDB) Management Playbook

How to create, apply, and verify PDBs for a set of deployments — safely, from live
state. Load this for "create PDBs / PodDisruptionBudget" tasks.

## Why PDBs, and why `maxUnavailable: 1`

A PDB caps how many pods of a workload can be voluntarily disrupted at once (e.g. during a
node drain in a rolling EKS upgrade), so a service never loses all its pods.

- **Use `maxUnavailable: 1`** for rolling-upgrade protection. It is **relative** — it stays
  valid even when the deployment's replicas are scaled down later for a maintenance window.
- **Do NOT use an absolute `minAvailable: <N>`** for workloads that will be scaled down. If
  `minAvailable` ≥ the (reduced) replica count, the drain can **deadlock** (no pod may be
  evicted) and the node never drains. This is the classic upgrade-blocker.
- One healthy replica must be evictable at a time; with `maxUnavailable: 1` and ≥2 ready
  replicas, `ALLOWED DISRUPTIONS` is ≥1.

## The rules (for this class of task)

1. Every PDB uses **`maxUnavailable: 1`** (never an absolute `minAvailable`).
2. `spec.selector.matchLabels` MUST be taken from the **deployment's own**
   `spec.selector.matchLabels` (query per deployment) — never assumed to be `app: <name>`.
3. PDB name: **`<deployment-name>-pdb`**; namespace matches the deployment (e.g. `mypertamina`).
4. Save each manifest as a separate `.yaml` in a folder (e.g. `pdb-manifests/`), then apply.
5. Do NOT change any deployment's replica count. Do NOT start a node drain/upgrade — PDBs only.
6. If a deployment is missing or has no selector, **SKIP and record the name** — don't
   fabricate a PDB.

## Procedure

For each deployment in the list:

```bash
NS=mypertamina
DEP=<deployment-name>

# 1. Read the REAL selector from the live deployment
SEL=$(kubectl -n "$NS" get deploy "$DEP" -o json 2>/dev/null \
      | jq -c '.spec.selector.matchLabels')

# 2. Skip + log if the deployment is missing or has no selector
if [ -z "$SEL" ] || [ "$SEL" = "null" ] || [ "$SEL" = "{}" ]; then
  echo "SKIP: $DEP (not found or no selector)" >> pdb-skipped.log
  continue
fi

# 3. Generate the manifest (maxUnavailable: 1 + the real selector) via jq → YAML-safe JSON
mkdir -p pdb-manifests
jq -n --arg name "${DEP}-pdb" --arg ns "$NS" --argjson sel "$SEL" '
  { apiVersion:"policy/v1", kind:"PodDisruptionBudget",
    metadata:{name:$name, namespace:$ns},
    spec:{ maxUnavailable:1, selector:{ matchLabels:$sel } } }' \
  > "pdb-manifests/pdb-${DEP}.json"
# (kubectl accepts JSON manifests; or convert to .yaml if you prefer)
```

Then validate and apply the whole folder:

```bash
kubectl apply -f pdb-manifests/ --dry-run=server   # validate first
kubectl apply -f pdb-manifests/                     # apply only if dry-run is clean
kubectl -n mypertamina get pdb                      # verify
```

> Note: a PDB manifest is valid JSON or YAML. If the task specifies `.yaml`, emit YAML
> (e.g. build the JSON then `yq -P`, or template the YAML directly) — the content is the
> same: `apiVersion: policy/v1`, `kind: PodDisruptionBudget`, `maxUnavailable: 1`, the real
> `matchLabels`.

## Alternate source: a GitOps deployment repo (e.g. myptm-service-deployment)

When the deployments live in a Git repo of manifests (GitOps), source the selector from the
**repo's deployment manifest**, not the cluster, and **save the PDB file into the repo**:

1. Explore the repo structure; find each service's deployment manifest and the placement
   convention (next to the deployment, or a `pdb/` folder per service). Follow that convention.
2. For each deployment, read its manifest and copy `spec.selector.matchLabels` **exactly**
   into the PDB. Skip + log if the manifest is missing or has no selector.
3. Write one PDB `.yaml` per deployment following the repo convention. Same content:
   `apiVersion: policy/v1`, `kind: PodDisruptionBudget`, `metadata.name: <deploy>-pdb`,
   `metadata.namespace: mypertamina`, `spec.maxUnavailable: 1`, and the real `matchLabels`.
4. Validate each YAML (`kubectl apply --dry-run=client -f <file>` if available, else static
   YAML + PDB-schema check).
5. **Do not apply to the cluster.** Commit the files on a feature branch and open a PR;
   the repo's GitOps pipeline applies them after review/merge. Don't merge your own PR.

Sourcing the selector from the real manifest (not `app: <name>`) is what makes the PDB
correct — read it, never assume.

## Pre-implementation verification via Datadog (when a Datadog MCP is connected)

Before generating/committing PDBs, cross-check each service in Datadog — a PDB is only safe
and useful if the service is actually running with enough replicas:

- **Live & monitored?** Confirm the service exists in Datadog (service catalog / emitting
  metrics or pods). A name in the list that Datadog doesn't know may be dead/renamed → FLAG,
  don't silently create a PDB for a ghost.
- **≥ 2 ready replicas?** A PDB with `maxUnavailable: 1` on a service with only **1** replica
  yields `ALLOWED DISRUPTIONS = 0` → it will **block** the node drain. For any service with
  < 2 ready pods, FLAG it (raise replicas before the window, or exclude it) — report it
  clearly rather than shipping a PDB that deadlocks the upgrade.
- **Health/traffic sanity:** a quick error-rate/throughput look distinguishes real
  high-traffic services from idle ones (informs priority, not whether to make the PDB).

Summarize the Datadog check per service (live? / #ready replicas / healthy? / note) and put
it in the PR description. This verification does not change the PDB content (still
`maxUnavailable: 1` + real selector) — it catches services where a PDB would be wrong or
counterproductive.

## Verify (acceptance)

- Every PDB applied; `ALLOWED DISRUPTIONS` column is **≥ 1**.
- A PDB showing **`ALLOWED DISRUPTIONS = 0`** means it's blocking eviction — usually the
  **selector is wrong** (matches 0 pods or the wrong set) or **pods aren't Ready**. Report
  it; don't leave it silently.
- Cross-check count: PDBs created + skipped = deployments in the list.

## Report
- Number of PDBs created; list of SKIPPED (with reason).
- `kubectl -n mypertamina get pdb` output.
- Any PDB with `ALLOWED DISRUPTIONS = 0` and the likely cause.

## Context to remember
PDBs are installed now while replicas are still high; because they use `maxUnavailable: 1`
(relative), they remain valid when replicas are scaled down for the maintenance window — no
need to edit them again before the upgrade.
