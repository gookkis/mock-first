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
| `--link-entry` | Also append the "privacy policy lives on a separate page" line to the portfolio entry body (both locales), if an entry exists and neither its body nor its `highlights.action` links the page yet. The wording is copied from the entries that already carry the line, not from this file. |
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
- Same site-repo hygiene as `/portfolio:publish`: commit local by default, push only
  with `--push`, `--pr` for review. A dirty tree is a stop only when the dirt touches
  the files this command writes; anything else (another session's edits, an unrelated
  deletion) is left alone, staged around by adding files by path, and named in the report.
- The Android side is committed too, locally, in its own repo
  (`docs(<app>): add the Play Data Safety summary` or `update ...`). `--push` pushes both
  repos; without it, print both `git push` lines as next steps and say the site one deploys.
- Run `date +%Y-%m-%d` for the date; the page shows it in the locale's long form
  (`2 September 2026` / `September 2, 2026`).

## Step 1 — Resolve

App module, `applicationId`, `app_name`, `versionName` as in `/portfolio:publish` Step 1.
Site config from `~/.claude/portfolio.yml`. Slug per the argument rules. If
`src/pages/apps/<slug>/privacy-policy.astro` already exists, this run is an update: read
it, keep its wording where the inventory hasn't changed, and show a diff before writing.

Slug lookup: an unreleased app has no Play link yet, so an entry can exist without the
`applicationId` appearing anywhere in it. Check `<collection>/<default_locale>/<derived
slug>.md` and the image folder `<images>/<derived slug>/` before concluding there is no
entry; `/portfolio:publish` often runs before this command, as a draft.

## Step 2 — Data inventory (the part that matters)

Scan the app module **and every module it depends on** (`implementation(project(...))`,
resolved through `settings.gradle`; shared modules often hold the SDKs). Record each
finding with the file that proves it.

### 2a. Permissions
`AndroidManifest.xml` of every module in the dependency tree, plus permissions merged by
libraries you can infer (AdMob adds `AD_ID`; Firebase adds `INTERNET`,
`ACCESS_NETWORK_STATE`, `WAKE_LOCK`). If `build/intermediates/merged_manifests/` exists
from a previous build, prefer the merged manifest there. Keep two lists: what the app's
own manifest declares, and what the merge added (`WAKE_LOCK`, `FOREGROUND_SERVICE`, the
`ACCESS_ADSERVICES_*` trio from the Mobile Ads SDK, the app's own
`DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION`, which is not user-facing and is not listed on
the page). The page names both groups, but separately. Classify:

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
| `user-messaging-platform` | Google UMP | consent form for EEA/UK/CH users before ads load; whether the app lets users reopen it (grep `showPrivacyOptions` / `isPrivacyOptionsRequired` in the settings wiring) decides what section 7 may promise |
| `retrofit`, `okhttp`, `ktor` | HTTP client | there is a backend or API: find base URLs (`BuildConfig`, `strings.xml`, `@Url`, `HttpUrl`, `baseUrl(`, string literals starting `https://`) and list them. Tell the two cases apart: **our backend** (data stored on our side: retention and deletion questions) vs a **public third-party API called directly** (GET, no auth, no account, e.g. an exchange-rate feed): that gets its own section like an SDK, stating what the request contains and that the host sees the device IP |
| `play-services-location` | location | location section, precise vs coarse, foreground vs background |
| `facebook`, `appsflyer`, `adjust`, `mixpanel`, `amplitude`, `sentry` | analytics/attribution | their own section, link their policy |
| `room`, `datastore`, `sqldelight`, `realm`, `SharedPreferences` in code | local storage | "data stored on your device" section |
| `androidx.work` | WorkManager | background processing mention if it touches data |

For each SDK found, note the version and the module it came from.

An interface is not an SDK. Projects like android-factory declare `Analytics`,
`CrashReporter` and `BillingManager` from day one but wire `NoOp*` implementations until
the real SDK lands. Check what the DI container actually instantiates (grep the
`AppContainer` / module for `NoOp`, `Debug`, `Fake`) and record those as **not in build**,
so the page can state the absence and the Data Safety table can say why a row is "no".
A store listing that promises a purchase while billing is `NoOp` is a report item, not
something the page mentions.

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
- Contact: the app's own support address (the feedback email it passes to its settings
  screen, e.g. `SUPPORT_EMAIL` in android-factory) vs the contact on the template page.
  When they differ, list both under `Unknown` and ask which one the page should carry;
  don't pick silently.
- Backup: `allowBackup="true"` with no backup rules means local preferences ride along in
  the user's Google backup. That is a bullet in section 4 of the page, not a data type in
  the Data Safety form.

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

Exception: the inventory was already confirmed earlier in this same session and nothing
under the app module or its dependency tree has changed since (check `git log` and
`git status` against that moment). Then print the inventory with "unchanged since
confirmation at <time>" and continue without asking again; the pages are not rewritten
and `lastUpdated` stays. This is what a re-run for `--link-entry` or `--push` alone
looks like.

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
- **A public third-party API gets a section of its own** in 5, named after the service:
  what each request carries (only the currency codes, say), that the host sees the device
  IP as with any connection, when the request fires (only on that screen), a link to the
  service's privacy page, and that the rest of the app works offline if it does.
- **Section 6 lists app-declared permissions first**, then the ones the merge added, under
  a sentence saying the SDK added them and only the SDK uses them. A reader comparing the
  page with the Play permissions list must find every entry.
- **Consent (UMP) is a paragraph in the AdMob section** and a bullet in 7, but the "you
  can change it again from Settings" promise is made only when the inventory found the
  privacy-options wiring.
- **Every "you can do X in the app" claim in 7** (clear history, change consent, export)
  is verified against a screen or ViewModel before it is written.
- The `en/` page says the same things in English; it's not a translation of sentences
  but of facts.
- Import depth: `../../../layouts/BaseLayout.astro` for `id`, one more `../` for `en/`.
- Write the pages with the file tools, not a shell heredoc: the pages are full of quotes
  and `$` template literals, and a heredoc mangled by a shell hook produces a half-written
  page that the build then accepts.

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

Rows and notes that recur:
- A GET-only public API (rate feeds and the like) is not a declared data type: nothing
  leaves the device to be stored or processed on our behalf. Say so in a note, with the
  hosts, so the form and the page tell the same story.
- Interfaces wired to `NoOp` get a "no" row with the reason in *By whom*
  (`no Crashlytics; NoOpCrashReporter`), so the next run can see what changed.
- `allowBackup=true` goes in the notes, not the table: it is the user's own backup.
- If the listing promises a purchase the build cannot make yet, add a "when Billing
  lands" note naming the row to add (*Financial info → Purchase history*) and the page
  section to write, and say the command must be re-run then.

## Step 5 — Validate, commit, report

1. `--dry-run`: print both pages and the summary, write nothing.
2. Write the files. If `--link-entry` and the portfolio entry exists with neither a
   privacy line in its body nor a `highlights.action` pointing at the page, append one
   blank line and the line to the body of both locales. Take the wording from an entry
   that already has it (`grep -rl privacy-policy <collection>`), per locale; only if no
   entry has one yet use:
   `Kebijakan privasi aplikasi ini tersedia di [halaman terpisah](/apps/<slug>/privacy-policy).`
   / `The app's privacy policy lives on [its own page](/en/apps/<slug>/privacy-policy).`
3. Build check as in `/portfolio:publish` Step 4.3 (`npm run build` if `node_modules`
   exists; otherwise offer `npm ci`). A page with a wrong import depth fails here, which
   is the point.
4. Commit, staging by path: `Add <App> privacy policy page (ID+EN)` or `Update ...`; when
   only the entries changed, `Link the <App> portfolio entry to its privacy policy (ID+EN)`.
   Append the attribution lines the session provides. Push/PR only per flag.
5. Report:
   - the two URLs (live after deploy) and the one to paste into Play Console -> App
     content -> Privacy policy: `<site_url>/apps/<slug>/privacy-policy`
   - where the app should show it too. Find where the app already wires its support
     links (the object that carries the feedback email, e.g. `SupportLinks` passed to
     `SettingsRoute` in android-factory) and name the field to fill
     (`privacyPolicyUrl`); fall back to suggesting a `privacy_policy_url` string and a
     Settings row only when no such object exists. Suggest, don't edit.
   - open questions from the inventory (unknown retention, no account-deletion path,
     SDK not recognised)
   - if a portfolio entry already exists for this slug, note that re-running
     `/portfolio:publish <app>` will refresh its `highlights` section from the confirmed
     inventory in `data-safety.md` and point its action button at the new page
   - the one-line disclaimer: draft to be reviewed by the owner; not legal advice.

## Re-runs

Re-run after adding or removing an SDK or a permission. The inventory diff is shown, the
page is updated in place, `lastUpdated` moves to today, and the Data Safety summary is
rewritten. An empty diff rewrites nothing: the run then only does what its flags ask
(`--link-entry`, `--push`) and says the pages were left as they are.
