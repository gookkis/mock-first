# Walkthrough — one feature from `/prd` to commit

A short example of the base workflow (non-Android) on a small feature: "export list to CSV".

## 1. `/prd export CSV`

Claude asks one question at a time. On question 2 (problem) it offers up to 3 brainstormed
angles, e.g.:

> 1. Users currently copy-paste from the table — error prone on large lists.
> 2. Some users want this for accounting reconciliation, not just backup.
> 3. If this is really about accounting, a filtered export (date range) might matter more
>    than a full-table export.

You answer, it moves to question 3, and so on through scope, stack, and success criteria.
At the end it writes `docs/PRD.md` and flags the riskiest part of the scope before asking
you to confirm.

## 2. `/mockup export-dialog`

Claude reads the PRD, then writes one self-contained HTML file:
`docs/mockups/export-dialog.html` — a modal with column checkboxes. It explains the call in
two lines (column selection was implied by "some users only need certain fields"). You open
it in a browser; the checkbox list needs a "select all", so Claude edits that same file.
Once you're happy it's renamed to `export-dialog-approved.html`.

## 3. `/break-task`

Reads the PRD and the approved mockup, writes `docs/TASKS.md`:

```markdown
### Phase 1 — Data
- [ ] T01 — CSV serialization for the list model
### Phase 4 — UI components
- [ ] T02 — Export dialog (column checkboxes, per mockup)
### Phase 5 — Wiring
- [ ] T03 — Wire dialog to export button, trigger download
```

## 4. `/coding`

Takes T01, implements it, runs the build, shows a 3-line summary, waits for your OK, stages only
the touched file, commits `feat(export): add CSV serialization (T01)`, checks the box, asks
if you want to continue. Repeats for T02 and T03.

## 5. Plans change: `/revise add an "include headers" toggle`

Categorized as "B. Change an existing feature". It reports:

```
Impact:
- PRD               : Features (In Scope), the CSV export item
- Mockups           : export-dialog needs revision (add the toggle)
- Tasks not started : T03 — just needs editing
- Tasks ALREADY DONE: T02 — has to be redone (the dialog UI)
- Committed code    : the T02 commit
- Estimated new work: 1 new task
```

Then it waits for your decision and — once confirmed — marks T02 `⚠️ SUPERSEDED`, adds T04
as its replacement, appends a `_Revised <date>_` note to the PRD, and points you back to
`/mockup export-dialog` to update the approved mockup before continuing with `/coding`.
