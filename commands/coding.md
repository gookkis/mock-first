---
description: Work the next task from TASKS.md — implement, review, commit
allowed-tools: all
---

# Coding Iteration

## Ground rules

- For dates, ALWAYS run `date +%Y-%m-%d` first. Never guess the date.
- This phase is pure execution. **No brainstorming.** If an idea outside the task's scope
  comes up, note it in one line — don't act on it yourself.
- If the user asks for a change that touches the PRD, a mockup, or the task list (not just a
  small fix inside this task), DON'T do it here. Send them to `/revise` so the documents
  stay in sync.

## Checks before starting

1. Read `docs/TASKS.md`. If it doesn't exist, point the user to `/break-task` and stop.
2. Run `git status`:
   - Uncommitted changes -> list them and ask: commit first, stash, or carry on? **Don't
     start before the user answers.**
   - Not a git repo -> tell the user, and ask whether to `git init` first or continue
     without commits.
3. Read `docs/PRD.md` and the relevant mockups (`docs/mockups/*-approved.html`) for this
   task.

## Each iteration

1. Take the **topmost unchecked task**.
   - `$ARGUMENTS` contains a task code (e.g. `T03`) -> work that one.
   - A task marked `[needs decision]` -> ask the user before starting.
   - Everything is checked -> jump to "When every task is done".
2. Name the task you're about to do plus a short plan (3 lines max). Then go — no approval
   needed for this part.
3. Implement:
   - Follow the project's existing conventions (naming, folder structure, style).
   - **Don't touch files outside this task's scope.** If it turns out you must, stop and ask.
   - For UI, follow the approved mockup — don't improvise your own layout.
   - **Don't write tests.** This pipeline doesn't use automated tests — verify through the
     build plus a manual check against the task's success criterion. If the user does want
     tests, that's its own task via `/revise`.
4. Build the code you touched (if the project has a build step). Fix failures. **If it fails
   3 times in a row for the same reason, stop and report** — don't keep trying and risk
   breaking something else.
5. Show a summary of the changes: files touched + what changed (not a full diff unless
   asked).
6. Ask the user to confirm before committing.

## Commit (after the user agrees)

1. **Stage only the files you touched for this task**, named one by one:
   `git add <file1> <file2>`. **NEVER use `git add .` or `git add -A`** — unrelated changes
   (env files, logs, build artifacts, the user's own work in progress) must not ride along.
2. Run `git status` once more to confirm what's staged is right.
3. Commit: `<type>(<area>): <short task title> (<T##>)`, e.g.
   `feat(checkout): add coupon validation (T03)`
4. Check the task off in `docs/TASKS.md`.
5. Update `docs/progress-report.md` with ONE line:
   `- [<date>] T## <title> — done (area: <area>)`. No long transcripts.
6. Ask: move on to the next task, or stop here?

## When every task is done

- Report the feature/MVP as complete and show the summary from the progress report.
- Mention any notes or ideas that came up during coding but fell outside scope.
- Suggest what's next: a full manual pass, or `/prd` for the next feature.
