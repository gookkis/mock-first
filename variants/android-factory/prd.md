---
description: Write a PRD through guided Q&A, scoped to one app or module
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(date*)
---

# Writing a PRD — scoped

**Follow `/mock-first:prd`.** It owns the Q&A discipline — one question at a time, brainstorm
only on questions 2, 4, 5 and 6, suggestions stay suggestions, partner notes at the end — and
this file does not restate it. What a factory changes is the four things below.

## 1. Scope (REQUIRED)

Read `$ARGUMENTS`. Format: `app:<name>`, `core:<module>`, or `module:<:gradle:path>` (for
local modules). If empty, list the options from `docs/INDEX.md` and ask which scope.

You write to the `PRD.md` inside that scope's docs folder — the "Docs" column in `INDEX.md`;
the layout rules live in `/android-factory:docs`. **Never the parent PRD.** The parent is read
for context only and never changed here; if the change affects the overall product direction,
send the user to `/android-factory:revise`.

If that scope's docs folder doesn't exist yet, create it through the `/android-factory:docs
module:<path>` flow — never write a PRD into a folder that isn't listed in `INDEX.md`.

Before asking anything, **read the `MODULE.md` in that folder** if it exists: the module's
responsibility, dependencies and public surface are the context that makes question 6 far
sharper.

## 2. Question 6 is a different question

Where the base command asks about the technical stack, ask instead:

> **Which module does this code belong in — shared/core, or local to an app?** *(brainstorm —
> name the trade-off: shared means writing it once but every change risks other apps; local is
> safe but duplicated. Check `INDEX.md` for the modules that already exist.)*

The stack is not in question here: it is Android, and `platforms/android.md` is the profile
every command uses.

## 3. The document has a different structure

```markdown
# PRD — <Feature Name>
Created: <date from the date command>  |  Status: Draft
Feature code: <F01/F02/... — used for task numbering later>

## 1. Overview
## 2. Problem Statement
## 3. Target User
## 4. Features (In Scope)
   - [number + one-line description per feature]
## 5. Out of Scope
## 6. Modules Affected
   - [shared / local, name them. If a NEW module is needed, write its Gradle path]
## 7. MVP Success Criteria
## 8. Screens Needed
   - [list of screens/pages. Screens that ALREADY EXIST in code are marked [existing]]
## 9. Notes / Assumptions
```

This replaces the empty stub left by `/android-factory:docs`. The same "don't overwrite a
filled PRD" rule as the base command applies — archive to `docs/archive/PRD-<date>.md` first,
or route to `/revise`.

## 4. A new module has to be registered before anything else is written

If question 6 surfaced a **new module** not yet in `INDEX.md`: register it there (type, owner,
prefix, Docs column) and create its docs folder through the `/android-factory:docs
module:<path>` flow. No code yet — the Gradle module itself is born in Phase 0 of
`/android-factory:break-task` — but its documents stand up now, so no module ever exists
without a PRD.

## Next step

`/mock-first:mockup` for an app scope, `/android-factory:break-task` for a core or module
scope. The mockup command reads `INDEX.md` through its Android profile and writes into that
scope's `mockups/` folder on its own.
