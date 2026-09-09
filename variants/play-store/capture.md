---
description: Build, install, and capture raw screenshots of one app through ADB, screen by screen (Views or Compose)
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(date*), Bash(mkdir*), Bash(ls*), Bash(cat*), Bash(uname*), Bash(echo*), Bash(test*), Bash(find*), Bash(adb*), Bash(emulator*), Bash(./gradlew*), Bash(gradlew*), Bash(sleep*), Bash(node*)
---

# Capture — raw screenshots of one app

Drives an emulator or a USB device through ADB, opens every screen listed in
`docs/store/<app>/screens.yml`, and saves an untouched capture of each one. Nothing here is
resized or decorated; that's `/play-store:enhance`'s job.

Run `/play-store:doctor` first if this is the first time on this machine.

**Compose apps work, with one difference.** A single-Activity Compose app has no Activity
per screen, so `am start -n <pkg>/<Activity>` reaches only the entry point. Two routes
still lead to an inner screen: a **declared deep link** (`navDeepLink` on the composable
plus a matching `<intent-filter>` in the manifest), which is by far the most reliable, or
**driving the UI** with taps. Step 2 detects which of the two this app supports before
anything is captured, and the tap resolver in Step 6 is written for Compose's accessibility
tree rather than for Android Views.

## Arguments

Read `$ARGUMENTS`:

| Argument | Meaning |
|---|---|
| `<app>` | Required. Gradle path (`:tokoku`) or app name (`tokoku`). Must be a `com.android.application` module. |
| `--device <serial or AVD name>` | Which device to use. Default: the `device:` in `screens.yml`, else the first running device, else the first AVD. |
| `--locale <xx>` | Capture in this language (`id`, `en`, ...). Default: every locale in `screens.yml`, or `en` if none. |
| `--screen <id>` | Capture only this screen id. Repeatable. |
| `--tablet` | Use the tablet profile (`tablet_device:` in `screens.yml`) and write into `raw-tablet/`. |
| `--no-build` | Skip the Gradle build/install (the app is already on the device). |
| `--variant <name>` | Gradle variant to install. Default `debug` -> task `install<Variant>`. |
| `--keep-demo` | Leave the status bar in demo mode after finishing (default: restore it). |
| `--from-previews` | Render `@Preview` composables instead of driving a device. Only when the project already has Compose preview screenshot testing configured; see the last section. |
| `--debug-ui` | On a failed tap, also save the accessibility dump and a screenshot next to the capture folder, for fixing `screens.yml`. |

## Absolute rules

- **Never build without saying so.** Print the exact Gradle task before running it; a build
  on a multi-app project can take minutes.
- **Never write anything into the app's source tree.** Everything lives under `docs/store/`.
  This includes not adding a Gradle plugin, a test source set, or a `testTag` to make
  capturing easier: report what would help and let the user decide.
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
3. Read `src/main/AndroidManifest.xml`: the launcher activity (`MAIN` + `LAUNCHER`), every
   other exported activity, and every `<intent-filter>` carrying a `<data>` scheme or host.
4. If `docs/apps/<app>/PRD.md` exists (android-factory variant), read its "Screens" section.

If `<app>` doesn't match exactly one app module, list the app modules and stop.

## Step 2 — Detect the UI toolkit and how screens can be reached

Run these checks across the app module **and every module it depends on**; a Compose app
usually keeps its screens in feature modules.

| # | Check | How |
|---|---|---|
| T1 | Compose in use | `buildFeatures { compose = true }`, a `androidx.compose` dependency, or any `@Composable` in the sources |
| T2 | Views in use | any `setContentView`, layout XML under `res/layout/`, or a Fragment subclass |
| T3 | Activities per screen | activities in the manifest beyond the launcher one |
| T4 | Compose Navigation | `NavHost`, `composable("route"`, or `navigation-compose` in the deps |
| T5 | Declared deep links | `navDeepLink` / `deepLinks = listOf(` in the nav graph **and** a matching `<data>` element in the manifest. Both halves are required: without the manifest side, `am start` cannot hand the URI to the app. |
| T6 | testTags visible to ADB | `testTagsAsResourceId` anywhere in the sources |
| T7 | Preview screenshot testing | `com.android.compose.screenshot` plugin applied and `android.experimental.enableScreenshotTest=true` in `gradle.properties` |

Report the result plainly, then pick the navigation mode:

- **T5 true** -> `deeplink`. Best case. List the URIs you found; each one opens a screen
  directly with no tapping and no ordering constraints.
- **T5 false, T3 true** -> `activity` for the screens that have their own Activity,
  `ui` for the rest.
- **T5 false, T3 false** (single-Activity Compose, no deep links) -> `ui`. Every inner
  screen is reached by tapping from the launcher. Say this explicitly, because it makes
  capture slower and more fragile, and add one line to the report: declaring
  `navDeepLink` on the three or four screens worth capturing would make this reliable.
- **T6 false** and mode is `ui` -> warn that `resource-id` will be empty in the
  accessibility tree, so `screens.yml` must select nodes by visible text or content
  description. Mention that a one-line `Modifier.semantics { testTagsAsResourceId = true }`
  on the app's root would expose `testTag`s to ADB, and that this is the user's call.

## Step 3 — The screens manifest

Path: `docs/store/<app>/screens.yml`.

**If it exists**, read it, reconcile it against Step 2 (an `open: activity` for an app with
no such activity is an error worth reporting), and go to Step 4.

**If it's missing**, draft it from Step 1, Step 2, and the code: nav graph routes,
`composable("route")` calls, screen-level composable names (`fun HomeScreen(`), or the
activity list. Pick 4 to 8 screens, main value first, settings last. For mode `ui`, order
matters: each screen is reached from the one before it or from the launcher, so write the
sequence the way a person would walk it.

Show the draft, ask the user to correct it, then write it.

```yaml
# docs/store/<app>/screens.yml — read by /play-store:capture and /play-store:enhance
app: ":tokoku"
applicationId: com.heri.tokoku
ui_toolkit: compose             # compose | views | mixed   (from Step 2)
navigation: ui                  # deeplink | activity | ui  (default for screens below)
test_tags_as_resource_id: false # from Step 2; when false, never select by id/
device: Pixel_7_API_34          # AVD name or serial; --device overrides
tablet_device: Pixel_Tablet
locales: [id, en]
status_bar: demo                # demo = clean clock/battery/signal, off = leave as is
disable_animations: true        # strongly recommended for Compose; see Step 4
before_all:                     # runs once after install, e.g. to clear onboarding
  - waitFor: "Selamat datang"
  - tap: "Lewati"
  - waitFor: "Beranda"
screens:
  - id: home
    title: Beranda
    open: launcher              # launcher | deeplink <uri> | activity <.Class> | ui
    waitFor: "Pesanan hari ini" # prefer this over a fixed wait
  - id: catalog
    title: Katalog produk
    open: deeplink tokoku://catalog
    waitFor: "Semua produk"
  - id: checkout
    title: Checkout
    open: ui                    # reached by walking the UI
    from: catalog               # start from that screen's end state; omit to start at launcher
    steps:
      - tap: "Keranjang"
      - waitFor: "Ringkasan pesanan"
      - scrollTo: "Bayar sekarang"
      - tap: "Bayar sekarang"
      - waitFor: "Metode pembayaran"
    note: needs the demo account seeded by before_all
```

Screen order is the store-page order. The `id` becomes the file name, so keep it short and
stable; `/enhance` and `/portfolio:publish` match captions and labels by `id`.

### Step types

| Step | Meaning |
|---|---|
| `waitFor: "<text>"` | Poll the accessibility tree until a node matching the text appears, up to 15 s. **Use this instead of `wait:` wherever possible.** Compose has no idle broadcast, so a fixed sleep is a guess. |
| `wait: <n>s` | Fixed sleep. For animations that finish on their own and expose no new text. |
| `tap: "<selector>"` | Tap a node (resolution in Step 6). |
| `tapAt: "<x%>,<y%>"` | Tap at a percentage of the screen, e.g. `50%,90%`. Resolution-independent. The fallback when nothing in the tree is selectable, such as a canvas or a chart. |
| `scrollTo: "<text>"` | Swipe up in the largest scrollable area until the text appears, up to 8 swipes. Needed for lazy lists, whose off-screen items do not exist in the tree at all. |
| `type: "<text>"` | Type into the focused field. |
| `key: back\|home\|enter\|tab` | Key event. |
| `swipe: up\|down\|left\|right` | One swipe across the middle 60 % of the screen. |

A `tap` selector is matched, in order: exact visible text, exact content description,
`id/<testTag>` **only when `test_tags_as_resource_id` is true**, then a case-insensitive
retry of the first two. `tap: "540,1200"` still taps absolute coordinates.

## Step 4 — Device

1. `adb devices -l`. Pick the device per the rules in Arguments. If the chosen name is an
   AVD that isn't running: `emulator -avd <name> -no-snapshot-load -no-boot-anim` in the
   background, then `adb wait-for-device` and poll `adb shell getprop sys.boot_completed`
   until it prints `1` (up to 120 s, `sleep 5` between polls). Record the serial and use
   `adb -s <serial>` from here on.
2. Screen size: `adb -s <serial> shell wm size`. Print it. Any ratio is fine.
3. **Animations off**, when `disable_animations: true`:
   ```
   adb shell settings put global window_animation_scale 0
   adb shell settings put global transition_animation_scale 0
   adb shell settings put global animator_duration_scale 0
   ```
   This matters more for Compose than for Views: a running animation keeps the accessibility
   tree busy, and `uiautomator dump` then fails with "could not get idle state". Restore the
   three to `1` at the end.
4. Locale, when `--locale` or `locales:` is set: try
   `adb shell settings put system system_locales <xx>-<XX>` (`id-ID`, `en-US`). Verify with
   `adb shell getprop persist.sys.locale` or by reopening the app and checking a known
   string. If it didn't take effect, say so and ask the user to switch the language in
   Settings; don't capture the wrong language silently.
5. Status bar, when `status_bar: demo` (default):
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

## Step 5 — Build and install (skip with `--no-build`)

Print, then run: `./gradlew :<app>:install<Variant>` (`gradlew.bat` on Windows). If it
fails, show the last 30 lines and stop; don't try to "fix" the build from here.

Then run `before_all`, if present.

## Step 6 — Reading the screen and tapping

This is the part that differs for Compose. Compose renders into one `AndroidComposeView`,
so the accessibility tree is not a view hierarchy: `class` is `android.view.View` almost
everywhere and is useless for selection, and `resource-id` is empty unless the app opted
into `testTagsAsResourceId`. What is reliable is `text` and `content-desc`.

**Dump the tree**

```
adb -s <serial> shell uiautomator dump --compressed /sdcard/ui.xml
adb -s <serial> pull /sdcard/ui.xml <scratch>/ui.xml
```

`--compressed` drops the nodes that carry no semantics, which makes a Compose dump far
smaller and faster. If the dump fails with "could not get idle state", the screen has a
continuous animation (a progress indicator, an infinite transition, a video). Retry once
after 2 s; if it fails again, fall back to `tapAt` for this step and note it in the report.

**Resolve a selector to a tap point**

1. Find the first node whose `text` equals the selector, then whose `content-desc` equals
   it, then, only when `test_tags_as_resource_id` is true, whose `resource-id` ends in
   `/<selector>`. Retry the first two case-insensitively before giving up.
2. **Walk up to the clickable node.** In Compose, the node holding the text is often not
   the node handling the click; a card with three `Text`s exposes them separately while
   only an ancestor is `clickable="true"`. From the matched node, walk up to the nearest
   ancestor with `clickable="true"`, and tap that. If neither the node nor any ancestor is
   clickable, tap the matched node's own centre and note it.
3. Tap the centre of `bounds="[x1,y1][x2,y2]"`: `adb shell input tap <cx> <cy>`.
4. **Confirm the tap did something.** Dump again after 500 ms. If the tree is byte-identical
   to the one before the tap, the tap missed: retry once, then fail the screen. Silently
   capturing the previous screen twice is the failure mode this prevents.

**When nothing matches**

Print every `text` and `content-desc` currently on screen, so the user can fix the
selector, and skip this screen. With `--debug-ui`, also save `ui.xml` and a screenshot to
`docs/store/<app>/raw/_debug/<screen-id>/`. Never guess a different selector and continue.

**Lazy lists**

An item outside the viewport is absent from the tree, not merely invisible. `scrollTo:`
swipes up in the largest node with `scrollable="true"` and re-dumps after each swipe, up to
8 times. If the text never appears, say how many swipes ran and stop the screen.

## Step 7 — Capture each screen

Output folder: `docs/store/<app>/raw/<locale>/` (or `raw-tablet/<locale>/` with `--tablet`).
File name: `<NN>-<id>.png`, `NN` = two-digit position in `screens:`.

For each screen (or only the `--screen` ids):

1. **Reach the start state.**
   - `from: <id>` present: continue from the previous screen's end state, without
     relaunching. This only works if that screen was captured in this run, in order; if it
     wasn't, walk from the launcher instead and say so.
   - otherwise `adb shell am force-stop <applicationId>` first, so the app starts clean.
2. **Open**
   - `launcher`: `adb shell monkey -p <applicationId> -c android.intent.category.LAUNCHER 1`
   - `deeplink <uri>`: `adb shell am start -W -a android.intent.action.VIEW -d "<uri>" <applicationId>`.
     Check the output: `Status: ok` with the app's own activity means it worked; a
     `Warning: Activity not started` or a chooser means the intent filter doesn't match, so
     fail the screen and report the URI rather than capturing whatever is on screen.
   - `activity <name>`: `adb shell am start -W -n <applicationId>/<name>` (a leading `.` is
     relative to the manifest `package`, which may differ from `applicationId`).
   - `ui`: nothing to launch here; the `steps` do the work.
3. **Settle**: run `waitFor:` if present, else `wait:`, else poll the dump until two
   consecutive dumps 500 ms apart are identical, up to 6 s.
4. **Steps**, in order, per Step 6. Wait 500 ms after each.
5. **Capture**: `adb -s <serial> exec-out screencap -p > <file>`. On Windows Git Bash this
   redirect is binary-safe; only fall back to `adb shell screencap` + `pull` if `exec-out`
   fails.
6. **Verify**: the file is larger than 10 KB, and open it with the Read tool. Check that the
   intended screen is showing: not the launcher, not a crash dialog, not a permission
   prompt, not a half-drawn frame, and not the previous screen. If it's wrong, retry once
   with 2 s more settling; if still wrong, delete the file and record the failure.

## Step 8 — Wrap up

1. Restore the animation scales and exit demo mode (Step 4) unless `--keep-demo`.
2. Write `docs/store/<app>/raw/<locale>/index.json`:
   ```json
   { "app": ":tokoku", "locale": "id", "device": "Pixel_7_API_34", "size": "1080x2400",
     "toolkit": "compose", "navigation": "ui", "date": "<date>",
     "screens": [ { "id": "home", "file": "01-home.png", "title": "Beranda" } ] }
   ```
   `/enhance` and `/portfolio:publish` read this, so include only captures that passed
   verification.
3. Report a table: screen id, file, size, how it was opened, status
   (`captured` / `skipped: <reason>`).
4. If any screen failed on a selector, list the on-screen texts you saw there.
5. Next step: `/play-store:enhance <app>`.

## Rendering `@Preview` composables instead (`--from-previews`)

A Compose app can produce screen images with no device at all, through Compose preview
screenshot testing. It renders `@Preview` functions, so what you get is the preview's data,
not the running app with real content. That makes it good for a clean, deterministic image
of a screen's design, and wrong for anything whose value is the real data in it.

Only usable when the project already has it configured (Step 2, T7). This command never
adds the plugin.

1. Find the real task name; it has changed between AGP versions:
   `./gradlew :<app>:tasks --all` and look for the screenshot update task
   (`update<Variant>ScreenshotTest` on current versions).
2. Run it, then locate the rendered PNGs from the task's own output rather than assuming a
   path; they land under the module's `build/outputs/`.
3. Match each rendered file to a screen `id` in `screens.yml` by the preview function name,
   asking once if a mapping is ambiguous, and copy them into `raw/<locale>/` under the
   usual `<NN>-<id>.png` names.
4. In `index.json` set `"source": "preview"`, and say in the report that these are preview
   renders, so the user can decide whether they're honest enough for a store listing.

If T7 is false, don't run anything. Print what enabling it takes
(`android.experimental.enableScreenshotTest=true` plus the `com.android.compose.screenshot`
plugin on the module) and stop.

## Re-runs

Re-running overwrites the files for the screens it captures and leaves the others. Use
`--screen <id>` to redo a single one after fixing its steps in `screens.yml`. For mode
`ui`, a screen with `from:` needs the screens before it in the same run, so `--screen`
on such a screen walks from the launcher instead.
