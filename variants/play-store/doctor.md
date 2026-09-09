---
description: Check that every tool the Play Store commands need is installed (read-only)
allowed-tools: Read, Glob, Grep, Bash(echo*), Bash(which*), Bash(where*), Bash(ls*), Bash(test*), Bash(uname*), Bash(date*), Bash(java*), Bash(adb*), Bash(emulator*), Bash(sdkmanager*), Bash(avdmanager*), Bash(node*), Bash(npm*), Bash(npx*), Bash(fastlane*), Bash(magick*), Bash(convert*), Bash(cat*), Bash(find*)
---

# Doctor — dependency check for the Play Store commands

Run this before `/play-store:capture`, `/play-store:enhance`, `/play-store:listing`, or
`/play-store:aso`. It verifies that every tool those commands rely on exists on this
machine and in this project, and prints exact install commands for whatever is missing.

## Arguments

Read `$ARGUMENTS`:

| Argument | Meaning |
|---|---|
| empty | Full check: system tools + project + devices. |
| `--system` | Only the machine-level tools (skip the project and device sections). |
| `--project` | Only the project-level checks (Gradle wrapper, app modules, Playwright in `node_modules`). |
| `--fix` | After the report, offer to run the **safe, project-local** fixes (see "Fixes"). Never runs system installers. |

## Absolute rules

- **Read-only by default.** Without `--fix` this command changes nothing: no installs, no
  file writes, no env changes.
- **Never guess a version.** Run the tool and read its output. If a tool can't be run,
  report it as missing, not as "probably installed".
- **Never run a system installer** (winget, brew, apt, choco, `sdkmanager --install`). Print
  the command for the user instead. The only thing `--fix` may run is listed under "Fixes".
- One check that fails must not stop the others. Run every check, then report once.
- Detect the OS first (`uname -s`; on Windows the Bash tool is Git Bash, so `uname` prints
  `MINGW*` or `MSYS*`) and pick the install commands for that OS.
- Run `date +%Y-%m-%d` for the date in the report header. Never guess it.

## Step 1 — Detect the environment

1. OS: `uname -s`. Map to `windows` / `macos` / `linux`.
2. The Android SDK location. Check in this order and record which one hit:
   - `$ANDROID_HOME`
   - `$ANDROID_SDK_ROOT`
   - the OS default: Windows `$LOCALAPPDATA/Android/Sdk`, macOS `~/Library/Android/sdk`,
     Linux `~/Android/Sdk`
   - `local.properties` in the project root (`sdk.dir=...`)
3. Whether the current directory is an Android project: `settings.gradle` or
   `settings.gradle.kts` exists.

## Step 2 — System tools

Run each check. Record `OK` / `MISSING` / `WARN` plus the version string or the reason.

### Required

| # | Tool | How to check | Pass condition |
|---|---|---|---|
| S1 | Java (JDK) | `java -version` | runs; version 17 or newer (Android Gradle Plugin 8+ needs 17) |
| S2 | Android SDK root | the path found in Step 1 exists and contains `platform-tools/` | folder exists |
| S3 | ADB | `adb --version`, else `<sdk>/platform-tools/adb --version` | runs. WARN if it only works via the full path (not on PATH). |
| S4 | Emulator binary | `emulator -version`, else `<sdk>/emulator/emulator -version` | runs. WARN if only via full path. |
| S5 | At least one AVD | `emulator -list-avds` (or full path) | prints at least one name. WARN (not MISSING) if a physical device is connected instead, see D1. |
| S6 | Node.js | `node --version` | runs; version 18 or newer |
| S7 | npm | `npm --version` | runs |

### Optional (only affects specific commands)

| # | Tool | How to check | Needed by |
|---|---|---|---|
| O1 | `sdkmanager` | `sdkmanager --version`, else `<sdk>/cmdline-tools/latest/bin/sdkmanager --version` | creating AVDs from the CLI. Otherwise the user creates them in Android Studio. |
| O2 | fastlane | `fastlane --version` | `/play-store:listing --upload` only |
| O3 | ImageMagick | `magick -version` (Windows/macOS) or `convert -version` (Linux) | fallback resizer when Playwright is unavailable |

## Step 3 — Project checks (skip with `--system`)

Only if Step 1 found a `settings.gradle(.kts)`.

| # | Check | How | Pass condition |
|---|---|---|---|
| P1 | Gradle wrapper | `gradlew` and `gradlew.bat` exist; `gradle/wrapper/gradle-wrapper.properties` exists | present |
| P2 | App modules | Grep `settings.gradle(.kts)` for `include`, then for each included path check whether its `build.gradle(.kts)` applies `com.android.application` (the id string, `alias(libs.plugins.android.application)`, or an `android-application` convention plugin) | at least one app module found. List them: Gradle path + `applicationId` read from the build file. |
| P3 | Playwright installed in the project | `node_modules/playwright/package.json` or `node_modules/playwright-core/package.json` exists (check `package.json` `devDependencies` too) | present. MISSING is normal on first run; `--fix` handles it. |
| P4 | Playwright Chromium browser | look for a `chromium-*` folder under the Playwright browsers cache: Windows `$LOCALAPPDATA/ms-playwright`, macOS `~/Library/Caches/ms-playwright`, Linux `~/.cache/ms-playwright`. If `PLAYWRIGHT_BROWSERS_PATH` is set, look there instead. | at least one `chromium-*` folder |
| P5 | Screens manifest | `docs/store/<app>/screens.yml` for each app module from P2 | WARN if missing (it's created by `/play-store:capture` on first run, so this is informational) |

## Step 4 — Devices (skip with `--system` or `--project`)

| # | Check | How | Result |
|---|---|---|---|
| D1 | Connected devices | `adb devices -l` | every line in `device` state is OK; `unauthorized` -> WARN "accept the USB debugging prompt on the phone"; `offline` -> WARN. Zero devices AND zero AVDs (S5) -> MISSING. Zero devices but AVDs exist -> OK with the note "no emulator running; `/play-store:capture` will start one". |
| D2 | Screen size of each connected device | `adb -s <serial> shell wm size` | informational: print `Physical size`. Note which ones are 9:16 vs taller (1080x2400 = 9:20). Nothing fails here; `/play-store:enhance` fits any ratio into the 1080x1920 canvas. |

## Step 5 — Report

Print one table, grouped, in this exact shape (statuses: `✅ OK`, `⚠️ WARN`, `❌ MISSING`,
`— skipped`):

```
Play Store doctor — <OS>, <date>

System
  ✅ S1  Java            17.0.11 (Temurin)
  ✅ S2  Android SDK     C:/Users/me/AppData/Local/Android/Sdk  (via ANDROID_HOME)
  ⚠️ S3  ADB             35.0.2 — works only via full path, not on PATH
  ...
Optional
  ❌ O2  fastlane        not found — only needed for --upload
Project  (<project name>)
  ✅ P1  Gradle wrapper  gradle-8.7
  ✅ P2  App modules     :tokoku (com.heri.tokoku), :warungku (com.heri.warungku)
  ❌ P3  Playwright      not in node_modules
  ❌ P4  Chromium        no chromium-* under <path>
Devices
  ✅ D1  emulator-5554   Pixel_7_API_34, 1080x2400 (9:20)

Result: 2 required missing, 1 warning.
Ready for:  /play-store:listing  /play-store:aso  /play-store:capture
Blocked:    /play-store:enhance (needs P3, P4)
```

The "Ready for / Blocked" lines use this mapping. A WARN never blocks; only MISSING does.

| Command | Requires |
|---|---|
| `/play-store:capture` | S1, S2, S3, and (S5 or D1) |
| `/play-store:enhance` | S6, S7, P3, P4 |
| `/play-store:listing` | nothing on the machine (P2 to know the apps); O2 only for `--upload` |
| `/play-store:aso` | nothing on the machine |

## Step 6 — Fix instructions

For every `❌` and `⚠️`, print the fix for the detected OS. Use exactly these:

**Java 17+**
- Windows: `winget install EclipseAdoptium.Temurin.17.JDK`
- macOS: `brew install --cask temurin@17`
- Linux (Debian/Ubuntu): `sudo apt install openjdk-17-jdk`
- Or use the JDK bundled with Android Studio: set `JAVA_HOME` to `<Android Studio>/jbr`.

**Android SDK / ADB / emulator not found**
- Install Android Studio (https://developer.android.com/studio), then in SDK Manager
  install *Android SDK Platform-Tools* and *Android Emulator*.
- Set the env var. Windows (PowerShell, persistent):
  `[Environment]::SetEnvironmentVariable('ANDROID_HOME', "$env:LOCALAPPDATA\Android\Sdk", 'User')`
  macOS/Linux: add `export ANDROID_HOME=~/Library/Android/sdk` (or `~/Android/Sdk`) to the
  shell profile.

**ADB / emulator only via full path (WARN)**
- Add `<sdk>/platform-tools` and `<sdk>/emulator` to PATH. Windows (PowerShell):
  `[Environment]::SetEnvironmentVariable('Path', $env:Path + ";$env:LOCALAPPDATA\Android\Sdk\platform-tools;$env:LOCALAPPDATA\Android\Sdk\emulator", 'User')`
- Not blocking: the other `/play-store` commands fall back to the full path automatically.

**No AVD**
- Android Studio -> Device Manager -> Create device. Recommended profile for Play Store
  shots: *Pixel 7* (1080x2400) or *Pixel 2* (1080x1920, exact 9:16), API 34, Google APIs
  image. For tablet shots add *Pixel Tablet* (2560x1600).
- CLI alternative (needs O1):
  `sdkmanager "system-images;android-34;google_apis;x86_64"` then
  `avdmanager create avd -n Pixel_7_API_34 -d pixel_7 -k "system-images;android-34;google_apis;x86_64"`

**Node.js 18+**
- Windows: `winget install OpenJS.NodeJS.LTS`
- macOS: `brew install node`
- Linux: https://nodejs.org/en/download/package-manager (or `nvm install --lts`)

**Playwright + Chromium (P3, P4)** — project-local; this is what `--fix` runs:
```
npm init -y            # only if package.json is missing
npm install -D playwright
npx playwright install chromium
```

**fastlane (optional)**
- macOS/Linux: `brew install fastlane` or `gem install fastlane`
- Windows: `gem install fastlane` (needs Ruby: `winget install RubyInstallerTeam.Ruby.3.2`)

**ImageMagick (optional)**
- Windows: `winget install ImageMagick.ImageMagick`
- macOS: `brew install imagemagick`
- Linux: `sudo apt install imagemagick`

## Fixes (`--fix` only)

The only fixes this command may run itself, each one after showing the exact command and
asking for confirmation:

1. P3 missing -> `npm init -y` (only if no `package.json`) then `npm install -D playwright`.
2. P4 missing -> `npx playwright install chromium`.

Both are project-local and reversible (delete `node_modules/` and the browsers cache). If the
project has no `package.json` and the user doesn't want one at the Android project root,
suggest putting it in `docs/store/` instead and note that path for `/play-store:enhance`.

After running a fix, re-run the corresponding check and show the updated row. Everything else
(Java, SDK, Node, fastlane) stays as printed instructions, never executed.

## Re-runs

Safe to run as often as needed. It has no state and writes nothing without `--fix`.
