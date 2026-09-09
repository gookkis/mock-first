---
description: Work the next task from TASKS.md — implement, review, commit
allowed-tools: all
---

# Coding Iteration — Android Multi-Module

## Scope (REQUIRED)

Read `$ARGUMENTS`. Format: `app:<name>` / `core:<module>` / `module:<:gradle:path>`,
optionally followed by a task code (e.g. `app:tokoku TOKOKU-T05`). If empty, list the scopes
that still have unfinished tasks.

The working folder is that scope's "Docs" column in `docs/INDEX.md`. Task file:
`<docs folder>/TASKS.md`.

## Ground rules

- For dates, ALWAYS run `date +%Y-%m-%d` first. Never guess the date.
- This phase is pure execution. **No brainstorming.** If an idea outside the task's scope
  comes up, note it in one line — don't act on it yourself.
- If the user asks for a change that touches the PRD, a mockup, or the task list (not just a
  small fix inside this task), DON'T do it here. Send them to `/revise` so the documents
  stay in sync.

## Checks before starting

1. Read `<docs folder>/TASKS.md`. If it doesn't exist, point the user to
   `/mock-first:break-task` and stop.
2. Run `git status`:
   - Uncommitted changes -> list them and ask: commit first, stash, or carry on? **Don't
     start before the user answers.**
   - Not a git repo -> tell the user, and ask whether to `git init` first or continue
     without commits.
3. Read `<docs folder>/PRD.md`, `<docs folder>/MODULE.md`, and the relevant mockups
   (`docs/apps/<app>/mockups/*-approved.html`). `MODULE.md` decides which folder new files
   go in, the module's local conventions, and the correct build command — follow it, don't
   invent your own structure.

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
   - **A new Gradle module?** The moment `settings.gradle` grows, register that module in
     `docs/INDEX.md` (type, owner, prefix, Docs column) and create its docs folder
     (`PRD.md`, `MODULE.md`, `TASKS.md`) through the `/mock-first:docs module:<path>` flow —
     **in the same iteration, before the commit.** A module without documents must never be
     committed.
4. Build **only the affected modules**, not the whole project (slow):
   `./gradlew :<module>:assembleDebug`. If the task touches a shared module, also build the
   apps that use it (see the "Used by" column in `docs/INDEX.md`) — a shared change can
   break another app even when the module itself compiles. Fix failures. **If it fails 3
   times in a row for the same reason, stop and report** — don't keep trying and risk
   breaking something else.
5. Show a summary of the changes: files touched + what changed (not a full diff unless
   asked).
6. Ask the user to confirm before committing.

## Commit (after the user agrees)

1. **Refresh the module documents first**, before staging — so the code and its documents
   land in the same commit:
   - Public declarations added/changed/removed -> update the `<!-- auto: api -->` block in
     that module's `MODULE.md`.
   - `build.gradle` dependencies changed -> update the `<!-- auto: depends-on -->` block in
     `MODULE.md` **and** that module's row in `INDEX.md` (including "Change risk" if it
     gained a consumer).
   - New folder under `src/main` -> update the `<!-- auto: layout -->` block.
   - Update the "Docs refreshed" date.
   Leave "Responsibility", "Local conventions", and "Constraints" **untouched** — those are
   the user's writing. If nothing changed, skip this step without writing anything.
2. **Stage only the files you touched for this task** (including `MODULE.md` and `INDEX.md`
   if they changed above), named one by one: `git add <file1> <file2>`. **NEVER use
   `git add .` or `git add -A`** — unrelated changes (env files, logs, build artifacts, the
   user's own work in progress) must not ride along.
3. Run `git status` once more to confirm what's staged is right.
4. Commit: `<type>(<module>): <short task title> (<PREFIX>-T##)`, e.g.
   `feat(core-ui): add EmptyState component (CORE-UI-T03)`
5. Check the task off in `<docs folder>/TASKS.md`.
6. Update `docs/progress-report.md` with ONE line:
   `- [<date>] <PREFIX>-T## <title> — done (module: <module>)`. No long transcripts.
7. Ask: move on to the next task, or stop here?

## When every task is done

- Report the MVP as complete and show the summary from the progress report.
- Mention any notes or ideas that came up during coding but fell outside scope.
- Suggest what's next: a full manual pass, or `/prd` for the next feature.
