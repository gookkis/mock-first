---
description: Break a PRD + mockups into tasks, aware of Android modules
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(date*)
---

# Task Breakdown — Android Multi-Module

**Follow `/mock-first:break-task` with the `android` profile.** Those own the ground rules
(`[existing]` screens are not new work, every screen needs an approved mockup first, never
overwrite an existing `TASKS.md` — continue its numbering), the brainstorming limits, and the
phase order, which lives in `platforms/android.md` and is not restated here.

This file adds what only a factory has: the dependency map, the scope prefixes, and the rule
that one module owns one task file.

## Scope (REQUIRED)

Read `$ARGUMENTS`. Format: `app:<name>`, `core:<module>`, or `module:<:gradle:path>`.
If empty, list the apps and modules from `docs/INDEX.md` and ask.

The working folder is that scope's "Docs" column in `INDEX.md`. Documents used, beyond the
ones the base command reads:

- `docs/INDEX.md` — the dependency map.
- The parent PRD (path in `INDEX.md`) — the big picture, read-only.
- `<docs folder>/MODULE.md` — dependencies, public surface, folder layout, build command.
  This is where you learn where new files go and which verification command to use. If it's
  missing, run the `/android-factory:docs` flow first.

## Task numbers carry a scope prefix

`TOKOKU-T01`, `CORE-UI-T01`. That's what stops numbers colliding across apps. The prefix comes
from the "Scope prefix" column in `INDEX.md`.

## Cross-app impact check (core: scope only)

If the scope is a shared module, BEFORE writing any tasks:

1. Read the "Used by" column in `INDEX.md`.
2. Report: "This module is used by <N> apps: <list>. A change here affects all of them."
3. Ask: must every app adapt now, or should the change be backward-compatible first?
4. If the user picks "every app adapts", create a separate integration task **per app** with
   that app's prefix, e.g. `TOKOKU-T14 — adapt to the :core:ui change`.

## Module placement is the brainstorming topic that matters here

Alongside the base command's three categories, ask whether the code should be shared or local
to the app, and name the trade-off: shared means writing it once but every change risks other
apps; local is safe but duplicated. The Android-specific work people forget — Room migrations,
R8 rules, manifest permissions, build variants — is listed in the profile's "Hidden work".

## Output — `<docs folder>/TASKS.md`

```markdown
# Task List — <scope>
Last updated: <date> | Prefix: <PREFIX> | Platform: android
Modules affected: <list>

### Phase 1 — Data
- [ ] <PREFIX>-T01 — <title>
  Module: :core:data
  Files: <estimate>
  Done when: <concrete criterion + the verification command, e.g. ./gradlew :core:data:assembleDebug>
```

**One module, one TASKS.md.** A task that changes another module's contents belongs in that
module's `TASKS.md` under that module's prefix, not merged into this one. What stays here is
the integration task.

Phase 0 exists here: every NEW module needs a companion task, "register in INDEX.md + create
the docs folder (PRD.md, MODULE.md, TASKS.md)" — so no module is ever born without documents.
And the last verification task is always: refresh `MODULE.md` for the affected modules
(`/android-factory:docs`), since the public surface and dependencies change once the code is
written.

## After it's written

- Summarize: task count per phase, modules touched, apps affected.
- Update `docs/progress-report.md`:
  `- [<date>] Breakdown <scope>: <N> tasks (<PREFIX>-T## through T##), modules: <list>`
- Suggest `/android-factory:coding <scope>`
