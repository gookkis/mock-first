# Mock-First — A Visual-First Spec Workflow for Claude Code

_Concept document, v0.1_

## 1. Summary

Mock-First is a set of Claude Code slash commands for solo developers and small teams. They
run one cycle — PRD → mockup → task breakdown → iterative coding — plus commands for the
things that happen around it: adopting a running project, auditing drift, and handling
changes mid-flight.

Two things separate it from other spec-driven plugins:

1. **The visual mockup is a required gate**, not an optional extra. You agree on the shape
   of the thing before a line of code is written.
2. **Change has its own command** (`/revise`) with impact analysis — instead of editing the
   documents and hoping everything stays in sync.

## 2. The problem

Claude Code writes code well, but three things kept recurring in real use:

- **Prompt drift.** The same instructions get retyped every session in slightly different
  words, producing inconsistent output.
- **Visual interpretation misses.** A text PRD is easy to read two ways — what the developer
  pictured, and what the model understood. The mismatch only surfaces once the code exists,
  the most expensive point to fix it.
- **Documents fall out of sync.** Once coding is underway, a change of direction leaves the
  PRD, the task list, and the code contradicting each other. Nothing records that a task
  already checked off now produces something nobody wants.

Existing spec-driven plugins solve the first problem well (custom commands + CLAUDE.md). The
second and third are mostly left alone.

## 3. The existing landscape

| Project | Flow | Mockup | Change handling |
|---|---|---|---|
| GitHub Spec Kit | specify → plan → tasks → implement | — | — |
| ShipSpec | PRD → SDD → TASKS → implement | — | — |
| PRD Workflow Manager | PRD → tasks | — | — |
| Claude Code Spec Workflow | spec → tasks | — | — |
| interactive-spec (skill) | spec → HTML mockup | yes, but standalone, not part of a pipeline | — |
| **Mock-First** | PRD → mockup → tasks → coding | **required gate, one mockup iterated in place** | **`/revise` with impact analysis** |

This is a crowded category and some of the players are more mature (npm install, subagents,
dashboards). Mock-First doesn't compete on tooling breadth — the two differences above are
the reason it exists at all.

## 4. Design principles

**Collaborative while planning, obedient while executing.** The planning commands (`/prd`,
`/mockup`, `/break-task`) act as a thinking partner — at most three options per question,
risks named, a leaner version proposed when the scope is too big. The coding command
(`/coding`) has no opinions at all; ideas outside the task get noted, not built.

**Suggestions stay suggestions.** The model's ideas never enter a document unless the user
picked them explicitly. "Just go ahead" is respected without a second pitch.

**Nothing is lost.** Old documents are archived, not overwritten. Superseded tasks are
marked, not deleted. Committed code is never reverted without permission.

**Lightweight.** Plain markdown files. No npm, no dashboard, no subagents, no MCP server.

## 5. Architecture

```
/prd  ──▶  /mockup  ──▶  /break-task  ──▶  /coding
                ↺ per screen             ↺ per task
       └──────────────────┴──────────────────┘
                        /revise
```

Four commands going forward, one command for every path back. Two more sit outside the
cycle: `/adopt` installs the structure into a project that's already running, and `/review`
audits how far the PRD has drifted from the code.

### Artifacts

```
docs/
  PRD.md                        source of truth, persists across sessions
  TASKS.md                      a living checklist with continuous numbering
  progress-report.md            one line per event
  mockups/
    <screen>-approved.html      the visual reference for /coding
    archive/                    older mockup versions (from /revise)
  archive/                      older document versions
```

### What each command is for

| Command | Character | Output |
|---|---|---|
| `/adopt` | Read-only survey, run once | `docs/` skeleton + baseline |
| `/prd` | Collaborative Q&A | `PRD.md` |
| `/mockup` | Collaborative, one screen per call | `<screen>-approved.html` |
| `/break-task` | Collaborative, architecture-aware | `TASKS.md` |
| `/coding` | Pure execution, no opinions | Code + commits |
| `/review` | Analytical, read-only | A drift report |
| `/revise` | Analytical, must report before acting | Every document back in sync |

## 6. The mechanisms that make the difference

### 6.1 The mockup as a gate

`/break-task` checks mockup coverage before writing any tasks. Screens without an
`-approved.html` are reported, and the user chooses: continue without them (accepting less
precise UI tasks) or finish the mockups first.

`/coding` treats the approved mockup as a required reference, with explicit instructions not
to improvise the layout.

### 6.2 Impact analysis on change

`/revise` categorizes the change (new feature, changed feature, UI change, scope change,
technical change), then **stops and reports** before touching any file: which PRD sections
are affected, which mockups need revision, tasks not started (just edit them) vs tasks
already done (the code has to be redone), which commits became irrelevant, and how many new
tasks to expect. Only after the user decides does anything change.

## 7. Limitations

- **Not tested across many stacks.** The phases in `/break-task` assume a common order
  (data → logic → UI → integration) that may not fit every architecture.
- **Mockups are static HTML.** Flows that hinge on complex interaction aren't captured.
- **No automated visual verification.** Whether the code matches the mockup depends on the
  model following instructions; there's no screenshot diff.
- **Solo-dev oriented.** No multi-developer conflict handling or branch orchestration.

## 8. Where it's going

**Near term:** testing on more real projects; per-stack phase tuning.

**Medium term:** safety hooks to replace instruction-based rules.

**Longer term:** visual verification (screenshot diff between mockup and implementation); a
separate skill for project conventions that activates without being called.

## 9. The Android Apps Factory variant

For brownfield multi-module/multi-app projects, see `variants/android-factory/`. The main
differences: the parent PRD is never touched (the commands add a layer underneath it),
existing code gets no tasks (it's marked `[existing]`), every Gradle module gets its own
`PRD.md` + `MODULE.md` + `TASKS.md` generated from the Gradle and source scan, and
`INDEX.md` — the module → app dependency map — is the basis of the cross-app impact analysis
in `/revise`. See `docs/BRANDING.md` for the naming and repo publication guide.
