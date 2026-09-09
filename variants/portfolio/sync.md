---
description: Compare the Android apps with the site's portfolio entries and report what's missing or stale (read-only)
allowed-tools: Read, Glob, Grep, Bash(date*), Bash(ls*), Bash(cat*), Bash(git*), Bash(test*), Bash(stat*), Bash(cmp*), Bash(md5sum*), Bash(sha1sum*)
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
- Matching is by `applicationId` inside the entry's `link:` or `hero.actions[].url`. An
  unreleased app has no Play URL yet, so a second, weaker match is allowed: the entry's
  filename equals the slug `/portfolio:publish` would derive, or its `image` / `hero.icon`
  live under `<images>/<that slug>/`. A title match alone is only a "maybe" and is
  reported as such; compare `app_name` with the title's part before any `:` too
  (`Convrt: Konverter Satuan` is `Convrt`).
- Template modules are not apps: skip any include whose path contains `template` or whose
  `applicationId` ends in `templateapp`, and list them on a `Skipped:` line so the skip is
  visible.
- Run `date +%Y-%m-%d` for the date.

## Step 1 — Inventory

1. Android side: every include in `settings.gradle(.kts)` whose build file applies the
   Android application plugin (the project's own convention plugin id counts, e.g.
   `appfactory.android.application`) -> Gradle path, `applicationId`, `app_name`. For
   each, the newest of: `docs/store/<app>/listing.md`, `docs/apps/<app>/PRD.md`,
   `docs/store/<app>/out/<default_locale>/featureGraphic.*`,
   `src/main/ic_launcher-playstore.png` (git commit **timestamp**, `--format=%ci`, if
   tracked, else file mtime). Dates alone are not enough: an entry and its sources
   usually land on the same day, and only the time says which came last.
2. Site side: every `<collection>/<default_locale>/*.md` -> slug, `title`, `link`,
   `type`, `draft`, `featured`, `image`, last commit timestamp
   (`git -C <site> log -1 --format=%ci -- <file>`), whether the same filename exists
   in every other locale, and which of the optional landing-page sections (`hero`,
   `screenshots`, `features`, `stats`, `highlights`, `note`) the entry actually sets.
3. Privacy, per slug: whether `src/pages/apps/<slug>/privacy-policy.astro` exists in
   every locale, and whether the entry links it (a `privacy-policy` line in the body or
   `highlights.action` pointing at it).
4. Icon, per matched pair: compare the hash of `<images>/<slug>/icon.png` with the app's
   `ic_launcher-playstore.png`. A launcher redesign after publishing leaves the site
   showing the old icon with no date to reveal it.

## Step 2 — Compare

For each app:

| State | Meaning |
|---|---|
| `missing` | No entry matched by `applicationId`, slug, or image folder. |
| `maybe` | No id or slug match, but an entry title equals `app_name` (case-insensitive, before any `:`). Needs a human look. |
| `stale` | Entry exists and its last commit is older than the newest Android source from Step 1.1. |
| `basic` | Entry exists and is current, but sets no landing-page section, while the app has captures or a listing that could fill them. |
| `draft` | Entry exists with `draft: true`. A slug-only match with no `link` is the normal shape of an unreleased app and is reported here, not as `maybe`; the note says "no link yet". |
| `ok` | Entry exists, is newer than its sources, all locales present. |

Also flag, per entry: a locale file missing (`id/x.md` without `en/x.md` or vice
versa), a locale whose sections differ from the default locale's (one language richer
than the other), any `image`, `hero.icon`, or `screenshots[].src` path that doesn't
resolve to a file under `public/`, `link` pointing at a package that no longer exists in
`settings.gradle`, an icon hash that differs from the app's Play icon, no privacy page
for the slug, and a privacy page the entry does not link.

## Step 3 — Report

```
Portfolio sync — <date>
Android project: <root>      Site: <site path> (<branch>, <last commit date>)

App            applicationId              Entry            State    Sections            Privacy        Note
:tokoku        com.gookkis.tokoku         —                missing  —                   —              /portfolio:publish tokoku
:siaptbslpdp   com.gookkis.siaptbslpdp    tbs-lpdp         stale    hero stats note     page, linked   listing.md 2026-09-07 > entry 2026-09-03
:warungku      com.gookkis.warungku       warungku         basic    —                   no page        8 captures available, none used
:apps:convrt   com.gookkis.convrt         convrt           draft    hero shots features page, unlinked no link yet (unreleased); matched by slug
:kasirku       com.gookkis.kasirku        kasirku          ok       hero shots features page, linked

Skipped: :template-app (template module)

Entries without an Android app (expected for non-Android work):
  dodolanan (web) · gookkis-cms (web) · istiqomah (mobile, iOS+Android — no link match: the link is a landing page)

Problems:
  tpa-verbal: en/tpa-verbal.md missing
  dodolanan: hero.icon /images/portfolio/dodolanan/icon.png not found
  kasirku: icon.png differs from apps/kasirku/src/main/ic_launcher-playstore.png
```

End with the commands to run, one per `missing`, `stale`, or `basic` app, plus
`/portfolio:privacy <app>` for each app without a page, `/portfolio:privacy <app>
--link-entry` for each unlinked page, and `/portfolio:publish <app>` for a changed icon.
