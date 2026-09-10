---
description: Create/refresh the PRD + technical docs for every Gradle module (idempotent)
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(date*), Bash(mkdir*), Bash(git log*)
---

# Module Docs — the per-module generator

Guarantees that **every Gradle module has its own documentation folder**: `PRD.md` (why &
what) + `MODULE.md` (what's inside) + `TASKS.md`.

Called automatically by `/android-factory:adopt` (all modules at once), `/android-factory:new-app`,
and `/android-factory:coding` (when a new module is born in Phase 0). Can also be run by hand.

## Arguments

Read `$ARGUMENTS`:

| Argument | Meaning |
|---|---|
| empty / `--all` | Every module in `settings.gradle`. Missing folders get created, existing ones get refreshed. |
| `app:<name>` / `core:<module>` / `module:<:gradle:path>` | A single module. |
| `--check` | **Read-only.** Only reports modules without docs or with stale docs. Writes nothing. |
| `--new` | Only creates what's missing. Modules that already have docs are left alone. |

## Absolute rules

- Run `date +%Y-%m-%d` for the date.
- **Idempotent.** Running it again must never break anything.
- **Touch no code.** This command only reads code and writes documents.
- **Never overwrite human-written sections.** In `MODULE.md` only the blocks marked
  `<!-- auto -->` may be rewritten; the rest is edited only when it's empty.
- **A `PRD.md` with content is never overwritten** — if one exists, skip it; its content
  changes through `/android-factory:prd` or `/android-factory:revise`.
- Don't invent anything. If the code doesn't tell you, write `_not filled in_` — never guess.

## Folder layout (used by every command)

The module slug is the Gradle path without the leading `:`, with `:` replaced by `-` ->
`:core:ui` -> `core-ui`. For a local module, if the first segment matches its owning app's
name, that segment is dropped -> `:tokoku:feature:checkout` (owned by tokoku) ->
`feature-checkout`.

```
docs/apps/<app>/                              app module
docs/apps/<app>/modules/<slug>/               local module (owned by that app)
docs/core/<slug>/                             shared module (used by 2+ apps)
```

Each folder holds `PRD.md`, `MODULE.md`, `TASKS.md` — plus `mockups/` for app folders.

When a local module becomes shared (a second app starts using it), **move** its folder to
`docs/core/<slug>/` and update `INDEX.md`. Report the move to the user; don't do it silently.

## Step 1 — Gather the inputs

Per module, read once:

1. The module's `build.gradle(.kts)` -> plugins, `dependencies`, namespace, any special
   `minSdk`/build variants.
2. `settings.gradle` -> the module's canonical path.
3. `docs/INDEX.md` -> module type, "Used by", scope prefix. If the module isn't in
   `INDEX.md` yet, add its row (and recompute the "Change risk" column).
4. The `src/main/` structure -> folder list + key files.
5. **Public** declarations in the source (`public`/default Kotlin, `@Composable`,
   `interface`, `data class`, `@Entity`, `@Dao`, Hilt `@Module`). `internal`/`private` are
   **not** recorded.

If a module has more than 30 public symbols, record the 30 most used by other modules (check
the imports in consuming modules) and close the table with
`_… and <N> more public symbols_`.

## Step 2 — Write `MODULE.md`

````markdown
# Module `<gradle path>`
Type: app | shared | local   |   Owner: <app / —>   |   Task prefix: <PREFIX>
Created: <date>  |  Docs refreshed: <date>

## Responsibility
_One or two sentences: what this module does, and what is NOT its job._

<!-- auto: used-by --> ## Used by
| Consumer | Type |
|---|---|
| :app:tokoku | app |
_(from INDEX.md; `—` when nothing uses it yet)_

<!-- auto: depends-on --> ## Depends on
| Internal module | Why |
|---|---|
| :core:data | product data source |

| External library | Version |
|---|---|
| androidx.room | 2.6.1 |

<!-- auto: api --> ## Public surface
| Symbol | Kind | Summary |
|---|---|---|
| EmptyState | @Composable | placeholder for an empty list |
_Public declarations only. Changing or removing a row here is a breaking change for the
consumers listed above._

<!-- auto: layout --> ## Layout
```
src/main/java/<ns>/
  ui/          <- Composables
  data/        <- Room + DTOs
```
New code for <X> goes in: <folder>

## Local conventions
_Rules specific to this module that the code can't tell you — e.g. all Composables are
stateless, state lives in the ViewModel. Written by a human; leave it empty until there
are any._

<!-- auto: build --> ## Build & verify
```
./gradlew <path>:assembleDebug
```
Shared modules: build the consuming apps too — <list of commands>.

## Constraints
_Anything that must not change without `/android-factory:revise`. Written by a human._
````

When refreshing a module that already has a `MODULE.md`: rewrite **only** the blocks marked
`<!-- auto: ... -->`, update the "Docs refreshed" date, and leave "Responsibility", "Local
conventions", and "Constraints" exactly as they are.

## Step 3 — The `PRD.md` and `TASKS.md` stubs

Only when they're **missing**. `PRD.md`:

```markdown
# PRD — <gradle path>
Parent: <path to the parent PRD from INDEX.md>  |  Scope prefix: <PREFIX>
Technical docs: ./MODULE.md

## Scope of this module
_(filled in via `/android-factory:prd module:<path>`)_

## Screens Needed
_(app/feature modules only; screens that ALREADY exist are marked `[existing]`)_

## Notes
- Complements the parent PRD; it does not replace it.
```

`TASKS.md`: empty, with the header `# Task List — <gradle path>` + `Prefix: <PREFIX>`.

## Step 4 — Detect staleness (always runs, including under `--check`)

Compare the documents against reality and report every mismatch:

| Symptom | Report as |
|---|---|
| Module in `settings.gradle` but no docs folder | `NO DOCS` |
| Docs folder exists but the module is gone from `settings.gradle` | `ORPHANED` — suggest archiving to `docs/archive/` |
| `build.gradle` dependencies ≠ the "Depends on" table | `DEPENDENCIES CHANGED` |
| A local module turns out to be used by 2+ apps | `NOW SHARED` — suggest moving the folder |
| A public symbol is gone but consumers still reference it | `API REMOVED` — route to `/android-factory:revise` |
| "Docs refreshed" is 30+ days older than the last commit touching the module (`git log -1 --format=%ad -- <path>`) | `STALE` |

`--check` stops here: print the findings table and write nothing.

## Step 5 — Report

```
Module docs — <date>
Created   : <N> modules (<list>)
Refreshed : <N> modules
Findings  : <N> (<NO DOCS: …>, <NOW SHARED: …>)
```

Then one line into `docs/progress-report.md`:
`- [<date>] Module docs: <N> created, <N> refreshed, <N> findings`

If a finding needs a decision (now shared, API removed, orphaned module), name the follow-up
command — `/android-factory:revise` — and **don't decide it yourself**.
