# Automation Playbook — Playwright (+ Jest/RTL) for Web

How to build a reliable automated suite. Playwright is the recommended e2e tool (no
existing setup). Jest + React Testing Library already cover unit/component. Load this for
"write an e2e test", "Playwright", "automate this".

## Layer the automation

- **Unit / component** (Jest + React Testing Library): the bulk. Render a component,
  interact via accessible queries, assert behavior. Fast, run on every commit.
- **End-to-end** (Playwright): a small set of critical journeys through the real app
  (login, core flow, checkout/payment). Slower — keep the list short and high-value.
- **API checks** (Playwright `request` fixture): hit REST endpoints to set up state or
  verify the backend effect of a UI action.

## Component tests (Jest + RTL) — principles

- Query by **accessible role/label/text** (`getByRole`, `getByLabelText`), not test-ids or
  CSS, so tests mirror how users (and assistive tech) find things.
- Test **behavior, not implementation** — a refactor that keeps behavior shouldn't break it.
- Cover the states: loading, empty, error, success; user interactions; edge inputs.
- Mock network at the boundary (e.g. MSW) rather than mocking internals.

## Playwright e2e — setup & structure

- `npm i -D @playwright/test && npx playwright install`. Config: `baseURL`, projects per
  browser (chromium/webkit/firefox), trace/screenshot/video on failure, retries in CI.
- Organize by user journey; use the **Page Object Model** (or component objects) to keep
  selectors and actions in one place.
- Use **stable selectors**: prefer `getByRole`/`getByLabel`/`getByText`; add
  `data-testid` for elements with no good accessible handle. Never depend on brittle CSS
  paths or auto-generated class names.

## Anti-flake rules (non-negotiable)

- **No fixed sleeps.** Use Playwright's web-first assertions and auto-waiting
  (`await expect(locator).toBeVisible()`), never `waitForTimeout`.
- **Isolate tests**: each test sets up its own state and can run alone and in any order.
  No dependence on another test's leftovers.
- **Control state via API/fixtures**, not by clicking through setup UI — seed data through
  the `request` fixture or a test endpoint, then test the one thing.
- **Deterministic data**: unique values per run; reset/clean up. Mock time/randomness where
  needed.
- Authenticate once and reuse storage state (`storageState`) instead of logging in per test.

## Asserting the real outcome

- Assert on what the user sees (`toHaveText`, `toBeVisible`, URL) **and**, for important
  flows, verify the backend effect via an API call (record created, status changed).

## CI

- Run unit/component on every push; run e2e on PRs/nightly. Headless, with retries,
  parallel workers, and trace/screenshot artifacts on failure for debugging.
- Quarantine genuinely flaky tests (don't let them erode trust) and fix them — a flaky
  suite gets ignored.

## What to automate vs not

- Automate: stable critical journeys, regression-prone areas, repetitive checks.
- Don't automate (yet): rapidly-changing UI, pure visual/usability judgment, one-offs.

## Quality bar

- Pyramid respected: many component tests, few high-value e2e.
- Accessible/stable selectors; behavior-focused; no fixed sleeps; isolated tests.
- State seeded via API/fixtures; auth reused; data deterministic.
- Real outcome asserted (UI + backend); green and stable in CI with failure artifacts.
