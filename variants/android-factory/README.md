# Mock-First — Android Apps Factory variant

For a multi-module / multi-app Android project **that's already running** and already has a
PRD.

## Principles
- **The parent PRD is never touched.** These commands add a layer underneath it.
- **Existing code gets no tasks.** It's marked `[existing]` and treated as done.
- **Every module has its own documents.** Every entry in `settings.gradle` — app, shared, or
  local — gets a folder holding `PRD.md` (why & what) + `MODULE.md` (what's inside) +
  `TASKS.md`. Created automatically, not by hand.
- **`INDEX.md` is the heart of it** — the module → app dependency map. Every cross-app impact
  analysis depends on it. If the map goes stale, `/revise` reports the wrong thing.

## Install
```bash
mkdir -p ~/.claude/commands/mock-first
cp *.md ~/.claude/commands/mock-first/
```
The commands become `/mock-first:adopt`, `/mock-first:prd`, and so on.

## First run
```bash
/mock-first:adopt          # ONCE — scan Gradle, build INDEX.md + docs for every module
```
After that, as needed:
```bash
/mock-first:new-app                            # a new app following an existing app's pattern
/mock-first:prd app:tokoku checkout feature    # a feature in one app
/mock-first:prd core:ui                        # a shared-module change
/mock-first:mockup app:tokoku checkout
/mock-first:break-task app:tokoku
/mock-first:coding app:tokoku
/mock-first:docs --check                       # which modules have incomplete or stale docs
/mock-first:revise change the button in core:ui   # any change from here on
```

## Document layout
```
docs/
  <parent PRD>                 <- yours, never touched
  INDEX.md                     <- module → app map, including a Docs column per module
  progress-report.md           <- global log
  apps/<app>/
    PRD.md  MODULE.md  TASKS.md  mockups/
    modules/<slug>/            <- local modules owned by this app
      PRD.md  MODULE.md  TASKS.md
  core/<slug>/                 <- shared modules
    PRD.md  MODULE.md  TASKS.md
  archive/
```

The folder slug is the Gradle path without the leading `:`, with `:` replaced by `-`
(`:core:ui` -> `core-ui`). For local modules, the owning app's segment is dropped
(`:tokoku:feature:checkout` -> `feature-checkout`).

## Two documents per module
| File | Contents | Written by |
|---|---|---|
| `PRD.md` | the module's product scope, screens, success criteria | the user, via `/mock-first:prd` |
| `MODULE.md` | responsibility, dependencies, public surface, folder layout, build command | generated from the Gradle + source scan |

In `MODULE.md`, only the blocks marked `<!-- auto: ... -->` are ever machine-rewritten. The
"Responsibility", "Local conventions", and "Constraints" sections belong to the user and are
never overwritten.

## How the docs stay in sync
There's no manual "remember to update the docs" step — each command handles it:

| Event | What happens automatically |
|---|---|
| `/adopt` | every module in `settings.gradle` gets a docs folder, `MODULE.md` filled from the code |
| `/new-app` | the app and its local modules are registered in `INDEX.md` and given docs |
| `/prd` | a new module surfacing in the Q&A is registered and documented immediately |
| `/break-task` | Phase 0 requires a "register module + create docs folder" task; the last task refreshes `MODULE.md` |
| `/coding` | a new Gradle module gets documented before the commit; `MODULE.md` is refreshed and staged alongside its code |
| `/revise` | a module that moves, becomes shared, or is deleted takes its docs folder with it |
| `/docs --check` | read-only audit: modules without docs, orphaned docs, changed dependencies, stale docs |

## Task numbering
A per-scope prefix prevents collisions: `TOKOKU-T01`, `RESEPKU-T01`, `CORE-UI-T01`.
One module, one `TASKS.md` — a task that changes another module belongs in that module's file.

## Task phases (Android)
```
0 Module & build → 1 Data → 2 Domain → 3 DI → 4 Presentation
→ 5 UI (Compose) → 6 Integration → 7 Verification
```

## What's specific to multi-app
**Changing a shared module** triggers a per-app impact analysis in `/mock-first:revise`:
which tasks are superseded in which app, and three ways to handle it (change in place / add
a new API / copy into a local module) with their trade-offs. The "Public surface" table in
`MODULE.md` is the list of what can break.

**Cross-module builds**: `/mock-first:coding` builds the affected module, then the apps that
use it — a shared module can compile fine on its own while breaking its consumers.

## Maintenance
`INDEX.md` and `MODULE.md` are kept current by the commands above. If you change a
`build.gradle` or add a module by hand outside the pipeline, run `/mock-first:docs` to
refresh (safe to repeat — only `<!-- auto -->` blocks are rewritten, and a `PRD.md` with
content is never overwritten). `/mock-first:adopt` also stays safe to re-run to rebuild the
map.
