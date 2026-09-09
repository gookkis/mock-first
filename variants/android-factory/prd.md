---
description: Write a PRD through guided Q&A
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(date*)
---

# Writing a PRD — scoped

## Scope (REQUIRED)

Read `$ARGUMENTS`. Format: `app:<name>`, `core:<module>`, or `module:<:gradle:path>` (for
local modules). If empty, list the options from `docs/INDEX.md` and ask which scope.

You write to the `PRD.md` inside that scope's docs folder (the "Docs" column in `INDEX.md`;
the layout rules live in `/mock-first:docs`) — **not the parent PRD.** The parent PRD (path
in `INDEX.md`) is read for context only and never changed here. If the change affects the
overall product direction, send the user to `/mock-first:revise`.

If that scope's docs folder doesn't exist yet, create it through the `/mock-first:docs
module:<path>` flow — never write a PRD into a folder that isn't listed in `INDEX.md`.

Before asking anything, **read the `MODULE.md` in that folder** if it exists: the module's
responsibility, dependencies, and public surface are the context that makes question 6
(which module) far sharper.

## Rules

- Ask ONE question at a time and wait for the answer before moving on.
- Never invent answers. If an answer is unclear, ask one short follow-up.
- Write the document in the user's language; default to English.
- If `$ARGUMENTS` is given, use it as the initial feature name/description and skip
  question 1.
- For dates, ALWAYS run `date +%Y-%m-%d` first. Never guess the date.

## Brainstorming mode

You are not a form-filler — act as a thinking partner.

**Brainstorm ONLY on questions 2, 4, 5, and 6.** Answer the rest and move on, so the session
doesn't drag.

On those questions, after the user answers:

1. Offer **at most 3 options/ideas** built on the user's stated goal.
2. For each option: **why it helps** + **what it costs** (complexity, time, dependencies).
   One line per option.
3. If the user's answer looks problematic (scope too wide, risky assumption), say so plainly.
4. Close with: "Want to take one of these, or stick with your answer?"

**Suggestions stay suggestions.** The decision is the user's. Never put your own idea into
the PRD unless the user picked it explicitly. If the user says "just go ahead", go ahead —
don't pitch again.

## Question order

1. What feature / app are we building? *(no brainstorming)*
2. What problem does it solve? (who struggles, and why) *(brainstorm)*
3. Who are the target users? *(no brainstorming)*
4. Which core features must be there? (a rough list is fine) *(brainstorm)*
5. What is explicitly OUT of scope this time? *(brainstorm)*
6. Which module does this code belong in — shared/core or local to an app? *(brainstorm —
   name the trade-off: shared means writing it once but every change risks other apps;
   local is safe but duplicated. Check INDEX.md for the modules that already exist)*
7. What counts as "done" for this MVP? *(no brainstorming)*

## Once everything is answered

1. Read the existing project structure (if there is code already) to ground the context.
2. Check whether the scope's `PRD.md` already has content:
   - **Empty/missing** -> write it.
   - **Already filled** -> DO NOT overwrite. If the goal is to add or change a feature in an
     existing PRD, send the user to `/revise` (safer, and it reports impact first). `/prd` is
     only for a PRD written from scratch — first ask whether the old one should be archived
     to `docs/archive/PRD-<date>.md`.
3. Structure of the scope's `PRD.md` (this replaces the empty stub from `/mock-first:docs`):

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

4. Show a summary of the PRD, then give your **partner notes**:
   - Which part is riskiest or least defined.
   - If the scope feels too large for an MVP, propose a leaner version (what can wait for
     v2) with your reasoning. Then ask whether the PRD is good or needs revision.
5. Once the user agrees, update `docs/progress-report.md` (create it if missing) with a
   single line: `- [<date>] PRD <feature code> created: <feature name> — <N> features in scope`
6. If the answer to question 6 surfaced a **new module** not yet in `INDEX.md`: register it
   there (type, owner, prefix, Docs column) and create its docs folder through the
   `/mock-first:docs module:<path>` flow. No code yet — the Gradle module itself is born in
   Phase 0 of `/mock-first:break-task` — but its documents stand up now, so no module ever
   exists without a PRD.
7. Suggest the next step: `/mock-first:mockup` (app scope) or `/mock-first:break-task`
   (core/module scope).
