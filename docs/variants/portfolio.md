# Portfolio — publish an app to the Astro site

> **Kenapa berkas ini tidak ada di dalam `variants/<nama>/`?** Claude Code memperlakukan
> **setiap** `.md` di folder command sebagai sebuah command — sebuah `README.md` di sana akan
> muncul sebagai `/<nama>:README` di daftar skill. Dokumentasi varian karena itu hidup di
> `docs/variants/`. Jangan dipindahkan kembali.


Companion to the [Play Store variant](../play-store/). Once an app has a listing and
marketing images, one command turns them into a portfolio entry on a personal Astro site:
one Markdown file per locale plus a cover image, validated against the site's own
content-collection schema, committed in the site repo.

Built for a site with this shape (Astro content collections, one file per locale with
matching filenames, deploy on push to `main`):

```
src/content.config.ts                 <- portfolio schema: title, description, technologies,
                                         link?, type, problem?, solution?, image?, featured, draft
src/content/portfolio/id/<slug>.md
src/content/portfolio/en/<slug>.md
public/images/portfolio/<slug>.jpg
```

Any Astro site with the same conventions works; the command reads the schema and paths
from the repo at run time rather than hardcoding them.

## Commands

| Command | What it does |
|---|---|
| `/portfolio:publish <app>` | Gathers title, description, technologies, Play Store link, problem/solution, body, and cover from the Android project (`listing.md`, PRD, Gradle deps, `/play-store:enhance` output). Drafts both locales, shows them, writes on approval, validates against the schema, builds, commits. `--push` deploys, `--pr` opens a pull request, default leaves the commit local. |
| `/portfolio:privacy <app>` | Scans the app and its modules for permissions, SDKs (AdMob, Firebase, billing, HTTP clients), accounts, local storage, and network targets; shows that data inventory for confirmation; then writes `src/pages/apps/<slug>/privacy-policy.astro` in both locales following the site's existing per-app privacy page, and a Data Safety summary for the Play Console in `docs/store/<app>/data-safety.md`. The URL to paste into Play Console is `<site_url>/apps/<slug>/privacy-policy`. |
| `/portfolio:sync` | Read-only. Every app module vs every site entry: `missing`, `stale`, `draft`, `ok`, plus entries with a locale file missing or a broken image path. |

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
| `link` | `https://play.google.com/store/apps/details?id=<applicationId>` |
| `type` | `mobile` |
| `problem`, `solution` | `docs/apps/<app>/PRD.md`; one question to the user if the PRD has no problem statement |
| body | full description + captions, rewritten in the site's first-person voice |
| `image` | `docs/store/<app>/out/<locale>/featureGraphic.*`, else the first enhanced screenshot, else `--image` |

Nothing is pasted from the store listing verbatim; the site's existing entries set the
tone, and the command reads two of them before drafting.

## Safety

- Only the two entry files and one image are ever written in the site repo.
- The site must have a clean working tree before the command starts.
- A push to `main` deploys to production, so pushing needs `--push` explicitly; `--pr`
  is the review path.
- An existing entry is never overwritten without showing the diff first.
