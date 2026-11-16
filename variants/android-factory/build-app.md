---
description: Build a new Android app from a brief and a reference app — PRD, mockups, tasks, scaffold, every task implemented, release build hardened — and stop when it is ready for the owner to try
allowed-tools: all
---

# Build App — from brief + reference to "ready to try"

One command for the whole build half of a new app. It runs the Mock-First loop
(`prd` → `mockup` → `break-task` → `coding`) end to end **without stopping at the usual
approval gates**, then hardens the release build, and stops at a single gate: the owner
tries the app. Store work (capture → release) starts only after that review, with
`/play-store:ship` (play-store plugin).

## Arguments

| Argument | Meaning |
|---|---|
| `"<brief>"` | Required. What the app is: audience, core job, monetisation if known. A path to a research or brief file is fine. |
| `--ref <app>` | The reference app to follow: a folder in `apps_dir`, a repo path, or a factory app name. Default: the most recently changed app of the same shape. |
| `--name "<Store title>"`, `--package <id>` | Optional. Otherwise proposed in Step 2 and recorded. |
| `--locales en,de` | Locales to ship. Default: the reference's. |
| `--in-factory app:<name>` | Build as a new app inside the current factory (uses `/android-factory:new-app` + `INDEX.md`). Default: a **standalone repo** in `apps_dir/<slug>-android`. |
| `--ask` | Restore the normal gates (one question at a time, wait at each document). |

## The autonomy contract

Unless `--ask` is given, or `publisher.yml` says `autonomy: ask`, invoking this command is the
owner's approval for every gate up to the hand-off:

- Q&A questions are answered with the **recommended** option, researched first when unsure
  (competitors, store reviews, library choice). Each answer goes in the PRD under
  **"Decisions taken without asking"**, one line each with the reason, so the owner can
  overturn any of them at review.
- Mockups, tasks and each task's commit go ahead without waiting.
- Still stop for: signing in, creating accounts, paying, anything published or pushed
  outside the machine (no store, AdMob or git remote work here), and a brief that is
  ambiguous about *what the app is* (one question, then continue).

## Step 0 — Publisher settings

Read `~/.claude/mock-first/publisher.yml`. Create it with the owner on first use (it is shared
by `play-store:*` and `portfolio:*`). Schema:

```yaml
publisher: Example Studio            # developer name shown on Play
package_prefix: com.example          # new packages: <prefix>.<slug>
apps_dir: ~/Projects                 # standalone repos live here
dname: "CN=Example Studio, O=Example, C=US"   # upload-key certificate
keys_dir: ~/Keys/example             # one "<slug> key/" folder per app
keys_readme: README.md               # table of keys inside keys_dir
key_validity_days: 10000
support_email: support@example.com
website: https://example.com
privacy_url: https://example.com/apps/{slug}/privacy-policy/
site_repo: ~/Projects/example-web    # portfolio/privacy site (portfolio plugin)
git: { org: example-org, visibility: private, branch: main }
testers_list: Internal               # existing Play Console tester list
autonomy: recommended                # recommended | ask
monetisation: ads + one-time premium_lifetime, no subscription
art: "how illustrations and icons are made (e.g. a local image-generation workflow)"
```

## Step 1 — Read the reference

From `--ref`, record (in the PRD's technical section, not a new file):

- Gradle layout (modules, version catalog, AGP/Kotlin/SDK levels, `build-logic`), the
  wrapper version, `local.properties` key names it reads, signing block, R8 rules.
- Reusable pieces to copy and adapt — not wholesale: ads + consent (UMP) wrapper,
  billing, in-app review, preferences, theme scaffolding, share/export, the store tooling
  under `docs/store/_tools/`.
- What the owner corrected in that app (its `docs/progress-report.md`, PRD decisions,
  git log) — those corrections are requirements here.

## Step 2 — PRD

Run the `/mock-first:prd` flow (or `/android-factory:prd app:<name>` with `--in-factory`) with
the brief as input, answering per the autonomy contract. Decide and record: name (check the
Play search for collisions), package, locales, free vs premium limits, ad placements with
frequency caps, product ids and prices, target audience age, permissions and why. Write
`docs/PRD.md` (standalone) or the scoped PRD.

## Step 3 — Mockups and tasks

1. `/mock-first:mockup` for the main screens with the `android` profile (phone; add a tablet
   width view for the busiest screen). No wait for approval; they are the reference the
   code is checked against.
2. `/mock-first:break-task` (or `/android-factory:break-task`) into `docs/TASKS.md`, phases per
   `platforms/android.md`. Put the risky logic in a pure-JVM `:domain` module with unit
   tests (generators, calculators, schedulers, validators).

## Step 4 — Scaffold

Standalone (default):

1. `apps_dir/<slug>-android/`, `git init -b <git.branch>`. Copy from the reference: wrapper,
   version catalog, root build files, `.gitignore` (must ignore `local.properties`,
   `*.jks`, `build/`, `docs/store/**/raw*/`, `node_modules`), then rename package and
   namespace.
2. Modules: `:app` (+ `:domain` when Step 3 asked for it).
3. Build config that every app needs from day one:
   - AdMob ids read from `local.properties`; missing keys fall back to Google's test ids;
     the **debug** build type is pinned to the test ids even when real ones exist.
   - Release signing read from `RELEASE_*` (see `/play-store:signing`), applied only when set.
   - `minifyEnabled true` for release, with keep rules for Room, Billing, serialization —
     and `-keep class * extends androidx.room.RoomDatabase { <init>(); }` even without your
     own Room, because the ads SDK's WorkManager needs it.
   - `locales_config.xml` for the shipped locales, Auto Backup rules, adaptive icon.
4. `JAVA_HOME=<jbr> ./gradlew :app:assembleDebug` must pass before the first task.
   Commit: `chore: scaffold from <ref>`.

`--in-factory`: run `/android-factory:new-app` with the answers already decided, then the
same build-config checklist against the convention plugin.

## Step 5 — Implement every task

Loop `/mock-first:coding` (or `/android-factory:coding`) task by task until `TASKS.md` has
none left: implement, build the affected modules, run the module tests, commit with the task
code, one progress line. No waiting between tasks. A task that turns out wrong is fixed in
`TASKS.md` with a line in the PRD's decisions, not silently skipped.

## Step 6 — Quality pass (the bar an owner review applies)

Go through each screen on an emulator (phone, then a tablet AVD) and fix before hand-off:

- **Looks like a product, not a template:** no placeholder shapes or stock-looking sample
  images, icons consistent in one family, themed illustrations made the way `publisher.yml`
  `art:` says (fallback: a consistent icon set), motion only where it explains state.
- **Every state:** loading, empty, error with retry, permission denied (and "don't ask
  again"), process death restore, dark theme, large font.
- **Tablets:** content max width (~760dp) or a two-pane layout; nothing stretched edge to edge.
- **Locales:** every string translated including defaults and sample data; numbers,
  decimals, dates and units formatted per locale; no text overflow in the longest language.
- **Time-based logic:** default dates/plans never in the past; notifications actually fire
  for the first event (test it); exact alarms when allowed, a bounded window otherwise.
- **Hardware:** optional features declared `required="false"` with a manual fallback; no
  Play-services modules that download on first use for a core flow.
- Honest self-check: list what still looks generic and fix what can be fixed now.

## Step 7 — Release hardening

1. Unit tests: `./gradlew test` (or the modules') pass.
2. `./gradlew :app:assembleRelease` with test ad ids (no real ids exist yet), install it on
   an emulator, set the device locale to each shipped locale, walk the main flow end to end,
   then `adb logcat -d | grep -E "FATAL|AndroidRuntime"` — must be empty. R8 breaks Room,
   Billing and reflection only in this build.
3. Debug build installed on the emulator for the owner, with demo data if the app needs
   history to look alive (a debug-only intent extra such as `--ez demo true`).

## Step 8 — Hand-off (stop here)

Report, in the owner's language:

- What the app does now, screen by screen, and how to try it: emulator already running
  with the debug build, or `./gradlew :app:installDebug`, or the APK path; the demo switch.
- **Decisions taken without asking** (from the PRD) — the list to overturn.
- What is still generic or missing, honestly.
- Commits made (local only; nothing pushed).
- Next, after the owner says OK: `/play-store:ship <app>` — signing, privacy page, AdMob,
  listing, captures (phone, 7", 10"), Console, release, products. Change requests go
  through `/mock-first:revise` (or `/android-factory:revise`).

Update the project memory/progress file with the state, then stop.
