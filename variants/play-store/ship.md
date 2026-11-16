---
description: After the owner has tried and approved the app, run the whole store pipeline in order — signing, privacy page, AdMob, listing, phone + 7" + 10" captures, Console, release, in-app products — stopping only where the owner must act
allowed-tools: all
---

# Ship — capture to release, in one pass

The store half of a new app, after `/android-factory:build-app` handed it over and the owner
said it is good. Each step is its own command; this one fixes the order (each step unblocks
the next), carries the state between them, and keeps going without asking — invoking
`/play-store:ship` is the owner's approval for every step below, including
`/play-store:release`. Updates to a live app use `/play-store:release` directly.

## Arguments

| Argument | Meaning |
|---|---|
| `<app>` | Required. |
| `--from <step>` | Resume at a step (`signing`, `privacy`, `admob`, `listing`, `capture`, `enhance`, `console`, `release`, `products`). Default: the first step whose output is missing. |
| `--until <step>` | Stop after that step. |
| `--track internal\|closed` | Passed to `/play-store:release`. Default `internal`. |
| `--ask` | Normal behaviour: each sub-command asks where it would ask. |

## Rules

- Autonomy as in `/android-factory:build-app` ("The autonomy contract"): recommended options,
  decisions recorded in `docs/store/<app>/console.md` under "Decisions taken without asking".
- **Owner-only stops** — explain and give the exact action, then continue with whatever does
  not depend on it:
  - Chrome not signed in, or signed in to an account that is not the publisher's (AdMob
    shows a sign-up page, Console shows no developer account). Never sign in or sign up.
  - A git push or other action the session's permission system denies: print the exact
    command for the owner (`! git -C <repo> push origin main`); do not work around it.
  - A bundle over 10 MB (`/play-store:release` Step 3.4).
  - Declarations with legal weight that `console.md` cannot answer from the code (trader
    status, a question new to the Console): ask once, with the recommended answer.
- Run `/play-store:doctor` first; a missing tool is fixed or reported before anything starts.

## Order

| # | Step | Command | Done when |
|---|---|---|---|
| 1 | Upload key | `/play-store:signing <app>` | release AAB fingerprint = the app's key |
| 2 | Privacy page + Data safety | `/portfolio:privacy <app>` → push the site → wait for the deploy | both locale URLs return **200** (with the trailing slash the app uses) and `data-safety.md` exists |
| 3 | AdMob | `/play-store:admob <app> --gdpr` | real ids in the release manifest; app attached to the published consent message. Skip when the app has no ads |
| 4 | Listing + answer sheet | `/play-store:listing <app>` | fastlane text per locale within budget; `console.md`, `changelogs.txt` written |
| 5 | Captures | `/play-store:capture <app>` then `--tablet 7` and `--tablet 10` | `raw/`, `raw-tablet7/`, `raw-tablet/` per locale, every image opened and checked |
| 6 | Store graphics | `/play-store:enhance <app>` then `--tablet 7`, `--tablet 10`, `--feature` | `out/` 1080×1920, `out-tablet7/` 1080×1920, `out-tablet/` 2560×1440, feature 1024×500; captions match the screens; re-run `/play-store:listing` to copy them into fastlane |
| 7 | Console | `/play-store:console <app>` | "Set up your app" gone; every App content item Actioned |
| 8 | Release | `/play-store:release <app> --track internal` | release available to testers (or draft + owner hand-off for >10 MB) |
| 9 | Products | `/play-store:release <app> --products` | each product the code sells is Active at its price |
| 10 | Record | — | commit store assets and docs; push to `git.org` as `git.visibility` on `git.branch` after a secret scan (no `.jks`, credentials, `local.properties`, real ad ids in tracked files); project memory/progress updated |

Rebuild the bundle after step 3 — anything built before the AdMob ids carries test ids.

## Report

One table: step → result → ids/paths → owner action left. Then the road to production:
closed test (12 testers, 14 days when the Console requires it) → production release with
`/play-store:release <app> --track production`, then link AdMob to the live listing and run
`/play-store:aso <app>` once there is install data.
