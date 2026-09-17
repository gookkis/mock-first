---
description: Build a plain HTML mockup from the PRD for visual review
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(date*), Bash(mkdir*), Bash(mv*), Bash(cp*)
---

# Building a Mockup

## Platform (REQUIRED — before anything else)

A web screen, a phone-browser screen, an Android screen, an iOS screen and each of the three
desktop OSes are different drawings.

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

Then read the profile for that platform:

```
${CLAUDE_PLUGIN_ROOT}/platforms/<platform>.md
```

(or `~/.claude/platforms/<platform>.md` if the commands were copied in without the plugin).
It owns the canvas, the design language, the component names, the states and the output
path — where it differs from the generic guidance below, **the profile wins**.

If you had to ask at step 4, add a single `Platform: <name>` line to the PRD's "Technical
Stack" so the next command doesn't ask again. Nothing else in the PRD changes.

## Scope (REQUIRED)

Read the rest of `$ARGUMENTS` as the screen name. If empty, read the "Screens Needed"
section of `docs/PRD.md` and ask which screen to work on.

## Ground rules

- For dates, ALWAYS run `date +%Y-%m-%d` first. Never guess the date.
- One `/mockup` run = one screen, on one platform. Don't try to do every screen at once,
  and don't mock the same screen for two platforms in a single run.
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
  loading, error, onboarding), **mention it** — but don't build it until asked. The
  profile's "states this platform always needs" is the checklist to run it against.
- If something in the PRD is genuinely ambiguous (not just a matter of taste), ask now —
  one question, not a queue of them.

## Steps

1. Read `docs/PRD.md`. If it doesn't exist, point the user to `/prd` and stop.
2. Work out the output folder from the profile's "Where things live" — on Android and iOS
   that depends on whether the project is a single app or a factory of several, so run the
   profile's project-shape check before you write anything.
3. Check that folder to see which screens already have an `-approved.html`.
4. Build the mockup directly. Don't ask about visual direction unless the PRD is ambiguous.

## Building it

- Write **one file**, at the path and under the naming rule the profile gives (plain
  `<screen-name>.html` for a single-platform project, `<screen-name>-<platform>.html` once
  the PRD targets more than one).
- Mockup file rules:
  - Self-contained: HTML + inline CSS, no external dependencies, no build step.
  - Built at the profile's canvas size — not a generic full-width page.
  - Realistic dummy data (not "Lorem ipsum").
  - Focus on layout, hierarchy, and flow — not animation or polish.
  - Name the component the code will use, in HTML comments, the way the profile shows.
- Explain in 2-3 lines: the platform you're building for, the main layout decision, and
  what you prioritized.
- Ask the user to open the file in a browser and review it.

## Iterate until it fits

1. If something is off, **edit the same file** — change only what the user named, don't
   rewrite the whole file (token waste).
2. Repeat until the user says it's right.

## Once the user approves

1. Rename the file to `<screen-name>-approved.html` in the same folder
   (`mv <folder>/<screen-name>.html <folder>/<screen-name>-approved.html`).
2. Update the "Screens Needed" section of the PRD: mark this screen as having an approved
   mockup, and name the platform when the PRD targets more than one.
3. Update `docs/progress-report.md`: `- [<date>] Mockup <screen-name> (<platform>) approved`
4. Check which screens are still without a mockup:
   - Some left -> suggest `/mockup <next screen>`.
   - None left -> suggest `/break-task`.
