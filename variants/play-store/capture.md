---
description: Build, install, and capture raw screenshots of one app through ADB, screen by screen
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(date*), Bash(mkdir*), Bash(ls*), Bash(cat*), Bash(uname*), Bash(echo*), Bash(test*), Bash(adb*), Bash(emulator*), Bash(./gradlew*), Bash(gradlew*), Bash(sleep*), Bash(node*)
---

# Capture — raw screenshots of one app

Drives an emulator or a USB device through ADB, opens every screen listed in
`docs/store/<app>/screens.yml`, and saves an untouched capture of each one. Nothing here is
resized or decorated; that's `/play-store:enhance`'s job.

Run `/play-store:doctor` first if this is the first time on this machine.

## Arguments

Read `$ARGUMENTS`:

| Argument | Meaning |
|---|---|
| `<app>` | Required. Gradle path (`:tokoku`) or app name (`tokoku`). Must be an `com.android.application` module. |
| `--device <serial or AVD name>` | Which device to use. Default: the `device:` in `screens.yml`, else the first running device, else the first AVD. |
| `--locale <xx>` | Capture in this language (`id`, `en`, ...). Default: every locale in `screens.yml`, or `en` if none. |
| `--screen <id>` | Capture only this screen id. Repeatable. |
| `--tablet` | Use the tablet profile (`tablet_device:` in `screens.yml`) and write into `raw-tablet/`. |
| `--no-build` | Skip the Gradle build/install (the app is already on the device). |
| `--variant <name>` | Gradle variant to install. Default `debug` -> task `install<Variant>` (e.g. `installDebug`). |
| `--keep-demo` | Leave the status bar in demo mode after finishing (default: restore it). |

## Absolute rules

- **Never build without saying so.** Print the exact Gradle task before running it; a build
  on a multi-app project can take minutes.
- **Never write anything into the app's source tree.** Everything lives under `docs/store/`.
- **Never fake a capture.** If a screen can't be opened or a tap target isn't found, save
  nothing for that screen, say why, and move on to the next one.
- **Look at every capture** with the Read tool after saving it. A black, white, or
  half-rendered frame is a failure, not a result.
- Run `date +%Y-%m-%d` for the date. Never guess it.
- ADB may not be on PATH. Resolve it once at the start: `adb` if it runs, else
  `<sdk>/platform-tools/adb` where `<sdk>` is `$ANDROID_HOME`, `$ANDROID_SDK_ROOT`,
  `local.properties`'s `sdk.dir`, or the OS default. Same for `emulator`. Use that resolved
  path for every later call.

## Step 1 — Resolve the app

1. Read `settings.gradle(.kts)`; find the include whose last segment matches `<app>`.
2. Read that module's `build.gradle(.kts)`: confirm it applies the Android application
   plugin, read `applicationId` (and `applicationIdSuffix` for the chosen variant, if any).
3. Read `src/main/AndroidManifest.xml` of that module: the launcher activity (the one with
   `MAIN` + `LAUNCHER`), and every `<data android:scheme=... android:host=...>` deep link.
4. If `docs/apps/<app>/PRD.md` exists (android-factory variant), read its "Screens" section
   for the list of screens the product is supposed to have.

If `<app>` doesn't match exactly one app module, list the app modules and stop.

## Step 2 — The screens manifest

Path: `docs/store/<app>/screens.yml`.

**If it exists**, read it and go to Step 3.

**If it's missing**, build a draft from what Step 1 found and from the code:
- Navigation: `navigation/*.xml` graphs, Compose `NavHost`/`composable("route")` calls,
  activities in the manifest.
- Deep links from the manifest are the preferred way to open a screen: they're stable and
  need no tapping.
- Pick 4 to 8 screens: the ones a first-time visitor to the store page must see (main value
  first, then the flow, then settings-type screens last). Don't list every screen.

Show the draft, ask the user to correct it, then write it. Format:

```yaml
# docs/store/<app>/screens.yml — read by /play-store:capture and /play-store:enhance
app: ":tokoku"
applicationId: com.heri.tokoku
device: Pixel_7_API_34          # AVD name or serial; --device overrides
tablet_device: Pixel_Tablet     # used with --tablet
locales: [id, en]
status_bar: demo                # demo = clean clock/battery/signal, off = leave as is
before_all:                     # optional, runs once after install (e.g. skip onboarding)
  - open: launcher
  - wait: 3s
  - tap: "Lewati"
screens:
  - id: home                    # file name: 01-home.png
    title: Beranda              # human label, reused as a caption fallback by /enhance
    open: launcher              # launcher | deeplink <uri> | activity <.Class or full name>
    wait: 2s
  - id: catalog
    title: Katalog produk
    open: deeplink tokoku://catalog
    wait: 2s
    steps:
      - tap: "Semua"            # text, content-desc, or resource-id (id/foo); or "x,y"
      - wait: 1s
  - id: checkout
    title: Checkout
    open: activity .ui.checkout.CheckoutActivity
    steps:
      - type: "user@example.com"   # types into the focused field
      - key: back                  # back | home | enter | tab
      - swipe: up                  # up | down | left | right
      - wait: 1s
    note: needs a logged-in demo account
```

Screen order in the file is the order on the store page. The `id` becomes the file name,
so keep it short and stable; `/enhance` matches captions by `id`.

## Step 3 — Device

1. `adb devices -l`. Pick the device per the rules in Arguments. If the chosen name is an
   AVD that isn't running: `emulator -avd <name> -no-snapshot-load -no-boot-anim` in the
   background, then `adb wait-for-device` and poll
   `adb shell getprop sys.boot_completed` until it prints `1` (up to 120 s, `sleep 5`
   between polls). Record the serial (`emulator-5554` etc.) and use `adb -s <serial>` from
   here on.
2. Screen size: `adb -s <serial> shell wm size`. Print it. Any ratio is fine.
3. Locale, when `--locale` or `locales:` is set: try
   `adb shell settings put system system_locales <xx>-<XX>` (`id-ID`, `en-US`). It applies
   without root on most emulator images. Verify with `adb shell getprop persist.sys.locale`
   or by reopening the app and checking a known string. If it didn't take effect, say so and
   ask the user to switch the language in Settings before continuing; don't capture the
   wrong language silently.
4. Status bar, when `status_bar: demo` (default):
   ```
   adb shell settings put global sysui_demo_allowed 1
   adb shell am broadcast -a com.android.systemui.demo -e command enter
   adb shell am broadcast -a com.android.systemui.demo -e command clock -e hhmm 1200
   adb shell am broadcast -a com.android.systemui.demo -e command battery -e level 100 -e plugged false
   adb shell am broadcast -a com.android.systemui.demo -e command network -e wifi show -e level 4
   adb shell am broadcast -a com.android.systemui.demo -e command network -e mobile show -e datatype none -e level 4
   adb shell am broadcast -a com.android.systemui.demo -e command notifications -e visible false
   ```
   At the end (unless `--keep-demo`): `... -e command exit`.

## Step 4 — Build and install (skip with `--no-build`)

Print, then run: `./gradlew :<app>:install<Variant>` (`gradlew.bat` on Windows). If it fails,
show the last 30 lines and stop; don't try to "fix" the build from here.

Then `before_all` steps, if any (Step 5 explains each step type).

## Step 5 — Capture each screen

Output folder: `docs/store/<app>/raw/<locale>/` (or `raw-tablet/<locale>/` with `--tablet`).
File name: `<NN>-<id>.png`, `NN` = two-digit position in `screens:`.

For each screen (or only the `--screen` ids):

1. **Open**
   - `launcher`: `adb shell monkey -p <applicationId> -c android.intent.category.LAUNCHER 1`
   - `deeplink <uri>`: `adb shell am start -W -a android.intent.action.VIEW -d "<uri>" <applicationId>`
   - `activity <name>`: `adb shell am start -W -n <applicationId>/<name>` (a leading `.` is
     relative to the manifest `package`, which may differ from `applicationId`; use the
     manifest package for the class part).
2. **Wait** the `wait:` value (default `2s`) with `sleep`.
3. **Steps**, in order:
   - `tap: "<text>"`: dump the UI (`adb shell uiautomator dump /sdcard/ui.xml` then
     `adb pull /sdcard/ui.xml <scratch>`), find the first node whose `text`,
     `content-desc`, or `resource-id` (suffix after `id/`) equals the value, case-sensitive
     first, then case-insensitive. Compute the center of its `bounds="[x1,y1][x2,y2]"` and
     `adb shell input tap <cx> <cy>`. `tap: "540,1200"` taps coordinates directly. If no
     node matches, print the texts that were on screen and skip this screen.
   - `type: "<text>"`: `adb shell input text '<text>'` with spaces written as `%s`.
   - `key: back|home|enter|tab`: `input keyevent 4|3|66|61`.
   - `swipe: up|down|left|right`: a 300 ms swipe across the middle 60 % of the screen.
   - `wait: <n>s`.
   After every step wait 500 ms before the next.
4. **Capture**: `adb -s <serial> exec-out screencap -p > <file>`. On Windows Git Bash this
   redirect is binary-safe; don't go through `adb shell screencap` + `pull` unless
   `exec-out` fails.
5. **Verify**: the file is larger than 10 KB, and open it with the Read tool. Check that the
   app is actually showing (not the launcher, not a crash dialog, not a permission prompt,
   not mid-animation). If it's wrong, retry once with a 2 s extra wait; if still wrong,
   delete the file and record the failure.

Between screens, if the next screen's `open:` is `launcher` or the same activity, press
`back` enough times or `am force-stop <applicationId>` first so the app starts fresh.

## Step 6 — Wrap up

1. Exit demo mode (Step 3.4) unless `--keep-demo`.
2. Write `docs/store/<app>/raw/<locale>/index.json`:
   ```json
   { "app": ":tokoku", "locale": "id", "device": "Pixel_7_API_34", "size": "1080x2400",
     "date": "<date>", "screens": [ { "id": "home", "file": "01-home.png", "title": "Beranda" } ] }
   ```
   `/enhance` reads this, so include only captures that passed verification.
3. Report a table: screen id, file, size, status (`captured` / `skipped: <reason>`).
4. Next step: `/play-store:enhance <app>`.

## Re-runs

Re-running overwrites the files for the screens it captures and leaves the others. Use
`--screen <id>` to redo a single one after fixing its steps in `screens.yml`.
