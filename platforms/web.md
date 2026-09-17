# Platform profile — Web (desktop-first)

A screen people open in a desktop browser, with a pointer and a keyboard. Read this
alongside `/mock-first:mockup`, `/mock-first:break-task` or `/mock-first:coding` — those
own the flow, this file only says what "web" means.

## Where things live (check the project shape FIRST)

A single app keeps the plain layout; a monorepo with several apps (several packages under
`apps/` or `packages/`, or an existing `docs/INDEX.md`) uses the per-app one, so a second
app's screen of the same name can't overwrite the first.

| Shape | Mockups | PRD | Tasks | Task numbers |
| --- | --- | --- | --- | --- |
| Single app | `docs/mockups/` | `docs/PRD.md` | `docs/TASKS.md` | `T01` |
| Multi-app | `docs/apps/<app>/mockups/` | `docs/apps/<app>/PRD.md` | `docs/apps/<app>/TASKS.md` | `<APP>-T01` |

One platform in the PRD -> plain `<screen>.html`. More than one -> `<screen>-web.html`, so
the mobile-web version of the same screen sits beside it instead of overwriting it.

Code: follow the routing convention already in the repo (`app/`, `pages/`, `src/routes/`).
Don't invent a second one.

## Canvas

Build the mockup at a **1280x800 viewport** with the content column capped around
1140px. Set it explicitly so the file looks the same on every reviewer's monitor:

```html
<body style="max-width:1280px;margin:0 auto">
```

It must not fall apart at 1024px (laptop) — if the layout only works wide, say so in your
2-3 line explanation instead of quietly designing something that breaks. A screen that has
to work on a phone too is **not** this profile; that is `mobile-web`, mocked as its own
file.

## Design language

- Persistent navigation that never moves: top bar, or left sidebar for more than ~7
  destinations. Breadcrumbs once the hierarchy is deeper than two levels.
- Density is the point. Tables with sortable headers, multi-column forms, side-by-side
  detail panels — a desktop user is scanning, not thumbing.
- Modal dialogs for a focused decision, inline expansion for anything the user needs to
  compare against the page behind it.
- Show the pointer affordances the platform has and mobile does not: hover states, right
  click menus, drag to reorder, tooltips on truncated text.
- A visible focus ring on every interactive element, and a keyboard path through the whole
  screen. Name the shortcut in the mockup where one is obvious (`/` to search, `Esc` to
  close).

## Naming the component

Comment the component the code will use, taken from the PRD's stack:

```html
<!-- <DataTable columns={...}> -->
<!-- <Dialog open> -->
```

If the stack names no component library, name the element's role instead
(`<!-- table, sortable by column -->`).

## States this platform always needs

Loading skeleton (not a spinner over the whole page), empty, error with a retry, inline
form validation, and the permission-denied / signed-out variant of anything gated. A 404
for a route that takes an id.

## Task phases

```
Phase 1 — Data: types, API client, queries
Phase 2 — Logic: business rules, derived state
Phase 3 — Routing & layout shell
Phase 4 — Components matching the approved mockup
Phase 5 — Forms, state wiring, error handling
Phase 6 — Responsive + accessibility pass (keyboard path, focus order, labels)
Phase 7 — Verification against the PRD's success criteria (manual, no automated tests)
```

## Verify

The project's own build and typecheck (`npm run build`, `npm run typecheck`, or whatever
`package.json` actually defines — read it, don't guess), then open the route in a browser
and check it against the approved mockup.

## Shipping it

No store, so no store commands. The release step is whatever the repo already deploys with —
read it (`vercel.json`, `netlify.toml`, a Dockerfile, a GitHub Actions workflow) rather than
proposing a new one. What still has to be true before a deploy: the production build passes,
the routes the PRD names resolve, and anything behind auth actually redirects when signed
out.

## Hidden work this platform always drags in

Empty and error states for every list, focus management when a dialog opens and closes,
what the browser Back button does after a modal, long-text truncation, and the first paint
of a page whose data is not ready yet.
