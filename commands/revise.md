---
description: Handle changes after the documents exist or coding is underway — with impact analysis
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(date*), Bash(mkdir*), Bash(cp*), Bash(mv*), Bash(git status*), Bash(git log*)
---

# Revise

Handles changes AFTER the documents exist or coding is underway — a feature the PRD doesn't
cover, a changed feature, a mockup/UI change, a scope change, or a change of technical
approach.

## Absolute rules

- Run `date +%Y-%m-%d` for the date.
- NEVER overwrite a file without archiving it to `docs/archive/` first.
- NEVER revert committed code without the user's permission.
- CHANGE NOTHING before the user has confirmed Step 2.

## Step 1 — Categorize

From `$ARGUMENTS`, or ask if it's empty. Categories:

- **A. New feature** not yet in the PRD
- **B. Change an existing feature**
- **C. Change a mockup / the UI**
- **D. Change scope** (drop, defer, or promote something into the MVP)
- **E. Change the technical approach / architecture**

## Step 2 — Impact analysis (REQUIRED, and stop after it)

Read `docs/PRD.md`, `docs/TASKS.md`, and `git log --oneline -20`. Report in this format:

```
Change: <summary>

Impact:
- PRD              : <affected sections>
- Mockups          : <screens needing revision, or "none">
- Tasks not started : <list — just edit them>
- Tasks ALREADY DONE: <list — the code has to be redone>
- Committed code   : <related commits, or "none">
- Estimated new work: <N> new tasks
```

Then offer **at most 3 options** when there's more than one way to handle it, each with a
real trade-off (cost vs how clean the result is).

**STOP. Wait for the user's decision before Step 3.**

## Step 3 — Apply

### Archive first
`docs/archive/<filename>-<date>.<ext>` — old mockups go to `docs/mockups/archive/`.

### PRD
- Add the feature/change with the next number in sequence (don't reuse existing numbers).
- Don't erase history — append `_Revised <date>: <summary>_` to the affected section.

### TASKS.md
- Task not started -> edit it directly.
- Task already checked off -> **don't remove the checkmark**. Mark it:
  `- [x] T05 — <title> ⚠️ SUPERSEDED (revised <date>, see T12)` then add the replacement
  task with the next number in sequence (T12 -> T13, not a reset to T01).

### Mockups
Don't build them here. If a screen is new or changed, finish the documents first, then point
the user to `/mockup <screen>`.

## Step 4 — Record it

```
- [<date>] REVISION (<category>): <summary>
  Impact: <N> new tasks, <N> superseded tasks
```

Then suggest the next step for that category (`/mockup` if there's a new screen, `/coding`
otherwise).
