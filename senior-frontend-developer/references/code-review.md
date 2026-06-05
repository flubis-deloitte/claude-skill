# Frontend Code Review

Read this when reviewing someone else's frontend code or a pull request. Review
like a senior: catch real problems, prioritize ruthlessly, and give feedback that
teaches rather than nitpicks.

## Contents
1. How to review (process)
2. The review checklist (what to look for)
3. Severity — what blocks vs what's a nudge
4. How to write the feedback

---

## 1. How to review (process)

1. **Understand the intent first.** Read the issue/PR description. What problem is
   this solving? Review against that goal, not against how *you* would've done it.
2. **Read the diff in context.** Open the surrounding files, not just the changed
   lines. A change can be locally fine but break a contract elsewhere.
3. **Run it in your head (or for real).** Trace the data flow and the states.
   What happens on error? On empty? On slow network? On mobile? With a keyboard?
4. **Check it against the repo's conventions**, not just generic best practice.
5. **Separate must-fix from nice-to-have** before you write anything.

## 2. The review checklist

Go through these lenses. Not every point applies to every PR — focus on what's
relevant to the change.

### Correctness & logic
- Does it actually solve the stated problem? Any obvious bugs or off-by-one/edge cases?
- Are async flows correct (await, error handling, race conditions, cleanup)?
- Are loading / error / **empty** states all handled — not just the happy path?

### Types
- Any `any`, unexplained `@ts-ignore`, or unsafe casts (`as`) hiding a real type gap?
- Are props, API responses, and state fully and accurately typed?
- Are external/untrusted inputs validated (e.g. Zod) before being trusted?

### Accessibility (a11y) — often the most-missed
- Semantic HTML? (`<button>` not `<div onClick>`, real `<label>`s, headings in order)
- Keyboard operable? Visible focus? No keyboard traps?
- Images have meaningful `alt` (or `alt=""` if decorative)? Icon-only buttons labeled?
- Color contrast adequate? Errors announced (`role="alert"`/`aria-describedby`)?

### React / Next.js correctness
- Is the Server/Client boundary right? Any needless `"use client"` bloating the bundle?
- `useEffect` doing data fetching that belongs server-side or in a query lib?
- Stable `key`s in lists (not array index when items reorder)?
- Any obvious hydration mismatch (reading `window`/random/`Date.now()` during render)?
- Dependency arrays correct? Effects cleaned up?

### Performance
- Unnecessary client JS, large imports, or missing code-splitting for heavy bits?
- Images via `next/image`? Any layout shift (CLS) from unsized media?
- Premature memoization adding noise — or a genuine hot path missing it?
- Long lists rendered without virtualization?

### Styling & design system
- Hardcoded colors/spacing instead of tokens/theme? (Red flag.)
- Reinventing a component/util that already exists in the repo?
- Responsive and consistent with surrounding UI?

### Security
- User-controlled HTML rendered with `dangerouslySetInnerHTML` (XSS risk)?
- Secrets/keys leaking into client components or the bundle?
- Links with `target="_blank"` missing `rel="noopener"`?

### Tests & maintainability
- Tests added/updated for new logic and components, per repo norms?
- Is the code readable — clear names, reasonable function size, no dead/commented code?
- Does it follow the repo's structure and naming conventions?

## 3. Severity — what blocks vs what's a nudge

Label your comments so the author knows what's required:

- **Blocking** (must fix before merge): bugs, broken a11y (keyboard/labels),
  type holes (`any` on a public boundary), security issues, missing error/empty
  states on a user-facing async surface, hardcoded values that break theming.
- **Should fix**: missing tests, perf issues that will bite under real data,
  reinventing existing primitives, convention violations.
- **Nit / optional** (clearly marked, non-blocking): naming preferences, minor
  style, "consider…" suggestions. Don't let nits drown the important comments.

If something is genuinely fine, say so — approve cleanly. Don't manufacture
comments to look thorough.

## 4. How to write the feedback

- **Lead with what's good and the overall verdict** (approve / approve-with-nits /
  request-changes), then the issues grouped by severity.
- **Be specific and actionable.** Point to the line, explain *why* it matters
  (user impact, not "I prefer"), and show the fix when it's quick.
- **Teach, don't dunk.** Frame as the change, not the person. "This `<div onClick>`
  isn't keyboard-accessible — a `<button>` gets focus + Enter/Space for free" beats
  "wrong, use a button".
- **Ask, don't assume,** when intent is unclear: "Was the empty state intentional
  here, or should we show a placeholder?"
- Keep it proportional — a two-line fix doesn't need a five-paragraph essay.

Example structure for the review summary:

> **Overall:** Solid — clean component split and good use of Server Components.
> Requesting changes on two accessibility items, rest are nits.
>
> **Blocking**
> - `UserMenu.tsx:42` — the trigger is a `<div onClick>`; keyboard users can't open
>   the menu. Make it a `<button>` (gets focus + Enter/Space).
> - `ProductList.tsx:18` — no empty state; an empty `products` array renders a blank
>   page. Add an empty placeholder.
>
> **Should fix**
> - `card.tsx:7` — hardcoded `#1d4ed8`; use the `primary` token so theming works.
>
> **Nits**
> - Consider renaming `data2` → `relatedProducts` for clarity. (optional)
