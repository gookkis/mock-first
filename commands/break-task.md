---
description: Turn an approved PRD + mockups into a task breakdown
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(date*)
---

# Task Breakdown

## Documents used

- `docs/PRD.md` — source of truth for features and scope.
- `docs/mockups/*-approved.html` — the visual reference for each screen.
- `docs/TASKS.md` — if it exists, continue the numbering; don't start over.

## Ground rules

- Run `date +%Y-%m-%d` for the date.
- A screen marked `[existing]` in the PRD is **not** new work — don't write a task for it
  unless the user explicitly asked for that screen to change. This is what stops a running
  project from suddenly sprouting dozens of tasks for features that have shipped years ago.
- Check first: does every screen under "Screens Needed" have an approved mockup? If some
  don't, tell the user which ones and ask whether to break down only the ready screens or
  wait for the remaining mockups.
- If `TASKS.md` already exists, **do not overwrite it** — continue numbering from the last
  task (T08 -> T09, not a reset to T01).

## Brainstorming mode

At most 3 points per category:

- **Alternative technical approaches** — when there's more than one reasonable way to build
  a feature, name the trade-off (complexity vs flexibility vs time).
- **Hidden tasks** that rarely make it into a PRD but are usually needed: input validation,
  error handling, environment/config setup, empty/loading/error states in the UI.
- **Cost warnings** when a feature is far more expensive than it looks (data migration,
  third-party integration, an architectural change).

If the user says "just follow the PRD", get on with it. **Suggestions stay suggestions.**

## Task phases

The usual order (skip phases that don't apply; don't force content into every phase):

```
Phase 1 — Data model / data structures
Phase 2 — Logic / business rules
Phase 3 — API / integrations (if any)
Phase 4 — UI components (matching the approved mockups)
Phase 5 — Wiring the pieces together
Phase 6 — Verification (manual checks against the PRD's success criteria — no automated tests)
```

## Output — docs/TASKS.md

```markdown
# Task List
Last updated: <date>

## Phase 1 — Data
- [ ] T01 — <task title>
  Files: <the files this will likely touch>
  Done when: <a concrete, verifiable criterion>
```

## After it's written

1. Summarize: task count per phase, which areas of the code get touched.
2. Ask whether anything needs changing.
3. Update `docs/progress-report.md`:
   `- [<date>] Breakdown <feature name>: <N> tasks (T## through T##)`
4. Suggest `/coding`.
