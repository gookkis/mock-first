---
description: Install Mock-First into a project that's already running (run once)
allowed-tools: Read, Glob, Grep, Write, Bash(date*), Bash(mkdir*), Bash(git log*), Bash(git status*)
---

# Adopt — Bootstrap a running project

Run this ONCE, at the start. Your job is to lay the Mock-First structure on top of an
existing project **without rewriting anything the user already wrote**.

## Absolute rules

- **Existing documents belong to the user.** NEVER change, move, or overwrite an existing
  PRD / spec / README. If `docs/PRD.md` already exists, treat it as the parent PRD: read it
  for context, never rewrite it.
- **What already exists counts as done.** DON'T create tasks for code that's already built.
- **Touch no code at all** in this command. Read only, plus new documents.
- Run `date +%Y-%m-%d` for the date. Never guess it.
- Suggest the user run this on a separate branch (`git checkout -b setup/mock-first`) so the
  result is easy to throw away if the scan gets things wrong.

## Step 1 — Scan the project

1. Look for existing planning documents: `docs/`, `README*`, `*.md` in the root, `spec*` or
   `design*` folders. If it's unclear which one acts as the PRD, ask the user.
2. Detect the stack and how it runs from whatever manifest exists (`package.json`,
   `pyproject.toml`, `go.mod`, `Cargo.toml`, `*.csproj`, `build.gradle`, ...):
   - the run / build commands (tests aren't part of this pipeline),
   - the main folder layout (where the UI lives, where the logic lives, where data lives).
3. List the **screens/pages/features that ALREADY exist** in code. Depending on the stack,
   that comes from route files, a pages/screens/views folder, or an endpoint list.
4. Read `git log --oneline -20` to see what's actively being worked on.
5. **Report the scan to the user and ask for corrections before continuing.** Don't guess at
   anything ambiguous. The part the user most needs to check is the list of screens you
   marked as "already exists" — if that's wrong, every breakdown after it is wrong too.

## Step 2 — Create the document skeleton

Create only what's missing (empty for now, filled in as it gets used):

```
docs/
  PRD.md                 <- only if MISSING. If it exists, leave it exactly as it is.
  TASKS.md               <- empty, with a header and numbering starting at T01
  progress-report.md     <- baseline (new)
  mockups/               <- empty
  archive/               <- empty
```

If `docs/PRD.md` is missing, write a **stub** — don't invent PRD content from the code:

```markdown
# PRD — <project name>
Created: <date> | Status: adopted from a running project
Feature codes: F01, F02, ... (for task numbering)

## Overview
_(filled in when /mock-first:prd is run for the next feature)_

## Stack & How It Runs
- Run: <command>
- Build: <command>

## Screens Needed
_(screens that ALREADY exist are marked [existing] — they are not new work)_
- <screen A> [existing]
- <screen B> [existing]

## Notes
- This document was created by /mock-first:adopt from a running project.
- Code predating the baseline date is not tracked as tasks.
```

If `docs/PRD.md` **already exists**: don't touch it. Instead put the screen list in a new
`docs/SCREENS.md` with every screen marked `[existing]`, and note there that the parent PRD
lives at its original path.

## Step 3 — Mark the baseline

Record the starting state in `docs/progress-report.md`:

```
# Progress Report
Baseline: <date> — Mock-First installed into a running project.
Starting state: <stack>, <N> existing screens, last commit <hash>.
Code predating this date is not tracked as tasks.
```

## Step 4 — Report

Show a summary: the stack you detected, the run/build commands, how many screens you marked
`[existing]`, which documents you created (and which you deliberately left alone). Then
suggest what's next:

- Adding a feature -> `/mock-first:prd <feature name>`
- Changing an existing feature/screen -> `/mock-first:revise`

Safe to re-run: running `/mock-first:adopt` again won't overwrite an existing PRD or TASKS —
it only refreshes the screen list and the baseline.
