# Tailwind CSS

Read this when the repo uses **Tailwind CSS** (commonly with `shadcn/ui`). Tailwind
is utility-first: you style with classes, and the design system lives in the config/
theme. The senior rules still hold (type safety, a11y, resilient states, no hardcoded
values); this file is *how* they map onto Tailwind.

This is **cross-cutting**: pair it with `nextjs-react.md` (when building),
`figma-to-code.md` (when converting designs), or `architecture.md` (when shaping the
system). If a repo mixes Tailwind and MUI, see the note at the end of `mui.md`.

**Detect the major version first** — it changes how config works:
- **v3**: JS config (`tailwind.config.{js,ts}`) + `@tailwind base/components/utilities`.
- **v4**: CSS-first — `@import "tailwindcss"` and a `@theme` block in your CSS; the JS
  config is optional/removed. New shadcn projects use v4 + React 19; v3 still works.

## Contents
1. The config/theme is the source of truth
2. Composing classes: the `cn()` helper
3. Variants with `cva`
4. shadcn/ui
5. Responsive & state variants
6. Next.js App Router (the RSC advantage)
7. TypeScript
8. Performance
9. Pitfalls (catch these in yourself)

---

## 1. The config/theme is the source of truth

Tokens (colors, spacing, radius, fontSize, shadows) come from the theme. Reference
**token utility classes**, and **don't scatter arbitrary values** (`bg-[#1d4ed8]`,
`p-[17px]`) — if a value repeats, add it to the theme instead.

**v3** — extend the JS config:
```ts
// tailwind.config.ts
import type { Config } from "tailwindcss";
export default {
  content: ["./app/**/*.{ts,tsx}", "./components/**/*.{ts,tsx}"],
  theme: { extend: { colors: { primary: "hsl(var(--primary))" }, borderRadius: { lg: "var(--radius)" } } },
} satisfies Config;
```

**v4** — define tokens in CSS with `@theme`:
```css
/* app/globals.css */
@import "tailwindcss";
@theme {
  --color-primary: oklch(0.55 0.2 264);
  --radius: 0.5rem;
}
```

**Semantic CSS-variable theming (the shadcn pattern)** — name colors by *role*
(`background`, `foreground`, `primary`, `muted`, `border`), not by hue, so light/dark
and rebranding are just variable swaps:
```css
@import "tailwindcss";
@import "tw-animate-css";          /* shadcn animations on v4 */
@custom-variant dark (&:is(.dark *));

:root   { --background: hsl(0 0% 100%); --foreground: hsl(222 84% 5%); --primary: hsl(221 83% 53%); }
.dark   { --background: hsl(222 84% 5%); --foreground: hsl(210 40% 98%); }

@theme inline {                    /* maps vars → utilities like bg-background, text-primary */
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-primary:    var(--primary);
}
```
On v4, the `@theme inline` mapping is what *generates* the `bg-*`/`text-*` utilities —
without it the classes won't exist. Keep `:root`/`.dark` at the top level (not inside
`@layer base`).

## 2. Composing classes: the `cn()` helper

Use a `cn()` helper (clsx + tailwind-merge), **never string concatenation**. It also
resolves conflicting utilities (`p-2 p-4` → `p-4`) deterministically — important when
merging a base class string with caller-provided `className`.

```ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";
export function cn(...inputs: ClassValue[]) { return twMerge(clsx(inputs)); }
```
```tsx
<div className={cn("rounded-md p-4", isActive && "bg-primary text-white", className)} />
```

## 3. Variants with `cva`

For components with style variants, use **class-variance-authority** instead of ternary
soup. It gives a typed, discoverable variant API and pairs with `cn`:

```tsx
import { cva, type VariantProps } from "class-variance-authority";

const button = cva("inline-flex items-center justify-center rounded-md font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 disabled:opacity-50", {
  variants: {
    variant: { primary: "bg-primary text-white hover:bg-primary/90", ghost: "hover:bg-muted" },
    size:    { sm: "h-8 px-3 text-sm", md: "h-10 px-4" },
  },
  defaultVariants: { variant: "primary", size: "md" },
});

type ButtonProps = React.ComponentPropsWithoutRef<"button"> & VariantProps<typeof button>;
export function Button({ variant, size, className, ...rest }: ButtonProps) {
  return <button className={cn(button({ variant, size }), className)} {...rest} />;
}
```

## 4. shadcn/ui

shadcn/ui is **not a dependency** — it's a CLI that copies component source into your
repo (usually `components/ui`), built on **Radix primitives + Tailwind + cva**. You own
and edit the code directly. **Reuse these primitives; don't rebuild a Dialog/Select.**

On the current (v4) version: components are updated for **React 19** (no `forwardRef`,
cleaned-up types), every primitive exposes a **`data-slot`** attribute for styling,
the default style is **`new-york`**, toasts are handled by **`sonner`**, and animations
use **`tw-animate-css`**. Theme is CSS-variable based (works with the §1 pattern). When
adding components, prefer the CLI so they match the repo's setup.

## 5. Responsive & state variants

- **Mobile-first**: unprefixed utilities apply at all sizes; `sm: md: lg: xl: 2xl:`
  layer on at larger breakpoints. Design the base for mobile, then add breakpoints.
- **State / conditional variants** instead of JS where possible: `hover:`,
  `focus-visible:`, `active:`, `disabled:`, plus `aria-[…]:`, `data-[state=open]:`,
  and `group-*` / `peer-*` for parent/sibling-driven styling.
- **Dark mode**: the `dark:` variant (class strategy). On v4, wire it with
  `@custom-variant dark (&:is(.dark *))` and toggle a `.dark` class on `<html>`.
- Use `focus-visible:` (not bare `focus:`) so keyboard focus shows without firing on
  mouse clicks — an accessibility detail seniors get right.

## 6. Next.js App Router (the RSC advantage)

Tailwind compiles to **plain CSS with zero runtime and zero client JS**, so — unlike
MUI — **styling never forces a component to be a Client Component.** You can style
Server Components freely; this is a real architectural win in the App Router (contrast
with `mui.md` §6, where MUI pushes the client boundary upward). `"use client"` is then
driven only by genuine interactivity, not styling.

Setup: import `globals.css` once in the root layout. On **v4**, use the
`@tailwindcss/postcss` plugin (Next.js) with `@import "tailwindcss"` in that file; the
old three `@tailwind` directives are v3-only.

## 7. TypeScript

Tailwind classes are plain strings (untyped), but you get type safety where it matters
via **cva**: derive prop types with `VariantProps<typeof buttonVariants>` (see §3) so a
component's allowed variants are checked at compile time. Recommended tooling: the
Tailwind CSS IntelliSense extension, `tailwind-merge`, and `prettier-plugin-tailwindcss`
to auto-sort classes for consistent, reviewable diffs.

## 8. Performance

- Only the classes you actually use end up in the final CSS, and utilities are deduped
  — bundles stay small. Heavy use of **arbitrary values** bloats output and breaks the
  token system; prefer theme tokens.
- Prefer **extracting a component** over `@apply`. Overusing `@apply` recreates the
  bespoke-CSS problem Tailwind avoids and loses dedup benefits; reserve it for a few
  base-layer rules.
- Let `prettier-plugin-tailwindcss` order classes so reviews diff cleanly.

## 9. Pitfalls (catch these in yourself)

- **Dynamic class names break the JIT scan** — the #1 Tailwind bug. The compiler only
  generates classes it can find as complete strings in source, so interpolation fails:
  ```tsx
  <div className={`text-${color}-500`} />                       // ❌ never generated
  const c = { red: "text-red-500", blue: "text-blue-500" }[color]; // ✅ full classes
  ```
  Use complete static classes, a lookup map, or `safelist` for truly dynamic cases.
- **Arbitrary-value soup** (`w-[473px]`, `bg-[#3b82f6]`) — defeats the design system;
  same red flag as hardcoding hex in MUI. Add a token instead.
- **Skipping `cn`/tailwind-merge** — conflicting utilities resolve unpredictably; always
  merge so caller `className` wins.
- **Unreadable mega class strings** — extract a component (preferred) rather than reach
  for `@apply`.
- **v4 migration gotchas** — the default border color became `currentColor` (set explicit
  border colors or add compatibility styles), and `@tailwind base/components/utilities`
  is replaced by `@import "tailwindcss"`. Don't mix v3 and v4 config styles in one repo.
