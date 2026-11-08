---
description: Create the app in Play Console through the user's signed-in Chrome and fill everything up to the first bundle upload — App content, store settings, and the store listing text and graphics per locale
allowed-tools: Read, Glob, Grep, Bash(date*), Bash(cat*), Bash(ls*), Bash(cp*), Bash(mkdir*), Bash(rm -r docs/store/*/.upload*), Bash(curl*), Bash(python3*), Bash(node*), Bash(find*)
---

# Console — Play Console, from "Create app" to "upload your bundle"

Drives **Claude in Chrome** in the browser where the user is signed in to Play Console and
transcribes what `/play-store:listing` already decided: the fastlane text and images, and the
App content answer sheet in `docs/store/<app>/console.md`. It stops where only the user can
act: choosing testers and uploading a bundle signed with their upload key.

## Arguments

| Argument | Meaning |
|---|---|
| `<app>` | Required. Gradle path or module name. |
| `--only content\|settings\|listing` | Run one part (Steps 3, 4 or 5). Default: all of them, in order. |
| `--locale <a,b>` | Listing locales to fill. Default: every folder under `fastlane/metadata/android/`. |
| `--update` | The app already exists; change only what differs from the local files (Step 2). |

## Absolute rules

- **Transcribe, don't decide.** Every answer comes from `docs/store/<app>/console.md`, the
  fastlane files, or `data-safety.md`. A question with no written answer is asked, not
  guessed. If `console.md` is missing, stop and run `/play-store:listing <app>` first.
- **The invocation is the user's consent to what creating an app requires** — the Developer
  Program Policies and US export declarations on the create form, and the IARC terms on the
  rating questionnaire. It covers nothing else: no payment profile, no account setting, no
  new tester list, no release, no "Send for review".
- **Never upload a bundle, create a release, pick testers, or send anything for review.**
  Tester emails are personal data the user chooses; a release is theirs to cut.
- **Never invent a tester, contact, or website.** Contact email and website come from
  `console.md` / the app's support links.
- **Verify every Save.** Each form reports "Change saved". A Save that silently did nothing
  is the common failure here (see Step 0).
- The session's browser rules apply: choose the browser with the user if several are
  connected, work in a new tab, close it at the end. Run `date +%Y-%m-%d` for the date.

## Step 0 — How this console behaves (read before clicking anything)

Play Console is an Angular app. What reliably works, learned the hard way:

- **Reading.** `get_page_text` often returns only the nav. Read the part after the nav:
  `document.body.innerText.split('Monetize with Play')[1]`.
- **Radios and checkboxes.** Click the inner `input` of `material-radio` /
  `material-checkbox`: `el.querySelector('input').click()`. Read state from
  `input.checked`, not `aria-checked` (often null).
- **Text fields.** `computer type` sometimes lands nowhere. The native value setter fills the
  field but the form validator may not see it ("Add a short description" stays red):
  ```js
  const set = (i, v) => { i.focus();
    Object.getOwnPropertyDescriptor(Object.getPrototypeOf(i), 'value').set.call(i, v);
    i.dispatchEvent(new Event('input', { bubbles: true })); };
  ```
  Follow it with one **real** keyboard event on the field: click it, `ctrl+End`, type a
  space, `BackSpace`, `Tab`. Short fields (app name, short description) can simply be
  typed after `ctrl+a`.
- **Dropdowns** (category): open by clicking the field, then click the option by exact
  text — `[...document.querySelectorAll('*')].filter(e => e.children.length === 0 &&
  e.textContent.trim() === 'Education').pop().closest('[role=option]').click()`.
- **Keep each `javascript_tool` call short** — under ~30 s. A call that times out at 45 s
  keeps running in the page; a timed-out "select six assets and Add" ran on and added a
  second set (a slot read 12 / 8). After any timeout, re-read the state before acting again.
- **Screenshots time out** on this console now and then; wait and retry rather than assume
  the click failed. Never resize the window: it desyncs the capture frame from the page.
- **Dialogs** (`material-dialog`) hold their own Save; the page's Save is a different button.

## Step 1 — Inputs

1. Resolve the module: `applicationId`, `versionCode`.
2. Read `docs/store/<app>/console.md` (answer sheet), `docs/store/<app>/listing.md`,
   `docs/store/<app>/data-safety.md` if present, and every
   `fastlane/metadata/android/<locale>/` folder: `title.txt`, `short_description.txt`,
   `full_description.txt`, `images/icon.png`, `images/featureGraphic.*`,
   `images/phoneScreenshots/`, `images/sevenInchScreenshots/`, `images/tenInchScreenshots/`.
3. **Tablet screenshots.** Play marks the 7-inch and 10-inch slots **required**, even for a
   phone-only app. If either folder is empty, stop and say so: capture with
   `/play-store:capture <app> --tablet` (a 7" portrait AVD and a 10" landscape one), then
   `/play-store:enhance` with the tablet templates. Moving the app's DataStore from the phone
   AVD with `run-as` (debug build) saves replaying the demo history.
4. The privacy policy URL must be live: `curl -sL -o /dev/null -w '%{http_code}' <url>` → 200.

## Step 2 — Find or create the app

1. `https://play.google.com/console/developers` → pick the developer account (ask if more
   than one) → the app list. Search for the package; with `--update` it must exist, otherwise
   it must not.
2. **Create app**: App name (store title, ≤ 30), **App package name** = `applicationId` →
   **Check availability** must say "Package name available" (a taken name is a stop) →
   default language (the listing's default locale) → **App** → **Free** (or Paid per
   `console.md`) → tick both declarations → **Create app**. Wait until the URL leaves
   `create-new-app` (it can take 20 s); note the numeric app id in
   `/app/<id>/app-dashboard`. Every URL below is `…/developers/<account>/app/<id>/…`.

## Step 3 — App content (`app-content/overview`)

Work through **Need attention** until it is empty. Each "Start declaration" opens one of
these pages; answer from `console.md`, Save, confirm "Change saved", go back.

| Page (`app-content/…`) | What to set |
|---|---|
| `privacy-policy` | the URL (native setter, then Save) |
| `ads-declaration` | Yes / No |
| `testing-credentials` (sign-in details) | "Is any part of your app restricted?" — No when there is no login and no paywall |
| `content-rating-overview` → questionnaire | email (support address); category (`All Other App Types` for a utility); IARC terms; then **every** question answered from `console.md` — watch "Is the app primarily a news or educational product?": an education app is **Yes**. Next is disabled until the page is saved: **Save**, then **Next**, then **Save** on the summary. Record the ratings shown |
| `target-audience-content` | the age groups ticked in `console.md`; under-13 groups pull the app into the Families policy, so only if `console.md` says so. The later steps are skipped when no child group is ticked |
| `data-privacy-security` (Data safety) | Step 2: collects/shares yes/no, encryption in transit, account creation method ("My app does not allow users to create an account"), external login, deletion request. Step 3: expand each category with its **Show** button (collapsed sections do not render their checkboxes) and tick the data types. Step 4: per type, **Start** → collected / shared, ephemeral, required/optional, purposes — same for both lists — Save in the dialog. Step 5: check the preview, Save |
| `ad-id-declaration` | Yes + purposes, when the merged manifest has `AD_ID` |
| `government-apps` | No, unless stated |
| `finance` | the last checkbox, "My app doesn't provide any financial features", after "Or" — Next — Save |
| `health` | "My app does not have any health features" — Next — Save (verify which box is ticked by screenshot; its label reads as "Or" in the DOM) |

Finish when the overview shows **Actioned** only and no "Start declaration" button remains.

## Step 4 — Store settings (`store-settings`)

- **App category** → Edit → category from `console.md` → Save (inside the dialog).
- **Store listing contact details** → Edit → email, website. The website field keeps
  whatever scheme it already shows; if it saved as `http://`, edit again and enter the full
  `https://…`. Phone only if `console.md` gives one.
- Leave External marketing as it is.

## Step 5 — Store listing (`main-store-listing`)

Default locale first, then each translation.

**Text.** Set title, short and full description (Step 0 recipe), confirm the counters
(`n / 30`, `n / 80`, `n / 4000`) and that no "Add a … description" error remains.

**Graphics.** One slot at a time — App icon, Feature graphic, Phone, 7-inch, 10-inch:

1. **Asset names must be unique across slots.** The asset library matches by file name, so
   a phone `01-home.jpg` and a tablet `01-home.jpg` collide. Copy the files to a scratch
   `docs/store/<app>/.upload/` with prefixes (`t7-`, `t10-`, `<locale>-p-`, …) and upload those.
2. Open the slot's **Add assets** (the buttons are in page order; identify a slot by its
   section heading, not by index — a filled slot keeps its button). The panel's
   `input[type=file]` exists only while the panel is open: `find` it, then `file_upload`.
3. A panel opened from the **feature graphic** slot accepts a single file; the rest of a
   multi-file upload silently disappears. Upload screenshot sets from a screenshot slot.
   The icon panel refuses the 1024×500 feature graphic as "too small" — that is the wrong
   panel, not a bad file.
4. Wait until every uploaded tile is present, click **deselect**, then tick the files in
   store order, **three per call**: `tile.querySelector('a[aria-label="Select button"]').click()`,
   checking `Selected asset` in the tile text after each.
5. **Add**, then read the slot back: its counter (`6 / 8`) and
   `[...section.querySelectorAll('img')].map(i => i.alt)` in order. Before the page is saved
   the alts are the file names; after Save they become generic. A duplicated set is removed
   with the slot's own **delete** buttons, last ones first.
6. **Save** the page and confirm "Change saved".

**Translations.** **Manage translations** → **Select languages** → tick the locale (find the
checkbox by its label, e.g. `Indonesian – id`) → **Apply**. The language switcher now opens
that listing. Its graphics slots show the default's images greyed out: **Add assets**
replaces them for that locale only — no delete needed. Fill text and graphics as above,
Save.

## Step 6 — What is left for the user

Open the dashboard and expand **View tasks**. Report, and stop:

- **Testers** — internal testing needs an email list; if the account has none, the user
  creates it. Do not create one.
- **The bundle** — built with the upload key. An AAB signed with the debug key (no
  `keystore.properties` on this machine) is rejected at upload; check with
  `keytool -printcert -jarfile <aab>` and say so if the owner is `CN=Android Debug`.
- **Release notes** — paste `docs/store/<app>/changelogs.txt` into the release's notes box
  as is: it already holds every locale in `<locale>…</locale>` tags. Check that
  `docs/store/<app>/changelogs.version` matches the bundle's `versionCode`; if the file is
  missing or older, run `/play-store:listing <app> --notes` first; the per-locale copies are
  in `fastlane/metadata/android/<locale>/changelogs/<versionCode>.txt`.
- Countries are chosen when the release is created.
- A personal developer account created after November 2023 needs a closed test (12
  testers, 14 days) before production; the Console says so if it applies.

## Step 7 — Report and record

- Table: part → page → value set → "Change saved" seen (yes/no). Content rating results.
- Listing: per locale, text counters and per slot the file order as read back.
- Anything skipped, and why.
- Record the state where the project keeps it (android-factory:
  `docs/monetization-setup.md`, one section per app), and delete
  `docs/store/<app>/.upload/`.
