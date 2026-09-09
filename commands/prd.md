---
description: Write a PRD through guided Q&A
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(date*)
---

# Writing a PRD

## Check for existing documents (REQUIRED before asking anything)

1. Check whether `docs/PRD.md` already exists:
   - **Doesn't exist** -> go ahead and create it.
   - **Already exists** -> DO NOT overwrite it. Ask the user: add the new feature to the
     existing PRD, or archive the old one to `docs/archive/PRD-<date>.md` and start over?
     `/prd` is only for a PRD written from scratch — if the goal is to add or change a
     feature in an existing PRD, point the user to `/revise` instead (safer, and it reports
     impact first).

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
6. What technical stack is used or planned? *(brainstorm — name the trade-offs when more
   than one choice is reasonable)*
7. What counts as "done" for this MVP? *(no brainstorming)*

## Once everything is answered

1. Read the existing project structure (if there is code already) to ground the context.
2. Write `docs/PRD.md` with this structure:

```markdown
# PRD — <Feature/App Name>
Created: <date from the date command> | Status: Draft
Feature code: <F01/F02/... — used for task numbering later>

## 1. Overview
## 2. Problem Statement
## 3. Target User
## 4. User Stories
## 5. Features (In Scope)
   - [number + one-line description per feature]
## 6. Out of Scope
## 7. Technical Stack
## 8. MVP Success Criteria
## 9. Screens Needed
   - [list of screens/pages. Screens that ALREADY EXIST in code are marked [existing]]
## 10. Notes / Assumptions
```

3. Show a summary of the PRD, then give your **partner notes**:
   - Which part is riskiest or least defined.
   - If the scope feels too large for an MVP, propose a leaner version (what can wait for
     v2) with your reasoning. Then ask whether the PRD is good or needs revision.
4. Once the user agrees, update `docs/progress-report.md` (create it if missing) with a
   single line: `- [<date>] PRD <feature code> created: <feature name> — <N> features in scope`
5. Suggest the next step: `/mockup`
