---
description: Turn an approved PRD + mockups into a task breakdown
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(date*)
---

# Task Breakdown

## Platform (REQUIRED — before anything else)

The phase order, the file layout and the verification command are all platform-specific.

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

Then read the profile:

```
${CLAUDE_PLUGIN_ROOT}/platforms/<platform>.md
```

(or `~/.claude/platforms/<platform>.md` if the commands were copied in without the plugin).
Its "Task phases", "Verify", "Where things live" and "Hidden work this platform always drags
in" sections **replace** the generic versions below. Its "Where things live" opens with a project-shape check —
that decides whether the documents live in `docs/` or `docs/apps/<app>/`, and whether task
numbers carry an app prefix.

If the PRD targets two platforms, ask whether to break down one of them or both — both means
two sets of UI tasks, not one shared set.

## Documents used

Paths come from the profile's "Where things live" — the ones below are the single-app
layout; a multi-app Android or iOS project uses `docs/apps/<app>/` instead.

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
  error handling, environment/config setup, empty/loading/error states in the UI — plus
  whatever the profile's "Hidden work this platform always drags in" lists, which is the
  part a PRD written by a non-specialist never mentions.
- **Cost warnings** when a feature is far more expensive than it looks (data migration,
  third-party integration, an architectural change).

If the user says "just follow the PRD", get on with it. **Suggestions stay suggestions.**

## Task phases

**The phase order comes from the profile's "Task phases"** — data-first on every platform,
but Android adds DI and module setup, iOS adds target/package setup, and the two web
profiles add a responsive/accessibility pass. Skip phases that don't apply; don't force
content into every phase. The last phase is always verification against the PRD's success
criteria, done by hand — this pipeline writes no automated tests.

## Output — docs/TASKS.md

```markdown
# Task List
Last updated: <date> | Platform: <platform>

## Phase 1 — Data
- [ ] T01 — <task title>
  Files: <the files this will likely touch>
  Done when: <a concrete, verifiable criterion + the profile's verify command>
```

When the PRD targets more than one platform, add a `Platform: <platform>` line to every task
whose work differs per platform (anything from the UI phase onward), and prefix its number
so the two sets don't collide — `WEB-T04`, `IOS-T04`. Data and domain tasks that are
genuinely shared stay unprefixed.

## After it's written

1. Summarize: task count per phase, which areas of the code get touched.
2. Ask whether anything needs changing.
3. Update `docs/progress-report.md`:
   `- [<date>] Breakdown <feature name> (<platform>): <N> tasks (T## through T##)`
4. Suggest `/coding`.
