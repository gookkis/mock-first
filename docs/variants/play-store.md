# Play Store — screenshots, listing, and ASO for a multi-app Android project

> **Kenapa berkas ini tidak ada di dalam `variants/<nama>/`?** Claude Code memperlakukan
> **setiap** `.md` di folder command sebagai sebuah command — sebuah `README.md` di sana akan
> muncul sebagai `/<nama>:README` di daftar skill. Dokumentasi varian karena itu hidup di
> `docs/variants/`. Jangan dipindahkan kembali.


Companion command set for the [Android apps factory variant](../android-factory/). It takes
an app module from `settings.gradle` and produces everything the Play Console asks for:
raw screenshots at the right sizes, "marketing" screenshots (device frame + background +
caption), the text listing, and an ASO audit.

## Commands

| Command | What it does |
|---|---|
| `/play-store:doctor` | Checks that every tool below is installed and reports what's missing, with install commands per OS. Read-only; `--fix` installs Playwright into the project after asking. |
| `/play-store:capture <app>` | Detects whether the app is Compose or Views and how its screens can be reached (declared deep link, own Activity, or walking the UI), then builds, installs, sets a clean demo status bar, turns animations off, walks the screens listed in `docs/store/<app>/screens.yml`, captures each via ADB, and looks at every capture to reject blank or broken ones. Raw output only. `--from-previews` renders `@Preview` composables instead, when the project already has Compose preview screenshot testing configured. |
| `/play-store:enhance <app>` | Renders each raw capture into a 1080x1920 marketing image (device frame, brand gradient, localized headline) plus the 1024x500 feature graphic. HTML templates rendered by Playwright; styles `bleed`, `tilt`, `clean`, `plain`; tablet layout with `--tablet`. |
| `/play-store:listing <app>` | Guided Q&A, then writes title / short / full description (and release notes) per locale into `fastlane/metadata/android/<locale>/`, copies the images alongside, and enforces the 30/80/4000/500 budgets by script. It also scans the app's permissions and SDKs and writes `docs/store/<app>/console.md`: the answer sheet for Store settings and every App content declaration — privacy policy, app access, ads, content rating, target audience, data safety, advertising ID, and the government / financial / health / news forms. `--upload` runs `fastlane supply` for metadata; declarations stay manual because the Console has no API for them. |
| `/play-store:aso <app>` | Scores the listing on keywords, budgets, policy risk, localization, and screenshots; fetches competitor store pages for a side-by-side; writes a dated report with ranked rewrites. Honest about what it can't know (search volume). |

## Pipeline
```
/play-store:doctor
/play-store:capture tokoku            # -> docs/store/tokoku/raw/<locale>/NN-id.png
/play-store:enhance tokoku            # -> docs/store/tokoku/out/<locale>/NN-id.jpg + featureGraphic.jpg
/play-store:listing tokoku            # -> tokoku/fastlane/metadata/android/<locale>/{title,short_description,full_description}.txt + images/
                                      #    + docs/store/tokoku/listing.md + docs/store/tokoku/console.md (App content answers)
/play-store:aso tokoku --competitors com.example.a,com.example.b
```
Each app module in `settings.gradle` gets its own `docs/store/<app>/` folder; the render
tooling in `docs/store/_tools/` is shared by all of them.

## Install
```bash
mkdir -p ~/.claude/commands/play-store
cp *.md ~/.claude/commands/play-store/
```
The commands become `/play-store:doctor`, `/play-store:capture`, and so on.

## First run
```bash
/play-store:doctor            # what's installed, what's missing, what each command needs
/play-store:doctor --fix      # also installs Playwright + Chromium into the project (asks first)
```

## What the machine needs

| Tool | Required by | Notes |
|---|---|---|
| JDK 17+ | capture | Android Gradle Plugin 8 needs 17. The JDK bundled with Android Studio works. |
| Android SDK: platform-tools (ADB), emulator, at least one AVD or a USB device | capture | Set `ANDROID_HOME`, or let doctor read `local.properties`. |
| Node.js 18+ and npm | capture (resize), enhance | |
| Playwright + Chromium | enhance | Project-local: `npm i -D playwright && npx playwright install chromium`. Doctor's `--fix` does this. |
| fastlane | listing `--upload` only | Optional. |
| ImageMagick | none | Optional fallback resizer. |

`/play-store:listing` and `/play-store:aso` need nothing installed.

## Play Store asset specs the commands target

| Asset | Spec |
|---|---|
| Phone screenshots | JPEG or 24-bit PNG (no alpha), 320-3840 px per side. 2 minimum to publish; 4 or more at 1080 px+ (9:16 portrait 1080x1920, 16:9 landscape 1920x1080) to be eligible for Play's promotional surfaces. `/enhance` renders 1080x1920. |
| Tablet / Chromebook screenshots | **At least 4**, 16:9 or 9:16, 1080-7680 px. Missing them downranks the app on large screens. |
| Feature graphic | 1024x500 JPEG or 24-bit PNG (no alpha), required |
| App icon | 512x512 32-bit PNG, 1 MB max |
| Preview video | YouTube URL, public or unlisted, ads off, not age-restricted, embeddable |

Modern phones render at 9:20 (1080x2400). That's fine: `/play-store:enhance` places the raw
capture inside a 1080x1920 canvas, so the device ratio never has to match the store ratio.

## Document layout
```
docs/store/<app>/
  screens.yml            <- which screens to capture and how to reach them (deep link / activity / taps)
  captions.<locale>.yml  <- one headline per screenshot, per language
  theme.yml              <- brand colors, font, frame style for /enhance
  raw/                   <- untouched ADB captures
  out/                   <- final 1080x1920 images + feature graphic
  listing.md             <- the approved text per locale + keywords (read by /aso)
  console.md             <- Store settings + every App content declaration, pre-answered
  data-safety.md         <- Data safety answers (shared with /portfolio:privacy)
fastlane/metadata/android/<locale>/
  title.txt  short_description.txt  full_description.txt  video.txt
  changelogs/<versionCode>.txt
  images/phoneScreenshots/  images/tenInchScreenshots/  images/featureGraphic.png  images/icon.png
```
