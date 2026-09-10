---
description: Build a plain HTML mockup from the PRD for visual review
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(date*), Bash(mkdir*), Bash(mv*), Bash(cp*)
---

# Building a Mockup

## Scope (REQUIRED)

Read `$ARGUMENTS`. Format: `app:<name> [screen-name]`. If empty, ask which app. Mockups are
for `app:` scopes only, never `core:`.

Working path: `docs/apps/<app>/mockups/`. The PRD you read: `docs/apps/<app>/PRD.md`.

## Android notes

The mockup stays static HTML (fast to write, opens straight in a browser) — not Compose. But
build it at a **phone viewport width (360-412px)** using Material 3 patterns (top app bar,
FAB, bottom nav, cards) so the translation to Compose is direct. Name the intended Compose
component in an HTML comment, e.g. `<!-- LazyColumn -->`.

## Ground rules

- For dates, ALWAYS run `date +%Y-%m-%d` first. Never guess the date.
- One `/mockup` run = one screen. Don't try to do every screen at once.

## One mockup only — save tokens

- Produce **one** mockup, not several variants to choose from. Preparing alternatives that
  get thrown away burns tokens and slows the review down.
- Pick the visual direction that best serves the PRD's goal yourself, then **state your
  choice in 1-2 lines** (e.g. "LazyColumn + cards, because the PRD stresses fast scanning").
  Don't present a menu.
- If it misses, the user says what's wrong and you **fix the same file** — don't create a
  new file each iteration.
- If a screen or state is missing from the PRD but almost certainly needed (empty state,
  loading, error), **mention it** — but don't build it until asked.
- If something in the PRD is genuinely ambiguous (not just a matter of taste), ask now —
  one question, not a queue of them.

## Steps

1. Read `docs/apps/<app>/PRD.md`. If it's missing, point the user to `/android-factory:prd
   app:<name>` and stop. A screen marked `[existing]` is already in the code; a mockup for
   it is a REVISION, not a new screen — route to `/android-factory:revise` first.
   **If the requested screen already has an `-approved.html`**: check
   `docs/apps/<app>/TASKS.md` for a completed task that depends on it. If there is one, stop
   and route to `/revise` first, so the superseded task gets recorded. If no completed task
   uses it, carry on (overwriting is fine).
2. Check `docs/apps/<app>/mockups/` to see which screens already have an `-approved.html`.
3. Decide which screen to work on:
   - `$ARGUMENTS` names one -> use it.
   - Empty -> list the screens from the PRD's "Screens Needed" with their status
     (mocked/not mocked) and ask which one.
4. Build the mockup directly. Don't ask about visual direction unless the PRD is ambiguous.

## Building it

- Write **one file** at `docs/apps/<app>/mockups/<screen-name>.html`.
- Mockup file rules:
  - Self-contained: HTML + inline CSS, no external dependencies, no build step.
  - Realistic dummy data (not "Lorem ipsum").
  - Focus on layout, hierarchy, and flow — not animation or polish.
  - Add HTML comments wherever the content will be dynamic.
- Explain in 2-3 lines: the main layout decision and what you prioritized.
- Ask the user to open the file in a browser and review it.

## Iterate until it fits

1. If something is off, **edit the same file** — change only what the user named, don't
   rewrite the whole file (token waste).
2. Repeat until the user says it's right.

## Once the user approves

1. Rename the file to `<screen-name>-approved.html` in the same folder
   (`mv docs/apps/<app>/mockups/<screen-name>.html` -> `...-approved.html`).
2. Update the "Screens Needed" section of `docs/apps/<app>/PRD.md`: mark this screen as
   having an approved mockup.
3. Update `docs/progress-report.md`: `- [<date>] Mockup <screen-name> approved`
4. Check which screens are still without a mockup:
   - Some left -> suggest `/mockup`.
   - None left -> suggest `/break-task`.
