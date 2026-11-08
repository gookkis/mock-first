---
description: Write the Play Store listing per locale (fastlane metadata) plus the App content answer sheet the Console asks for — rating, privacy, data safety, declarations
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(date*), Bash(mkdir*), Bash(ls*), Bash(cat*), Bash(cp*), Bash(node*), Bash(wc*), Bash(fastlane*), Bash(test*), Bash(find*)
---

# Listing — store text, store settings, and every App content declaration

Writes the two things the Play Console asks for before an app can go live:

1. **Store listing text** per locale, in the layout `fastlane supply` reads, plus the images
   produced by `/play-store:enhance`.
2. **An answer sheet for the rest of the Console** — Store settings, App content
   (privacy policy, ads, app access, content rating, target audience, data safety,
   advertising ID, and the government / financial / health / news declarations) — derived
   from the code where it is derivable and marked `ASK` where it is not.

Nothing here is uploaded automatically except metadata with `--upload`; the declarations
are forms only a human can submit. The answer sheet exists so filling them is transcription,
not guesswork.

## Arguments

Read `$ARGUMENTS`:

| Argument | Meaning |
|---|---|
| `<app>` | Required. Same form as in `/play-store:capture`. |
| `--locale <a,b>` | Locales to write. Default: `locales:` from `screens.yml`, else `en`. |
| `--from <path>` | Use an existing listing text (a `.md` or `.txt` the user wrote) as the source instead of the Q&A. Never rewrite the source file. |
| `--text` | Only the store listing text (Steps 2-4). Skip the declarations. |
| `--console` | Only the App content answer sheet (Steps 5-7). Skip the listing text. |
| `--images` | Only refresh the image folders from `docs/store/<app>/out/`. |
| `--notes <text>` | Only (re)write the release notes for this version: `changelogs/<versionCode>.txt` per locale (≤500 chars each) plus the combined `docs/store/<app>/changelogs.txt`. Without a value, draft them from the commits since the previous version. A full run writes them too, and every run writes them when the version went up (Step 1). |
| `--upload` | After writing, run `fastlane supply` for metadata only (asks first; needs fastlane + a configured `Appfile`/JSON key). |

## Absolute rules

- **Character budgets are hard limits, counted by script, not by eye.** Title 30,
  short description 80, full description 4000, release notes 500. Count with
  `node -e "process.stdout.write(String([...require('fs').readFileSync(process.argv[1],'utf8').trim()].length))" <file>`
  (code points, which is what the Console counts). Over budget = rewrite, not truncate.
- **Google Play metadata policy** (title, short description, icon, and the visible start of
  the full description): no "free", "best", "#1", "top", "sale", ALL-CAPS words, emoji or
  emoticons, repeated punctuation, no Play-store-performance claims ("Top 10 app",
  "#1 in Finance"), no promotional or pricing text ("download now", "50% off", "limited
  time"), no competitor names or brands the app isn't part of, no testimonials, no keyword
  lists, no claims the app doesn't fulfil, no reference to other stores or platforms.
  If the user insists on one of these, say it may be rejected and write it anyway.
- **No invented features and no invented declarations.** Everything in the description and
  every answer in the answer sheet must trace to the code, the PRD, or the user's answers.
  An answer you cannot prove is `ASK — <the question>`, never a guess. A wrong Data safety
  or Content rating answer is a policy violation, not a typo.
- **Each locale is written in that language**, not translated word-for-word. Idiomatic
  Indonesian for `id`, plain English for `en`.
- **Existing listing files belong to the user.** If `fastlane/metadata/android/<locale>/`
  already has text, show a diff-style comparison and ask before overwriting.
- **Not legal advice.** Say once in the report that the declarations are a prepared draft
  the user must read and own before submitting.
- Run `date +%Y-%m-%d` for the date.

## Step 1 — Gather

1. Resolve the app module (as in `/play-store:capture` Step 1): `applicationId`, `app_name`
   from `strings.xml`, `versionName`, `versionCode`, `targetSdk`, `minSdk`.
2. Read, if they exist: `docs/apps/<app>/PRD.md` (android-factory), `docs/store/<app>/screens.yml`,
   `docs/store/<app>/captions.<locale>.yml` (headlines are already benefit-first copy),
   `docs/store/<app>/data-safety.md` (written by `/portfolio:privacy` — if it exists it is
   the source of truth for Step 6; never re-derive it, only fill the gaps), `README` of the
   app module, and the feature modules it depends on (names are a good feature inventory).
3. Existing listing: `fastlane/metadata/android/*/` or `docs/store/<app>/listing.md`.
4. Privacy policy URL: `docs/store/<app>/data-safety.md`, the portfolio entry, or
   `~/.claude/portfolio.yml` (`<site_url>/apps/<slug>/privacy-policy`). If there is none,
   this is a blocker — say so early and point to `/portfolio:privacy <app>`.
5. **Version upgrade check (every run, whatever the flags).** Compare the module's current
   `versionCode` with the newest `fastlane/metadata/android/*/changelogs/<n>.txt` and with
   `docs/store/<app>/changelogs.version` (the version `changelogs.txt` belongs to, see
   Step 3). If the current
   `versionCode` has no changelog in every locale, or `changelogs.txt` belongs to an older
   version, the version went up: write the release notes for the new `versionCode` (Step 3,
   release notes) and rebuild `changelogs.txt` in this run, even under `--text`,
   `--console` or `--images`. Draft them from the commits since the previous version
   (`git log <previous-version-tag>..HEAD`, or since the commit that last touched
   `changelogs.txt` when there is no tag) and show the draft before writing.

## Step 2 — Q&A (skip what's already answered by the documents or `--from`)

Ask one at a time, at most six questions:

1. Who is this app for, in one sentence? (the audience shapes the vocabulary)
2. The one thing it does better than the alternatives?
3. Play category (e.g. Business, Shopping, Finance, Productivity) and up to five store tags?
4. Three to five search terms a user would type to find it. These are the ASO keywords;
   `/play-store:aso` reuses them.
5. Anything that must be mentioned (login required, subscription, region-only, needs a
   companion device) or must not be mentioned?
6. Tone: neutral, friendly, or formal?

## Step 3 — Write the text per locale

Draft all fields per locale, show them with the character count next to each, and ask for
approval before writing files.

**Title (≤30):** `<App name>` or `<App name>: <2-3 word descriptor>` when the name alone
doesn't say what the app does. The descriptor carries the top keyword. No punctuation
tricks, no repeated brand.

**Short description (≤80):** one sentence, the main benefit plus the top two keywords,
readable on its own; it shows above the fold on the store page.

**Full description (≤4000):**
- First 2 lines (about 160 chars): the pitch. They're visible before "Read more" and are
  what the ranking reads first. Include the top keyword naturally.
- Then 3 to 6 short feature blocks, each a plain-text line as a heading (the Console renders
  no markdown, only line breaks) followed by 1 to 3 sentences. Order them the same way as
  the screenshots, so the text and images tell one story.
- Then "who it's for" and any required disclosures (permissions and why, account required,
  subscription and price, in-app purchases, ads, region limits, companion hardware).
  A subscription app must state the price, period, and that it renews.
- Close with a single line of contact / support — the same address as the Console's store
  listing contact email.
- Keyword use: each target keyword 2 to 4 times across the full text, never in a row, never
  in a list. The Console penalizes stuffing.
- Plain text only. Line breaks separate paragraphs. Unicode bullets (`•`) are fine.

**Release notes (≤500 per locale, every full run and `--notes`):** what changed, user-visible,
in that locale. No version numbers alone ("bug fixes" is a wasted field), no promotional
text. On a first listing, describe the release itself. Read `versionCode` from the app
module's `build.gradle(.kts)`; never guess it.

Write them twice from the same text: once per locale for `fastlane supply`, and once as
**`docs/store/<app>/changelogs.txt`: one combined file for every supported locale** — the
block the Console's release-notes box takes when pasted, one tag per locale (Play Console
locale code), in `--locale` order. It always holds the current version only; a version
upgrade rewrites it (git keeps the old ones):

```
<en-US>
• ...
</en-US>
<id>
• ...
</id>
```

Build the combined file by script from the per-locale files, never by hand, so the two
can't drift:

```
{ for l in <locales>; do echo "<$l>"; cat <app-module>/fastlane/metadata/android/$l/changelogs/<versionCode>.txt; echo "</$l>"; done; } \
  > docs/store/<app>/changelogs.txt
echo "<versionCode> <versionName>" > docs/store/<app>/changelogs.version
```

The file must contain nothing but the tagged blocks (no header, no version line): it is
pasted as is. The version it belongs to lives in `changelogs.version`, which the Step 1
upgrade check reads. If an older `docs/store/<app>/release-notes/` folder exists, leave it.

It lives under `docs/store/`, not `fastlane/metadata/android/`: `supply` treats every folder
there as a locale.

## Step 4 — Write the listing files

```
<app-module>/fastlane/metadata/android/<locale-code>/
  title.txt
  short_description.txt
  full_description.txt
  video.txt                <- only when the user has a YouTube URL (public/unlisted, ads off, not age-restricted)
  changelogs/<versionCode>.txt   <- per locale, for supply
  images/
    phoneScreenshots/      <- from docs/store/<app>/out/<locale>/01-*.jpg ...
    sevenInchScreenshots/  <- from out-tablet7/<locale>/ if present
    tenInchScreenshots/    <- from out-tablet/<locale>/ if present
    featureGraphic.<ext>   <- from docs/store/<app>/out/<locale>/featureGraphic.*
    icon.png               <- ic_launcher-playstore.png (512x512) if found
```

Plus the combined, paste-ready notes for all locales: `docs/store/<app>/changelogs.txt`
and its `changelogs.version` (format above).

Locale folder codes: `id`, `en-US`, `ms`, `en-GB`, `ja-JP`, etc. (the Play Console codes,
not Android resource qualifiers). Map `en` -> `en-US`.

For a multi-app project, the fastlane folder is per app module. If the project already has
a `fastlane/` at the root with a per-app layout, follow the existing convention instead.

Write all files, then re-count every text file with the script and print the table:

| Locale | Title | Short | Full | Notes | Images |
|---|---|---|---|---|---|
| id | 24/30 | 71/80 | 1980/4000 | 180/500 | 6 + feature |

Asset gate (state pass/fail, don't fix silently):
- at least 2 screenshots to publish at all; 4 or more at 1080px+ to be eligible for Play's
  promotional surfaces;
- large-screen (tablet/Chromebook) listings need **at least 4** screenshots, 16:9 or 9:16,
  1080-7680px, or the app is downranked on those devices;
- feature graphic 1024x500 JPEG or 24-bit PNG (no alpha) — required;
- icon 512x512 32-bit PNG, ≤1MB;
- screenshots JPEG or 24-bit PNG, no alpha, 320-3840px per side.

Also write `docs/store/<app>/listing.md`: every locale's fields in one readable page with
the counts, the keyword list from Q&A item 4, and the date. This is the file
`/play-store:aso` audits.

## Step 5 — Scan for the declarations

Read the app module **and every module it depends on** (`implementation(project(...))`,
resolved through `settings.gradle`); shared modules carry the SDKs. Prefer
`build/intermediates/merged_manifests/` when a previous build left one. Record, each with
the file that proves it:

| Signal | Where | Feeds |
|---|---|---|
| Permissions, own + merged | every `AndroidManifest.xml` | Data safety, permissions declaration, privacy policy |
| `com.google.android.gms.permission.AD_ID` | manifest (added by AdMob/Firebase Analytics) | Advertising ID declaration |
| Ad SDKs (AdMob, AppLovin, Unity Ads, IronSource) | version catalog, `build.gradle` | Ads declaration, content rating, Data safety (sharing) |
| Analytics/crash SDKs (Firebase, Crashlytics, Sentry, Amplitude) | same | Data safety (collection), privacy policy |
| Billing (`com.android.billingclient`) | same | Content rating (digital purchases), listing disclosure |
| Auth (Credential Manager, Google Sign-In, Firebase Auth, own login screen) | code + manifest | App access, Data safety (account), account deletion URL |
| Network hosts | Retrofit/Ktor base URLs, `network_security_config.xml` | Data safety (sharing, transit encryption) |
| Local storage, DataStore, Room, `EncryptedSharedPreferences` | code | Data safety (security practices) |
| WebView loading remote content, user-to-user messaging, UGC | code | Content rating, target audience |
| Foreground service types, exact alarms, all-files access, SMS/Call Log, `QUERY_ALL_PACKAGES`, broad photo/video access | manifest | Sensitive permission declarations |
| Generative AI: image/text/voice generation, a model SDK, or a backend that returns generated media | code, network hosts, version catalog | AI-generated content policy (Step 7b), content rating |
| `targetSdk`, `minSdk` | `build.gradle` | Release readiness |

Show the inventory and ask the user to confirm it before Step 6. Anything you could not
prove stays `unknown` and becomes a question — never assume an SDK collects nothing because
you found no evidence.

## Step 6 — Write `docs/store/<app>/console.md`

One page, ordered exactly like the Console's left menu, so the user fills the forms top to
bottom without hunting. Every line is either an answer, or `ASK — <question>`. Mark each
item `ready` / `needs answer` / `blocker`.

```
# Play Console — <app_name> (<applicationId>)
Prepared <date> from <versionName> (<versionCode>). Draft; read and own it before submitting.

## Store settings
- App or game / Free or paid / Category / Tags (≤5, from Q&A 3)
- Store listing contact details: email (required, public), phone (optional), website
- External marketing: on/off — ASK

## Store listing
- Points at the fastlane files written in Step 4, per locale.

## App content
1. Privacy policy       — URL, live and reachable, names the app, the data, and the contact
2. App access           — "all functionality available without special access", or the demo
                          credentials + the exact steps to reach the gated screens
3. Ads                  — yes/no, from the SDK scan; if yes, the ad SDKs by name
4. Content ratings      — Step 7
5. Target audience      — age bands; whether the store listing appeals to children;
                          if any band is under 13, the Families policy applies (certified
                          ad SDK, no ad-ID collection, extra data rules)
6. News apps            — yes/no
7. Data safety          — Step 6b
8. Government apps      — yes/no
9. Financial features   — none, or which: personal loans, payments/e-money, banking,
                          crypto exchange/wallet, investment/trading, insurance, debt
                          management. Any "yes" needs an organisation account and extra docs.
10. Health apps         — none, or which: health research, telehealth, mental health,
                          medical device, drug dosage, health data. Declaration is required
                          even when the answer is "no health features".
11. Sensitive permissions — one row per declared permission that needs a form: all-files
                          access, SMS/Call Log, exact alarm, broad photo/video access,
                          `QUERY_ALL_PACKAGES`, foreground service types (each needs a
                          purpose string), plus the in-app disclosure they require.
12. Advertising ID      — yes only if `AD_ID` is present; list the purposes it is used for
13. AI-generated content — Step 7b. Only when the app generates content; the in-app
                          reporting path is a release blocker, not a form field

## Release readiness
- targetSdk vs the current Play requirement; signing; account type (a personal developer
  account still needs 12 testers for 14 days of closed testing before production access).
```

### Step 6b — Data safety

If `docs/store/<app>/data-safety.md` exists (from `/portfolio:privacy`), reuse it verbatim
and only add what the Console form needs on top. Otherwise write it from the Step 5
inventory, one row per data type the form lists:

| Data type | Collected | Shared | Ephemeral | Required/Optional | Purposes | Proof |
|---|---|---|---|---|---|---|

Purposes are the Console's own list: App functionality, Analytics, Developer communications,
Advertising or marketing, Fraud prevention/security/compliance, Personalisation, Account
management. Then the security section:

- Data encrypted in transit — yes/no, with the evidence (HTTPS only, cleartext disabled).
- Users can request data deletion — yes/no, and the mechanism.
- **Account deletion:** an app that lets users create an account must offer deletion
  in-app *and* at a public web URL. If the scan found auth and there is no URL, that is a
  `blocker`.
- Committed to the Play Families policy — only when the target audience includes children.
- Independently validated against a security standard — usually no.

Note in the file that the Console can import these answers from its own exported CSV: tell
the user to export the template from Data safety → Export, drop it at
`docs/store/<app>/data-safety-template.csv`, and re-run; only then fill that exact header
row into `docs/store/<app>/data-safety.csv`. Never invent the CSV headers.

## Step 7 — Content rating draft

The rating is an IARC questionnaire; its answers are legally binding. Draft them, don't
submit them. Write into `console.md`:

- Email address for the rating certificate (ASK if unknown).
- Category: the questionnaire's own list (e.g. Utility/Productivity/Communication/Other,
  Social, Reference/News, Game and its subtype).
- One line per question group with the proposed answer **and the evidence**: violence,
  sexuality, nudity, language, controlled substances, gambling and simulated gambling,
  horror/fear, crude humour, user interaction (does the app let users communicate or share
  content?), does it share the user's location with other users, digital purchases,
  in-app ads, and the miscellaneous group.
- When the app generates content with AI, answer yes wherever the questionnaire asks about
  it, and keep the UGC answer honest: sharing through the system share sheet sends content
  *out of* the app, which is not the same as users publishing to each other inside it.
- Flag every answer that follows from Step 5 rather than from the user (ads SDK → "contains
  ads", billing client → "digital purchases", chat/UGC → "user interaction"), because these
  are the ones a wrong answer gets the app pulled for.
- Note that changing an answer later re-issues the rating and can change the age bands.

## Step 7b — AI-generated content

Applies when Step 5 found generative AI: the app turns a prompt, photo, or voice into new
content, whether the model runs on-device, in your own backend, or behind a third-party API.
Play's AI-Generated Content policy covers text-to-text, text-to-image, image-to-image, and
apps that generate voice or video of real people. It does **not** cover an app that merely
displays AI content created elsewhere.

The requirement that fails releases, quoted so it is not paraphrased away:

> "Apps that generate content using AI must contain in-app user reporting or flagging
> features that allow users to report or flag offensive content to developers without
> needing to exit the app."

Check the code for it. A reporting path exists only if all three hold, so record the file for
each:

1. It is reachable from wherever generated content is shown — the result screen, and any
   gallery or history of past results.
2. It works **without leaving the app** to find an address. Opening a mail composer already
   filled in from a chooser inside the app is fine; a support email printed in Settings, a
   link to a web form, or a Play Store review is not.
3. It identifies the content being reported. A report with no id cannot be acted on, and
   acting on reports is what Google actually measures.

If any of the three fails, write it into `console.md` as a **blocker**, not a suggestion:
this is a policy violation that gets a release rejected, and it is app code, not a form the
user can fill in later.

Also record in `console.md`:

- Whether generated content passes a moderation check before the user sees it, and whether
  that check is automated, human, or both.
- Marketing claims: the store text must not promise content the safeguards are built to
  refuse. Check the Step 3 text against it.
- That the developer must actually read incoming reports after launch. Suggest a fixed
  subject line for the report mail so it can be filtered.

## Step 8 — Report

Print two tables: the character/asset table from Step 4, and the Console checklist with one
row per item and its status. Print the path of the combined release notes
(`docs/store/<app>/changelogs.txt`), the `versionCode` it belongs to, whether this run
detected a version upgrade, and the per-locale counts. End with the
blockers in order, then:

- `--upload` next step, or `/play-store:console <app>` to fill the Console through the
  browser (listing, store settings, and every App content declaration from `console.md`), or
  the manual path: Console → Store presence → Store listing, then Policy → App content.
- `/play-store:aso <app>` to audit the text once it's written.

## Step 9 — `--upload` (optional)

Only if `fastlane --version` runs and `<app-module>/fastlane/Appfile` (or the root one)
names `json_key_file` and `package_name`. Show the command, ask, then run:

```
fastlane supply --skip_upload_apk --skip_upload_aab \
  --metadata_path <app-module>/fastlane/metadata/android -p <applicationId>
```

Add `--skip_upload_changelogs` when no changelog exists for the current `versionCode`. `supply` uploads store
listing text and images only — **no App content declaration can be uploaded**; those stay
manual (or `/play-store:console`), which is what `console.md` is for.

Otherwise print the command and the two setup steps (service account JSON in the Play
Console, `fastlane supply init`) and stop.

## Re-runs

Re-running with the same answers rewrites the same text. Use `--images` after every
`/play-store:enhance` run to refresh only the pictures, `--console` after a permission or
SDK change to re-derive the declarations, and `--notes` to redo a release's notes. Any run
after a `versionCode` bump writes the new notes and rewrites `changelogs.txt` by itself.
