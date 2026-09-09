---
description: Write the Play Store text listing per locale into the fastlane metadata layout
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(date*), Bash(mkdir*), Bash(ls*), Bash(cat*), Bash(cp*), Bash(node*), Bash(wc*), Bash(fastlane*), Bash(test*)
---

# Listing — title, short description, full description

Writes the text the Play Console asks for, per locale, in the layout `fastlane supply`
reads, so the result can be uploaded without copy-paste. Also copies the images produced by
`/play-store:enhance` into the same layout. Needs nothing installed except for `--upload`.

## Arguments

Read `$ARGUMENTS`:

| Argument | Meaning |
|---|---|
| `<app>` | Required. Same form as in `/play-store:capture`. |
| `--locale <a,b>` | Locales to write. Default: `locales:` from `screens.yml`, else `en`. |
| `--from <path>` | Use an existing listing text (a `.md` or `.txt` the user wrote) as the source instead of the Q&A. Never rewrite the source file. |
| `--images` | Only refresh the image folders from `docs/store/<app>/out/`. |
| `--upload` | After writing, run `fastlane supply` for metadata only (asks first; needs fastlane + a configured `Appfile`/JSON key). |

## Absolute rules

- **Character budgets are hard limits, counted by script, not by eye.** Title 30,
  short description 80, full description 4000. Count with
  `node -e "process.stdout.write(String([...require('fs').readFileSync(process.argv[1],'utf8').trim()].length))" <file>`
  (code points, which is what the Console counts). Over budget = rewrite, not truncate.
- **Google Play metadata policy** (title, short description, and the first lines of the
  full description): no "free", "best", "#1", "top", "new", ALL CAPS words, emoji, or
  promotional phrases ("download now", "50% off"); no competitor names; no claims the app
  doesn't fulfil; no keyword lists. If the user insists on one of these, say it may be
  rejected and write it anyway.
- **No invented features.** Everything in the description must exist in the code, the PRD,
  or the user's answers. When unsure, ask.
- **Each locale is written in that language**, not translated word-for-word. Idiomatic
  Indonesian for `id`, plain English for `en`.
- **Existing listing files belong to the user.** If `fastlane/metadata/android/<locale>/`
  already has text, show a diff-style comparison and ask before overwriting.
- Run `date +%Y-%m-%d` for the date.

## Step 1 — Gather

1. Resolve the app module (as in `/play-store:capture` Step 1): `applicationId`, `app_name`
   from `strings.xml`, version name.
2. Read, if they exist: `docs/apps/<app>/PRD.md` (android-factory), `docs/store/<app>/screens.yml`,
   `docs/store/<app>/captions.<locale>.yml` (headlines are already benefit-first copy),
   `README` of the app module, and the feature modules it depends on (names are a good
   feature inventory).
3. Existing listing: `fastlane/metadata/android/*/` or `docs/store/<app>/listing.md`.

## Step 2 — Q&A (skip what's already answered by the documents or `--from`)

Ask one at a time, at most six questions:

1. Who is this app for, in one sentence? (the audience shapes the vocabulary)
2. The one thing it does better than the alternatives?
3. Play category (e.g. Business, Shopping, Finance, Productivity)?
4. Three to five search terms a user would type to find it. These are the ASO keywords;
   `/play-store:aso` reuses them.
5. Anything that must be mentioned (login required, subscription, region-only, needs a
   companion device) or must not be mentioned?
6. Tone: neutral, friendly, or formal?

## Step 3 — Write per locale

Draft all three fields per locale, show them with the character count next to each, and
ask for approval before writing files.

**Title (≤30):** `<App name>` or `<App name>: <2-3 word descriptor>` when the name alone
doesn't say what the app does. The descriptor carries the top keyword. No punctuation
tricks, no repeated brand.

**Short description (≤80):** one sentence, the main benefit plus the top two keywords,
readable on its own; it shows above the fold on the store page.

**Full description (≤4000):**
- First 2 lines (about 160 chars): the pitch. They're visible before "Read more" and are
  what the ranking reads first. Include the top keyword naturally.
- Then 3 to 6 short feature blocks, each a bold-free plain-text line as a heading (the
  Console renders no markdown, only line breaks) followed by 1 to 3 sentences. Order them
  the same way as the screenshots, so the text and images tell one story.
- Then "who it's for" and any required disclosures (permissions, account, subscription).
- Close with a single line of contact / support.
- Keyword use: each target keyword 2 to 4 times across the full text, never in a row, never
  in a list. The Console penalizes stuffing.
- Plain text only. Line breaks separate paragraphs. Unicode bullets (`•`) are fine.

## Step 4 — Write files

```
fastlane/metadata/android/<locale-code>/
  title.txt
  short_description.txt
  full_description.txt
  images/
    phoneScreenshots/      <- copied from docs/store/<app>/out/<locale>/01-*.jpg ...
    featureGraphic.<ext>   <- copied from docs/store/<app>/out/<locale>/featureGraphic.*
    icon.png               <- ic_launcher-playstore.png (512x512) if found
```

Locale folder codes: `id`, `en-US`, `ms`, `en-GB`, `ja-JP`, etc. (the Play Console codes,
not Android resource qualifiers). Map `en` -> `en-US`.

For a multi-app project, the fastlane folder is per app module:
`<app-module>/fastlane/metadata/android/...`. If the project already has a `fastlane/` at
the root with a per-app layout, follow the existing convention instead.

Write all files, then re-count every text file with the script and print the table:

| Locale | Title | Short | Full | Images |
|---|---|---|---|---|
| id | 24/30 | 71/80 | 1980/4000 | 6 + feature |

Also write `docs/store/<app>/listing.md`: every locale's three fields in one readable page
with the counts, the keyword list from Q&A item 4, and the date. This is the file
`/play-store:aso` audits.

## Step 5 — `--upload` (optional)

Only if `fastlane --version` runs and `<app-module>/fastlane/Appfile` (or the root one)
names `json_key_file` and `package_name`. Show the command, ask, then run:

```
fastlane supply --skip_upload_apk --skip_upload_aab --skip_upload_changelogs \
  --metadata_path <app-module>/fastlane/metadata/android -p <applicationId>
```

Otherwise print the command and the two setup steps (service account JSON in the Play
Console, `fastlane supply init`) and stop.

## Re-runs

Re-running with the same answers rewrites the same text. Use `--images` after every
`/play-store:enhance` run to refresh only the pictures.
