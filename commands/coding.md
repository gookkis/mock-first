---
description: Work the next task from TASKS.md — implement, review, commit
allowed-tools: all
---

# Coding Iteration

## Platform (REQUIRED — before anything else)

Where new files go, how the code is verified and what the commit area is called are all
platform-specific.

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
Use its "Where things live" for paths, its "Verify" for the build command and the commit
area, and its "States this platform always needs" as the checklist before you call a UI task
done. Run its project-shape check first — on a multi-app project the documents live under
`docs/apps/<app>/` and task numbers carry an app prefix.

## Ground rules

- For dates, ALWAYS run `date +%Y-%m-%d` first. Never guess the date.
- This phase is pure execution. **No brainstorming.** If an idea outside the task's scope
  comes up, note it in one line — don't act on it yourself.
- If the user asks for a change that touches the PRD, a mockup, or the task list (not just a
  small fix inside this task), DON'T do it here. Send them to `/revise` so the documents
  stay in sync.

## Checks before starting

1. Read `docs/TASKS.md` (or `docs/apps/<app>/TASKS.md` on a multi-app project). If it
   doesn't exist, point the user to `/break-task` and stop.
2. Run `git status`:
   - Uncommitted changes -> list them and ask: commit first, stash, or carry on? **Don't
     start before the user answers.**
   - Not a git repo -> tell the user, and ask whether to `git init` first or continue
     without commits.
3. Read the PRD and the relevant mockups for this task, at the paths the profile gives. When
   the PRD targets more than one platform, read the mockup for **this** platform —
   `<screen>-ios-approved.html`, not whichever file matches the screen name first.

## Each iteration

1. Take the **topmost unchecked task**.
   - `$ARGUMENTS` contains a task code (e.g. `T03`) -> work that one.
   - A task marked `[needs decision]` -> ask the user before starting.
   - Everything is checked -> jump to "When every task is done".
2. Name the task you're about to do plus a short plan (3 lines max). Then go — no approval
   needed for this part.
3. Implement:
   - Follow the project's existing conventions (naming, folder structure, style), and the
     profile's layout for anything the repo has no precedent for yet.
   - **Don't touch files outside this task's scope.** If it turns out you must, stop and ask.
   - For UI, follow the approved mockup — don't improvise your own layout.
   - **Don't write tests.** This pipeline doesn't use automated tests — verify through the
     build plus a manual check against the task's success criterion. If the user does want
     tests, that's its own task via `/revise`.
4. Build the code you touched, with the profile's verify command — the narrow one
   (`./gradlew :<module>:assembleDebug`, `xcodebuild -scheme <scheme> build`, the project's
   own `npm run build`), never the whole project when a module will do. On a multi-app
   Android or iOS project, a change inside a shared module also has to build every app that
   depends on it. Fix failures. **If it fails 3 times in a row for the same reason, stop and
   report** — don't keep trying and risk breaking something else.
5. Show a summary of the changes: files touched + what changed (not a full diff unless
   asked).
6. Ask the user to confirm before committing.

## Commit (after the user agrees)

1. **Stage only the files you touched for this task**, named one by one:
   `git add <file1> <file2>`. **NEVER use `git add .` or `git add -A`** — unrelated changes
   (env files, logs, build artifacts, the user's own work in progress) must not ride along.
2. Run `git status` once more to confirm what's staged is right.
3. Commit: `<type>(<area>): <short task title> (<T##>)`, e.g.
   `feat(checkout): add coupon validation (T03)`. The profile says what `<area>` is on this
   platform — the Gradle module on Android, the target or package on iOS, the route or
   feature on web.
4. Check the task off in the task file.
5. Update `docs/progress-report.md` with ONE line:
   `- [<date>] T## <title> — done (<platform>, area: <area>)`. No long transcripts.
6. Ask: move on to the next task, or stop here?

## When every task is done

- Report the feature/MVP as complete and show the summary from the progress report.
- Mention any notes or ideas that came up during coding but fell outside scope.
- Suggest what's next: a full manual pass, the profile's "Shipping it" step (on Android that
  is the `/play-store:*` and `/portfolio:*` chain), or `/prd` for the next feature.
