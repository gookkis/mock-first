# Command reference

Every command in the four plugins, with its arguments and a usage example. The command files
under [`commands/`](../commands/) and [`variants/`](../variants/) are the source of truth; this
page is the quick lookup.

| Plugin | Commands |
| --- | --- |
| [`mock-first`](#mock-first--any-platform) | `prd` · `mockup` · `break-task` · `coding` · `revise` · `review` · `adopt` |
| [`android-factory`](#android-factory--android-apps-and-multi-module-projects) | `build-app` · `new-app` · `adopt` · `docs` · `prd` · `break-task` · `coding` · `revise` |
| [`play-store`](#play-store--from-screenshots-to-a-release) | `doctor` · `ship` · `signing` · `admob` · `capture` · `enhance` · `keywords` · `listing` · `aso` · `console` · `release` |
| [`portfolio`](#portfolio--the-site-entry-and-privacy-page) | `publish` · `privacy` · `sync` |

`<app>` in the Android commands is a Gradle path (`:apps:tokoku`) or a module name (`tokoku`).
Shared settings for `build-app`, `play-store:*` and `portfolio:*` live in
`~/.claude/mock-first/publisher.yml` — see [Configuration](#publisheryml).

---

## Make a new Android app — the short path

```bash
# 1. Build the app: PRD → mockups → tasks → code → release smoke test. Stops once for you.
/android-factory:build-app "Offline sourdough timer with feeding log" --ref tpa-numerik-android --locales en,de

# 2. Try it on a device. Anything off → ask for the fix, or /mock-first:revise.

# 3. Publish: signing → privacy page → AdMob → listing → captures → Console → release → products.
/play-store:ship sourdough
```

`ship` stops only where you have to act (signing in to Chrome, a choice only you can make).
If it stops halfway, run it again — it resumes at the first step whose output is missing, or
use `--from <step>`.

---

## mock-first — any platform

The base workflow: PRD → mockup → tasks → code, one gate at a time. `mockup`, `coding` and
`revise` accept a platform profile as the first word (`web`, `mobile-web`, `android`, `ios`,
`desktop-linux`, `desktop-macos`, `desktop-windows`); otherwise the platform is read from the
PRD or the repo.

### `/mock-first:prd`

Writes `docs/PRD.md` through Q&A, one question at a time, with up to 3 brainstormed options on
the questions that matter.

| Argument | Meaning |
| --- | --- |
| *(none)* | Starts the Q&A. |

```bash
/mock-first:prd
```

### `/mock-first:mockup`

Builds one self-contained HTML mockup of a screen at the platform's canvas, revises it with you,
then saves it as `docs/mockups/<screen>-approved.html`.

| Argument | Meaning |
| --- | --- |
| `[<platform>]` | Optional platform profile. |
| `[<screen>]` | The screen to mock up. If empty, lists "Screens Needed" from the PRD and asks. |

```bash
/mock-first:mockup android home
```

### `/mock-first:break-task`

Reads the PRD and the approved mockups and writes the checklist `docs/TASKS.md`.

| Argument | Meaning |
| --- | --- |
| *(none)* | Breaks down the whole PRD. |

```bash
/mock-first:break-task
```

### `/mock-first:coding`

Works one task: implement, build, short diff summary, wait for your OK, commit only that task's
files, tick the box.

| Argument | Meaning |
| --- | --- |
| `[<platform>]` | Optional platform profile. |
| `[<task code>]` | Work this task (e.g. `T03`). Default: the next unchecked one. |

```bash
/mock-first:coding T03
```

### `/mock-first:revise`

A change after the documents exist: categorizes it (new feature, changed feature, UI change, …),
reports which tasks and commits it affects, and waits for your decision before editing.

| Argument | Meaning |
| --- | --- |
| `[<platform>]` | Optional platform profile. |
| `[<change>]` | The change in plain words. Asked if empty. |

```bash
/mock-first:revise "add a dark mode toggle in settings"
```

### `/mock-first:review`

Read-only audit of drift between the PRD, the code and `TASKS.md`. Marks each item `DONE`,
`PARTIAL`, `MISSING` or `? UNVERIFIED`. Changes nothing.

```bash
/mock-first:review
```

### `/mock-first:adopt`

One-time install into a project that already has code: writes the PRD and tasks from what is
already there.

```bash
/mock-first:adopt
```

---

## android-factory — Android apps and multi-module projects

Scoped versions of the base commands, plus the one-pass app builder. A **scope** is
`app:<name>`, `core:<module>` or `module:<:gradle:path>`; when it is empty, the command lists
the scopes from `docs/INDEX.md` and asks.

### `/android-factory:build-app`

A new app from a brief and a reference app in one pass: PRD, mockups, tasks, scaffold, every
task implemented and committed, a quality pass (icons, ads, states, tablets, locales,
`review_checklist`) and a release smoke test. Takes the recommended option at each gate and
records those decisions in the PRD; stops once, for you to try the app.

| Argument | Meaning |
| --- | --- |
| `"<brief>"` | Required. What the app does, in one or two sentences. |
| `--ref <app>` | Reference app to follow: a folder in `apps_dir`, a repo path, or a factory app. Default: the most recently changed app of the same shape. |
| `--name "<Store title>"` | Store title. Default: proposed and recorded. |
| `--package <id>` | Package id. Default: `<package_prefix>.<slug>`. |
| `--locales en,de` | Locales to ship. Default: the reference's. |
| `--in-factory app:<name>` | Build inside the current factory instead of a standalone repo in `apps_dir/<slug>-android`. |
| `--ask` | Restore the normal gates (one question at a time, wait at each document). |

```bash
/android-factory:build-app "Kanji flashcards with spaced repetition" --ref tpa-verbal-android --locales en,id
/android-factory:build-app "Daily habit tracker" --in-factory app:habit --ask
```

### `/android-factory:new-app`

Adds an app to an existing factory, following the closest existing app. Asks name and package,
the reference app, the shared modules it uses, and what makes it different; then registers it
in `docs/INDEX.md`.

```bash
/android-factory:new-app
```

### `/android-factory:adopt`

One-time bootstrap for a multi-module project: scans `settings.gradle`/`build.gradle` and builds
the module → app dependency map at `docs/INDEX.md`.

```bash
/android-factory:adopt
```

### `/android-factory:docs`

Creates or refreshes `PRD.md`, `MODULE.md` and `TASKS.md` for every Gradle module. Idempotent.

| Argument | Meaning |
| --- | --- |
| *(none)* / `--all` | Every module. |
| `app:<name>` / `module:<path>` | One scope. |
| `--new` | Only create what is missing. |
| `--check` | Read-only: report modules without docs, orphaned or stale docs. |

```bash
/android-factory:docs --check
/android-factory:docs module::core:ads
```

### `/android-factory:prd` · `break-task` · `coding` · `revise`

Same as the base commands, scoped to one app or module. `revise` adds a cross-app impact
analysis when a shared module changes.

| Command | Argument | Example |
| --- | --- | --- |
| `prd` | `<scope>` | `/android-factory:prd app:tokoku` |
| `break-task` | `<scope>` | `/android-factory:break-task core:billing` |
| `coding` | `<scope> [<task code>]` | `/android-factory:coding app:tokoku TOKOKU-T05` |
| `revise` | `[<change>]` | `/android-factory:revise "move the ad wrapper into core:ads"` |

---

## play-store — from screenshots to a release

Run `doctor` once per machine. `ship` runs the rest in order; each step also works alone.

### `/play-store:doctor`

Checks that the JDK, Android SDK, emulator, Node.js and Playwright are installed and prints the
install command for what is missing. Read-only.

| Argument | Meaning |
| --- | --- |
| `--system` | Only machine-level tools. |
| `--project` | Only project checks (Gradle wrapper, app modules, Playwright). |
| `--fix` | Offer the safe, project-local fixes. Never runs system installers. |

```bash
/play-store:doctor --fix
```

### `/play-store:ship`

After you have tried and approved the app: signing → privacy page → AdMob → listing → phone,
7" and 10" captures → enhance → Console → release → in-app products.

| Argument | Meaning |
| --- | --- |
| `<app>` | Required. |
| `--from <step>` | Resume at `signing`, `privacy`, `admob`, `listing`, `capture`, `enhance`, `console`, `release` or `products`. Default: the first step whose output is missing. |
| `--until <step>` | Stop after that step. |
| `--track internal\|closed` | Passed to `release`. Default `internal`. |
| `--ask` | Each sub-command asks where it normally would. |

```bash
/play-store:ship sourdough
/play-store:ship sourdough --from console --until release
```

### `/play-store:signing`

Creates the app's own upload key in `keys_dir` (or reuses the existing one — never re-keys a
live app), wires it into the release build through `local.properties`, and checks the
fingerprint of the signed bundle.

| Argument | Meaning |
| --- | --- |
| `<app>` | Required. |
| `--alias <name>` | Alias for a new key. Default `upload`. |
| `--verify` | Only build and check the fingerprint. Changes nothing. |

```bash
/play-store:signing sourdough
/play-store:signing sourdough --verify
```

### `/play-store:admob`

In your signed-in Chrome: creates the AdMob app and one ad unit per format the code loads,
writes the ids into `local.properties`, and proves they reach the release build.

| Argument | Meaning |
| --- | --- |
| `<app>` | Required. |
| `--name "<AdMob app name>"` | Default: the store title, else `app_name`. |
| `--formats banner,interstitial,rewarded` | Override the formats detected in the code. |
| `--gdpr` | Also attach the app to the GDPR consent message (needs a live privacy URL). |
| `--no-build` | Skip the release-build check. |

```bash
/play-store:admob sourdough --gdpr
```

### `/play-store:capture`

Builds, installs and captures raw screenshots through ADB, screen by screen, driven by
`docs/store/<app>/screens.yml`.

| Argument | Meaning |
| --- | --- |
| `<app>` | Required. |
| `--device <serial\|AVD>` | Default: `device:` in `screens.yml`, else the first running device. |
| `--locale <xx>` | Default: every locale in `screens.yml`. |
| `--screen <id>` | Only this screen. Repeatable. |
| `--tablet [7\|10]` | Tablet profile: `10` landscape → `raw-tablet/`, `7` portrait → `raw-tablet7/`. Play needs both, even for phone-only apps. |
| `--no-build` | The app is already installed. |
| `--variant <name>` | Gradle variant to install. Default `debug`. |
| `--keep-demo` | Leave the status bar in demo mode. |
| `--from-previews` | Render Compose `@Preview`s instead of driving a device. |
| `--debug-ui` | On a failed tap, save the accessibility dump and a screenshot. |

```bash
/play-store:capture sourdough --locale de
/play-store:capture sourdough --tablet 7
```

### `/play-store:enhance`

Turns raw captures into marketing screenshots (device frame, background, caption) and the
feature graphic.

| Argument | Meaning |
| --- | --- |
| `<app>` | Required. |
| `--locale <xx>` | Default: every captured locale. |
| `--style <name>` | One style for the set: `bleed` (default), `clean`, `tilt`, `plain`. |
| `--vary` | A different layout per screen (`hero`, `bleed`, `float`, `tilt`, `clean`, `zoom`, `duo`, optional `panorama`). Default for new themes. |
| `--variants 2\|3` | Render alternative sets A/B/C with a contact sheet each. |
| `--pick A\|B\|C` | Promote one variant set and record it in `theme.yml`. |
| `--screen <id>` | Only this screen. Repeatable. |
| `--feature` | Only the feature graphic. |
| `--tablet [7\|10]` | Tablet output: 10" at 2560x1440, 7" at 1080x1920. |
| `--png` | 24-bit PNG instead of JPEG. |
| `--dry-run` | Write the job file and the first screen's HTML only. |

```bash
/play-store:enhance sourdough --variants 3
/play-store:enhance sourdough --pick B
/play-store:enhance sourdough --tablet 10
```

### `/play-store:listing`

Writes the store listing per locale (`fastlane/metadata/android/`) and the App content answer
sheet (rating, privacy, data safety, declarations).

| Argument | Meaning |
| --- | --- |
| `<app>` | Required. |
| `--locale <a,b>` | Default: `locales:` from `screens.yml`, else `en`. |
| `--from <path>` | Use your own listing text as the source instead of the Q&A. |
| `--text` | Only the listing text. |
| `--console` | Only the App content answer sheet. |
| `--images` | Only refresh the image folders from `docs/store/<app>/out/`. |
| `--notes [<text>]` | Only the release notes for this version. Without text, drafted from the commits. |
| `--upload` | Then push the metadata with `fastlane supply` (asks first). |

```bash
/play-store:listing sourdough --locale en,de
/play-store:listing sourdough --notes
```

### `/play-store:keywords`

Which searches can a new app win? Demand from autocomplete, the ten holders of each search, top-10
odds from LazyASO's multipliers, and a go / maybe / skip / long-tail call. Writes one report per
country; the listing reads the latest.

| Argument | Meaning |
| --- | --- |
| `<app>` | Required. |
| `--terms <a,b,c>` | Candidate searches. Default: seeds from the PRD and competitors, expanded by autocomplete. |
| `--country <US,GB,...>` | One report per country. Default: the countries of the app's locales. |
| `--names` | Draft three titles and short descriptions from the go terms. |

```bash
/play-store:keywords sky-map --names
/play-store:keywords aurora --country US,GB,CA
```

### `/play-store:aso`

Read-only ASO audit of the listing, compared with competitors; writes one report.

| Argument | Meaning |
| --- | --- |
| `<app>` | Required. |
| `--locale <xx>` | Default: every locale in `listing.md`. |
| `--competitors <a,b,c>` | Package ids or Play URLs, up to 5. |
| `--keywords <a,b,c>` | Override the target keywords. |
| `--search-terms <csv>` | Play Console search-terms export, to weight keywords. |
| `--live` | Audit the published Play page instead of the local files. |

```bash
/play-store:aso sourdough --competitors com.example.bread,com.example.starter
```

### `/play-store:console`

In your signed-in Chrome: creates the app in Play Console and fills App content, store
settings, and the listing text and graphics per locale — up to the first bundle upload.

| Argument | Meaning |
| --- | --- |
| `<app>` | Required. |
| `--only content\|settings\|listing` | Run one part. Default: all, in order. |
| `--locale <a,b>` | Default: every folder under `fastlane/metadata/android/`. |
| `--update` | The app already exists; change only what differs. |

```bash
/play-store:console sourdough
/play-store:console sourdough --update --only listing
```

### `/play-store:release`

In your signed-in Chrome: uploads the signed bundle to a track, fills release notes and
testers, sends for review, then creates the one-time in-app products the code sells. Also
handles updates to a live app.

| Argument | Meaning |
| --- | --- |
| `<app>` | Required. |
| `--track internal\|closed\|production` | Default: `internal` for a new app, `production` for a live one. |
| `--testers "<list>"` | Default: `testers_list` in `publisher.yml`. The list must exist. |
| `--products` | Only create the in-app products. |
| `--no-review` | Save the release but leave Send for review to you. |

```bash
/play-store:release sourdough --track internal
/play-store:release sourdough --products
```

---

## portfolio — the site entry and privacy page

### `/portfolio:publish`

Adds or updates the app's portfolio entry (id + en) in the Astro site from the listing and
screenshots.

| Argument | Meaning |
| --- | --- |
| `<app>` | Required. |
| `--site <path>` | Default: `site:` in `~/.claude/portfolio.yml`. |
| `--slug <slug>` | Entry filename and image folder. |
| `--basic` | Only the base fields. |
| `--shots <n>` | Number of screenshots. Default: all, capped at 8. |
| `--image <path>` | Card thumbnail override. |
| `--featured` / `--no-featured` | Set `featured:`. |
| `--draft` | Write with `draft: true`. |
| `--push` | Push to `main` after the commit — **deploys to production**. |
| `--pr` | Commit on a branch and open a PR with `gh`. |
| `--dry-run` | Show what would be written. |

```bash
/portfolio:publish sourdough --dry-run
/portfolio:publish sourdough --push
```

### `/portfolio:privacy`

Writes the app's privacy policy page (id + en) from what the code actually does (permissions,
SDKs, accounts, local data), plus a Data Safety summary.

| Argument | Meaning |
| --- | --- |
| `<app>` | Required. |
| `--slug <slug>` | Page folder. |
| `--template <path>` | Page to copy the structure from. Default: the newest one in the site. |
| `--contact <email>` | Contact address. |
| `--link-entry` | Also link the page from the portfolio entry. |
| `--push` / `--pr` / `--dry-run` | As in `publish`. |

```bash
/portfolio:privacy sourdough --push
```

### `/portfolio:sync`

Read-only report of apps with no entry, a stale entry, or a draft: `missing`, `maybe`, `stale`,
`basic`, `draft`, `ok`.

| Argument | Meaning |
| --- | --- |
| `[<app>]` | Only this app. |
| `--site <path>` | Site repo. |

```bash
/portfolio:sync
```

---

## publisher.yml

Created with you on the first `build-app` or `play-store:*` run, at
`~/.claude/mock-first/publisher.yml`:

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
review_checklist:                    # your recurring revisions; checked before hand-off
  - "UI reads as a paid, production app, not a template"
  - "no emoji used as icons or decoration"
```
