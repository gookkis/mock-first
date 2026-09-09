---
description: Break a PRD + mockups into tasks, aware of Android modules
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(date*)
---

# Task Breakdown — Android Multi-Module

## Scope (REQUIRED)

Read `$ARGUMENTS`. Format: `app:<name>`, `core:<module>`, or `module:<:gradle:path>`.
If empty, list the apps and modules from `docs/INDEX.md` and ask.

The working folder is that scope's "Docs" column in `INDEX.md`. Documents used:

- `docs/INDEX.md` — the dependency map.
- The parent PRD (path in `INDEX.md`) — the big picture.
- `<docs folder>/PRD.md` — the specific scope.
- `<docs folder>/MODULE.md` — dependencies, public surface, folder layout, build command.
  This is where you learn where new files go and which verification command to use. If it's
  missing, run the `/mock-first:docs` flow first.
- `docs/apps/<app>/mockups/*-approved.html` (app scope only).

## Ground rules

- Run `date +%Y-%m-%d` for the date.
- A screen marked `[existing]` in the PRD is **not** new work — don't write a task for it
  unless the user explicitly asked for it to change.
- **Task numbers carry a scope prefix**: `TOKOKU-T01`, `CORE-UI-T01`. That's what stops
  numbers colliding across apps. The prefix comes from the "Scope prefix" column in
  `INDEX.md`.

## Cross-app impact check (core: scope only)

If the scope is a shared module, BEFORE writing any tasks:

1. Read the "Used by" column in `INDEX.md`.
2. Report: "This module is used by <N> apps: <list>. A change here affects all of them."
3. Ask: must every app adapt now, or should the change be backward-compatible first?
4. If the user picks "every app adapts", create a separate integration task **per app** with
   that app's prefix, e.g. `TOKOKU-T14 — adapt to the :core:ui change`.

## Brainstorming mode

At most 3 points per category:

- **Module placement** — should this code be shared or local to the app? Name the trade-off
  (shared = written once but every change risks other apps; local = safe but duplicated).
- **Unwritten tasks** specific to Android: Room migrations, ProGuard/R8 rules, manifest
  permissions, build variant configuration.
- **Ordering risk** — which module has to compile first.
- A cost warning when a feature is far more expensive than it looks.

If the user says "just follow the PRD", get on with it. **Suggestions stay suggestions.**

## Task phases (Android order)

```
Phase 0 — Module & build: new Gradle module setup, dependencies, build variants.
          Every NEW module needs a companion task, "register in INDEX.md + create the
          docs folder (PRD.md, MODULE.md, TASKS.md)" — so no module is ever born
          without documents.
Phase 1 — Data: Room/DataStore/network sources
Phase 2 — Domain: use cases, models, repository interfaces
Phase 3 — DI: Hilt/Koin modules and bindings
Phase 4 — Presentation: ViewModel, state, events
Phase 5 — UI: Compose screens matching the approved mockup
Phase 6 — Integration: navigation, wiring into the app
Phase 7 — Verification: manual checks against the PRD's success criteria (no automated tests)
```

Skip phases that don't apply. Don't force content into every phase.

## Output — `<docs folder>/TASKS.md`

```markdown
# Task List — <scope>
Last updated: <date> | Prefix: <PREFIX>
Modules affected: <list>

### Phase 1 — Data
- [ ] <PREFIX>-T01 — <title>
  Module: :core:data
  Files: <estimate>
  Done when: <concrete criterion + the verification command, e.g. ./gradlew :core:data:assembleDebug>
```

If `TASKS.md` already exists, DON'T overwrite it — continue numbering from the last task.

**One module, one TASKS.md.** A task that changes another module's contents belongs in that
module's `TASKS.md` under that module's prefix, not merged into this one. What stays here is
the integration task.

The last Phase 7 task is always: `refresh MODULE.md for the affected modules
(/mock-first:docs)` — the public surface and dependencies change once the code is written.

## After it's written

- Summarize: task count per phase, modules touched, apps affected.
- Ask whether anything needs changing.
- Update `docs/progress-report.md`:
  `- [<date>] Breakdown <scope>: <N> tasks (<PREFIX>-T## through T##), modules: <list>`
- Suggest `/mock-first:coding <scope>`
