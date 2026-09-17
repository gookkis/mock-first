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
/play-store:doctor       # check the JDK, SDK, emulator, Node, Playwright first
/play-store:capture      # raw screenshots through ADB, screen by screen
/play-store:enhance      # device frame, background, caption + the feature graphic
/play-store:listing      # per-locale copy + the App content answer sheet the Console asks for
/play-store:aso          # score the listing and compare it with named competitors
/portfolio:privacy       # privacy policy page + Data Safety summary, from what the code does
/portfolio:publish       # the portfolio entry on the site
/portfolio:sync          # which apps have no entry yet, or a stale one
```

These are Android-specific by construction: they read a Gradle app module, drive ADB, and
write `fastlane/metadata/android/`. Don't reach for them from another profile — name the gap
instead.

## Hidden work this platform always drags in

Room migrations, R8/ProGuard rules for anything reflective, manifest permissions and their
rationale dialog, build variants and signing config, configuration changes (rotation,
theme, font size), and back-stack behaviour after a deep link.
