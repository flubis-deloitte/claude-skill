# Jira Query Playbook — finding new & reopened issues

How to find exactly the issues that need intake, and nothing else. Load this for the
"find candidates" step.

## Scope

- **Assignee**: the user. Use `assignee = currentUser()` if the connector runs as the
  user, or `assignee = "<accountId/email>"` from the agent config.
- **Project(s)**: restrict to the relevant project key(s) (e.g. `project = KANBAN`) to
  avoid noise. Get the key(s) from the agent instructions/env.

## Two candidate sets

### 1. Newly assigned / created since last run
Issues that appeared on the user's plate since the previous run window.

```
assignee = currentUser()
AND project in (<KEYS>)
AND (created >= "-70m" OR updated >= "-70m")
AND statusCategory != Done
ORDER BY updated DESC
```

- Use a window a bit larger than the autopilot interval to avoid gaps (e.g. run every
  60m, query the last 70m). De-dup (see `dedup-and-safety.md`) handles the overlap.
- `statusCategory != Done` avoids pulling closed tickets; adjust if you want closed-but-
  assigned too.

### 2. Reopened
Issues that moved back from a done/closed state to an active one.

```
assignee = currentUser()
AND project in (<KEYS>)
AND status changed FROM ("Done", "Closed", "Resolved") TO ("Reopened", "To Do", "In Progress")
AFTER "-70m"
ORDER BY updated DESC
```

- Adjust the exact status names to the project's workflow. If the project has an explicit
  `Reopened` status, `status = Reopened AND updated >= "-70m"` is simpler.

## How to run it

- Use the Atlassian/Jira connector's JQL search tool (e.g. searchJiraIssuesUsingJql) —
  don't scrape the UI.
- Request the fields you need for replication: `key, summary, description, priority,
  issuetype, status, assignee, reporter, created, updated, labels, components,
  fixVersions, issuelinks, subtasks, attachment`, plus any custom fields for acceptance
  criteria / FSD / Figma links the project uses.
- Page through all results (don't stop at the first page) but cap sensibly; if a run
  returns an unusually large set (e.g. first run over a big backlog), replicate the most
  recent N and note the rest in the summary for a follow-up.

## Time-window strategy

- Prefer an **overlapping window** (query slightly more than the interval) + robust
  dedup, over trying to track an exact "last run" timestamp — it's simpler and safe.
- If you do keep a last-run marker (state file / label), use it to set the lower bound
  and still keep a small overlap.

## Quality bar
- Query scoped to the user + relevant project(s); both new and reopened covered.
- Window overlaps the schedule so nothing falls between runs.
- All needed fields fetched; results fully paged; huge first-run backlogs handled gracefully.
