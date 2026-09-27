---
description: Create the AdMob app and one ad unit per format the code uses, through the user's signed-in Chrome, then wire the ids into local.properties and prove they reach the release build
allowed-tools: Read, Edit, Glob, Grep, Bash(date*), Bash(cat*), Bash(grep*), Bash(ls*), Bash(git check-ignore*), Bash(sh ./gradlew*), Bash(./gradlew*), Bash(find*), Bash(unzip*), Bash(*aapt2*)
---

# AdMob — app, ad units, and ids into the build

Creates the app in AdMob, one ad unit for each ad format the app's code actually loads,
and writes the ids where the build reads them. Driven through **Claude in Chrome** in the
browser where the user is already signed in to AdMob; nothing here asks for a password.

## Arguments

| Argument | Meaning |
|---|---|
| `<app>` | Required. Gradle path (`:apps:tokoku`) or module name (`tokoku`). |
| `--name "<AdMob app name>"` | Name shown in AdMob. Default: the store title from `docs/store/<app>/listing.md`, else `app_name`. |
| `--formats banner,interstitial,rewarded` | Override the formats detected from the code. |
| `--gdpr` | Also add the app to the existing GDPR consent message (Step 6). Needs a live privacy policy URL. |
| `--no-build` | Skip the release-build check in Step 5. |

## Absolute rules

- **Only formats the code loads.** An ad unit nobody requests is noise in the AdMob
  reports and a question at review. Detect them (Step 1); `--formats` overrides.
- **Never create a second AdMob app for the same package.** List the apps first (Step 2);
  if one exists, reuse it and only add the missing units.
- **Ids go in `local.properties` only**, which must be git-ignored. Never in a manifest,
  a source file, a committed properties file, or a core module. Debug builds must keep the
  Google test ids (the android-factory convention plugin pins them); do not add a way around it.
- **Never click an ad, and never load the release build with real ids on a physical
  device you own.** An emulator counts as a test device; a phone does not.
- The session's browser rules apply: choose the browser with the user if several are
  connected, work in a new tab, close it at the end.
- Run `date +%Y-%m-%d` for the date.

## Step 1 — Read the app

1. Resolve the module; read `applicationId` from its `build.gradle(.kts)`.
2. Formats in use: grep the app module (not core) for how ads are wired —
   `AdBanner(` / banner view → **banner**; `showInterstitial` / `recordAction()` feeding a
   frequency cap → **interstitial**; `showRewarded` / `rewardedReady` / `AdFreePass` →
   **rewarded**; `AppOpenAd` → **app open**; `NativeAd` → **native**. Print what you found
   and where.
3. Key names: find how the build reads ids. In android-factory it is
   `<module-name>.admob.appId`, `.banner`, `.interstitial`, `.rewarded` in the root
   `local.properties` (see the convention plugin). Confirm by grepping `build-logic/` for
   `admob`; use whatever naming it actually reads.
4. `git check-ignore local.properties` must print the path. If not, stop.

## Step 2 — AdMob: find or create the app

Open `https://admob.google.com/v2/apps/list` in a new tab.

- The app table renders late. Read rows with `javascript_tool`
  (`[...document.querySelectorAll('tr,[role=row]')].map(r => r.innerText)`) after a 3 s
  wait; `get_page_text` often returns only the footer.
- If a row shows this `applicationId`, note its app id and skip to Step 3.
- Otherwise: **Add app** → Platform **Android** → "Is the app listed on a supported app
  store?" **No** (it is not published yet) → **Continue** → App name → **Add app** → **Done**.
  The app lands on `/v2/apps/<numeric id>/overview`. The full app id is
  `ca-app-pub-<publisher>~<numeric id>`; read it from the page rather than assembling it.

Once the app is on Play, AdMob offers to link it to the store listing. That is a later
step, not part of this command.

## Step 3 — One ad unit per format

For each format: open `https://admob.google.com/v2/apps/<numeric id>/adunits/create`.

- **Select the format with JavaScript.** Coordinate clicks on the format cards stop
  working after the first navigation. Use
  `[...document.querySelectorAll('material-button')].find(b => b.getAttribute('aria-label') === 'Interstitial').click()`
  (labels: `Banner`, `Interstitial`, `Rewarded`, `Rewarded interstitial`, `Native advanced`, `App open`).
- **Typing into the name field often does nothing.** Set it with the native setter plus an
  `input` event, then check the counter reads `n / 80` before creating:
  ```js
  const i = [...document.querySelectorAll('input')].find(x => x.offsetParent && x.type === 'text');
  i.focus(); i.value = '<app-id> interstitial';
  i.dispatchEvent(new Event('input', { bubbles: true })); i.blur();
  ```
- Name units `<app-id> <format>` so they sort together.
- **Rewarded** also asks for a reward amount and item. Amount `1`; the item field is
  prefilled `Reward` — leave it. The reward the app grants is decided in code
  (`AdFreePass`, quota), not by this value.
- Leave "Partner bidding" unticked.
- Click **Create ad unit** via JavaScript; the success page shows
  `ca-app-pub-…/…`. Read it from `document.body.innerText`.

Afterwards open `https://admob.google.com/v2/apps/<numeric id>/adunits/list` and verify, per
row, name → id → **format**. A unit created as the wrong format cannot be changed; archive
it and create another.

## Step 4 — Write the ids

Append to `local.properties`, one block per app, in the file's own style (read it first:
android-factory keeps a comment line per app):

```properties
# <App> — AdMob app "<AdMob app name>", created <date>.
<module>.admob.appId=ca-app-pub-XXXXXXXXXXXXXXXX~NNNNNNNNNN
<module>.admob.banner=ca-app-pub-XXXXXXXXXXXXXXXX/NNNNNNNNNN
<module>.admob.interstitial=ca-app-pub-XXXXXXXXXXXXXXXX/NNNNNNNNNN
<module>.admob.rewarded=ca-app-pub-XXXXXXXXXXXXXXXX/NNNNNNNNNN
```

The app id uses `~`, ad units use `/`. They are easy to swap and the SDK crashes at start
if the app id is wrong. Print the block back (the file is ignored, so this is the only
record in the conversation).

## Step 5 — Prove it reaches the release build (skip with `--no-build`)

1. Say you are building, then run `./gradlew :<app>:assembleRelease` (on Linux,
   `sh ./gradlew` if the wrapper has no executable bit).
2. `aapt2 dump xmltree --file AndroidManifest.xml <release apk>` and grep for
   `APPLICATION_ID`: it must be the new `~` id, not the Google test app id
   `ca-app-pub-3940256099942544~…`.
3. Check the debug build still carries the test id. If the build logs a "missing AdMob id"
   warning for release, the key names in Step 1.3 are wrong.

Real ads may take up to an hour to fill after units are created; empty fills before then are
normal.

## Step 6 — GDPR consent message (`--gdpr`)

UMP shows the consent form only for apps attached to a published message.

1. `https://admob.google.com/v2/privacymessaging/gdpr`. Usually one message covers every
   app; open it.
2. **Your apps** → pencil → tick the app. A yellow banner "You need to add a privacy policy
   URL" appears and **shifts every row down**: re-measure before clicking the row's
   **Add URL**, and zoom to confirm the popup belongs to this app's row.
3. Enter the app's privacy policy URL in the same form the other rows use (trailing slash
   or not) → **Save** → **Confirm** (the button greys while it saves; wait) → **Your apps**
   now counts one more.
4. **Publish changes** → **Publish**. The list shows the new count and today's date. It can
   take an hour to reach devices.

Do not publish a message whose URL is not live yet: `curl -sL -o /dev/null -w '%{http_code}'`
must print 200.

## Step 7 — Report

- AdMob app name, numeric id, and a table: format → unit name → unit id → verified format.
- The `local.properties` block written, and whether the release manifest carried the id.
- GDPR: attached and published, or skipped (and why).
- Next: `/play-store:console <app>` to create the Play Console app, then link AdMob to the
  store listing once the app is published.
