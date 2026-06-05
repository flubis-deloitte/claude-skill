# Figma → Code

Read this when turning a Figma design, frame, or component into working frontend
code. The goal is a faithful, **maintainable** implementation — not a pixel-traced
dump of absolutely-positioned divs. A senior reproduces the *intent* of the design
using the codebase's real components and tokens.

## Contents
1. Get the design context
2. Read the design before you code
3. Map to the design system (don't hardcode)
4. Build layout with flow, not absolute positioning
5. Responsiveness and states
6. Component decomposition
7. Verification against the design

---

## 1. Get the design context

If a **Figma MCP / Dev Mode** connection is available, use it to pull real data
instead of eyeballing a screenshot:

- Get the selected node's structure/metadata (layer names, hierarchy, auto-layout).
- Get variable/token definitions (colors, spacing, typography) so you can map them
  to the repo's tokens.
- Get a screenshot of the node for visual reference.
- If the repo has Code Connect mappings, use them — they tell you which code
  component a Figma component maps to.

If you only have a static image/screenshot, work from that, but be explicit that
exact spacing/tokens are approximated and should be verified.

Always prefer structured Figma data (auto-layout, constraints, variables) over
guessing pixel offsets from an image.

## 2. Read the design before you code

Spend a moment analyzing, like a senior would in a design handoff:

- **Layout system**: Is it auto-layout (→ flex/grid) or freeform? What's the
  direction, gap, padding, alignment?
- **Spacing scale**: Are spacings multiples of a base (4/8px)? Map to token scale.
- **Typography**: Which text styles repeat? These become tokens/classes, not one-offs.
- **Color**: Which fills are theme tokens vs literal colors? Map to the theme.
- **Repetition**: What repeats? Those are components (cards, list items, chips).
- **Responsive intent**: Constraints/“hug vs fill” hint at how it should reflow.
- **States**: Are hover/focus/active/disabled/error variants in the file?

## 3. Map to the design system (don't hardcode)

This is the #1 thing that separates senior from junior Figma conversion:

- **Reuse existing components.** If the design shows a button and the repo has a
  `Button` primitive, use it (with the right variant) — do not build a new button.
  In an **MUI** repo, that means using MUI's `Button`/`TextField`/etc. with the
  right `variant`/`color`, not a bespoke component.
- **Map raw values to tokens.** `#2563EB` → `text-primary`/`bg-primary` (whatever
  the token is), `gap: 16px` → the spacing token, font sizes → the typography scale.
  In **MUI**, map to the **theme**: colors → `palette`, spacing → `theme.spacing`,
  text → `typography`. Hardcoded hex/px is a code-review red flag either way.
- If a needed primitive doesn't exist, build it *as a reusable primitive* in the
  design-system layer (or register it on the MUI theme), then use it — see
  `architecture.md` and `mui.md`.

## 4. Build layout with flow, not absolute positioning

- Translate Figma **auto-layout → flexbox/grid**. Vertical auto-layout → `flex-col`;
  horizontal → `flex-row`; gap → `gap-*`; padding → `p-*`; alignment → `items-*` /
  `justify-*`.
- Avoid `position: absolute` for general layout. Reserve it for genuine overlays
  (badges on avatars, tooltips), and even then prefer the design system's primitives.
- Use semantic HTML for the structure (`<nav>`, `<section>`, `<article>`, `<ul>`,
  headings in order) — the Figma layer name is a hint, not the final tag.

## 5. Responsiveness and states

Designs are usually one or two breakpoints; you own the in-between:

- Build **mobile-first**, then layer `sm:`/`md:`/`lg:` adjustments.
- Decide reflow behavior for grids/rows (stack on mobile, columns on desktop).
- Ensure no fixed widths cause horizontal scroll on small screens.
- Implement the **interaction states** even if the Figma file only shows the
  default: hover, focus-visible, active, disabled, and (for inputs) error.
- Handle **content variability**: long strings, missing images, empty lists,
  many items — the design's "perfect" content won't match production.

## 6. Component decomposition

- Break the frame into a sensible component tree; don't produce one 400-line file.
- Extract anything that repeats into its own component with a typed props API.
- Keep presentational components dumb (props in, UI out); lift data fetching up to
  the page/Server Component (see `nextjs-react.md`).
- Name components by role (`PricingCard`, `NavBar`), not by Figma layer ids.

## 7. Verification against the design

Before declaring done:

- [ ] Spacing, type, and color come from tokens — no stray hardcoded values
- [ ] Existing primitives reused; no duplicate Button/Input/Card reinvented
- [ ] Layout uses flex/grid (from auto-layout), not absolute hacks
- [ ] Responsive from mobile to the design's largest breakpoint; no overflow
- [ ] Hover/focus/active/disabled/error states implemented and keyboard-accessible
- [ ] Semantic HTML; images have `alt`; headings in order
- [ ] Visually matches the frame (compare against the screenshot/Dev Mode)
- [ ] Passes the global Definition of Done in `SKILL.md`

Then summarize any deviations from the design and why (e.g. "Figma used a one-off
color; mapped to the nearest theme token for consistency").
