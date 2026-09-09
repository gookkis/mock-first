# Changelog

## Unreleased

### Added

- `variants/portfolio/` — publish an app to a personal Astro site's portfolio.
  - `/portfolio:publish <app>` — gathers title, description, technologies (from Gradle
    deps), Play Store link, problem/solution (from the PRD), body, and cover (from
    `/play-store:enhance`) and writes one Markdown entry per locale with matching
    filenames plus the image, reading the schema from the site's `content.config.ts` at
    run time. Drafts are shown before writing; the site is built; the commit stays local
    unless `--push` (deploys) or `--pr` (review branch) is given.
  - `/portfolio:privacy <app>` — builds a data inventory from the code (merged
    permissions, third-party SDKs, accounts, local storage, network targets, backup
    flags), asks for confirmation, then writes the app's privacy policy page in both
    locales at `src/pages/apps/<slug>/privacy-policy.astro` following the site's existing
    per-app page, plus a Play Console Data Safety summary in
    `docs/store/<app>/data-safety.md`. Sections exist only for SDKs actually found.
  - `/portfolio:sync` — read-only comparison of app modules vs site entries: missing,
    stale, draft, locale file missing, broken image path.
  - Site location and paths live in `~/.claude/portfolio.yml`.
- `variants/play-store/` — a companion command set for Play Store assets on a multi-app
  Android project, one `docs/store/<app>/` folder per app module:
  - `/play-store:doctor` — read-only check that the JDK, Android SDK (ADB, emulator, AVDs),
    Node.js, and Playwright + Chromium are installed, that the project has a Gradle wrapper
    and at least one app module, and which devices are connected. Prints per-OS install
    commands and says which commands are ready or blocked. `--fix` offers the two
    project-local fixes (install Playwright, download Chromium) and nothing else.
  - `/play-store:capture <app>` — builds and installs the app, drives the emulator through
    ADB from a `screens.yml` manifest (deep link / activity / tap-by-text steps), sets a
    clean demo status bar, and saves raw captures it has looked at.
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
