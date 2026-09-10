---
description: Add a new app to the factory, following an existing app's pattern
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(date*), Bash(mkdir*), Bash(cp*)
---

# New App

## Step 1 — Gather the details

Ask one at a time:

1. App name + package id
2. Which existing app is closest? (as a pattern to follow, not to copy wholesale)
3. Which shared modules will it use? — show the list from `docs/INDEX.md` as options
4. What makes this app different from the one it's modeled on?

Brainstorm (3 options max): if something this app needs could reasonably become a new shared
module — so the next app benefits too — say it now, before the code lands locally.

## Step 2 — Analyze

Read the reference app: module structure, dependencies, naming conventions. Report the plan:
which modules get created, where the pattern comes from, what has to be written fresh. Wait
for confirmation.

## Step 3 — Prepare the documents (not the code)

1. **Update** `docs/INDEX.md` first: add the new app's row, add each planned local module to
   the "Local Modules" table, and add this app's name to the "Used by" column of every shared
   module it uses. Update "Change risk" too — a module that was used by 2 apps is now used by
   3, so its risk goes up. Fill the "Docs" column for every new row.
2. Create the docs folders for the app **and each of its local modules** through the
   `/android-factory:docs app:<name>` flow — each gets `PRD.md`, `MODULE.md`, `TASKS.md` (the app
   folder also gets `mockups/`).
   Since the code doesn't exist yet, `MODULE.md` here holds the plan: "Responsibility" from
   the Step 1 answers, "Depends on" from the chosen shared modules, "Public surface" left as
   `_not filled in_`. Refresh it later with `/android-factory:docs` once the Phase 0 code exists.
3. Record it in `docs/progress-report.md`:
   `- [<date>] New app: <name> (modeled on: <app>), shared modules: <list>, docs: <N> folders`

## Step 4 — Hand off

No code is written here. Point the user to:

- `/android-factory:prd app:<name>` to define the scope
- then `/android-factory:mockup`, `/android-factory:break-task`, `/android-factory:coding`
- once the Gradle modules exist: `/android-factory:docs app:<name>` to fill in the public surface
  and layout in `MODULE.md`
