# Frontend Architecture & Design Systems

Read this when designing component APIs, deciding folder structure, choosing a
state-management strategy, or building/extending a design system. This is about
decisions that are expensive to change later — make them deliberately.

## Contents
1. Component design (props APIs & composition)
2. The Server/Client boundary as architecture
3. State management strategy
4. Folder structure & co-location
5. Design system & tokens
6. Scaling and consistency

---

## 1. Component design (props APIs & composition)

A good component API is small, predictable, and hard to misuse.

- **Compose, don't configure.** Prefer composition (`children`, slots, sub-components)
  over a pile of boolean/config props. A `Card` with `<Card.Header>`, `<Card.Body>`
  scales better than `<Card showHeader headerSize="lg" footerVariant=…>`.
- **Avoid boolean explosion.** Many booleans that can't combine → a single `variant`
  union (`variant="primary" | "ghost" | "danger"`). Use discriminated unions for
  mutually exclusive modes.
- **Extend the native element** when wrapping one, so consumers keep the full API
  and `ref`:

```tsx
type InputProps = React.ComponentPropsWithoutRef<"input"> & {
  label: string;
  error?: string;
};
export const Input = React.forwardRef<HTMLInputElement, InputProps>(
  function Input({ label, error, id, ...rest }, ref) {
    const inputId = id ?? React.useId();
    return (
      <div>
        <label htmlFor={inputId}>{label}</label>
        <input id={inputId} ref={ref} aria-invalid={!!error}
               aria-describedby={error ? `${inputId}-err` : undefined} {...rest} />
        {error && <p id={`${inputId}-err`} role="alert">{error}</p>}
      </div>
    );
  }
);
```

- **Controlled vs uncontrolled:** pick one per component and document it. Support
  `defaultValue` (uncontrolled) *or* `value` + `onChange` (controlled), not a muddle.
- **Keep presentational components dumb.** They take props and render. Data fetching,
  routing, and business logic live higher up. This keeps them reusable and testable.
- **Separation of concerns:** primitives (Button, Input) → composites (SearchBar) →
  features (CheckoutForm) → pages. Lower layers never import from higher layers.

## 2. The Server/Client boundary as architecture

In the App Router, *where* the client boundary sits is an architectural decision,
not an afterthought:

- Default everything to Server Components. Treat `"use client"` as a deliberate
  choice at the **leaf** that needs interactivity.
- Design feature components so the data-fetching shell is a Server Component and
  only the interactive controls are client. Pass server data down as props; pass
  Server Components into client ones via `children` when needed.
- Keep secrets and heavy logic on the server side of the boundary.

## 3. State management strategy

Match the tool to the kind of state. Most "state management" problems are really
"wrong category" problems.

- **Server state** (data from your API): use **TanStack Query** (or RSC + Server
  Actions). Don't hand-roll it into `useState`/`useEffect`. You get caching, dedupe,
  revalidation, and loading/error states.
- **URL state** (filters, tabs, pagination, search): keep it in the URL (search
  params / route). It's shareable, bookmarkable, and survives refresh.
- **Local UI state** (open/closed, hover, form inputs): `useState`/`useReducer`,
  kept as close to where it's used as possible.
- **Global client state** (theme, auth user, cart): a light store like **Zustand**
  or React Context — only for things genuinely shared across the tree. Don't reach
  for a global store when local state or the URL would do.
- **Derive, don't store.** If a value can be computed from existing state/props,
  compute it during render instead of duplicating it in state (a classic bug source).

Decision shortcut: *Is it from the server? → query lib. Should it survive refresh /
be shareable? → URL. Used by one subtree? → local. Truly app-wide? → store/context.*

## 4. Folder structure & co-location

Follow the repo if it has a structure. If you're establishing one, **co-locate by
feature**, not by file-type:

```
app/
  (routes)/...                 # route segments: page.tsx, layout.tsx, loading.tsx, error.tsx
components/
  ui/                          # design-system primitives (Button, Input, Card)
features/
  checkout/
    components/                # feature-specific components
    hooks/
    actions.ts                 # Server Actions for this feature
    schema.ts                  # Zod schemas / types
    checkout.test.tsx          # tests live next to code
lib/                           # shared utils, clients, helpers
```

- Co-locate a component with its types, tests, and styles.
- Shared primitives go in the design-system layer (`components/ui`), feature code
  stays inside its feature folder.
- Use path aliases (`@/…`) consistently; don't deep-relative-import (`../../../`).

## 5. Design system & tokens

- **Tokens are the source of truth** for color, spacing, typography, radius,
  shadow, z-index. Define them once and reference them everywhere — in a **Tailwind**
  repo that's the Tailwind config / CSS variables; in an **MUI** repo it's the
  **theme** (`createTheme`). Components must consume tokens, never hardcode raw values.
- **Theming** (light/dark, brands) is just swapping token values — which only works
  if nothing hardcodes hex/px. This is why "no hardcoded values" is an architectural
  rule, not a style nit. (MUI: `cssVariables: true` makes this swap cleaner.)
- **Primitive layer**: build a small, well-tested set of accessible primitives
  (Button, Input, Select, Dialog, etc.). For complex interactive widgets (dialogs,
  menus, comboboxes), build on a headless/accessible base (e.g. Radix) rather than
  re-implementing focus traps and ARIA from scratch. **In MUI, the primitive layer
  already exists** — MUI components are your primitives; customize them through theme
  overrides instead of rebuilding (see `mui.md`).
- **Variants**: express component variants in a type-checked way — a typed utility
  like `cva` (Tailwind), or theme `components.*.variants` + module augmentation (MUI)
  — so allowed combinations are discoverable and safe.
- **Document usage** (even briefly): when to use which variant, do's and don'ts.
  A design system nobody understands gets bypassed.

## 6. Scaling and consistency

- **Consistency > cleverness.** One obvious way to build a form, fetch data, handle
  errors. Patterns the team can copy beat locally-optimal one-offs.
- **Make the right thing the easy thing.** If everyone hardcodes colors, the tokens
  are too hard to use — fix the ergonomics, don't just nag in review.
- **Boundaries:** features shouldn't import each other's internals; share through
  the `lib`/`ui` layers or explicit public exports.
- **Keep bundles honest:** watch what crosses into client components; lazy-load
  heavy, below-the-fold, client-only pieces.
- When proposing architecture, state the trade-offs explicitly (flexibility vs
  simplicity, build-time vs runtime, etc.) so the team can decide with eyes open —
  there's rarely one correct answer, only the right fit for this codebase.
