---
description: Add or update one app's portfolio entry (id + en) in the Astro site from the Android project
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(date*), Bash(mkdir*), Bash(ls*), Bash(cat*), Bash(cp*), Bash(test*), Bash(node*), Bash(npm*), Bash(npx*), Bash(git*), Bash(gh*)
---

# Portfolio publish — from an app module to a site entry

Reads one app module of the Android project plus whatever `/play-store` already produced,
and writes the matching portfolio entry into the Astro site: one Markdown file per locale
(same filename in `id/` and `en/`), an app icon, and a set of screenshots. Validates
against the site's content schema, builds the site, commits. Pushing is opt-in because a
push to `main` deploys straight to production.

The site renders a portfolio entry as a landing page: hero (icon, title, tagline, action
buttons, note line, tech chips, phone mockup) -> problem/solution -> screenshot scroller
-> feature grid -> stat cards -> checklist highlights -> disclaimer box -> Markdown body.
Every section past the hero is optional frontmatter and is skipped when absent, so a
minimal entry still renders correctly.

## Arguments

| Argument | Meaning |
|---|---|
| `<app>` | Required. Gradle path (`:tokoku`) or app name (`tokoku`). |
| `--site <path>` | Site repo. Default: `site:` in `~/.claude/portfolio.yml`. |
| `--slug <slug>` | Entry filename and image folder. Default: derived (see Step 2). |
| `--basic` | Write only the base fields (no hero/screenshots/features/stats/highlights/note). |
| `--shots <n>` | How many screenshots to include. Default: all that passed capture, capped at 8. |
| `--image <path>` | Card thumbnail, overriding the auto-picked one. |
| `--featured` / `--no-featured` | Set `featured:`. Default: keep existing, `false` for a new entry. |
| `--draft` | Write with `draft: true`. |
| `--push` | After committing on `main`, push. **This deploys to production.** |
| `--pr` | Commit on branch `portfolio/<slug>`, push it, open a PR with `gh`. |
| `--dry-run` | Show everything that would be written, write nothing. |

Without `--push` or `--pr`: files written, site built, changes committed on the current
branch of the site repo, not pushed.

## Absolute rules

- **Never touch site code.** Only the entry Markdown files and images under the slug's
  own folder are written. No `.astro`, no config, no other entry, no shared image.
- **Read the schema from the site, don't assume it.** Parse the `portfolio` collection in
  `src/content.config.ts` (or `src/content/config.ts`) at run time and produce exactly its
  fields. If it has a field this command doesn't map, name it in the report and ask
  whether to fill it rather than silently skipping it.
- **Same filename in every locale.** The language switcher maps `id/x.md` to `en/x.md` by
  name. Never write one locale without the other.
- **Two languages, two texts.** `en/` is written in English by you, not a literal
  translation. Same facts, natural phrasing. Section headings too.
- **No marketing copy.** The site is a personal portfolio in the first person's voice
  (read two existing entries first to match it). Store-listing text is a source of facts,
  not text to paste. No "best", no "#1", no exclamation marks.
- **Never invent a number.** `stats` values come from the PRD, the code, or the user. If
  a number can't be traced to one of those, leave the whole section out.
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
images: public/images/portfolio        # per-slug subfolder; URLs are /images/portfolio/<slug>/...
locales: [id, en]
default_locale: id
branch: main
site_url: https://gookkis.com
```

If the file is missing, ask for the site path, detect the rest from the repo (the `base:`
of the portfolio collection, the image path used by existing entries, the locale
subfolders, `site:` in `astro.config.mjs`), show it, and write the file.

Then in the site repo: `git status --short`. Dirt that touches the files this command
writes (the slug's entry files or image folder) is a stop: say what's dirty and let the
user resolve it. Dirt anywhere else (another session's edits, an unrelated deletion, an
untracked file) is left alone: stage this command's files by path, never `git add -A`,
and name the leftover dirt in the report. Note the branch.

## Step 1 — Gather from the Android project

1. Resolve the app module (as in `/play-store:capture` Step 1): Gradle path,
   `applicationId`, `app_name` from `strings.xml`, `versionName`.
2. Sources, in order of preference; read what exists:
   - `docs/store/<app>/listing.md` (from `/play-store:listing`): title, short and full
     description per locale, keyword list. The full description's feature blocks map
     onto `features.items`.
   - `docs/apps/<app>/PRD.md` (android-factory): overview, problem statement, target
     user, success criteria, any concrete numbers.
   - `docs/store/<app>/screens.yml` and `captions.<locale>.yml`: screen order, screen
     labels, and the benefit headlines.
   - `docs/store/<app>/data-safety.md` (from `/portfolio:privacy`): the confirmed data
     inventory, which is exactly what `highlights` should say.
   - The app module's `README.md`, if any.
3. Technologies from the build: read the app module's `build.gradle(.kts)` and the version
   catalog (`gradle/libs.versions.toml`). Map, keep at most 6, `Android` first unless the
   app is also on another platform (then list the languages, as the Istiqomah entry does):

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
   | `workmanager` / `work-runtime` | WorkManager |

4. Links: the Play Store URL is
   `https://play.google.com/store/apps/details?id=<applicationId>`. If the PRD or README
   names a landing page for the app, that's a second action button, not a replacement.
   An app that is not on Play yet (no `versionCode` bump past 1, the release checklist
   still open, or the user says so) gets **no** store URL anywhere in the entry: a link
   that 404s is worse than none. The entry is then written with `draft: true`, `link`
   and the store action are omitted, and the report says both must be added at release.

## Step 2 — Slug and existing entry

Slug: `--slug`, else the last segment of `applicationId` in kebab-case, unless the app
name gives a clearer one (`Siap TBS LPDP` -> `tbs-lpdp`). Propose it and confirm on the
first publish. It names both the Markdown files and the image folder.

Existing entry: grep every `<collection>/*/*.md` for the `applicationId` (it appears in
`link:` or in a `hero.actions` URL). An unreleased entry carries no URL, so also check
for `<collection>/<default_locale>/<derived slug>.md` and the image folder
`<images>/<derived slug>/`. If found either way, that file's name is the slug (ignore
`--slug`), and this run is an **update**: keep `featured`, `draft`, and any section the
sources haven't changed; show a diff before writing. Never create a second entry for an
app that already has one under another name.

## Step 3 — Images

The site draws its own phone frame around each screenshot and renders its own captions
underneath, so **screenshots come from `docs/store/<app>/raw/`, never from `out/`**. The
`out/` images already carry a burned-in headline and a drawn device frame; using them
would double both.

| Site file | Source | Format |
|---|---|---|
| `<images>/<slug>/icon.png` | `ic_launcher-playstore.png` in the app module (512x512), else the largest `mipmap-xxxhdpi/ic_launcher*.png` | PNG, kept square |

A vector-only adaptive icon (minSdk 26 projects, android-factory among them) has neither
file. Do not draw one by hand from the drawables inside this command. If the Android
project has `docs/store/_tools/icon.mjs` (android-factory renders the Play PNG from the
adaptive-icon drawables with it: `node icon.mjs <res dir> <out.png> <preview.png>`), run
it and commit the PNG at `src/main/ic_launcher-playstore.png` in the app module so the
next run finds it; otherwise stop and ask the user to export the Play Store icon from
Android Studio's Image Asset wizard to that path.
| `<images>/<slug>/<NN>-<id>.webp` | `docs/store/<app>/raw/<default_locale>/<NN>-<id>.png` | WebP, 540 px wide, height proportional |
| card thumbnail (`image:`) | `docs/store/<app>/out/<default_locale>/featureGraphic.*`, else `--image` | as-is |

Notes that matter:

- The phone frame is `height: auto`, so **any aspect ratio works**. Don't crop. Existing
  entries hold both 9:16 (540x960) and 9:20 (480x1067) shots. Downscale only.
- Target under 40 KB per screenshot. At 540 px wide, WebP quality 80 lands there.
- The card thumbnail is rendered `object-cover` in a 16:9 box on the index page. The
  1024x500 feature graphic fits that almost exactly. A square icon used here gets its top
  and bottom cropped, so prefer the feature graphic; if only an icon exists, say so in the
  report and let the user decide.
- Keep the raw capture order. `screenshots.items[0]` becomes the hero phone mockup and the
  rest become the scroller, so the first screenshot must be the app's main screen.

### Converting to WebP

The site repo has no image library, so conversion runs in the Android project where
Playwright already lives (installed by `/play-store:doctor --fix`). If
`docs/store/_tools/webp.mjs` is missing, write it exactly:

```js
// Downscale images to a target width and re-encode as WebP, preserving aspect ratio.
// usage: node webp.mjs <outDir> <width> <quality 0-100> <input...>
import { chromium } from 'playwright';
import fs from 'node:fs';
import path from 'node:path';

const [outDir, widthArg, qualityArg, ...inputs] = process.argv.slice(2);
if (!inputs.length) { console.error('usage: node webp.mjs <outDir> <width> <quality> <input...>'); process.exit(1); }
const width = Number(widthArg), quality = Number(qualityArg) / 100;

const dataUri = p => {
  const ext = path.extname(p).slice(1).toLowerCase();
  const mime = ext === 'jpg' ? 'image/jpeg' : `image/${ext}`;
  return `data:${mime};base64,${fs.readFileSync(p).toString('base64')}`;
};

const browser = await chromium.launch();
const page = await browser.newPage();
try {
  fs.mkdirSync(outDir, { recursive: true });
  for (const input of inputs) {
    const out = path.join(outDir, path.basename(input).replace(/\.[^.]+$/, '.webp'));
    const b64 = await page.evaluate(async ({ src, width, quality }) => {
      const img = new Image();
      img.src = src;
      await img.decode();
      const h = Math.round((img.naturalHeight / img.naturalWidth) * width);
      const c = document.createElement('canvas');
      c.width = width; c.height = h;
      c.getContext('2d').drawImage(img, 0, 0, width, h);
      return c.toDataURL('image/webp', quality).split(',')[1];
    }, { src: dataUri(input), width, quality });
    fs.writeFileSync(out, Buffer.from(b64, 'base64'));
    console.log(`${out} ${(fs.statSync(out).size / 1024).toFixed(0)}K @${width}px`);
  }
} finally {
  await browser.close();
}
```

Run it into a scratch folder, check the reported sizes, then copy the results into the
site. If Playwright is unavailable and ImageMagick is, `magick <in> -resize 540x -quality
80 <out>.webp` is the fallback; if neither exists, copy the PNGs unconverted, say so in
the report, and note they are several times larger than the site's other images.

## Step 4 — Draft both locales

Read two existing entries in each locale first (the newest by git date) to match tone,
length, and how the sections are phrased. Then draft, per locale.

### Base fields (always)

- `title`: the app name as users see it.
- `description`: one sentence, 120-200 characters, what it is + for whom + the one
  differentiator. Shown on the index card.
- `technologies`: from Step 1.3.
- `type: "mobile"`, `image`, `featured`, `draft`.
- `link`: the app's landing page if it has one, else the Play Store URL. It is the
  fallback CTA when `hero.actions` is absent, so set it whenever the URL is live. For an
  unreleased app (Step 1.4) omit it; the schema allows that and the page renders without a
  button.
- `problem`: 2-3 sentences. Who has what problem, why existing options fall short.
- `solution`: 2-3 sentences starting with what was built, the 3-4 capabilities that
  answer the problem, and the constraint that shaped it.

`--basic` stops here. Everything below is optional and omitted entirely when its source
is missing; never emit an empty section.

### `hero`

```yaml
hero:
  icon: "/images/portfolio/<slug>/icon.png"
  note: "Gratis · Tanpa akun · Bisa dipakai offline"
  actions:
    - label: "Unduh di Google Play"
      url: "https://play.google.com/store/apps/details?id=<applicationId>"
    - label: "Kunjungi situs aplikasi"
      url: "<landing page>"
      variant: "secondary"
```

`note` is three or four short claims joined by `·`, each one true per the data inventory
(free, no ads, no account, works offline). Drop the ones that don't hold. Omit
`actions` entirely if the app has only the store link; the bare `link` renders one button
on its own. For an unreleased app omit `actions` too, unless a landing page exists.

### `screenshots`

`title` and `subtitle` are written in the entry's own voice; when omitted the site falls
back to a generic heading. Per item: `src` is the site path, `alt` describes the screen
for someone who can't see it, `title` is the screen label from `screens.yml`, `caption`
is one short sentence from `captions.<locale>.yml`.

```yaml
screenshots:
  title: "Lihat tampilan aplikasinya"
  subtitle: "<one line on what the screens show>"
  items:
    - src: "/images/portfolio/<slug>/01-beranda.webp"
      alt: "Tampilan beranda checklist harian <app>"
      title: "Beranda"
      caption: "Checklist hari ini, sekali ketuk."
```

`alt` and `caption` must differ: `alt` names what is on screen, `caption` says why it
matters. Cap at 8 items (`--shots`).

### `features`

From the feature blocks of the full description, in the same order as the screenshots so
text and images tell one story. Each item gets one emoji `icon`, a 2-4 word `title`, and
one sentence `description`. Between 4 and 6 items; fewer than 3 isn't worth a grid.

### `stats`

Only when the PRD or the code gives real numbers (question counts, durations, surah
counts, supported cities). `value` carries the number, `unit` the qualifier, `accent`
cycles through `primary`, `violet`, `teal`, `amber`. If the numbers are estimates, say so
in `subtitle`. **No source, no section.**

### `highlights`

The privacy and constraints checklist. When `docs/store/<app>/data-safety.md` exists, take
the facts from there so the entry and the privacy page can't contradict each other; other-
wise derive them the same way `/portfolio:privacy` Step 2 does and say in the report that
they're unverified. Each item is a short claim as `title` plus one clause of detail as
`description`. `action` links to the app's privacy policy page:
`<site_url>/apps/<slug>/privacy-policy` when `/portfolio:privacy` has created it.

### `note`

Disclaimers only: not an official app, not affiliated, figures are estimates, results not
guaranteed. `body` is a list of paragraphs, and it is rendered as HTML, so `<strong>` and
`<a href>` are allowed and anything else should be avoided. Omit for an app with nothing
to disclaim.

### Body

1-3 short paragraphs after the frontmatter: detail that doesn't fit a section, how it's
distributed, what's planned. No headings; the page provides them. If
`src/pages/apps/<slug>/privacy-policy.astro` exists and `highlights.action` doesn't
already point at it, end with the site's usual line, copied per locale from an entry
that already has one (`grep -rl privacy-policy <collection>`); the current ones are
`Kebijakan privasi aplikasi ini tersedia di [halaman terpisah](/apps/<slug>/privacy-policy).`
and `The app's privacy policy lives on [its own page](/en/apps/<slug>/privacy-policy).`
If the page doesn't exist yet, say in the report that `/portfolio:privacy <app>
--link-entry` adds the line later.

Show both files in full, plus the image list and the slug. Wait for a yes, or for edits.
`--dry-run` stops here.

## Step 5 — Write and validate

1. Write `<collection>/<locale>/<slug>.md` for every locale; copy the icon and the
   converted screenshots into `<images>/<slug>/`.
2. Schema check without a build: parse each file's frontmatter and compare with the Zod
   fields read in Step 0. Required fields present, `type` in the enum, `link` a URL,
   `technologies` an array, every `variant` and `accent` in its enum, every `src` and
   `icon` path pointing at a file that now exists under `public/`. Report per file.
3. Build check: if `node_modules/` exists in the site repo, run `npm run build` and treat
   any error as a failure to fix before committing. If it's missing, say `npm ci` is
   needed for a real build, offer to run it, and if declined skip the build with a clear
   "not built" in the report.
4. `git status --short` must show the expected files: the locale Markdown files and the
   slug's own image folder. Anything else present must be the dirt noted in Step 0, and
   stays unstaged.

## Step 6 — Commit, then optionally push

Commit in the site repo, message in the repo's existing style:

```
Add <Title> portfolio entry (ID+EN)
```
or `Update <Title> portfolio entry (ID+EN)`. Body: one line on where the images came
from, one on where the text came from. Append the attribution lines the session provides.

- default: commit on the current branch, no push. Print `cd <site> && git push` as the
  next step and say it deploys.
- `--push`: state "this deploys to <site_url>", then `git push`.
- `--pr`: `git checkout -b portfolio/<slug>` before committing, `git push -u origin
  portfolio/<slug>`, `gh pr create --fill`, print the PR URL, `git checkout <branch>`.

## Step 7 — Report

Table of locale, file, `description` length, sections written, validation result. Then the
image list with file sizes, the commit hash, and what was or wasn't pushed. Live URLs:
`<site_url>/portfolio/<slug>` and `<site_url>/en/portfolio/<slug>`. Finally, anything
left open: sections skipped for lack of a source, a schema field with no mapping, or a
card thumbnail that had to fall back to the icon.

## Re-runs

Idempotent: the same app maps to the same entry (by `applicationId`). Re-run after
`/play-store:listing`, `/play-store:capture`, or `/portfolio:privacy` changes to refresh
the affected sections; the diff is shown before anything is overwritten.
