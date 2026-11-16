# Platform profile — Android

A native Android app. The mockup stays static HTML (fast to write, opens straight in a
browser) — not Compose — but it is built so the translation to Compose is direct. Read this
alongside `/mock-first:mockup`, `/mock-first:break-task` or `/mock-first:coding` — those own
the flow.

## Where things live (check the project shape FIRST)

Android is the one platform here that is often a factory of several apps over shared
modules, and that decides where every file goes. Work it out before anything else:

1. **`docs/INDEX.md` exists** -> multi-module factory, already mapped. Use the scope's
   "Docs" column verbatim and its "Scope prefix" for task numbers. This is the
   `android-factory` layout; if that plugin is installed, its commands are the better fit.
2. **No `INDEX.md`** -> read `settings.gradle` / `settings.gradle.kts` and count:
   - more than one module applying `com.android.application`, **or** any `:core:*` /
     `:feature:*` / `:shared:*` module -> treat as **multi-app**;
   - a single `:app` module (with or without a library or two) -> **single app**.
3. **No Gradle files** (greenfield, nothing generated yet) -> single app. Say so in one
   line, so the user can correct you before you write a path that later has to move.

| Shape | Mockups | PRD | Tasks | Task numbers |
| --- | --- | --- | --- | --- |
| Single app | `docs/mockups/` | `docs/PRD.md` | `docs/TASKS.md` | `T01` |
| Multi-app | `docs/apps/<app>/mockups/` | `docs/apps/<app>/PRD.md` | `docs/apps/<app>/TASKS.md` | `<APP>-T01` |

On a multi-app project every command needs to know **which app** — take it from
`$ARGUMENTS` (`app:<name>`), and if it is missing, list the apps from `settings.gradle` and
ask. Never write into `docs/mockups/` on a multi-app project: the next app's screen of the
same name would overwrite it.

## Canvas

Build at a **phone viewport of 360-412px**. Material 3 patterns: top app bar, FAB, bottom
navigation, cards, list items with a leading icon and supporting text.

## Design language

- Material 3, not a web page shrunk down: elevation and tonal surfaces instead of borders,
  an FAB for the one primary action, bottom nav for 3-5 destinations.
- The system Back gesture always exists — every screen must have an answer for it, and a
  destructive flow needs a confirm before it fires.
- Touch targets 48dp or more.
- Dark theme is not optional on Android. If the mockup only works on a light surface, say
  so.

## Naming the component

Name the Compose component the code will use, in an HTML comment:

```html
<!-- LazyColumn -->
<!-- ElevatedCard -->
<!-- ModalBottomSheet -->
```

## States this platform always needs

Loading, empty, error with a retry, plus the two Android ones people forget: **runtime
permission denied** (and the "don't ask again" variant) and **process death** — the screen
restored from a `SavedStateHandle` with no in-memory state left.

## Task phases

```
Phase 0 — Module & build (multi-app only): new Gradle module, dependencies, build variants
Phase 1 — Data: Room / DataStore / network sources
Phase 2 — Domain: use cases, models, repository interfaces
Phase 3 — DI: Hilt/Koin modules and bindings
Phase 4 — Presentation: ViewModel, state, events
Phase 5 — UI: Compose screens matching the approved mockup
Phase 6 — Integration: navigation, wiring into the app
Phase 7 — Verification against the PRD's success criteria (manual, no automated tests)
```

Skip phases that don't apply. Don't force content into every phase.

## Verify

Build **only the affected modules**, never the whole project:

```
./gradlew :<module>:assembleDebug
```

On a multi-app project, a change inside a shared module also has to build every app that
depends on it — a shared change can break another app even when the module itself compiles.

Commit area = the module: `feat(core-ui): add EmptyState component (CORE-UI-T03)`.

## Shipping it

Android is the one platform here with a full store pipeline in commands. Once the tasks are
done, the release path is its own plugin set — install `play-store@mock-first` (it pulls in
`android-factory`) and `portfolio@mock-first` on top:

```
/android-factory:build-app  # brief + reference app -> PRD, mockups, tasks, code, hardened, ready to try
/play-store:ship            # after the owner tried it: every step below, in order
/play-store:doctor          # check the JDK, SDK, emulator, Node, Playwright first
/play-store:signing         # the app's own upload key, wired via local.properties, fingerprint checked
/portfolio:privacy          # privacy policy page + Data Safety summary, from what the code does
/play-store:admob           # AdMob app + one unit per format the code loads, GDPR message
/play-store:listing         # per-locale copy + the App content answer sheet the Console asks for
/play-store:capture         # raw screenshots through ADB: phone, --tablet 7, --tablet 10
/play-store:enhance         # device frame, background, caption + the feature graphic
/play-store:console         # create the app, App content, store settings, listing
/play-store:release         # upload the bundle, notes, testers, Send for review; in-app products
/play-store:aso             # score the listing and compare it with named competitors
/portfolio:publish          # the portfolio entry on the site
/portfolio:sync             # which apps have no entry yet, or a stale one
```

These are Android-specific by construction: they read a Gradle app module, drive ADB, and
write `fastlane/metadata/android/`. Don't reach for them from another profile — name the gap
instead.

## Hidden work this platform always drags in

Room migrations, R8/ProGuard rules for anything reflective, manifest permissions and their
rationale dialog, build variants and signing config, configuration changes (rotation,
theme, font size), and back-stack behaviour after a deep link.

## Release hardening (before anything goes to the store)

- **Run the release build, not just debug.** Install the minified release APK on an
  emulator, walk the main flow, and check `adb logcat -d | grep -E "FATAL|AndroidRuntime"`.
  R8 breaks Room, Billing and reflection only there. An app that shows AdMob ads needs
  `-keep class * extends androidx.room.RoomDatabase { <init>(); }` even with no Room of its
  own — the ads SDK's WorkManager database crashes at start otherwise.
- **Debug never requests live ads.** Pin the debug build type to Google's test ad ids even
  when `local.properties` holds the real ones; release reads the real ids and falls back to
  test ids when they are missing (so check the release manifest before uploading).
- **Optional hardware is optional.** `uses-feature … android:required="false"` for camera,
  autofocus and the like, with a manual fallback — otherwise Play drops devices on update.
- **No on-demand Play-services modules in a core flow.** The GMS code scanner downloads a
  module on first use and failed or crashed on real devices; CameraX + ZXing in-app works
  everywhere.
- **Data that outlives a version** (share codes, deep links, backups) carries a version, and
  an older app reading a newer one offers "Update" with a store link instead of "invalid".
- **Locales are more than strings:** translate defaults and sample data, format numbers,
  decimals, dates and units per locale, and check the longest language for overflow.
- **Tablets:** cap content width (~760dp) or use two panes; the 7" and 10" screenshots are
  required by Play and show a stretched phone layout immediately.
- **Time-based features:** defaults never in the past; the first notification of a new
  schedule actually fires; exact alarms when allowed, a bounded window otherwise.
- **Build environment:** set `JAVA_HOME` to Android Studio's JBR for every Gradle call, run
  the wrapper as `sh ./gradlew` when it has no executable bit, and never trust a
  "verification" that ran after a failed build — it reads stale outputs.
- **Before a version bump**, fetch upstream: another machine may already have used the next
  `versionCode`.
