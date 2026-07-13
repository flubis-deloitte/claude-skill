# Safe kubectl Playbook

Patterns for operating a production cluster without causing an incident. Load this
alongside any hands-on task.

## Confirm context every time (before any write)

```bash
kubectl config current-context            # which cluster am I on?
kubectl config view --minify -o jsonpath='{..namespace}'   # default namespace
```

- Verify the cluster is the intended one (e.g. `ppnmp-eks-prd-coreapps`) and always pass
  `-n <namespace>` explicitly rather than relying on the default.
- If the context is wrong or ambiguous, STOP and fix it before running anything that writes.

## Read before write

- Inspect the live object as JSON and pull exactly what you need with `jq`:
  ```bash
  kubectl -n mypertamina get deploy <name> -o json | jq '.spec.selector.matchLabels'
  ```
- Never assume field values (labels, selectors, ports). Read them.
- List/scan safely: `kubectl -n mypertamina get deploy -o name`, `get pdb`, `get hpa`.

## Validate before applying

- **Always** `--dry-run=server` first (server-side validates against admission + schema):
  ```bash
  kubectl apply -f manifests/ --dry-run=server
  ```
  `--dry-run=client` only checks local syntax — prefer server.
- Review the diff when changing existing objects: `kubectl diff -f manifests/`.

## Apply idempotently

- Use `kubectl apply -f` with saved manifest files (declarative, re-runnable). Avoid
  imperative one-offs (`kubectl create`, `edit`, `patch`) for anything you want to be
  repeatable and auditable.
- Keep manifests in a folder; the folder is the record of the change.

## jq patterns

- Extract a selector: `jq -c '.spec.selector.matchLabels'`
- Guard for missing/empty: treat `null` / `{}` / empty as "skip".
- Build a manifest safely (avoids quoting bugs): `jq -n --arg ... --argjson sel "$SEL" '{...}'`.
- Iterate a list from a file: `while read DEP; do ...; done < deployments.txt`.

## Scope discipline

- Touch only the resource type the task defines. Creating PDBs does not mean scaling,
  editing deployments, or draining nodes.
- Disruptive verbs (`drain`, `cordon`, `delete`, `scale`, `rollout restart`) are OFF unless
  the task explicitly asks for them — and even then, per `change-safety.md`.

## After any change
Verify with a live query, confirm acceptance, and capture the output for the report.
