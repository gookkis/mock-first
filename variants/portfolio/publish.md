---
description: Add or update one app's portfolio entry (id + en) in the Astro site from the Android project
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(date*), Bash(mkdir*), Bash(ls*), Bash(cat*), Bash(cp*), Bash(test*), Bash(node*), Bash(npm*), Bash(npx*), Bash(git*), Bash(gh*)
---

# Portfolio publish — from an app module to a site entry

Reads one app module of the Android project plus whatever `/play-store` already produced,
and writes the matching portfolio entry into the Astro site: one Markdown file per locale
(same filename in `id/` and `en/`) and one cover image. Validates against the site's
content schema, builds the site, commits. Pushing is opt-in because a push to `main`
deploys straight to production.

## Arguments

Read `$ARGUMENTS`:

| Argument | Meaning |
|---|---|
| `<app>` | Required. Gradle path (`:tokoku`) or app name (`tokoku`). |
| `--site <path>` | Site repo. Default: `site:` in `~/.claude/portfolio.yml`. |
| `--slug <slug>` | Entry filename. Default: derived (see Step 2). |
| `--image <path>` | Cover image to use instead of the auto-picked one. |
| `--featured` / `--no-featured` | Set `featured:`. Default: keep the existing value, `false` for a new entry. |
| `--draft` | Write with `draft: true` (entry stays invisible on the site). |
| `--push` | After committing on `main`, push. **This deploys to production.** |
| `--pr` | Commit on branch `portfolio/<slug>`, push that branch, open a PR with `gh`. Nothing reaches `main`. |
| `--dry-run` | Show the two Markdown files and the image choice, write nothing. |

Without `--push` or `--pr`: files written, site built, changes committed on the current
branch of the site repo, not pushed. The user pushes when ready.

## Absolute rules

- **Never touch site code.** Only `src/content/portfolio/<locale>/<slug>.md` and
  `public/images/portfolio/<slug>.<ext>` are written. No `.astro`, no config, no other
  entry.
- **Read the schema from the site, don't assume it.** Parse the `portfolio` collection in
  `src/content.config.ts` (or `src/content/config.ts`) at run time and produce exactly its
  fields. If the schema has a field this command doesn't know, leave it out and mention it.
- **Same filename in every locale.** The site's language switcher maps `id/x.md` to
  `en/x.md` by name. Never write one locale without the other.
- **Two languages, two texts.** `en/` is written in English by you, not a literal
  translation. Same facts, natural phrasing.
- **No marketing copy.** The site is a personal portfolio in the first person's voice
  (read two existing entries first to match it). Store-listing text is a source of facts,
  not text to paste. No "best", no "#1", no exclamation marks.
- **Problem and solution are the entry.** If neither the PRD nor the user can say what
  problem the app solves, stop and ask; don't fill the fields with feature lists.
- **Never overwrite an existing entry without showing the diff** and getting a yes.
- **Never push without the flag.** `--push` deploys; say so before running it.
- Run `date +%Y-%m-%d` for the date.

## Step 0 — Site config

`~/.claude/portfolio.yml`:

```yaml
site: C:/Users/user/ClaudeProject/gookkis-web
collection: src/content/portfolio      # locale subfolders inside
images: public/images/portfolio        # written as /images/portfolio/<slug>.<ext> in frontmatter
locales: [id, en]
default_locale: id
branch: main
site_url: https://gookkis.com
```

If the file is missing, ask for the site path, detect the rest from the repo (the
`base:` of the portfolio collection, the `public/images/...` path used by existing
entries' `image:` fields, the locale subfolders, `site:` in `astro.config.mjs`), show it,
and write the file.

Then in the site repo: `git status --short` must be clean (or only untracked files
outside the portfolio folders); otherwise stop and say what's dirty. Note the current
branch.

## Step 1 — Gather from the Android project

1. Resolve the app module (as in `/play-store:capture` Step 1): Gradle path,
   `applicationId`, `app_name` from `strings.xml`, `versionName`.
2. Sources, in order of preference; read what exists:
   - `docs/store/<app>/listing.md` (from `/play-store:listing`): title, short and full
     description per locale, keyword list.
   - `docs/apps/<app>/PRD.md` (android-factory): overview, problem statement, target
     user, success criteria.
   - `docs/store/<app>/screens.yml` and `captions.<locale>.yml`: the feature order and
     benefit-first headlines.
   - The app module's `README.md`, if any.
3. Technologies from the build: read the app module's `build.gradle(.kts)` and the version
   catalog (`gradle/libs.versions.toml`). Map, keep at most 6, `Android` always first:

   | Dependency contains | Tag |
   |---|---|
   | `kotlin` plugin / `.kt` sources | Kotlin |
   | `compose` | Jetpack Compose |
   | `room` | Room |
   | `hilt` / `koin` / `dagger` | Hilt / Koin / Dagger |
   | `retrofit` / `ktor` | Retrofit / Ktor |
   | `firebase` | Firebase |
   | `play-services-ads` / `admob` | AdMob |
   | `billing` | Play Billing |
   | `sqldelight` / `realm` | SQLDelight / Realm |
   | `workmanager` / `work-runtime` | WorkManager |

   Order: language, UI, data, then services. Drop anything the entry's text doesn't touch.
4. Link: `https://play.google.com/store/apps/details?id=<applicationId>`. If the PRD or
   README names a landing page for the app, prefer that and put the Play link in the body.
5. Cover image, first hit wins:
   - `--image <path>`
   - `docs/store/<app>/out/<default_locale>/featureGraphic.*` (1024x500, landscape, same
     shape as the site's existing 1568x744 covers)
   - `docs/store/<app>/out/<default_locale>/01-*.jpg` (portrait; the site handles it, one
     existing entry uses a 540x960 shot)
   - none: write the entry without `image:` and say so.

   Copy as `public/images/portfolio/<slug>.<ext>` keeping the original extension. Don't
   re-encode; there's no image library in the site repo.

## Step 2 — Slug and existing entry

Slug: `--slug`, else the last segment of `applicationId` in kebab-case
(`com.gookkis.siaptbslpdp` -> `siaptbslpdp`), unless the app name gives a clearer one
(`Siap TBS LPDP` -> `siap-tbs-lpdp`). Propose it and confirm on the first publish.

Existing entry: grep every `<collection>/*/*.md` for `link:` containing the
`applicationId`. If found, that file's name is the slug (ignore `--slug`), and this run is
an **update**: keep `featured`, `draft`, and `image` unless a flag or a new image says
otherwise; keep the body unless the sources changed since the entry's last git commit
(`git log -1 --format=%cI -- <file>` vs the mtime of `listing.md` / `PRD.md`).

## Step 3 — Draft both locales

Read two existing entries in each locale first (the newest by git date) to match tone,
length, and how `problem`/`solution` are phrased there.

Per locale:

- `title`: the app name as users see it (no descriptor suffix from the store title).
- `description`: one sentence, 120-200 characters, what it is + for whom + the one
  differentiator. Shown on cards.
- `technologies`: from Step 1.3.
- `link`, `type: "mobile"`, `image`, `featured`, `draft`.
- `problem`: 2-3 sentences. Who has what problem, why existing options fall short. From
  the PRD's problem statement; if the PRD has none, ask the user one question:
  "Masalah apa yang app ini selesaikan, dan kenapa solusi yang ada tidak cukup?"
- `solution`: 2-3 sentences starting with what was built ("Membangun aplikasi Android
  ..." / "Built an Android app ..."), the 3-4 capabilities that answer the problem,
  and the constraint that shaped it (offline, no account, local data, ...).
- Body: 1-3 short paragraphs. What's inside in more detail, how it's distributed
  (Play Store, downloadable packs, ...), disclaimers (unofficial, not affiliated). No
  headings; the page already has them.
- If `src/pages/apps/<slug>/privacy-policy.astro` exists in the site, end the body with
  the privacy link line the existing entries use:
  `Kebijakan privasi aplikasi ini tersedia di [halaman terpisah](/apps/<slug>/privacy-policy).`
  and its English counterpart under `/en/apps/<slug>/privacy-policy`. If it doesn't
  exist yet, mention `/portfolio:privacy <app>` in the report.

Show both files in full, plus the image choice and the slug. Wait for a yes, or for
edits. `--dry-run` stops here.

## Step 4 — Write and validate

1. Write `<collection>/<locale>/<slug>.md` for every locale, and copy the image.
2. Schema check without a build: for each file, parse the frontmatter and compare with the
   Zod fields read in Step 0: required fields present, `type` in the enum, `link` a URL,
   `technologies` an array, no unknown keys. Report per file.
3. Build check: if `node_modules/` exists in the site repo, run `npm run build` and treat
   any error as a failure to fix before committing. If `node_modules/` is missing, say
   `npm ci` is needed for a real build, offer to run it (it's local and reversible), and
   if declined skip the build with a clear "not built" in the report.
4. `git status --short` must show only the expected files: 2 Markdown files (or as many
   as locales) and at most 1 image.

## Step 5 — Commit, then optionally push

Commit in the site repo, message in the repo's existing style:

```
Add <Title> portfolio entry (ID+EN)
```
or `Update <Title> portfolio entry (ID+EN)` for an update. Body: one line on the image
source and one on where the text came from. Append the attribution lines the session
provides.

- default: commit on the current branch, no push. Print
  `cd <site> && git push` as the next step and say it deploys.
- `--push`: state "this deploys to <site_url>", then `git push`.
- `--pr`: `git checkout -b portfolio/<slug>` before committing, `git push -u origin
  portfolio/<slug>`, `gh pr create --fill`, print the PR URL, `git checkout <branch>`.

## Step 6 — Report

Table: locale, file, characters in `description`, validation result. Then the image
path, the commit hash, and what was or wasn't pushed. Live URL once deployed:
`<site_url>/portfolio/<slug>` and `<site_url>/en/portfolio/<slug>`.

## Re-runs

Idempotent: the same app maps to the same entry (by `applicationId` in `link`). Re-run
after `/play-store:listing` changes to refresh the text; the diff is shown before anything
is overwritten.
