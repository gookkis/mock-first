---
description: Install Mock-First into a running Android project (run once)
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(date*), Bash(mkdir*)
---

# Adopt — Bootstrap a running project

Run this ONCE, at the start. Your job is to lay the Mock-First structure on top of an
existing project **without rewriting anything the user already wrote**.

## Absolute rules

- **The PRD is the parent document.** NEVER change, move, or overwrite an existing PRD.
- **What already exists counts as done.** DON'T create tasks for code that's already built.
- **Touch no code at all** in this command. Read only, plus new documents.
- Run `date +%Y-%m-%d` for the date.

## Step 1 — Scan the project

- Find the existing PRD (`docs/`, `README`, `*.md` in the root). If it's unclear, ask the
  user where it lives.
- Scan the Gradle structure:
  - `settings.gradle`/`settings.gradle.kts` -> the list of every module.
  - Each `build.gradle(.kts)` -> read the `dependencies` block for
    `implementation(project(":..."))`.
- Classify each module:
  - `app` — has the `com.android.application` plugin.
  - `shared/core` — used by more than one app.
  - `local` — used by exactly one app.
- Report the scan to the user and ask for corrections before continuing. Don't guess when a
  module is ambiguous.

## Step 2 — Build the dependency map

Write `docs/INDEX.md` — **the single most important artifact**, the basis of all cross-app
impact analysis.

````markdown
# Factory Map
Created: <date> | Source: settings.gradle + build.gradle
Parent PRD: <path to the existing PRD>

## Shared Modules
| Module | Used by | Change risk | Prefix | Docs |
|---|---|---|---|---|
| :core:ui | tokoku, resepku, kasirku | HIGH (3 apps) | CORE-UI | docs/core/core-ui/ |
| :core:data | tokoku, kasirku | MEDIUM (2 apps) | CORE-DATA | docs/core/core-data/ |

## Apps
| App | Modules used | Scope prefix | Docs |
|---|---|---|---|
| tokoku | :core:ui, :core:data | TOKOKU | docs/apps/tokoku/ |
| resepku | :core:ui | RESEPKU | docs/apps/resepku/ |

## Local Modules
| Module | Owned by | Prefix | Docs |
|---|---|---|---|
| :tokoku:feature:checkout | tokoku | TOKOKU-CHECKOUT | docs/apps/tokoku/modules/feature-checkout/ |

Update this file whenever a module, an app, or a dependency changes.
````

"Change risk": HIGH when used by 3+ apps, MEDIUM at 2, LOW at 1.

The "Docs" column is required for **every** module — that column is what guarantees each
module has its own documents. The folder slug rules live in `/mock-first:docs`.

## Step 3 — Create documents for EVERY module

Not just apps and shared modules — **every module in `settings.gradle` gets its own
folder**, local ones included. Run the `/mock-first:docs --all` flow: the folder layout, the
contents of `MODULE.md`, and the `PRD.md`/`TASKS.md` stubs are all defined there; don't
restate them differently here.

The expected result:

```
docs/
  INDEX.md                     <- the map (Step 2)
  progress-report.md           <- global log
  <the existing parent PRD>    <- DO NOT TOUCH
  apps/<app>/
    PRD.md  MODULE.md  TASKS.md  mockups/
    modules/<slug>/            <- local modules owned by this app
      PRD.md  MODULE.md  TASKS.md
  core/<slug>/                 <- shared modules
    PRD.md  MODULE.md  TASKS.md
```

How the two documents divide the work:

- `PRD.md` — **why & what**. The module's product scope, written through the
  `/mock-first:prd` Q&A. At adopt time it's still an empty stub.
- `MODULE.md` — **what's inside**. Responsibility, dependencies, public API surface, folder
  layout, build command. Filled automatically from the Gradle + source scan.

Because the project is already running, `MODULE.md` **can be filled in right away** — the
data is in the code. Leave what the code can't tell you ("Responsibility", "Local
conventions", "Constraints") empty; don't invent it.

When there are many modules, work in stages: shared modules first (highest risk), then apps,
then local modules. Report progress as you go; don't go quiet for long.

## Step 4 — Mark the baseline

1. Record the starting state in `docs/progress-report.md`:

```
# Progress Report
Baseline: <date> — Mock-First installed into a running project.
Starting state: <N> apps, <N> shared modules, <N> local modules, last commit <hash>.
Code predating this date is not tracked as tasks.
```

2. For each app, list the screens that **already exist** in its `PRD.md` marked `[existing]`
   — so `/mock-first:break-task` doesn't treat them as new work. Take the list from the
   existing Compose/Activity/Fragment files.

## Step 5 — Report

Show a summary: how many apps, how many shared modules, how many local modules, how many
docs folders you created, and which module is riskiest to change. Then suggest what's next:

- Adding a feature -> `/mock-first:prd app:<name>`
- A new app -> `/mock-first:new-app`
- Changing a shared module -> `/mock-first:revise`
- Filling in or refreshing module docs again -> `/mock-first:docs` (`--check` just reports
  which modules have incomplete docs)
