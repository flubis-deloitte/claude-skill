# Replicate-to-Multica Playbook

How to turn a Jira issue into a Multica issue the System Analyst can act on. Load this
for the "replicate" step.

## Field mapping

| Multica issue | From Jira |
|---------------|-----------|
| Title | `[<JIRA-KEY>] <summary>` — always prefix the key (also powers dedup) |
| Description | See the template below |
| Priority | Map Jira priority → Multica priority (Highest/High→High, Medium→Medium, Low/Lowest→Low) |
| Assignee | Leave unassigned, or assign to the **System Analyst** per config — never FE/BE |
| Labels/tags | Carry issue type + a `jira-intake` tag if supported |

## Description template

Write a self-contained description so the SA doesn't have to open Jira to triage:

```
Source: Jira <JIRA-KEY> — <URL>
Type: <issuetype>   Priority: <priority>   Status in Jira: <status>
Reporter: <reporter>   Created: <date>   (Reopened: yes/no)

## Summary
<Jira summary>

## Description
<Jira description, converted to readable markdown>

## Acceptance criteria
<from the AC field / description, if present>

## Links & attachments
- Figma: <links found in the ticket>
- FSD / docs: <links / attachment names>
- Linked issues: <keys + relationship>
- Sub-tasks: <keys + titles>

## Intake notes
Replicated by the Jira intake agent on <UTC datetime>. For the System Analyst to
triage and route to FE/BE.
```

- Convert Jira markup/ADF to clean markdown; keep tables/lists readable.
- Include **every link** in the ticket (Figma, FSD, Confluence) — those are what the SA
  needs. Note attachments by name (and that they live on the Jira ticket) since binaries
  may not transfer automatically.
- Don't editorialize the requirement; replicate faithfully. Add only the "Intake notes".

## Creating the issue

- Use the environment's Multica issue-creation path (e.g. the `multica` CLI on the
  daemon). Confirm the exact command in your runtime before the first real run; do a
  dry-run/test issue once.
- Set the title, description, and priority per the mapping. Record the created issue's
  ID/URL for the run summary and for dedup.

## Fallback (if you can't create issues programmatically)

If issue creation isn't available in the run environment, **do not fail silently**.
Post a structured, copy-ready block on the autopilot run issue — one section per Jira
ticket using the description template above — and flag it clearly: "Could not create
Multica issues automatically; please create these N issues." That still ensures nothing
is missed.

## Quality bar
- Title carries the Jira key; description is self-contained (summary, AC, all links).
- Priority mapped; left for SA to route (not assigned to FE/BE).
- Created issue ID/URL captured; attachments/links preserved; faithful to the ticket.
