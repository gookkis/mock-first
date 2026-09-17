---
description: Handle changes after the documents exist or coding is underway — with impact analysis
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(date*), Bash(mkdir*), Bash(cp*), Bash(mv*), Bash(git status*), Bash(git log*)
---

# Revise

Handles changes AFTER the documents exist or coding is underway — a feature the PRD doesn't
cover, a changed feature, a mockup/UI change, a scope change, or a change of technical
approach.

## Platform (REQUIRED — before anything else)

The document paths, the mockup file names and the verification command are platform-specific.

Seven profiles live in `platforms/`: `web`, `mobile-web`, `android`, `ios`,
`desktop-windows`, `desktop-macos`, `desktop-linux`. Resolve which one applies:

1. `$ARGUMENTS` starts with a profile name -> that is the platform, and the rest of
   `$ARGUMENTS` keeps its usual meaning. `desktop` on its own is **not** a platform — the
   three desktop OSes differ in menu bar, window chrome and packaging, so ask which one.
2. Otherwise a `Platform:` line — in `docs/TASKS.md` if the breakdown recorded one, then in
   the PRD's "Technical Stack".
3. Otherwise read the repo:

   | Signal | Platform |
   | --- | --- |
   | `settings.gradle*`, `AndroidManifest.xml` | `android` |
   | `*.xcodeproj` / `Package.swift` with an iOS destination | `ios` |
   | the same with a macOS destination, or AppKit | `desktop-macos` |
   | `*.csproj` / `*.sln` with WinUI, WPF or WinForms | `desktop-windows` |
   | `meson.build`, or GTK/libadwaita/Qt in the dependencies | `desktop-linux` |
   | `package.json` with `electron` or `@tauri-apps` | a desktop OS — ask which |
   | `package.json` with a web framework | `web` or `mobile-web` |

4. Still unclear, or the stack is cross-platform (Flutter, React Native, KMP, Capacitor,
   Electron, Tauri, Compose Multiplatform) -> **ask once**, naming the candidates. A
   cross-platform stack still ships one screen per platform; one run does one of them.

Then read the profile — `${CLAUDE_PLUGIN_ROOT}/platforms/<platform>.md`, or
`~/.claude/platforms/<platform>.md` if the commands were copied in without the plugin — and
run its project-shape check: a multi-app project keeps its documents under `docs/apps/<app>/`
with app-prefixed task numbers, and reading `docs/mockups/` there would report screens as
missing when they are simply somewhere else.

## Absolute rules

- Run `date +%Y-%m-%d` for the date.
- NEVER overwrite a file without archiving it to `docs/archive/` first.
- NEVER revert committed code without the user's permission.
- CHANGE NOTHING before the user has confirmed Step 2.

## Step 1 — Categorize

From `$ARGUMENTS`, or ask if it's empty. Categories:

- **A. New feature** not yet in the PRD
- **B. Change an existing feature**
- **C. Change a mockup / the UI**
- **D. Change scope** (drop, defer, or promote something into the MVP)
- **E. Change the technical approach / architecture**

## Step 2 — Impact analysis (REQUIRED, and stop after it)

Read the PRD and the task file at the paths the profile gives (`docs/` on a single-app
project, `docs/apps/<app>/` on a multi-app one), plus `git log --oneline -20`. Report in this
format:

```
Change: <summary>

Impact:
- PRD              : <affected sections>
- Mockups          : <screens needing revision, or "none" — name the platform when the
                     PRD targets more than one, since each has its own file>
- Tasks not started : <list — just edit them>
- Tasks ALREADY DONE: <list — the code has to be redone>
- Committed code   : <related commits, or "none">
- Estimated new work: <N> new tasks
```

Then offer **at most 3 options** when there's more than one way to handle it, each with a
real trade-off (cost vs how clean the result is).

**STOP. Wait for the user's decision before Step 3.**

## Step 3 — Apply

### Archive first
`docs/archive/<filename>-<date>.<ext>` — old mockups go to `docs/mockups/archive/`.

### PRD
- Add the feature/change with the next number in sequence (don't reuse existing numbers).
- Don't erase history — append `_Revised <date>: <summary>_` to the affected section.

### TASKS.md
- Task not started -> edit it directly.
- Task already checked off -> **don't remove the checkmark**. Mark it:
  `- [x] T05 — <title> ⚠️ SUPERSEDED (revised <date>, see T12)` then add the replacement
  task with the next number in sequence (T12 -> T13, not a reset to T01).

### Mockups
Don't build them here. If a screen is new or changed, finish the documents first, then point
the user to `/mockup <screen>`.

## Step 4 — Record it

```
- [<date>] REVISION (<category>): <summary>
  Impact: <N> new tasks, <N> superseded tasks
```

Then suggest the next step for that category (`/mockup` if there's a new screen, `/coding`
otherwise).
