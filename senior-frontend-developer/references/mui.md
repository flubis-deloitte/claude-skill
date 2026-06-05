# Material UI (MUI)

Read this when the repo uses **Material UI** (`@mui/material`). MUI is itself the
design system — its idioms are very different from Tailwind, so don't apply
utility-class habits here. The senior rules still hold (type safety, a11y,
resilient states, no hardcoded values); this file is *how* they map onto MUI.

This is **cross-cutting**: pair it with `nextjs-react.md` (when building),
`figma-to-code.md` (when converting designs), or `architecture.md` (when shaping
the system). If a repo mixes MUI and Tailwind, also read the "MUI + Tailwind"
note at the end.

## Contents
1. The theme is the source of truth
2. Styling: `sx` vs `styled()` vs theme overrides
3. Use MUI components — customize, don't fight
4. Layout primitives
5. Responsive design with breakpoints
6. Next.js App Router integration (the key gotcha)
7. TypeScript: theme augmentation
8. Accessibility with MUI
9. Performance
10. MUI + Tailwind in the same repo

---

## 1. The theme is the source of truth

In MUI, design tokens live in the **theme** (`createTheme`): `palette`,
`typography`, `spacing`, `breakpoints`, `shape` (radius), `shadows`, `zIndex`.
Reference theme values everywhere — **never hardcode hex colors or magic px**.

```ts
// app/theme.ts
import { createTheme } from "@mui/material/styles";

export const theme = createTheme({
  cssVariables: true, // emit tokens as CSS variables: better SSR/theming, less client work
  palette: { primary: { main: "#1d4ed8" }, mode: "light" },
  shape: { borderRadius: 8 },
  spacing: 8, // theme.spacing(2) === 16px
});
```

- Spacing: `theme.spacing(2)` (or `sx={{ p: 2 }}` → 16px), not `padding: "16px"`.
- Color: `theme.palette.primary.main` / `sx={{ color: "primary.main" }}`, not `#1d4ed8`.
- Custom tokens (brand colors, extra variants) go through **theme augmentation**
  (section 7), so they're type-safe — not as one-off literals.

## 2. Styling: `sx` vs `styled()` vs theme overrides

Pick the right tool — this is a senior signal:

- **`sx` prop** — instance-level, one-off styling with full theme access and
  shorthands. Great for layout tweaks and small adjustments. Slight runtime cost;
  fine for most cases, but avoid heavy dynamic `sx` inside large lists.
  ```tsx
  <Box sx={{ p: 2, color: "text.secondary", display: "flex", gap: 1 }} />
  ```
- **`styled()`** (from `@mui/material/styles`) — reusable styled components,
  extracted and named. Use when a style repeats or a component needs a stable
  identity. Use `shouldForwardProp` to keep custom props off the DOM.
  ```tsx
  const Card = styled("div")(({ theme }) => ({
    padding: theme.spacing(2),
    borderRadius: theme.shape.borderRadius,
    backgroundColor: theme.palette.background.paper,
  }));
  ```
- **Theme component overrides** — for app-wide consistency, set defaults and
  custom variants on MUI components once, in the theme:
  ```ts
  createTheme({
    components: {
      MuiButton: {
        defaultProps: { disableElevation: true },
        styleOverrides: { root: { textTransform: "none" } },
        variants: [{ props: { variant: "brand" }, style: { /* … */ } }],
      },
    },
  });
  ```
  This is the senior move: one place to enforce how every `Button` looks.
- **Avoid `style={{…}}`** (inline) — no theme access, no pseudo-classes, no media queries.

## 3. Use MUI components — customize, don't fight

Prefer MUI's components (`Button`, `TextField`, `Dialog`, `Menu`, `Autocomplete`,
`Select`, `Snackbar`, …). They're accessible and battle-tested — **don't
re-implement a Dialog or a Select.** Customize via:

- Props: `variant`, `color`, `size`.
- `sx` for instance styling.
- `slots` / `slotProps` to override or pass props to internal parts.

Reach for **MUI X** (DataGrid, Date/Time Pickers, Charts) for complex widgets
rather than hand-rolling. If a primitive genuinely doesn't exist, build it on
MUI's base and register it through the theme (section 2), not as a snowflake.

## 4. Layout primitives

Use theme-aware primitives instead of raw divs where they add clarity:

- **`Box`** — a themed `div`; accepts `sx`. The default building block.
- **`Stack`** — flexbox with `direction`, `spacing`, `divider`. Ideal for vertical/
  horizontal groups.
- **`Grid`** — responsive grid; in current MUI use the `size` prop (e.g. `size={{ xs: 12, md: 6 }}`).
- **`Container`** — centered, max-width page wrapper.

## 5. Responsive design with breakpoints

Responsiveness flows through the theme's breakpoints (`xs sm md lg xl`),
mobile-first:

- Breakpoint objects in `sx`:
  ```tsx
  <Box sx={{ width: { xs: "100%", md: 480 }, p: { xs: 2, md: 4 } }} />
  ```
- In `styled()`: `theme.breakpoints.up("md")`.
- For JS-driven layout decisions: `useMediaQuery(theme.breakpoints.down("sm"))`
  — but note it's a **hook**, so the component must be a Client Component, and it
  can cause a hydration flash; prefer CSS/`sx` responsiveness when possible.

## 6. Next.js App Router integration (the key gotcha)

**MUI's interactive components are Client Components** (they rely on React context,
hooks, and Emotion). Importing them makes a file effectively client, so **MUI
tends to push the client boundary upward.** Be deliberate: keep data-fetching
shells as Server Components and render MUI UI in client leaves (see
`nextjs-react.md` §1). Don't blanket the whole app in `"use client"` if you can
isolate the interactive parts.

Required setup (so styles land in `<head>` and don't flash):

```tsx
// app/providers.tsx
"use client";
import { ThemeProvider } from "@mui/material/styles";
import CssBaseline from "@mui/material/CssBaseline";
import { theme } from "./theme";

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      {children}
    </ThemeProvider>
  );
}
```

```tsx
// app/layout.tsx  — stays a Server Component
import { AppRouterCacheProvider } from "@mui/material-nextjs/v15-appRouter";
import { Providers } from "./providers";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <AppRouterCacheProvider>
          <Providers>{children}</Providers>
        </AppRouterCacheProvider>
      </body>
    </html>
  );
}
```

Notes:
- Install `@mui/material @emotion/react @emotion/styled @mui/material-nextjs`.
  Use the cache provider entry point matching your Next version
  (`@mui/material-nextjs/v15-appRouter`, or `v1X-appRouter`).
- `AppRouterCacheProvider` collects MUI's server-generated CSS as Next streams the
  HTML, so styles go to `<head>` instead of `<body>` (prevents FOUC). It is itself
  a client component meant to wrap the layout.
- Keep `theme` imported only by the client `providers.tsx` so it never has to cross
  a server→client prop boundary; `{children}` passes through (donut pattern) and may
  still contain Server Components.
- **Dark mode / CSS variables:** with `cssVariables: true`, add `InitColorSchemeScript`
  (from `@mui/material`) in the layout to prevent theme-flicker on load.
- Fonts: use `next/font` and set `typography.fontFamily` to the `var(--font-…)` CSS variable.

## 7. TypeScript: theme augmentation

Make custom tokens/variants type-safe with module augmentation instead of casting:

```ts
declare module "@mui/material/styles" {
  interface Palette { brand: Palette["primary"]; }
  interface PaletteOptions { brand?: PaletteOptions["primary"]; }
}
declare module "@mui/material/Button" {
  interface ButtonPropsVariantOverrides { brand: true; }
}
```

Now `sx={{ color: "brand.main" }}` and `<Button variant="brand">` type-check.

## 8. Accessibility with MUI

MUI components are accessible by default, but you can still break it:

- **`TextField`**: always give a `label` (or `aria-label` if visually hidden).
  Use the `error` + `helperText` props so validation errors are announced.
- **`IconButton`**: needs an `aria-label` (it has no text).
- Don't strip focus outlines; keep `:focus-visible` styling intact.
- Ensure custom `palette` colors meet contrast (MUI won't enforce it for you).
- Remember disabled MUI buttons aren't focusable — wrap in a `span` if you need a
  tooltip on them.

## 9. Performance

- Emotion has a runtime cost. `sx` and theme overrides are fine; avoid large amounts
  of dynamic `sx` in long lists — extract to `styled()` or virtualize.
- **Icons:** import individually (`import MenuIcon from "@mui/icons-material/Menu"`),
  never the barrel (`import { Menu } from "@mui/icons-material"`) — the barrel pulls
  in thousands of icons.
- `cssVariables: true` shifts theming to CSS variables, improving SSR and cutting
  some client work.
- MUI is a large dependency — `next/dynamic` heavy, below-the-fold MUI views.

## 10. MUI + Tailwind in the same repo

If both are present, establish clear ownership and don't mix them on one element:

- A common split: **MUI owns components + theme**, Tailwind handles utility layout.
- Beware CSS specificity/order conflicts between Emotion and Tailwind, and don't put
  both `sx` and conflicting Tailwind classes on the same node.
- Follow whatever convention the repo already uses; if there's a `tailwind.config`
  *and* a MUI theme, find out which surface each governs before adding more.
