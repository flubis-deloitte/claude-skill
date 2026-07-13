# Findings Intake Playbook

How to read, normalize, de-duplicate, and group a findings export. Load this for the
ingest step.

## Read the file

- Formats: `.xlsb`, `.xlsx`, `.csv`. For `.xlsb` use pandas + `pyxlsb`
  (`pd.read_excel(path, engine="pyxlsb")`); for `.xlsx` use `openpyxl`; CSV is direct.
- Confirm the header row and map columns to the canonical schema (Alarm ID, Notification
  ID, Product, Main Type, Sub Type, Alarm Title, Description, Affected Domain/Apps/Website,
  Severity, Alarm Date). If headers differ, map synonyms; don't assume position.
- Handle multi-line cells (Description / Affected asset often contain newlines).

## Normalize each row

- **Alarm ID** — the stable unique key (used for dedup and tracking).
- **Severity** — normalize to a common scale: INFO < LOW < MEDIUM < HIGH < CRITICAL.
- **Affected asset** — extract the concrete target (e.g. a permission string like
  `NSCameraUsageDescription` or `android.permission.CAMERA`, a domain, or an endpoint).
- **Date** — parse to ISO; track newest/oldest for the batch.

## De-duplicate

- Exact dupes: same Alarm ID → keep one.
- Re-exports: the same Alarm ID appearing in a later file is the same finding — reconcile
  with what's already tracked (see `tracking-reporting.md`), don't create a new item.

## Group (so you plan once, not per row)

Findings frequently repeat the same issue across many assets/permissions. Group by:
- **(Main Type, Sub Type, Alarm Title)** — the class of issue.
- **Notification ID** — the batch it came in.

Example: 12 rows of "Configuration Weakness / Mobile Application Security / Company Mobile
Application Permissions Disclosed", each a different permission
(`NSContactsUsageDescription`, `android.permission.CAMERA`, …), are **one issue** —
"the app declares these permissions" — with a list of affected permissions. Triage the
group once; list the members.

## Output of intake

A normalized, de-duplicated set:
- Per group: type, sub type, title, severity, count, list of affected assets, dates.
- Per finding: Alarm ID, affected asset, severity, date (kept for tracking/dedup).

## Quality bar
- Actual columns confirmed and mapped; multi-line cells handled.
- Severity normalized; affected asset extracted per row.
- De-duplicated by Alarm ID; grouped by type + notification so planning is per-issue.
