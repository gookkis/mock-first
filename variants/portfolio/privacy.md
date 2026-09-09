---
description: Generate an app's privacy policy page (id + en) on the site from what the code actually does, plus a Data Safety summary
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(date*), Bash(mkdir*), Bash(ls*), Bash(cat*), Bash(test*), Bash(node*), Bash(npm*), Bash(npx*), Bash(git*), Bash(gh*)
---

# Portfolio privacy — a policy page that matches the code

Play Console requires a privacy policy URL for every app. This command scans the app
module for what it really does with data (permissions, SDKs, network, accounts, local
storage), turns that into a **data inventory** the user confirms, and only then writes
two pages on the Astro site, one per locale, following the site's existing per-app
privacy page as the template. It also writes a Data Safety summary for the Play Console
form from the same inventory.

Result URL: `<site_url>/apps/<slug>/privacy-policy` (and `/en/apps/<slug>/privacy-policy`).

## Arguments

| Argument | Meaning |
|---|---|
| `<app>` | Required. Gradle path or app name. |
| `--slug <slug>` | Page folder. Default: the portfolio entry's slug if one exists, else derived as in `/portfolio:publish` Step 2. |
| `--template <path>` | Existing page to copy the structure from. Default: the newest `src/pages/apps/*/privacy-policy.astro` in the site. |
| `--contact <email>` | Contact address. Default: the one used in the template page. |
| `--link-entry` | Also append the "privacy policy lives on a separate page" line to the portfolio entry body (both locales), if an entry exists and doesn't link it yet. |
| `--push` / `--pr` / `--dry-run` | Same meaning as in `/portfolio:publish`. |

## Absolute rules

- **The page states facts found in the code, not a generic template.** Every claim
  ("no account", "no analytics SDK", "only these permissions") must trace back to the
  inventory in Step 2. If the code contradicts the template's wording, the code wins.
- **When unsure, say "unknown" in the inventory and ask.** Never assert that an SDK
  doesn't collect data because you didn't find evidence; absence of evidence is a
  question for the user.
- **Not legal advice.** Say so once in the report: this is a well-structured draft the
  user must read and own before publishing.
- **Only these files are written in the site repo:**
  `src/pages/apps/<slug>/privacy-policy.astro`, `src/pages/en/apps/<slug>/privacy-policy.astro`,
  and, with `--link-entry`, the two existing entry files. Nothing else.
- **The Android side gets one file:** `docs/store/<app>/data-safety.md`. No code changes.
- Same site-repo hygiene as `/portfolio:publish`: clean tree first, commit local by
  default, push only with `--push`, `--pr` for review.
- Run `date +%Y-%m-%d` for the date; the page shows it in the locale's long form
  (`2 September 2026` / `September 2, 2026`).

## Step 1 — Resolve

App module, `applicationId`, `app_name`, `versionName` as in `/portfolio:publish` Step 1.
Site config from `~/.claude/portfolio.yml`. Slug per the argument rules. If
`src/pages/apps/<slug>/privacy-policy.astro` already exists, this run is an update: read
it, keep its wording where the inventory hasn't changed, and show a diff before writing.

## Step 2 — Data inventory (the part that matters)

Scan the app module **and every module it depends on** (`implementation(project(...))`,
resolved through `settings.gradle`; shared modules often hold the SDKs). Record each
finding with the file that proves it.

### 2a. Permissions
`AndroidManifest.xml` of every module in the dependency tree, plus permissions merged by
libraries you can infer (AdMob adds `AD_ID`; Firebase adds `INTERNET`,
`ACCESS_NETWORK_STATE`, `WAKE_LOCK`). If `build/intermediates/merged_manifests/` exists
from a previous build, prefer the merged manifest there. Classify:

| Group | Permissions |
|---|---|
| network | INTERNET, ACCESS_NETWORK_STATE |
| ads | com.google.android.gms.permission.AD_ID |
| location | ACCESS_COARSE_LOCATION, ACCESS_FINE_LOCATION, ACCESS_BACKGROUND_LOCATION |
| media | CAMERA, RECORD_AUDIO, READ_MEDIA_IMAGES/VIDEO/AUDIO, READ/WRITE_EXTERNAL_STORAGE |
| contacts / calendar / phone / SMS | the respective permissions |
| notifications | POST_NOTIFICATIONS |
| background | FOREGROUND_SERVICE*, RECEIVE_BOOT_COMPLETED, SCHEDULE_EXACT_ALARM |
| billing | com.android.vending.BILLING |
| other | anything else, listed verbatim |

### 2b. Third-party SDKs (from Gradle deps and the version catalog)

| Dependency contains | SDK | What it means for the policy |
|---|---|---|
| `play-services-ads`, `admob` | Google AdMob | ads section, AAID, IP, device info, ad interactions; personalised-ads opt-out |
| `firebase-analytics`, `firebase-bom` + analytics | Firebase Analytics | usage analytics, app instance id, events |
| `firebase-crashlytics` | Crashlytics | crash logs, device state |
| `firebase-messaging` | FCM | push token |
| `firebase-auth`, `play-services-auth`, `credentials` | account sign-in | account section: what identifiers, deletion path |
| `firebase-firestore`, `firebase-database`, `firebase-storage` | cloud data | data stored on our side, retention, deletion |
| `billing` | Play Billing | purchases handled by Google, we don't see payment details |
| `retrofit`, `okhttp`, `ktor` | HTTP client | there is a backend or API: find base URLs (`BuildConfig`, `strings.xml`, `@Url`, `HttpUrl`, `baseUrl(`) and list them |
| `play-services-location` | location | location section, precise vs coarse, foreground vs background |
| `facebook`, `appsflyer`, `adjust`, `mixpanel`, `amplitude`, `sentry` | analytics/attribution | their own section, link their policy |
| `room`, `datastore`, `sqldelight`, `realm`, `SharedPreferences` in code | local storage | "data stored on your device" section |
| `androidx.work` | WorkManager | background processing mention if it touches data |

For each SDK found, note the version and the module it came from.

### 2c. Accounts and user data in code
- Login/sign-up screens: grep for `signIn`, `login`, `register`, `AuthCredential`,
  `GoogleSignIn`, `CredentialManager`.
- What the app stores locally: Room entities and DataStore keys (entity class names and
  field names tell you if it's progress/scores/preferences vs names/emails/photos).
- Anything sent out: every network call's target and payload type (from the API
  interface: request body classes and their fields).
- Exports/backups: `allowBackup`, `fullBackupContent`, `dataExtractionRules`.
- Children: age gating, `tagForChildDirectedTreatment`, Families policy signals; the
  audience from the PRD or `listing.md`.

### 2d. Show the inventory and stop for confirmation

```
Data inventory — <app> (<applicationId>)  <date>

Permissions (merged): INTERNET, ACCESS_NETWORK_STATE, AD_ID      [app/src/main/AndroidManifest.xml, +AdMob]
Third-party SDKs:     Google AdMob 23.6.0 (core:ads)            [gradle/libs.versions.toml]
Accounts / login:     none found                                 [grep signIn|login in app + modules]
Local data:           Room: Attempt(score, date), Setting(key)   [core:data/.../entities]
Network calls:        none besides ads; no base URL found        [grep retrofit|okhttp|ktor|baseUrl]
Backup:               allowBackup=false                          [AndroidManifest.xml]
Audience:             students preparing for exams, 13+          [docs/store/<app>/listing.md]
Unknown:              —
```

Ask: "Ada yang salah atau kurang?" Wait. Everything after this is derived from the
confirmed inventory.

## Step 3 — Write the two pages

Read the template page in both locales. Keep its frontmatter constants
(`appName`, `packageId`, `storeUrl`, `title`, `description`, `lastUpdated`), its
`BaseLayout` import path depth, the `<article>` wrapper, and the section order:

1. intro paragraph (who publishes it, where, what the page explains)
2. Ringkasan singkat / Quick summary: 4-6 bullets
3. Data yang kami kumpulkan / Data we collect
4. Data yang disimpan di perangkat Anda / Data stored on your device
5. one section per third-party SDK from the inventory (AdMob, Firebase, ...)
6. Izin aplikasi / App permissions: every merged permission with a one-line reason
7. Pilihan Anda / Your choices
8. Privasi anak-anak / Children's privacy
9. Keamanan / Security
10. Perubahan kebijakan / Changes
11. Kontak / Contact

Rules for adapting the template to the inventory:
- **No SDK -> no section.** An app without ads has no AdMob section and no AD_ID line.
- **A backend changes section 3** from "Tidak ada" to a concrete list: what is sent, to
  which host, why, how long it's kept, how to request deletion. If retention or deletion
  is unknown, that's a question for the user, not a sentence to invent.
- **Accounts add** an account subsection in 3 and a deletion path in 7 (Play requires an
  account-deletion path; if the app has none, flag it in the report).
- **Location, camera, microphone, contacts** each get a sentence in 6 saying when the
  permission is used and that it's only used in the foreground for that feature (verify
  foreground vs background from the inventory).
- The `en/` page says the same things in English; it's not a translation of sentences
  but of facts.
- Import depth: `../../../layouts/BaseLayout.astro` for `id`, one more `../` for `en/`.

## Step 4 — Data Safety summary (Android side)

Write `docs/store/<app>/data-safety.md`: the answers for the Play Console's Data Safety
form, derived from the same inventory, in the form's own vocabulary:

```markdown
# Data Safety — <app>  (<date>, from the confirmed inventory)

Does the app collect or share any of the required user data types?  Yes / No
Is all user data encrypted in transit?  Yes (HTTPS only) / N/A (no data leaves the device)
Do you provide a way for users to request that their data be deleted?  ...

| Data type | Collected | Shared | Purpose | Optional | By whom |
|---|---|---|---|---|---|
| Device or other IDs (Advertising ID) | yes | yes | Advertising | no | Google AdMob |
| App activity (in-app actions) | no | no | | | |
| ...

Notes: AdMob's data is collected by the SDK, which Play counts as "collected by the app".
Source: <sdk> data disclosure page URL for each SDK.
```

Link each SDK's official data disclosure page (AdMob and Firebase publish them) instead of
guessing what the SDK collects.

## Step 5 — Validate, commit, report

1. `--dry-run`: print both pages and the summary, write nothing.
2. Write the files. If `--link-entry` and the portfolio entry exists without a privacy
   link, append the existing entries' phrasing to the body of both locales:
   `Kebijakan privasi aplikasi ini tersedia di [halaman terpisah](/apps/<slug>/privacy-policy).`
   / `This app's privacy policy is available on [a separate page](/en/apps/<slug>/privacy-policy).`
3. Build check as in `/portfolio:publish` Step 4.3 (`npm run build` if `node_modules`
   exists; otherwise offer `npm ci`). A page with a wrong import depth fails here, which
   is the point.
4. Commit: `Add <App> privacy policy page (ID+EN)` or `Update ...`, plus the attribution
   lines the session provides. Push/PR only per flag.
5. Report:
   - the two URLs (live after deploy) and the one to paste into Play Console -> App
     content -> Privacy policy: `<site_url>/apps/<slug>/privacy-policy`
   - where the app should show it too: a "Kebijakan privasi" link in its About/Settings
     screen, and the same URL in `strings.xml` (`privacy_policy_url`) if the app has no
     such string yet. Suggest, don't edit.
   - open questions from the inventory (unknown retention, no account-deletion path,
     SDK not recognised)
   - if a portfolio entry already exists for this slug, note that re-running
     `/portfolio:publish <app>` will refresh its `highlights` section from the confirmed
     inventory in `data-safety.md` and point its action button at the new page
   - the one-line disclaimer: draft to be reviewed by the owner; not legal advice.

## Re-runs

Re-run after adding or removing an SDK or a permission. The inventory diff is shown, the
page is updated in place, `lastUpdated` moves to today, and the Data Safety summary is
rewritten.
