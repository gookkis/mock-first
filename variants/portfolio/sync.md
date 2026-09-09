---
description: Compare the Android apps with the site's portfolio entries and report what's missing or stale (read-only)
allowed-tools: Read, Glob, Grep, Bash(date*), Bash(ls*), Bash(cat*), Bash(git*), Bash(test*), Bash(stat*)
---

# Portfolio sync — what the site doesn't know yet

Read-only. Lists every app module in the Android project next to its portfolio entry in
the Astro site and says which apps have no entry, which entries are older than their
sources, and which entries have no matching app (those are fine: web, iOS, and Laravel
projects live on the site too; they're listed so nothing is silently ignored).

## Arguments

| Argument | Meaning |
|---|---|
| empty | Every app module in `settings.gradle(.kts)`. |
| `<app>` | Only this app. |
| `--site <path>` | Site repo. Default: `site:` in `~/.claude/portfolio.yml`. |

## Absolute rules

- Writes nothing, in either repo.
- Matching is by `applicationId` inside the entry's `link:`; a name match alone is only a
  "maybe" and is reported as such.
- Run `date +%Y-%m-%d` for the date.

## Step 1 — Inventory

1. Android side: every include in `settings.gradle(.kts)` whose build file applies the
   Android application plugin -> Gradle path, `applicationId`, `app_name`. For each, the
   newest of: `docs/store/<app>/listing.md`, `docs/apps/<app>/PRD.md`,
   `docs/store/<app>/out/<default_locale>/featureGraphic.*` (git commit date if tracked,
   else file mtime).
2. Site side: every `<collection>/<default_locale>/*.md` -> slug, `title`, `link`,
   `type`, `draft`, `featured`, `image`, last commit date
   (`git -C <site> log -1 --format=%cs -- <file>`), and whether the same filename exists
   in every other locale.

## Step 2 — Compare

For each app:

| State | Meaning |
|---|---|
| `missing` | No entry whose `link` contains the `applicationId`. |
| `maybe` | No link match, but an entry title equals `app_name` (case-insensitive). Needs a human look. |
| `stale` | Entry exists and its last commit is older than the newest Android source from Step 1.1. |
| `draft` | Entry exists with `draft: true`. |
| `ok` | Entry exists, is newer than its sources, all locales present. |

Also flag, per entry: a locale file missing (`id/x.md` without `en/x.md` or vice
versa), `image:` pointing at a file that doesn't exist under `public/`, and `link`
returning to a package that no longer exists in `settings.gradle`.

## Step 3 — Report

```
Portfolio sync — <date>
Android project: <root>      Site: <site path> (<branch>, <last commit date>)

App            applicationId              Entry            State    Note
:tokoku        com.gookkis.tokoku         —                missing  /portfolio:publish tokoku
:siaptbslpdp   com.gookkis.siaptbslpdp    tbs-lpdp         stale    listing.md 2026-09-07 > entry 2026-09-03
:warungku      com.gookkis.warungku       warungku         ok

Entries without an Android app (expected for non-Android work):
  dodolanan (web) · gookkis-cms (web) · istiqomah (mobile, iOS+Android — no link match: the link is a landing page)

Problems:
  tpa-verbal: en/tpa-verbal.md missing
```

End with the commands to run, one per `missing`/`stale` app.
