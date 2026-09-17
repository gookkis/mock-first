# Platform profile — Mobile web

The same web app, but opened in a phone browser. It is not a smaller desktop screen and it
is not a native app: there is no pull-to-refresh you control, no hover, and the browser
eats part of the viewport. Read this alongside `/mock-first:mockup`,
`/mock-first:break-task` or `/mock-first:coding` — those own the flow.

## Where things live (check the project shape FIRST)

A single app keeps the plain layout; a monorepo with several apps (several packages under
`apps/` or `packages/`, or an existing `docs/INDEX.md`) uses the per-app one, so a second
app's screen of the same name can't overwrite the first.

| Shape | Mockups | PRD | Tasks | Task numbers |
| --- | --- | --- | --- | --- |
| Single app | `docs/mockups/` | `docs/PRD.md` | `docs/TASKS.md` | `T01` |
| Multi-app | `docs/apps/<app>/mockups/` | `docs/apps/<app>/PRD.md` | `docs/apps/<app>/TASKS.md` | `<APP>-T01` |

The file is `<screen>-mobile-web.html` when the PRD also targets desktop web, so both live
side by side; plain `<screen>.html` when mobile web is the only target.

Code: the same codebase as web. A separate mobile route tree is a decision, not a default —
if the mockups diverge enough to need one, raise it in `/break-task` rather than building it
quietly.

## Canvas

Build the mockup at a **390px viewport** and check it still holds at **360px** — that is
the floor of phones still in use. Frame it so the reviewer sees the real thing:

```html
<body style="max-width:390px;margin:0 auto;border:1px solid #ddd">
```

Use `100dvh`, never `100vh`: the address bar collapses on scroll and `100vh` leaves a
sticky footer cut off. Respect `env(safe-area-inset-bottom)` — on iOS Safari the home
indicator sits on top of anything flush to the bottom.

## Design language

- One column. A second column on a phone is a table in disguise; turn it into a card or a
  definition list instead.
- The primary action goes in thumb reach: a sticky bottom bar, not the top right corner.
- Bottom sheet instead of dropdown, full-screen view instead of modal, accordion instead of
  a side-by-side panel.
- Tap targets 44px or more, with real spacing between them. Two destructive actions must
  never be neighbours.
- **No hover-only affordance.** Anything that only appears on hover does not exist here —
  if the desktop version hides actions behind hover, this screen needs them visible or
  behind an explicit menu.
- Navigation: bottom tab bar for 3-5 destinations, a drawer beyond that. A sticky header
  that collapses on scroll buys back vertical space.

## Naming the component

Same as web — comment the component from the PRD's stack — plus a note wherever the
behaviour differs from the desktop version of the same screen:

```html
<!-- <BottomSheet> — desktop uses <Dropdown> -->
```

## States this platform always needs

Everything web needs, plus: **offline** (a phone browser loses the network mid-session),
slow-network skeletons, and — if the PRD says PWA — the install / add-to-home-screen
prompt and what the app looks like launched standalone, without browser chrome.

## Task phases

```
Phase 1 — Data: types, API client, queries (shared with web if the codebase is shared)
Phase 2 — Logic: business rules, derived state
Phase 3 — Routing & layout shell
Phase 4 — Components matching the approved mockup
Phase 5 — Forms, state wiring, error handling
Phase 6 — Touch & viewport pass: tap targets, safe areas, dvh, offline, no hover-only paths
Phase 7 — Verification against the PRD's success criteria (manual, no automated tests)
```

## Verify

The project's own build, then open the route in a browser at a 390px viewport with device
emulation on — and, where the change touches gestures, scrolling or the address bar, on a
real phone. Desktop emulation does not reproduce the collapsing address bar honestly.

## Shipping it

Same deploy path as web — it is the same codebase. If the PRD says PWA, the release also
needs a manifest with the icon sizes Android and iOS each expect, a service worker whose
cache is versioned (a stale one is worse than no offline support), and a check that the app
launched from the home screen still routes correctly, without the browser's Back button.

## Hidden work this platform always drags in

The on-screen keyboard covering the field being typed into, a sticky footer fighting that
keyboard, momentum scrolling inside a bottom sheet, double-tap zoom on a tap target that is
too small, and what happens when the tab is restored hours later with stale data.
