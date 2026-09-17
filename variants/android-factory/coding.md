---
description: Work the next task from TASKS.md — implement, review, commit
allowed-tools: all
---

# Coding Iteration — Android Multi-Module

**Follow `/mock-first:coding` with the `android` profile.** Those own the loop: take the
topmost unchecked task, name it plus a three-line plan, implement, build only the affected
modules, summarize, wait for the user before committing, stage named files only (never
`git add .`), check the task off, log one line in `docs/progress-report.md`. The rules about
not brainstorming here and routing document changes to `/revise` hold as written there.

This file adds the four things a factory needs on top.

## 1. Scope (REQUIRED)

Read `$ARGUMENTS`. Format: `app:<name>` / `core:<module>` / `module:<:gradle:path>`,
optionally followed by a task code (e.g. `app:tokoku TOKOKU-T05`). If empty, list the scopes
that still have unfinished tasks.

The working folder is that scope's "Docs" column in `docs/INDEX.md`; the task file is
`<docs folder>/TASKS.md`. If it doesn't exist, point the user to
`/android-factory:break-task` and stop.

## 2. MODULE.md is the layout authority

Before implementing, read `<docs folder>/MODULE.md` alongside the PRD and the mockups. It
decides which folder new files go in, the module's local conventions, and the correct build
command — follow it, don't invent your own structure. If it's missing, run the
`/android-factory:docs` flow first.

## 3. A new Gradle module must be documented in the same iteration

The moment `settings.gradle` grows, register that module in `docs/INDEX.md` (type, owner,
prefix, Docs column) and create its docs folder — `PRD.md`, `MODULE.md`, `TASKS.md` — through
the `/android-factory:docs module:<path>` flow, **before the commit**. A module without
documents must never be committed.

## 4. Refresh the module documents before staging

So that code and documents land in the same commit, before `git add`:

- Public declarations added/changed/removed -> update the `<!-- auto: api -->` block in that
  module's `MODULE.md`.
- `build.gradle` dependencies changed -> update the `<!-- auto: depends-on -->` block **and**
  that module's row in `INDEX.md`, including "Change risk" if it gained a consumer.
- New folder under `src/main` -> update the `<!-- auto: layout -->` block.
- Update the "Docs refreshed" date.

Leave "Responsibility", "Local conventions" and "Constraints" **untouched** — those are the
user's writing. If nothing changed, skip this step without writing anything. Stage the changed
`MODULE.md` and `INDEX.md` together with the code.

## Commit message

`<type>(<module>): <short task title> (<PREFIX>-T##)`, e.g.
`feat(core-ui): add EmptyState component (CORE-UI-T03)`. The progress-report line carries the
module: `- [<date>] <PREFIX>-T## <title> — done (module: <module>)`.
