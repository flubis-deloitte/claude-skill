# De-duplication & Safety Playbook

The intake agent runs repeatedly and unattended, so preventing duplicates and staying
read-only on Jira is critical. Load this for the "de-duplicate" step.

## De-duplication (the golden rule: at most one Multica issue per Jira key)

Before creating a Multica issue for `<JIRA-KEY>`, confirm it isn't already there. Use one
of these, in order of preference:

1. **Search existing Multica issues by key** — the reliable method. Every replicated
   issue's title starts with `[<JIRA-KEY>]`. Search the Multica board for that key; if a
   match exists, skip (it's already replicated). This survives restarts and needs no
   external state.
2. **Local state file** — keep a small file (e.g. `~/.jira-intake/seen.json`) mapping
   `JIRA-KEY → multica_issue_id + last_seen`. Fast, but only valid on the same machine;
   pair with method 1 as the source of truth.
3. **Jira-side marker (opt-in only)** — add a label like `multica-synced` or a comment to
   the Jira ticket after replicating. This IS a write to Jira, so only do it if the user
   explicitly enables it in the agent config. Default is **read-only Jira**.

Because the query window overlaps between runs, the same ticket will appear in
consecutive runs — dedup is what makes that safe. Always dedup; never assume a fresh
window means fresh tickets.

## Handling reopened tickets

- A reopened ticket may already have an old Multica issue from its first pass. Decide the
  policy and apply it consistently:
  - **Default**: if a Multica issue for the key exists and is open, add a comment
    ("Reopened in Jira on <date> — <what changed>") instead of creating a duplicate.
  - If the previous Multica issue was closed/archived, create a fresh one titled
    `[<JIRA-KEY>] (reopened) <summary>` and note the link to the old one.
- State the reopen explicitly in the description/comment so the SA knows it's a
  re-triage, not a new feature.

## Safety

- **Read-only on Jira by default.** Don't change status, assignee, or fields. Only add a
  label/comment if the dedup-marker option is explicitly enabled.
- **Idempotent.** Re-running the same window must not create duplicates or side effects.
- **Least privilege.** Use a Jira token scoped to read the relevant project(s).
- **No secrets in output.** Don't print tokens or full auth context in the run summary.
- **Fail loud, not silent.** If dedup can't be verified (e.g. Multica search unavailable),
  do NOT blindly create issues — pause creation and report, so you don't spam duplicates.
- **Cap per run.** Guard against a runaway first run over a large backlog: replicate the
  most recent N, list the rest, and let the user confirm before a bulk backfill.

## Quality bar
- Every candidate checked against existing Multica issues (by Jira key) before creation.
- Reopened handling is consistent (comment vs new issue) and states the reopen.
- Jira left unmodified by default; writes only if explicitly enabled.
- Idempotent across overlapping windows; large backfills capped and surfaced.
