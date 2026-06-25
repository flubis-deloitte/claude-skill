# Manual & Exploratory Testing Playbook (Web)

How to run the human side of web QA — the part automation can't do well. Load this for
"exploratory testing", "cross-browser", "usability", "visual check".

## When manual is the right tool

- **Exploratory** — finding unknown issues in new/complex/risky features.
- **Usability** — is the flow clear, the copy understandable, the feedback adequate?
- **Visual / layout vs Figma** — pixel/spacing/typography/responsive judgment.
- **Cross-browser / cross-device** spot checks beyond what's automated.
- **One-off** verification or features changing too fast to automate yet.

## Exploratory testing (charter-driven)

- Time-box and set a **charter**: "Explore the checkout flow, focus on promo + insufficient
  balance + back-button". You design and execute as you learn.
- Apply heuristics: boundaries; interruptions (back, refresh, resize, offline, tab away);
  unusual sequences; double actions; role switching; stale/expired data; deep-link straight
  to a step.
- Take notes continuously; turn findings into defects and into new scripted/automated cases.

## Visual vs Figma

- Compare the built page to the Figma frame: layout, spacing, typography, color, states
  (empty/loading/error/success), and responsive behavior. Discrepancies are defects —
  reference the specific frame.
- Verify the dynamic/empty/error states, not just the populated happy view.

## Cross-browser & responsive

- Check target browsers (e.g. Chrome, Safari, Firefox) and viewports (mobile/tablet/
  desktop). Look for layout breaks, overflow, font rendering, and behavior differences.
- Test real interactions: keyboard navigation, focus states, form behavior, scrolling.

## Accessibility quick pass (manual)

- Keyboard-only: can you reach and operate everything? Visible focus?
- Screen-reader sanity on key flows; images have alt; form fields have labels; headings
  ordered; color isn't the only signal; contrast looks adequate. (Pair with an automated
  axe check where possible.)

## Executing & recording

- Follow the test case steps exactly; record pass/fail with **evidence** (screenshot/
  recording) at the point of failure.
- Verify the **end state**, not just the screen: was the API called, did the data change?
  (Check via the network panel or a read view.)
- Log defects with full reproduction (see `bug-reporting.md`).

## Quality bar

- Exploratory sessions are charter-driven and documented.
- Visual checks compare to Figma incl. all states; cross-browser/responsive covered.
- Basic accessibility checked; evidence captured; end state verified.
