---
description: Cut a release in Play Console through the user's signed-in Chrome — upload the signed bundle to a track, release notes, testers, Send for review — then create the one-time in-app products the code sells. Also handles updates to a live app.
allowed-tools: Read, Edit, Glob, Grep, Bash(date*), Bash(cat*), Bash(ls*), Bash(cp*), Bash(mkdir*), Bash(grep*), Bash(git fetch*), Bash(git log*), Bash(git status*), Bash(*keytool*), Bash(sh ./gradlew*), Bash(bash gradlew*), Bash(./gradlew*), Bash(find*), Bash(du*), Bash(curl*)
---

# Release — from a signed bundle to "Changes in review"

`/play-store:console` stops before anything that publishes. This command is the other half,
and **running it is the owner's explicit go-ahead to publish**: upload the bundle, attach it
to a track, fill the release, and press **Send for review** (or **Save and publish** on
internal testing). Nothing else is implied: no payment profile, no account settings, no
new tester emails the owner did not give.

## Arguments

| Argument | Meaning |
|---|---|
| `<app>` | Required. Gradle path or module name. |
| `--track internal\|closed\|production` | Default: `internal` for an app with no release yet, `production` for an app that is live. |
| `--testers "<list name>"` | Tester list for internal/closed. Default: `testers_list` in `publisher.yml`. The list must already exist. |
| `--products` | Only Step 6 (in-app products). |
| `--no-review` | Fill and Save the release, but leave Send for review to the owner. |

## Absolute rules

- **Only a bundle that passed the checks in Step 1.** Signed with this app's upload key,
  real ad ids in the release manifest (when the app shows ads), `versionCode` above every
  code already on Play and in the upstream branch.
- **Never create a tester list, type tester emails, or change countries/pricing templates**
  beyond what Step 6 states. Those are the owner's.
- **Bundles over 10 MB cannot go through the browser upload tool.** Do not look for API
  credentials, service-account keys or other side doors, and do not try to shrink the bundle
  to fit. Prepare the draft release, then hand over the exact path and clicks (Step 3.4).
- **Verify every Save** and read the release page back after a reload.
- Follow the browser rules of `/play-store:console` Step 0 (Angular inputs, short JS calls,
  re-read after a timeout).

## Step 1 — Preflight (local)

1. `git fetch` and compare `versionCode` with the upstream branch and with
   `fastlane/metadata/android/*/changelogs/`: another machine may already have shipped the
   next codes. Bump past the highest (rebase first if upstream is ahead).
2. Release notes: `docs/store/<app>/changelogs.txt` and `changelogs.version` must match the
   bundle's `versionCode`; otherwise run `/play-store:listing <app> --notes` first. Every
   locale of the listing needs a note.
3. Build: `JAVA_HOME=<jbr> ./gradlew :<app>:bundleRelease`.
4. Signature: `keytool -printcert -jarfile <aab>` = the app's upload key (`/play-store:signing --verify`).
5. Ads: when the code loads ads, the release manifest's `APPLICATION_ID` is not Google's
   test id `ca-app-pub-3940256099942544~…` (see `/play-store:admob` Step 5). A bundle built
   before the AdMob ids were written still carries test ids — rebuild.
6. **Release smoke test** (skip only if `/android-factory:build-app` Step 7 ran on this exact
   build): install the release APK on an emulator (not the owner's phone — real ad ids),
   walk the main flow, `adb logcat -d | grep -E "FATAL|AndroidRuntime"` must be empty.
7. Size: `du -h <aab>`. Over 10 MB → Step 3.4 path.

## Step 2 — Console state

Open the app (`…/app/<id>/app-dashboard`). New app: "Set up your app" must be gone (all
App content Actioned, listing saved with phone **and** 7" and 10" screenshots); if not, run
`/play-store:console <app>` first. Note whether the account needs a closed test before
production (the dashboard says so: 12 testers for 14 days on newer personal accounts).

## Step 3 — Create the release

1. Track page: `testing/internal` (`…/tracks/internal-testing`), closed, or `production`
   → **Create new release**.
2. Internal/closed only, first time: **Testers** tab → tick the existing list → Save.
3. **Upload** → `find` the `input[type=file]` → `file_upload` the AAB → wait for the
   processing row to show the `versionCode`. Errors to stop on and report verbatim:
   wrong signing key, versionCode already used, missing permissions declaration.
4. Bundle over 10 MB: fill the rest (name, notes), **Save as draft**, then stop and give the
   owner: the absolute AAB path, the Console path (track → draft release → Upload → drag the
   file), and that you will continue with Step 4 once it shows the version.
5. Release name: default (`<versionCode> (<versionName>)`). Notes: paste
   `changelogs.txt` as is (all `<locale>` tags).
6. **Next** → review the summary: errors are stops; "devices no longer supported" must be
   **0** on an update (a new `uses-feature` without `required="false"` is the usual cause);
   the native debug-symbols warning is not a blocker.

## Step 4 — Publish

- Internal testing: **Save and publish** — it goes out to testers within minutes, no review.
- Closed / production: **Save**, then **Publishing overview** → **Send N changes for
  review** → confirm. Quick checks run first (up to ~15 min), then review (usually hours,
  up to 7 days). With managed publishing off, an approved production release rolls out
  immediately at the percentage set; tell the owner "Remove changes" in Publishing
  overview still cancels it while in review.
- `--no-review`: stop after Save and say so.

## Step 5 — Update to a live app (production)

Same as Steps 3–4, plus: if permissions, SDKs or data handling changed since the last
release, the privacy page and Data safety must change first (`/portfolio:privacy`, then
`/play-store:console <app> --update --only content`) — a mismatch is a common rejection.
Listing screenshots that show the old UI are not a blocker; mention them.

## Step 6 — One-time in-app products

Play refuses to create products until a bundle with the `com.android.vending.BILLING`
permission is uploaded to any track — so this runs after Step 3.

1. Grep the code for product ids (`ProductDetails`, `setProductId(`, constants such as
   `premium_lifetime`). Only create products the code queries.
2. `…/app/<id>/one-time-products` (Monetize → Products → In-app products) → **Create
   product**: product id = the code's id exactly (cannot be changed later), name and
   description from `docs/store/<app>/console.md` (or the PRD's monetisation section), one
   **purchase option** id `lifetime` (buy, not rent).
3. Price: base country and price from `console.md`; let other countries **auto-convert**
   (do not hand-set them). Save, then **Activate**. Read back: status Active, price.
4. A product with no written price is asked, not guessed.

## Step 7 — Report and record

Track, versionCode/name, bundle size and fingerprint, notes locales, devices lost, status
("Changes in review" / "Available to internal testers"), products (id, price, status). Write
the same into the project's release record (`docs/progress-report.md` one line, and
`docs/monetization-setup.md` in android-factory). Remaining owner actions, if any, as an
exact list. Next for a new app: closed test → production; then link AdMob to the store
listing once the app is live.
