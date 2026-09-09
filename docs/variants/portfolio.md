# Portfolio — publish an app to the Astro site

> **Kenapa berkas ini tidak ada di dalam `variants/<nama>/`?** Claude Code memperlakukan
> **setiap** `.md` di folder command sebagai sebuah command — sebuah `README.md` di sana akan
> muncul sebagai `/<nama>:README` di daftar skill. Dokumentasi varian karena itu hidup di
> `docs/variants/`. Jangan dipindahkan kembali.


Companion to the [Play Store variant](play-store.md). Once an app has a listing and
captured screens, one command turns them into a portfolio entry on a personal Astro site:
one Markdown file per locale plus the app's icon and screenshots, validated against the
site's own content-collection schema, committed in the site repo.

Built for a site with this shape (Astro content collections, one file per locale with
matching filenames, deploy on push to `main`):

```
src/content.config.ts                 <- portfolio schema, base fields plus the optional
                                         landing-page sections listed below
src/content/portfolio/id/<slug>.md
src/content/portfolio/en/<slug>.md
public/images/portfolio/<slug>/icon.png
public/images/portfolio/<slug>/01-<screen>.webp
```

The detail page renders as a landing page: hero (icon, title, tagline, action buttons,
note line, tech chips, phone mockup) -> problem/solution -> screenshot scroller -> feature
grid -> stat cards -> checklist highlights -> disclaimer box -> Markdown body. Base fields
are `title`, `description`, `technologies`, `link`, `type`, `problem`, `solution`,
`image`, `featured`, `draft`; the sections past the hero come from the optional `hero`,
`screenshots`, `features`, `stats`, `highlights`, and `note` objects and are skipped when
absent, so a minimal entry still renders.

Any Astro site with the same conventions works; the command reads the schema and paths
from the repo at run time rather than hardcoding them.

## Commands

| Command | What it does |
|---|---|
| `/portfolio:publish <app>` | Builds the full landing-page entry: base fields plus the `hero`, `screenshots`, `features`, `stats`, `highlights`, and `note` sections, sourced from `listing.md`, the PRD, Gradle deps, the raw captures, and the privacy data inventory. Converts screenshots to WebP and copies them with the app icon into the slug's image folder. Drafts both locales, shows them, writes on approval, validates against the schema, builds, commits. `--basic` writes base fields only, `--push` deploys, `--pr` opens a pull request, default leaves the commit local. |
| `/portfolio:privacy <app>` | Scans the app and its modules for permissions, SDKs (AdMob, Firebase, billing, HTTP clients), accounts, local storage, and network targets; shows that data inventory for confirmation; then writes `src/pages/apps/<slug>/privacy-policy.astro` in both locales following the site's existing per-app privacy page, and a Data Safety summary for the Play Console in `docs/store/<app>/data-safety.md`. The URL to paste into Play Console is `<site_url>/apps/<slug>/privacy-policy`. |
| `/portfolio:sync` | Read-only. Every app module vs every site entry: `missing`, `stale`, `basic`, `draft`, `ok`, which landing-page sections each entry sets, plus entries with a locale file missing, uneven sections between locales, or a broken image path. |

## Install
```bash
mkdir -p ~/.claude/commands/portfolio
cp *.md ~/.claude/commands/portfolio/
```

## Config

`~/.claude/portfolio.yml` (created on first run if missing):

```yaml
site: C:/Users/user/ClaudeProject/gookkis-web
collection: src/content/portfolio
images: public/images/portfolio
locales: [id, en]
default_locale: id
branch: main
site_url: https://gookkis.com
```

## Where the content comes from

| Entry field | Source in the Android project |
|---|---|
| `title` | `app_name` in `strings.xml` |
| `description` | short description in `docs/store/<app>/listing.md`, rewritten as one sentence |
| `technologies` | app module `build.gradle` + version catalog, mapped to tags (Android, Kotlin, Jetpack Compose, Room, ...) |
| `link` | the app's landing page if it has one, else the Play Store URL |
| `type` | `mobile` |
| `problem`, `solution` | `docs/apps/<app>/PRD.md`; one question to the user if the PRD has no problem statement |
| `hero.icon` | `ic_launcher-playstore.png` in the app module |
| `hero.actions` | Play Store URL, plus the landing page as a secondary button |
| `hero.note` | short true claims joined by `·`, from the data inventory (free, no ads, no account, offline) |
| `screenshots` | `docs/store/<app>/raw/<locale>/`, downscaled to WebP; `title` from `screens.yml`, `caption` from `captions.<locale>.yml` |
| `features` | the feature blocks of the full description, one emoji each |
| `stats` | numbers from the PRD or the code; the section is omitted when there is no source |
| `highlights` | `docs/store/<app>/data-safety.md` from `/portfolio:privacy`, so the entry and the policy page agree |
| `note` | disclaimers: unofficial, not affiliated, figures are estimates |
| body | detail that doesn't fit a section, rewritten in the site's first-person voice |
| `image` | `docs/store/<app>/out/<locale>/featureGraphic.*`, else `--image` |

Nothing is pasted from the store listing verbatim; the site's existing entries set the
tone, and the command reads two of them before drafting.

Screenshots come from `raw/`, not from the `out/` images that `/play-store:enhance`
produces. The site draws its own phone frame and renders its own captions, so the
marketing images would arrive framed and captioned twice. The phone frame is
height-agnostic, so any aspect ratio works and nothing is cropped.

## Safety

- Only the entry Markdown files and the slug's own image folder are ever written in the
  site repo.
- The site must have a clean working tree before the command starts.
- A push to `main` deploys to production, so pushing needs `--push` explicitly; `--pr`
  is the review path.
- An existing entry is never overwritten without showing the diff first.
