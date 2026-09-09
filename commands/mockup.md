---
description: Build a plain HTML mockup from the PRD for visual review
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(date*), Bash(mkdir*), Bash(mv*), Bash(cp*)
---

# Building a Mockup

## Scope (REQUIRED)

Read `$ARGUMENTS` as the screen name. If empty, read the "Screens Needed" section of
`docs/PRD.md` and ask which screen to work on.

## Ground rules

- For dates, ALWAYS run `date +%Y-%m-%d` first. Never guess the date.
- One `/mockup` run = one screen. Don't try to do every screen at once.
- Check whether this screen already has an `-approved.html` that a **completed** task in
  `TASKS.md` depends on. If so this is a REVISION, not a new mockup — send the user to
  `/revise` first. If no completed task uses it, carry on as usual (overwriting is fine).

## One mockup only — save tokens

- Produce **one** mockup, not several variants to choose from. Preparing alternatives that
  get thrown away burns tokens and slows the review down.
- Pick the visual direction that best serves the PRD's goal yourself, then **state your
  choice in 1-2 lines** (e.g. "list view, because the PRD stresses fast scanning"). Don't
  present a menu.
- If it misses, the user says what's wrong and you **fix the same file** — don't create a
  new file each iteration.
- If a screen or state is missing from the PRD but almost certainly needed (empty state,
  loading, error, onboarding), **mention it** — but don't build it until asked.
- If something in the PRD is genuinely ambiguous (not just a matter of taste), ask now —
  one question, not a queue of them.

## Steps

1. Read `docs/PRD.md`. If it doesn't exist, point the user to `/prd` and stop.
2. Check `docs/mockups/` to see which screens already have an `-approved.html`.
3. Build the mockup directly. Don't ask about visual direction unless the PRD is ambiguous.

## Building it

- Write **one file** at `docs/mockups/<screen-name>.html`.
- Mockup file rules:
  - Self-contained: HTML + inline CSS, no external dependencies, no build step.
  - Realistic dummy data (not "Lorem ipsum").
  - Focus on layout, hierarchy, and flow — not animation or polish.
- Explain in 2-3 lines: the main layout decision and what you prioritized.
- Ask the user to open the file in a browser and review it.

## Iterate until it fits

1. If something is off, **edit the same file** — change only what the user named, don't
   rewrite the whole file (token waste).
2. Repeat until the user says it's right.

## Once the user approves

1. Rename the file to `docs/mockups/<screen-name>-approved.html`
   (`mv docs/mockups/<screen-name>.html docs/mockups/<screen-name>-approved.html`).
2. Update the "Screens Needed" section of `docs/PRD.md`: mark this screen as having an
   approved mockup.
3. Update `docs/progress-report.md`: `- [<date>] Mockup <screen-name> approved`
4. Check which screens are still without a mockup:
   - Some left -> suggest `/mockup <next screen>`.
   - None left -> suggest `/break-task`.
