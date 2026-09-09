---
description: Handle changes — with cross-app impact analysis
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(date*), Bash(mkdir*), Bash(cp*), Bash(mv*), Bash(git status*), Bash(git log*), Bash(git mv*)
---

# Revise — Multi-App

Handles changes AFTER the documents exist or coding is underway.
In a multi-app project, **cross-app impact is the main thing you have to report**.

## Absolute rules
- Run `date +%Y-%m-%d` for the date.
- NEVER overwrite a file without archiving it to `docs/archive/` first.
- NEVER revert committed code without the user's permission.
- CHANGE NOTHING before the user has confirmed Step 2.

## Step 1 — Categorize
From `$ARGUMENTS`, or ask if it's empty. Categories:
- **A. New app** -> route to `/mock-first:new-app` and stop.
- **B. New feature in one app** -> local impact.
- **C. Change an existing feature**
- **D. Change a shared/core module** -> **cross-app impact, needs the full analysis**
- **E. Change a mockup / the UI**
- **F. Change scope** (drop/defer/promote into the MVP)
- **G. Cross-app refactor**
- **H. Module added / removed / a local module promoted to shared** -> the structure changes,
  and the docs folders move with it

## Step 2 — Impact analysis (REQUIRED, and stop after it)
Read `docs/INDEX.md`, the parent PRD, the `MODULE.md` of every module touched (that's where
the public surface and the consumer list live), every relevant `TASKS.md`, and
`git log --oneline -20`.

### For categories D and G (touching a shared module)
Report in this format:

```
Change: <summary>
Module touched: :core:ui  (HIGH risk — used by 3 apps)

Impact per app:
┌ tokoku
│  Tasks not started : TOKOKU-T08 — just needs editing
│  Tasks ALREADY DONE: TOKOKU-T03 — has to be redone
│  Mockups affected  : dashboard
├ resepku
│  Tasks ALREADY DONE: RESEPKU-T05 — has to be redone
├ kasirku
│  Not affected (doesn't use the component being changed)

Total: 2 apps need adapting, 3 tasks superseded, ~4 new tasks
Commits affected: a3f9c1, b71e02
```

Then offer **at most 3 options** with real trade-offs, e.g.:
1. *Change it in place and adapt every app* — clean, but 3 tasks get redone.
2. *Add a new API and keep the old one* — nothing gets redone, but there's temporary duplication.
3. *Copy it into the local module of the app that needs it* — the other apps are safe, but the code is no longer single-source.

### For the other categories
Same format, but only one app.

**STOP. Wait for the user's decision before Step 3.**

## Step 3 — Apply
### Archive first
`docs/archive/<filename>-<date>.<ext>` — mockups go to `mockups/archive/`.

### PRD
- The parent PRD **is only changed** when the change affects the overall product direction.
  For app- or module-level changes, only that scope's PRD changes.
- Don't erase history — append `_Revised <date>: <summary>_`.

### TASKS.md (possibly more than one file)
- Task not started -> edit it directly.
- Task already checked off -> **don't remove the checkmark**:
  `- [x] TOKOKU-T03 — <title> ⚠️ SUPERSEDED (revised <date>, see TOKOKU-T15)`
  then add the replacement task with the next number in sequence.
- A new task in another app caused by the shared-module change -> write it in that app's
  `TASKS.md`, not merged into one file.

### INDEX.md
If a module or dependency was added or changed, **update the map**. This is the step people
forget, and it's what makes the next impact analysis wrong. The "Docs" column has to be
updated too.

### MODULE.md (every module touched)
- Public surface changed -> update the `<!-- auto: api -->` block, and mention the change in
  the **consuming** modules' `MODULE.md` too if they use that symbol.
- Dependencies changed -> update the `<!-- auto: depends-on -->` block and "Used by".
- "Responsibility", "Local conventions", and "Constraints" only change if the user asks.

### Docs folder structure (category H)
- **New module** -> register it in `INDEX.md` and create its docs folder through the
  `/mock-first:docs module:<path>` flow.
- **Local module becomes shared** -> move its folder from `docs/apps/<app>/modules/<slug>/`
  to `docs/core/<slug>/` (`git mv` if it's tracked), update the Docs column and "Change
  risk", and record the move. **Don't change the old task prefix** — already-completed tasks
  get hard to trace if their numbers change.
- **Module deleted** -> move its docs folder to `docs/archive/<slug>-<date>/` rather than
  deleting it. Remove its row from `INDEX.md` and name the tasks that are now orphaned.

### Mockups
Don't build them here. Finish the documents, then point the user to
`/mock-first:mockup app:<name> <screen>`.

## Step 4 — Record it
```
- [<date>] REVISION (<category>): <summary>
  Modules: <list> | Apps affected: <list>
  Impact: <N> new tasks, <N> superseded tasks
```
Then suggest the next step for that category.
