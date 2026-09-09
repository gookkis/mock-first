---
description: Audit drift between the PRD, the code, and TASKS — read-only, changes nothing
allowed-tools: Read, Glob, Grep, Bash(date*), Bash(git log*), Bash(git status*)
---

# Review — Does the PRD still match reality?

Answers one question: **does the PRD still match the code as it stands today?**

Use it when a project feels "half-done but hard to steer", before deciding where to go next.
How it differs from the other commands:

- `/adopt` inventories what exists (once, at the start).
- `/revise` applies a change you've **already** decided on.
- `/review` finds **what needs deciding**.

## Absolute rules

- **Read-only.** DO NOT change, create, or delete any file in this command — not the PRD,
  not TASKS, not the progress report. The output is a report on screen, nothing else.
- **Never invent a status.** A "done" status needs evidence you can name (file + function or
  class). If the evidence is thin, write `? UNVERIFIED` and say what needs a manual check. A
  review that guesses is more dangerous than no review at all.
- Run `date +%Y-%m-%d` for the date.
- Optional scope via `$ARGUMENTS` (e.g. a module or feature name) — if empty, review the
  whole PRD.

## Step 1 — Gather the sources

- The PRD (`docs/PRD.md`, or the parent PRD referenced from `docs/INDEX.md`/`README`).
- `docs/TASKS.md` and `docs/progress-report.md` if they exist.
- The code structure.
- `git log --oneline -20` — to spot whether the project ever changed direction.

## Step 2 — Match PRD → code

For every feature / functional requirement in the PRD, assign a status **with evidence**:

| Status | Meaning |
|---|---|
| `DONE` | an implementation covers it; you can point at the file and function |
| `PARTIAL` | the logic exists but isn't wired up, or has no UI surface |
| `MISSING` | no trace of it in the code |
| `? UNVERIFIED` | there are hints but the evidence is thin — say what to check |

Pay particular attention to **features whose logic exists but that have no UI**. That's the
most common reason a project feels stuck — the code has been paid for, but nobody can use it.

## Step 3 — Match the other direction (code → PRD)

- Code/modules the **PRD never mentions** — leftovers from an older direction, or features
  that grew on their own.
- Leftovers from a pivot: dependencies, folders, or config from an abandoned stack.
- Documents that **contradict reality**: a README or CLAUDE.md describing a project state
  that no longer holds. Quote the specific sentence — those documents get read first in the
  next session and seed wrong assumptions from the start.

## Step 4 — Report

```
PRD vs code review — <date>

Matching     : <N> features
Partial      : <N>  -> <short list + what's missing>
Missing      : <N>  -> <short list>
Unverified   : <N>  -> <list + what to check manually>

In the code but not in the PRD:
- <module/feature> — <where it probably came from>

Documents that contradict reality:
- <file>: "<quote>" — in fact <what's actually true>

Biggest gap: <one sentence — where the real work actually stopped>
```

Then offer **at most 3 recommendations**, each with its consequence — e.g. trim the PRD's
scope, finish one flow end-to-end so it's usable, or clean up the documents first.
**Suggestions stay suggestions** — apply nothing.

## Step 5 — Hand off

Based on the user's decision, point at the right command:

- Change the PRD's content/scope -> `/mock-first:revise`
- Work on what's `MISSING`/`PARTIAL` -> `/mock-first:break-task`
- A feature whose screen isn't defined yet -> `/mock-first:mockup <screen>`
- First time installing this in the project -> `/mock-first:adopt` first
