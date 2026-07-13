# Triage & Remediation Playbook

How to turn a normalized finding (or group) into a decision + action. Load this for the
triage step.

## The four decisions

Assign every finding/group exactly one:

- **Fix** — a real, worth-fixing issue. Needs concrete remediation + owner + priority.
- **Accept (risk accepted)** — real but acceptable for a documented reason (business need,
  compensating control, low impact). Record the rationale and who accepted it.
- **False positive** — the tool is wrong (misdetection, not applicable). Record why.
- **Needs investigation** — can't decide yet; assign an investigator and what to check.

## Assessing true severity (don't just trust the label)

Re-rate by **impact × exploitability**, then reconcile with the tool's severity:

- **Impact** — what an attacker gains if it's exploited (data exposure, account takeover,
  privilege, DoS). INFO-labeled items can still add up (e.g. excessive permissions widen
  attack surface).
- **Exploitability** — is it reachable/exploitable in practice? Public vs internal, auth
  required, preconditions.
- **Aggregation** — many low/INFO items of the same class can together be a medium issue
  (e.g. a pile of unnecessary permissions). Rate the group.

Use the higher of (your assessment, the tool's severity) when in doubt; explain any
downgrade.

## Remediation (for Fix)

Each Fix must state:
- **What to change** — concrete and specific (remove permission X, tighten config Y, add
  header Z), not "improve security".
- **Where** — the asset/repo/config location.
- **Owner** — who applies it (mobile app → mobile team; web → FE; infra/config → infra).
- **Priority** — by your re-rated severity + exploitability.
- **Verification** — how to confirm it's fixed (re-scan, manual check).

For type-specific guidance:
- **Configuration Weakness → Mobile Application Security** (app permissions): see
  `references/mobile-permission-weakness.md`.
- Other types (exposed endpoints, TLS/headers, leaked assets, etc.): apply the same
  impact×exploitability logic; give the concrete config/code change and owner.

## Accept / False positive — require a rationale

- **Accept**: state the business/technical reason, the compensating control if any, the
  residual risk, and who accepts it. Revisit on the next review.
- **False positive**: state why the tool is wrong (not applicable, already mitigated,
  misidentified asset) so it isn't re-triaged from scratch next time.

## Prioritization output

Rank the Fixes: CRITICAL/HIGH first, then MEDIUM, then LOW/INFO (individually or as a
group). Surface the top few clearly for the owner.

## Quality bar
- Each finding/group has one decision with a stated rationale.
- Severity re-assessed by impact × exploitability (+ aggregation), not raw label alone.
- Every Fix: concrete change, location, owner, priority, verification.
- Accept/FP carry explicit, revisitable rationale.
