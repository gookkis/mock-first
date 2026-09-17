# Changelog

## 0.3.0 — 2026-09-17

**Breaking for anyone already using `android-factory`:** `/android-factory:mockup` is gone and
the plugin now requires `mock-first`. Use `/mock-first:mockup app:<name> <screen>` — it writes
into the same folder the old command did.

### Removed

- `/android-factory:mockup`. `platforms/android.md` already reads `docs/INDEX.md` and writes
  into the scope's own `mockups/` folder, so the variant's copy added nothing but a second
  place to keep the Material 3 and Compose guidance in sync. `/mock-first:mockup app:<name>
  <screen>` covers the factory.

### Changed

- `android-factory` now declares `mock-first` as a dependency and **extends** it instead of
  standing alone. Installing it pulls the base plugin in, and `play-store` and `portfolio`
  inherit that through the chain. The README's claim that "there is no reason to install
  both" is gone — that was true only while the variant carried its own copy of the loop.
- `/android-factory:prd`, `:break-task` and `:coding` are rewritten as deltas on their base
  counterparts rather than near-copies of them: each now states what a factory changes and
  nothing else. Between them and the deleted mockup command that removes roughly 200 lines of
  duplicated instructions — the Q&A discipline, the brainstorming limits, the commit
  procedure, and an Android phase list that already lives in `platforms/android.md`.
- What stays in the variant is what only a factory has: `INDEX.md` as the dependency map,
  `MODULE.md` as the layout authority, scope prefixes on task numbers, the rule that one
  module owns one `TASKS.md`, the cross-app impact check, and the discipline that a Gradle
  module is never committed without its documents.
- `android-factory` 0.1.0 -> 0.2.0. The other three manifests are unchanged: `mock-first`
  0.2.0, `play-store` 0.2.0, `portfolio` 0.1.0 — nothing inside them changed in this release.

## 0.2.0 — 2026-09-17

### Fixed

- `author` in `.claude-plugin/plugin.json` was a string, which fails the plugin manifest
  schema. `/plugin install mock-first@mock-first` — the install path the README documents —
  stopped at `author: Invalid input`, so it had never worked. It is now an object.

### Added

- `platforms/` — one profile per target platform (`web`, `mobile-web`, `android`, `ios`,
  `desktop-macos`, `desktop-windows`, `desktop-linux`), each owning the canvas the mockup is
  drawn at, the design language, the component names to write into HTML comments, the states
  that platform always needs, where files live, the phase order and the command that verifies
  a task. `/mockup`, `/break-task` and `/coding`
  resolve the platform (explicit argument, then the PRD's Technical Stack, then the repo
  itself) and read the matching profile; they only ask when all three are inconclusive or
  the stack is cross-platform, and then record the answer as a `Platform:` line in the PRD.
- **Desktop is three profiles, not one.** The three OSes disagree on where the menu bar lives
  (top of the screen on macOS, a command bar on Windows, often nowhere on GNOME), which side
  the caption buttons sit on, the button order in a dialog, and how the app is packaged
  (notarized `.dmg`, MSIX, Flatpak). An Electron or Tauri shell does not erase any of that, so
  `desktop` on its own is rejected as a platform and the command asks which OS. The Linux
  profile asks one more question first — GTK/libadwaita or Qt/KDE — because Linux has no
  single HIG to fall back on.
- Every profile checks the project's shape before writing anything: a single app keeps
  `docs/mockups/` and `T01`, while a multi-app or multi-module project — more than one
  `com.android.application` module, any `:core:*`/`:feature:*` module, several app targets in
  an Xcode workspace or `.sln`, several apps in a JavaScript monorepo, or an existing
  `docs/INDEX.md` — switches to `docs/apps/<app>/mockups/` and app-prefixed task numbers, so
  a second app's screen of the same name can't overwrite the first.
- Each profile ends with a "Shipping it" section, which is where the platform's release path
  now hangs: Android names the `/play-store:*` and `/portfolio:*` chain in order; iOS, macOS,
  Windows and Linux state what that store or installer actually demands — notarization and a
  universal binary, MSIX and code signing against SmartScreen, an AppStream metainfo file for
  Flathub — and say plainly that no command set covers them here; the web profiles point at
  whatever the repo already deploys with. `/coding` suggests that step when the last task is
  checked off.
- The platform-resolution block is one shared table of repo signals across all five commands,
  rather than prose repeated five times, so an eighth platform is one row and one new profile.
- `/revise` and `/review` resolve the platform too, so on a multi-app Android or iOS project
  they read `docs/apps/<app>/` instead of reporting every screen as missing from
  `docs/mockups/`.
- A PRD that targets two platforms now produces one mockup per platform
  (`<screen>-web.html`, `<screen>-mobile-web.html`) and one set of UI tasks per platform,
  with shared data and domain tasks left unprefixed.
- The three variants ship as installable plugins from the same marketplace instead of
  copy-by-hand command folders: `android-factory@mock-first`, `play-store@mock-first` and
  `portfolio@mock-first`, each with its own `.claude-plugin/plugin.json`.
- Each variant declares the one below it in `dependencies`, so `/plugin install
  portfolio@mock-first` pulls in `play-store` and `android-factory` on its own, and
  `claude plugin prune` removes them once nothing needs them.
- `.github/workflows/validate-plugins.yml` runs `claude plugin validate --strict` over the
  marketplace and every plugin manifest, and fails on any namespaced command reference that
  points at a file that does not exist.

### Changed

- `/mockup` no longer produces a generic full-width HTML page. It builds at the platform's
  canvas — 1280x800 for web, 390px for mobile web, 360-412px for Android, 393x852 with safe
  areas for iOS — and names the component the code will use (`<!-- LazyColumn -->`,
  `<!-- NavigationStack -->`, `<!-- <DataTable> -->`) in a comment.
- `/break-task` takes its phase order from the profile instead of the one generic six-phase
  list, and `/coding` takes its build command and commit area from the profile rather than
  building whatever the project's default target is.
- The Android notes that only existed inside the `android-factory` variant — phone viewport,
  Material 3, Compose component names, module-scoped builds — are now in
  `platforms/android.md`, where the base commands can reach them on a single-app project too.
- The android-factory commands moved out of the `mock-first` command namespace into their
  own: `/mock-first:adopt` is now `/android-factory:adopt`, and so on for all 8. They had
  been sharing a namespace with the base plugin, which meant the two could not be installed
  together. They are now independent, and the cross-references from the play-store and
  portfolio commands were updated to match.

- `variants/portfolio/` — publish an app to a personal Astro site's portfolio.
  - `/portfolio:publish <app>` — writes one Markdown entry per locale with matching
    filenames, reading the schema from the site's `content.config.ts` at run time. Fills
    the base fields (title, description, technologies from Gradle deps, link, problem and
    solution from the PRD) and the landing-page sections the site renders: `hero` (icon,
    action buttons, note line), `screenshots` (raw captures downscaled to WebP, labelled
    from `screens.yml` and `captions.<locale>.yml`), `features`, `stats` (only from a real
    source, never invented), `highlights` (from the privacy data inventory, so the entry
    and the policy page agree), and `note` (disclaimers). Screenshots come from `raw/`
    rather than the `out/` marketing images, because the site draws its own phone frame
    and captions. `--basic` writes base fields only. Drafts are shown before writing; the
    site is built; the commit stays local unless `--push` (deploys) or `--pr` is given.
  - `/portfolio:privacy <app>` — builds a data inventory from the code (merged
    permissions, third-party SDKs, accounts, local storage, network targets, backup
    flags), asks for confirmation, then writes the app's privacy policy page in both
    locales at `src/pages/apps/<slug>/privacy-policy.astro` following the site's existing
    per-app page, plus a Play Console Data Safety summary in
    `docs/store/<app>/data-safety.md`. Sections exist only for SDKs actually found.
  - `/portfolio:sync` — read-only comparison of app modules vs site entries: missing,
    stale, basic (no landing-page section while captures exist), draft, locale file
    missing, sections uneven between locales, broken image path.
  - Site location and paths live in `~/.claude/portfolio.yml`.
- `variants/play-store/` — a companion command set for Play Store assets on a multi-app
  Android project, one `docs/store/<app>/` folder per app module:
  - `/play-store:doctor` — read-only check that the JDK, Android SDK (ADB, emulator, AVDs),
    Node.js, and Playwright + Chromium are installed, that the project has a Gradle wrapper
    and at least one app module, and which devices are connected. Prints per-OS install
    commands and says which commands are ready or blocked. `--fix` offers the two
    project-local fixes (install Playwright, download Chromium) and nothing else.
  - `/play-store:capture <app>` — builds and installs the app, drives the emulator through
    ADB from a `screens.yml` manifest, sets a clean demo status bar, and saves raw captures
    it has looked at. Handles Compose as well as Views: a detection step reports whether
    screens are reachable by declared deep link, by their own Activity, or only by walking
    the UI, and the tap resolver is written for Compose's accessibility tree (text and
    content description rather than `resource-id`, walking up to the clickable ancestor,
    `--compressed` dumps, animations disabled so dumps don't fail on a busy tree,
    `waitFor`/`scrollTo` steps for lazy lists, and a post-tap check that the screen
    actually changed). `--from-previews` renders `@Preview` composables with no device,
    when the project already has Compose preview screenshot testing configured.
  - `/play-store:enhance <app>` — HTML templates rendered by Playwright turn each capture
    into a 1080x1920 marketing image (device frame, brand gradient, localized headline)
    plus the 1024x500 feature graphic. The renderer and templates are written once to
    `docs/store/_tools/` and are the user's to edit.
  - `/play-store:listing <app>` — Q&A, then title / short / full description per locale in
    the `fastlane/metadata/android/` layout with images copied alongside; budgets counted
    by script; optional `--upload` via `fastlane supply`.
  - `/play-store:aso <app>` — scores the listing (keywords, budgets, policy, localization,
    screenshots), fetches competitor store pages for a side-by-side, and writes a dated
    report with ranked rewrites. States plainly that search volume needs Play Console data.
- `variants/android-factory/docs.md` — `/mock-first:docs`, per-module documentation
  generator. Every entry in `settings.gradle` (app, shared, and local modules alike) gets
  its own folder with `PRD.md`, `MODULE.md`, and `TASKS.md`. `MODULE.md` is filled from the
  Gradle + source scan (responsibility, consumers, dependencies, public API surface, folder
  layout, build command); only `<!-- auto: ... -->` blocks are ever rewritten, so the
  human-written sections survive a refresh. `--check` is a read-only audit: modules without
  docs, orphaned docs, changed dependencies, a local module that has quietly become shared,
  and stale entries.
- `module:<:gradle:path>` as a third scope form alongside `app:` and `core:`, so local
  modules are addressable by `/mock-first:prd`, `/mock-first:break-task`, and
  `/mock-first:coding`.

### Changed

- `variants/portfolio/`, after the first real run against an unreleased app:
  - **Unreleased apps.** `/portfolio:publish` no longer writes a Play URL that would 404:
    `link` and the store action are omitted and the entry is written as a draft. Existing
    entries are found by slug and image folder as well as by `applicationId`, so a
    draft written before the app has a link is updated, not duplicated. `/portfolio:sync`
    matches the same way and reports such an entry as `draft` ("no link yet") rather than
    `maybe`; template modules are skipped and listed.
  - **Dirty site tree.** All three commands stop only when the dirt touches the files
    they write. Anything else (another session's edits) is staged around by path and
    named in the report, instead of blocking the run.
  - **Play icon.** A vector-only adaptive icon has no `ic_launcher-playstore.png`;
    `/portfolio:publish` now renders it with the project's `docs/store/_tools/icon.mjs`
    when present, or asks for an Image Asset export, rather than falling back to a mipmap
    that doesn't exist. `/portfolio:sync` hashes the entry's `icon.png` against the app's
    Play icon so a launcher redesign shows up as a problem.
  - **Privacy on re-run.** `/portfolio:privacy` skips the confirmation stop when the
    inventory was confirmed earlier in the same session and nothing changed, and an empty
    inventory diff rewrites nothing, so a `--link-entry` or `--push` follow-up doesn't
    re-ask or bump `lastUpdated`. `--link-entry` copies the wording from entries that
    already carry the line and skips entries whose `highlights.action` already points at
    the page. The Android-side `data-safety.md` is committed in its own repo; `--push`
    pushes both.
  - **Privacy inventory.** Merged permissions are split into app-declared and
    SDK-injected and the page lists them that way; a public third-party API called
    directly (a rate feed) gets its own section and a Data Safety note instead of being
    treated as a backend; Google UMP is recognised and the "change consent from Settings"
    promise is made only when the privacy-options wiring exists; interfaces wired to
    `NoOp` are recorded as "not in build"; `allowBackup=true` becomes a bullet on the
    page; a support address that differs from the template page's contact is asked
    about; every "you can do X in the app" claim is verified against a screen first. The
    report names the app's own support-links object (e.g. `SupportLinks.privacyPolicyUrl`)
    rather than a generic `strings.xml` suggestion.
  - `/portfolio:sync` reports a `Privacy` column (no page / page unlinked / linked) and
    compares commit timestamps, not dates, since an entry and its sources usually land on
    the same day.
- **Everything is now in English** — every command file (base and Android variant),
  `docs/CONCEPT.md`, `docs/BRANDING.md`, and the walkthrough. The command files are prompts
  the model reads, so a single language keeps their behavior consistent; each command still
  tells the model to write the resulting documents in the user's own language.
- Superseded tasks are now marked `⚠️ SUPERSEDED` instead of `⚠️ USANG`.
- `docs/INDEX.md` gains a `Docs` column (and `Prefix` on the module tables) so every module
  row points at its documentation folder. Local modules are now listed with their own prefix.
- `/mock-first:adopt` scaffolds documentation for **all** modules, not just apps and shared
  modules, and reports the local-module count in the baseline. Its frontmatter now declares
  `Write`/`Edit`, which it always needed to create documents.
- `/mock-first:prd` registers a module that first appears during the Q&A and reads
  `MODULE.md` as context; it writes to the scope's folder from `INDEX.md` rather than
  `docs/PRD.md`.
- `/mock-first:new-app` updates `INDEX.md` before scaffolding, and creates docs for the app
  and each of its local modules.
- `/mock-first:break-task` requires a "register module + create docs folder" companion task
  in Phase 0 for every new module, keeps one `TASKS.md` per module, and closes Phase 7 with
  a `MODULE.md` refresh.
- `/mock-first:coding` documents a newly created Gradle module before committing it, and
  refreshes `MODULE.md`/`INDEX.md` before staging so docs and code land in the same commit.
- `/mock-first:revise` gains category H (module added / removed / promoted to shared): moves
  the docs folder when a local module becomes shared, archives it when a module is deleted,
  and updates `MODULE.md` for every touched module.

## 0.1.0 — 2026-09-07

Initial implementation, based on the Mock-First brainstorm
(https://claude.ai/share/f199189e-3ef1-4915-9058-01ced42185e5).

### Added

- Base workflow: `/prd`, `/mockup`, `/break-task`, `/coding`, `/revise`.
- `/review` — read-only drift audit: matches every PRD requirement against the code (with
  evidence, and an explicit `? BELUM YAKIN` status when evidence is weak), flags code that
  isn't in the PRD, and flags docs whose claims contradict reality. Changes nothing; routes
  findings to `/revise`, `/break-task`, or `/mockup`.
- `/adopt` (base, stack-agnostic) — one-time bootstrap for an existing project: read-only
  scan of docs, stack, and existing screens; marks them `[existing]` so `/break-task` never
  turns already-shipped work into tasks; writes a baseline to `progress-report.md`. Never
  touches existing PRD/docs/code. Safe to re-run.
- `/prd`, `/mockup`, `/break-task` run in a collaborative brainstorming mode (max 3
  options per question, suggestions never enter a document unless explicitly chosen).
  `/coding` is pure execution — no brainstorming, no scope creep.
- `/revise` categorizes any mid-project change and reports its impact (PRD sections,
  mockups, pending vs. completed tasks, affected commits) before touching any file.
- Git safety in `/coding`: never `git add .`/`git add -A`, stages only the files a task
  touched, checks `git status` before starting, stops after 3 consecutive failures for the
  same reason.
- Nothing is deleted: superseded tasks are marked `⚠️ SUPERSEDED` and kept; old mockup variants
  and PRDs move to `archive/` instead of being overwritten.
- `variants/android-factory/`: Android multi-module / multi-app ("apps factory") variant.
  - `/mock-first:adopt` — one-time bootstrap for a brownfield project: scans
    `settings.gradle`/`build.gradle`, classifies modules (app / shared / local), and builds
    `docs/INDEX.md` as the module → app dependency map. Never touches the existing parent
    PRD or existing code.
  - `/mock-first:new-app` — scaffolds a new app from an existing app's pattern and updates
    `INDEX.md`.
  - `/mock-first:prd`, `/mock-first:mockup`, `/mock-first:break-task`, `/mock-first:coding`
    scoped to `app:<name>` or `core:<module>`, with per-scope task numbering
    (`TOKOKU-T01`, `CORE-UI-T01`) to avoid collisions across apps.
  - `/mock-first:revise` reports cross-app impact for shared-module changes, reading
    `INDEX.md` to show which apps and which already-completed tasks are affected.
- `docs/CONCEPT.md` — design rationale and architecture.
- `docs/COMPARISON.md` — honest comparison against GitHub Spec Kit, ShipSpec, and others.
- `docs/BRANDING.md` — naming, positioning, and GitHub presentation guide.
- `.claude-plugin/plugin.json` and `marketplace.json` for plugin-style installation.
