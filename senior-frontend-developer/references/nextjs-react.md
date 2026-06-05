# Next.js (App Router) + React + TypeScript

Read this when building or changing components, pages, routes, data fetching, or
forms. Assumes the App Router. Adapt to the repo's actual version and conventions.

## Contents
1. Server vs Client Components — the most important decision
2. Data fetching
3. Routing, layouts, and special files
4. Forms and mutations (Server Actions)
5. TypeScript patterns for components
6. Styling
7. Performance
8. Common mistakes

---

## 1. Server vs Client Components

**Default to Server Components.** A component is a Server Component unless it has
`"use client"` at the top of the file. Only opt into a Client Component when you
need: event handlers (`onClick`, `onChange`), state/effects (`useState`,
`useEffect`), browser-only APIs (`window`, `localStorage`), or client-only libs.

**Push the client boundary to the leaves.** Don't make a whole page a Client
Component to get one interactive widget. Keep the page/layout as a Server
Component and extract the interactive bit into its own small `"use client"`
component. Server Components can render Client Components, and you can pass server
data down as props.

```tsx
// app/products/page.tsx  — Server Component (no "use client")
import { getProducts } from "@/lib/products";
import { AddToCartButton } from "./add-to-cart-button"; // client leaf

export default async function ProductsPage() {
  const products = await getProducts(); // runs on the server, no client JS shipped
  return (
    <ul>
      {products.map((p) => (
        <li key={p.id}>
          {p.name}
          <AddToCartButton productId={p.id} /> {/* only this is client */}
        </li>
      ))}
    </ul>
  );
}
```

Rules of thumb:
- Server Components can be `async` and `await` directly. Client Components cannot.
- You cannot pass functions (non-serializable) from a Server Component to a Client
  Component as props — pass data, or use Server Actions for behavior.
- Importing a Server Component into a Client Component doesn't work; pass it as
  `children` instead (the "donut" pattern).

## 2. Data fetching

- **Fetch on the server** in Server Components by default — closer to the data,
  no client waterfall, no exposed secrets.
- Use `fetch()` with Next's caching options, or call your data layer directly.
  Be explicit about caching intent (`cache: "no-store"` for dynamic, revalidation
  for ISR-style freshness).
- **Parallelize** independent requests with `Promise.all` instead of sequential
  `await`s (avoid waterfalls).
- Stream slow data with **`<Suspense>`** + a `loading.tsx` boundary so the shell
  paints immediately.
- For **client-side** server-state (mutations, polling, optimistic UI, infinite
  lists), use **TanStack Query** — not `useEffect` + `useState`. It gives you
  caching, dedupe, retries, and loading/error states for free.

```tsx
// Parallel, not waterfall
const [user, orders] = await Promise.all([getUser(id), getOrders(id)]);
```

## 3. Routing, layouts, and special files

App Router uses folders for routes. Know the special files:

- `page.tsx` — the route's UI.
- `layout.tsx` — shared shell that wraps children; persists across navigation.
- `loading.tsx` — instant loading UI (wraps the segment in Suspense automatically).
- `error.tsx` — error boundary for the segment (**must be a Client Component**).
- `not-found.tsx` — 404 UI; trigger with `notFound()`.
- `route.ts` — API route handlers (`GET`, `POST`, …).
- Dynamic segments: `[id]`, catch-all `[...slug]`, groups `(marketing)`.

Always provide `loading.tsx` and `error.tsx` for segments that fetch data — that's
how you get resilient states "for free" at the route level.

Use the metadata API (`export const metadata` or `generateMetadata`) for SEO/title
rather than manual `<head>` manipulation.

## 4. Forms and mutations (Server Actions)

Prefer **Server Actions** for mutations in the App Router. They work without
client JS and integrate with progressive enhancement.

```tsx
// app/contact/actions.ts
"use server";
import { z } from "zod";

const Schema = z.object({ email: z.string().email(), message: z.string().min(1) });

export async function submitContact(_prev: unknown, formData: FormData) {
  const parsed = Schema.safeParse(Object.fromEntries(formData));
  if (!parsed.success) {
    return { ok: false, errors: parsed.error.flatten().fieldErrors };
  }
  await saveMessage(parsed.data);
  return { ok: true, errors: {} };
}
```

```tsx
// app/contact/contact-form.tsx
"use client";
import { useActionState } from "react";
import { submitContact } from "./actions";

export function ContactForm() {
  const [state, action, pending] = useActionState(submitContact, { ok: false, errors: {} });
  return (
    <form action={action}>
      <label htmlFor="email">Email</label>
      <input id="email" name="email" type="email" required aria-describedby="email-err" />
      {state.errors?.email && <p id="email-err" role="alert">{state.errors.email}</p>}
      <button disabled={pending}>{pending ? "Sending…" : "Send"}</button>
    </form>
  );
}
```

- **Always validate on the server** (Zod), even if you also validate client-side.
- Wire up `aria-describedby` / `role="alert"` so errors are announced to screen readers.
- Disable the submit button while pending; reflect pending state in the label.
- Call `revalidatePath`/`revalidateTag` after a mutation to refresh cached data.

## 5. TypeScript patterns for components

- Type props with an explicit interface/type. Don't rely on inference for public APIs.
- Extend native element props when wrapping one, so consumers get the full API:

```tsx
type ButtonProps = React.ComponentPropsWithoutRef<"button"> & {
  variant?: "primary" | "ghost";
};

export function Button({ variant = "primary", className, ...rest }: ButtonProps) {
  return <button className={cn(buttonStyles[variant], className)} {...rest} />;
}
```

- Use discriminated unions for components with mutually exclusive prop modes.
- Avoid `React.FC` (implicit `children`, awkward generics); type props directly.
- Type `unknown` for untrusted/external data, then narrow (often with Zod).
- Prefer `satisfies` to validate object shapes without widening the type.

## 6. Styling

Follow the repo's styling system; the principle "never hardcode values that bypass
the design system" is constant across all of them.

- **Tailwind**: tokens come from the config/theme; never hardcode hex/px or lean on
  arbitrary values. Compose classes with a `cn` helper and express variants with `cva`.
  See `references/tailwind.md` for the full playbook (incl. v3 vs v4 and the RSC advantage).
- **Material UI (MUI)**: tokens live in the **theme**; style with `sx` / `styled()` /
  theme overrides, not utility classes. See `references/mui.md` for the full playbook,
  including the App Router client-boundary gotcha.
- Respect `prefers-reduced-motion` for animations regardless of styling system.

## 7. Performance

- Server Components ship zero JS — use them to keep bundles small.
- `next/image` for images (sizing, lazy-loading, format) — prevents layout shift.
- `next/font` for fonts (no FOUT, no layout shift).
- `next/dynamic` to lazy-load heavy client-only components below the fold.
- Memoize (`useMemo`/`memo`/`useCallback`) only when profiling shows a real cost;
  premature memoization adds noise and can hurt.
- Set explicit width/height (or aspect-ratio) on media to reserve space (CLS).
- Virtualize very long lists (e.g. TanStack Virtual) instead of rendering 1000 rows.

## 8. Common mistakes (catch these in yourself)

- `"use client"` at the top of a page just to use one hook → extract the leaf instead.
- `useEffect(() => { fetch(...) }, [])` for initial data → fetch server-side or use TanStack Query.
- `key={index}` in lists that reorder/filter → use a stable id.
- Forgetting `loading.tsx`/`error.tsx` → users see blank screens or unhandled errors.
- Passing event handlers from Server → Client component props → use Server Actions or move the boundary.
- Mutating state directly, or storing derived values in state instead of computing them.
- Reading `window`/`localStorage` during render without a client guard → hydration mismatch.
