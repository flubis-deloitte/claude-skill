# Tracking & Reporting Playbook

How to track triaged findings to closure and report the state. Load this for the track &
report steps.

## Track each Fix / Investigate as an item (e.g. Multica issue)

- Create one tracked item per **group** (preferred, less noise) or per finding when they
  need separate owners. Title with the class + a stable reference to the Alarm IDs.
- Include: type/sub-type, severity (your re-rated value + the tool's), decision, affected
  assets (list of Alarm IDs / permissions / domains), the remediation steps, the owner,
  and a link back to the source report/date.
- Assign to the owner (mobile team for app-permission findings; FE for web; infra for
  config/infra). *Accept* and *False positive* items don't need an action item — record
  them in the report/decision log instead.

## De-duplicate across runs (dedup by Alarm ID)

- Before creating an item, check whether its Alarm ID(s) are already tracked (search
  existing items for the Alarm ID / the group reference). If tracked, **update** status
  instead of creating a duplicate.
- Keep a decision log (which Alarm IDs are Accepted / False positive) so re-exports of the
  same finding aren't re-triaged from scratch — they map to the prior decision.

## Status lifecycle

`New → Triaged (decision set) → In Remediation → Fixed → Verified (re-scan clear) → Closed`
(or `Accepted` / `False positive` as terminal decisions). Update as owners act; a finding
is only Closed after re-scan/verification confirms it.

## Report

Produce a concise triage summary:

```
Findings review — <source>, <date>
Total: N findings (M after de-dup) across G groups.
By severity (re-rated): CRITICAL x, HIGH x, MEDIUM x, LOW x, INFO x.
By decision: Fix a · Accept b · False positive c · Investigate d.

Top priorities:
- [group] <title> (HIGH) → <remediation> — owner: <owner>
- …

Accepted (with rationale): …
Newly tracked: <links> · Already tracked: <count>
```

Keep it one screen; lead with what matters (the Fixes, highest severity first).

## Safety
- Advise & track only — do not apply production security changes yourself.
- Don't leak sensitive detail (tokens, internal hosts, exact exploit steps) beyond what an
  owner needs to act; keep reports need-to-know.

## Quality bar
- Each Fix/Investigate tracked with owner, severity, decision, remediation, source link.
- De-duplicated by Alarm ID; prior Accept/FP decisions preserved across re-runs.
- Status reflects reality; Closed only after verification.
- Report is prioritized, one screen, and states residual/accepted risk.
