---
description: Turn raw captures into Play Store marketing screenshots (frame, background, caption) and the feature graphic
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(date*), Bash(mkdir*), Bash(ls*), Bash(cat*), Bash(node*), Bash(npx*), Bash(npm*), Bash(test*), Bash(cp*), Bash(python3*)
---

# Enhance — store-ready images from raw captures

Takes the captures in `docs/store/<app>/raw/<locale>/` and renders, for each one, a
1080x1920 image in the style of the paid screenshot generators: brand background, headline,
a device frame around the capture. Also renders the 1024x500 feature graphic and, on
request, plain padded screenshots without decoration.

Rendering is an HTML template screenshotted by Playwright (Chromium), so every style is
just CSS you can edit. Needs Node.js + Playwright: `/play-store:doctor --fix` installs them.

## Arguments

Read `$ARGUMENTS`:

| Argument | Meaning |
|---|---|
| `<app>` | Required. Same form as in `/play-store:capture`. |
| `--locale <xx>` | Only this locale. Default: every locale that has a `raw/<locale>/index.json`. |
| `--style <name>` | `bleed` (default), `clean`, `tilt`, `plain`, or `strip`. One style for the whole set. Overrides `theme.yml`. |
| `--style strip` | The look of the paid generators (AppLaunchpad and the like): the whole set drawn as **one wide canvas** and cut into 1080x1920 images, so the gradient runs on from image to image; a **hero** first (big title, app icon, laurel fact badges) whose tilted device **straddles images 1 and 2**; every caption a heavy plain line plus the keyword in a tilted white **sticker pill** with a hard accent shadow; big devices with a thin bright ring, bleeding off the bottom (`fit: true` shows the whole device). One extra image (the hero), so at most 7 captures. RTL locales are mirrored (Play's carousel runs right-to-left there). |
| `--vary` | Vary the layout screen by screen (`showcase.html`): a hero first, then `bleed`, `float`, `tilt`, `clean`, `zoom` (callout on a detail), `duo` (two screens), optional `panorama` background running across the set. Default when `theme.yml` has `vary: true`, which new themes get. |
| `--variants <2\|3>` | Render 2–3 alternative sets (A, B, C: different layout sequence, panorama on/off, light surface) side by side for the owner to choose, plus one contact sheet per set. |
| `--pick <A\|B\|C>` | Promote one variant set into `out/<locale>/` and record it in `theme.yml` (`variant:`), so later runs reproduce it. |
| `--screen <id>` | Only this screen. Repeatable. |
| `--feature` | Only the feature graphic. |
| `--tablet [7\|10]` | `10` (default for a bare `--tablet`): `raw-tablet/` → `out-tablet/` at 2560x1440 with `tablet.html`. `7`: `raw-tablet7/` → `out-tablet7/` at **1080x1920** with `tablet7.html`. Play accepts only 16:9 or 9:16 for screenshots — a 1200x1920 render (the 7" AVD's own size) is rejected. |
| `--png` | Output 24-bit PNG instead of JPEG. |
| `--dry-run` | Write the job file and the HTML for the first screen, open nothing, render nothing. |

## Absolute rules

- **Never touch the raw captures.** Read from `raw/`, write to `out/`.
- **Captions are content, not decoration.** Never invent a claim the app doesn't deliver.
  Every headline must describe something visible in that screenshot.
- **No two neighbours look alike.** A set where every image is the same frame on the same
  gradient reads as a template (the most common owner revision). With `--vary`, two adjacent
  screens never share a layout and the first one is the hero.
- **Keep the headline short**: at most 5 words in the `bleed`/`tilt` styles, 7 in `clean`.
  A store visitor reads it in one second on a phone thumbnail.
- **Look at the output.** Open every rendered image with the Read tool. Overflowing text,
  a clipped headline, or a device frame that hides the content is a failure; fix the
  caption or the template and re-render.
- Never run `npm install` or `npx playwright install` from here. If Playwright is missing,
  say so and point to `/play-store:doctor --fix`.
- **Badges are facts, never awards.** The `strip` hero's stats, note and laurels carry things the app
  verifiably is ("140 questions, 115 minutes", "Offline, no account", "10 languages") - never "#1", "Editors' Choice",
  "5 star rating", download counts or press quotes the app has not earned. Play's metadata policy
  treats unearned rankings and awards in store graphics as misleading.
- Run `date +%Y-%m-%d` for the date.

## Step 1 — Inputs

1. Resolve the app module (as in `/play-store:capture` Step 1).
2. Read `docs/store/<app>/screens.yml` and each `raw/<locale>/index.json`. If there are no
   captures, stop and point to `/play-store:capture <app>`.
3. Check Playwright: `node -e "require.resolve('playwright')"` from the project root, and a
   `chromium-*` folder in the Playwright browsers cache (see doctor P4). If either is
   missing, stop.

## Step 2 — Theme (`docs/store/<app>/theme.yml`)

If missing, derive a draft from the app:
- Colors: `res/values/colors.xml`, Compose `Color.kt`/`Theme.kt`, or the launcher icon
  background (`ic_launcher_background.xml`). Pick the primary as `bg1`, a darker or hue-shifted
  companion as `bg2`. Text white on a dark gradient, near-black on a light one. Check the
  contrast: headline on background must be at least 4.5:1.
- Name: `app_name` in `strings.xml`.
- Icon: `ic_launcher-playstore.png` in the app module, if present.

Show the draft and ask the user to confirm or paste their own colors. Then write:

```yaml
# docs/store/<app>/theme.yml — read by /play-store:enhance
style: bleed              # bleed | clean | tilt | plain
font: Inter               # any Google Font name; falls back to system sans
bg1: "#1E3A8A"
bg2: "#7C3AED"
text: "#FFFFFF"
accent: "#F59E0B"         # underline / highlight color
frame: "#0B0B0F"          # device bezel color
name: Tokoku
tagline: Kelola toko dari genggaman     # feature graphic subtitle (per-locale override in captions)
icon: tokoku/src/main/ic_launcher-playstore.png
output: jpeg              # jpeg | png
vary: true                # per-screen layouts from showcase.html (see Step 5)
layouts: auto             # auto | a list per screen, e.g. [hero, bleed, zoom, float, duo, clean]
panorama: false           # one background running across the whole set
surface: "#F6F4EF"        # light-surface variant (variant C): background
ink: "#14141A"            #   and text colour on it
art:                      # optional decorations: the app's own illustrations (transparent PNG/WebP),
  - tokoku/src/main/res/drawable-nodpi/art_box.webp   # never stock clip art the app doesn't use
variant: A                # set by --pick
# --- style: strip (optional keys; defaults in brackets)
strip_font: Archivo       # heavy display face with a 900 weight [font]
bg3: "#14B8A6"            # brightest stop in the middle of the strip [bg2]
ring: "#5EEAD4"           # thin ring around the device bezel [white 35%]
pill: "#FFFFFF"           # keyword sticker [#FFFFFF]
pillink: "#0B1F1D"        # text on the sticker [#111111]
hero_style: normal        # normal | italic (the hero title) [normal]
```

## Step 3 — Captions (`docs/store/<app>/captions.<locale>.yml`)

If missing for a locale, draft one headline (+ optional sub-line) per screen from:
`title:` in `screens.yml`, the PRD's description of that screen, and what is visible in the
raw capture (open it). Write benefit-first ("Semua pesanan dalam satu layar"), not
feature-first ("Daftar pesanan"). Same language as the locale. Show the draft, ask, write:

```yaml
locale: id
tagline: Kelola toko dari genggaman
screens:
  home:     { headline: "Semua pesanan dalam satu layar", sub: "Pantau status tanpa buka-tutup aplikasi" }
  catalog:  { headline: "Katalog rapi, cari dalam sekejap" }
  checkout: { headline: "Checkout dua ketukan" }
  stats:    { headline: "Tahu produk terlaris", layout: zoom, zoom: { x: 50%, y: 35% } }   # magnify a detail
  orders:   { headline: "Dua layar, satu alur", layout: duo, pair: checkout }              # second screen behind
  promo:    { headline: "Diskon otomatis", chips: "Tanpa login · Offline · Gratis" }       # one pill line
```

`layout`, `zoom` (percent position of the detail inside the capture), `pair` (another
screen id) and `chips` are optional and only read with `--vary`.

**`--style strip`** reads a `strip:` block instead (add it next to `screens:`):

```yaml
strip:
  hero: { title: "Full ITP English\ntest simulation", note: "New packs added",
          stats: [{ v: "140", l: "questions" }, { v: "115", l: "minutes" }, { v: "310–677", l: "score scale" }] }
  screens:
    home:     { top: "Full simulation", pill: "115 minutes" }   # 1st screen: the device shared with the hero
    result:   { top: "Your score", pill: "by section" }
    explain:  { top: "Mistakes", pill: "explained", fit: true }  # whole device: the point sits low on screen
```

`top` is the plain line (2-3 words), `pill` the keyword in the sticker (1-2 words) - together at most
5 words, the keyword being the one the store visitor should remember. A hero `title` with `\n` keeps
exactly those lines (no wrapping) and the font shrinks until the longest one fits, so write short lines
(about 8-12 Latin characters, 4-6 CJK) - long lines come out small. Without `\n` the title wraps freely.
`\n` in `top` is a plain line break. Always break CJK and Thai titles yourself. Prefer `stats` (up to three number cards: big value in the accent colour, short label) over `badges`
(laurels): laurels read as awards, so plain facts inside them look borrowed and the small text
doesn't sell. Use numbers the app guarantees (format, time limit, scale) - never a content count that
grows with downloads or updates, it goes stale on the first new pack. `note` is a short pill under the
title (≤ ~22 characters, it must not reach the device). `badges` still works: two, ≤ ~20 characters. Use `fit: true` wherever the
thing the caption names is in the lower third of the capture - the bleeding device crops it.

A screen with no entry falls back to its `title:` from `screens.yml` and gets a warning in
the report.

## Step 4 — Tooling (written once, shared by every app)

If `docs/store/_tools/render.mjs` or any template is missing, write these files exactly.
They are plain files the user can edit; on later runs never overwrite one that exists.

### `docs/store/_tools/render.mjs`

```js
// Renders HTML templates to images with Playwright. Usage: node render.mjs <jobs.json>
// jobs.json: [{ template, width, height, out, format: "jpeg"|"png", quality, vars: {...}, images: { key: path } }]
import { chromium } from 'playwright';
import fs from 'node:fs';
import path from 'node:path';

const jobsFile = process.argv[2];
if (!jobsFile) { console.error('usage: node render.mjs <jobs.json>'); process.exit(1); }
const jobs = JSON.parse(fs.readFileSync(jobsFile, 'utf8'));
const base = path.dirname(path.resolve(jobsFile));
const abs = p => path.isAbsolute(p) ? p : path.resolve(base, p);

const dataUri = p => {
  const ext = path.extname(p).slice(1).toLowerCase();
  const mime = ext === 'jpg' ? 'image/jpeg' : ext === 'svg' ? 'image/svg+xml' : `image/${ext}`;
  return `data:${mime};base64,${fs.readFileSync(p).toString('base64')}`;
};
const esc = s => String(s).replace(/[&<>"]/g, c => ({ '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;' }[c]));

const browser = await chromium.launch();
try {
  for (const job of jobs) {
    const vars = {};
    for (const [k, v] of Object.entries(job.vars ?? {})) vars[k] = esc(v);
    for (const [k, p] of Object.entries(job.images ?? {})) vars[k] = fs.existsSync(abs(p)) ? dataUri(abs(p)) : '';
    const html = fs.readFileSync(abs(job.template), 'utf8').replace(/\{\{(\w+)\}\}/g, (_, k) => vars[k] ?? '');
    const page = await browser.newPage({ viewport: { width: job.width, height: job.height }, deviceScaleFactor: 1 });
    await page.setContent(html, { waitUntil: 'load' });
    await page.evaluate(() => document.fonts.ready);
    await page.waitForTimeout(150);
    // A block overflows when its text sticks out of its own box or out of the canvas. Measured
    // with a Range so overflow in every direction counts (scrollHeight ignores overflow upwards,
    // which is exactly what happens to a bottom-aligned headline that wraps too far). Descenders
    // may poke a few px past the line box, so the tolerance is a third of the font size.
    const overflow = await page.evaluate(() => [...document.querySelectorAll('[data-check]')]
      .filter(el => {
        const tol = parseFloat(getComputedStyle(el).fontSize) * 0.34;
        const box = el.getBoundingClientRect();
        const r = document.createRange(); r.selectNodeContents(el);
        const c = r.getBoundingClientRect();
        if (c.width === 0 && c.height === 0) return false;
        return c.top < box.top - tol || c.bottom > box.bottom + tol || c.left < box.left - tol || c.right > box.right + tol
          || c.top < -tol || c.bottom > innerHeight + tol || c.left < -tol || c.right > innerWidth + tol;
      })
      .map(el => el.getAttribute('data-check')));
    const format = job.format ?? 'jpeg';
    const opts = format === 'jpeg' ? { quality: job.quality ?? 95 } : {};
    // "parts": one wide page cut into several images (strip.html), so a background or a device can
    // run on from one screenshot into the next. Each part: { out, x, y?, width?, height? }.
    const parts = job.parts ?? [{ out: job.out }];
    for (const part of parts) {
      const out = abs(part.out);
      fs.mkdirSync(path.dirname(out), { recursive: true });
      const clip = job.parts ? { x: part.x ?? 0, y: part.y ?? 0, width: part.width ?? 1080, height: part.height ?? job.height } : undefined;
      await page.screenshot({ path: out, type: format, ...opts, ...(clip ? { clip } : {}) });
      console.log(`${overflow.length ? 'OVERFLOW ' + overflow.join(',') + ' ' : ''}wrote ${part.out}`);
    }
    await page.close();
  }
} finally {
  await browser.close();
}
```

An existing copy without `job.parts` predates `--style strip`: replace its screenshot block with
the `parts` loop above (the only edit made to an existing tool file).

### `docs/store/_tools/templates/phone.html`

Variables: `style`, `font`, `bg1`, `bg2`, `text`, `accent`, `frame`, `headline`, `sub`,
`shot` (image).

```html
<!doctype html>
<html><head><meta charset="utf-8">
<link href="https://fonts.googleapis.com/css2?family={{font}}:wght@500;800&display=swap" rel="stylesheet">
<style>
  html, body { margin: 0; width: 1080px; height: 1920px; overflow: hidden; }
  body { position: relative; font-family: "{{font}}", "Segoe UI", Roboto, system-ui, sans-serif;
         color: {{text}}; background: linear-gradient(160deg, {{bg1}} 0%, {{bg2}} 100%); }
  body::before { content: ""; position: absolute; inset: 0;
         background: radial-gradient(900px 700px at 20% 0%, rgba(255,255,255,.18), transparent 60%); }
  .headline { position: absolute; top: 130px; left: 72px; right: 72px; height: 230px; text-align: center;
         font-size: 84px; font-weight: 800; line-height: 1.08; letter-spacing: -1.5px;
         display: flex; align-items: flex-end; justify-content: center; }
  .headline em { font-style: normal; box-shadow: inset 0 -18px 0 {{accent}}; }
  .sub { position: absolute; top: 380px; left: 110px; right: 110px; height: 110px; text-align: center;
         font-size: 38px; font-weight: 500; line-height: 1.3; opacity: .88; }
  .device { position: absolute; left: 50%; background: {{frame}}; box-shadow: 0 60px 120px rgba(0,0,0,.45);
         border-radius: 112px; padding: 20px; }
  .screen { width: 100%; height: 100%; border-radius: 92px; overflow: hidden; background: #000; position: relative; }
  .screen img { display: block; width: 100%; height: 100%; object-fit: cover; object-position: top center; }
  .punch { position: absolute; top: 40px; left: 50%; width: 34px; height: 34px; margin-left: -17px;
         border-radius: 50%; background: {{frame}}; z-index: 2; }
  /* bleed: big device, cut off at the bottom */
  .bleed .device { top: 540px; width: 860px; height: 1870px; margin-left: -430px; }
  /* tilt: same, rotated */
  .tilt .device { top: 600px; width: 860px; height: 1870px; margin-left: -430px; transform: rotate(-7deg); }
  /* clean: whole device visible */
  .clean .headline { top: 110px; font-size: 76px; }
  .clean .sub { top: 340px; }
  .clean .device { top: 500px; width: 620px; height: 1348px; margin-left: -310px; border-radius: 84px; padding: 16px; }
  .clean .screen { border-radius: 68px; }
  .clean .punch { top: 30px; width: 26px; height: 26px; margin-left: -13px; }
</style></head>
<body class="{{style}}">
  <div class="headline" data-check="headline"><span>{{headline}}</span></div>
  <div class="sub" data-check="sub">{{sub}}</div>
  <div class="device"><div class="screen"><div class="punch"></div><img src="{{shot}}" alt=""></div></div>
</body></html>
```

### `docs/store/_tools/templates/showcase.html`

The varied phone template (`--vary`, `--variants`). Same 1080x1920 canvas and variables as
`phone.html`, plus `layout` (`hero|bleed|float|tilt|clean|zoom|duo`), `extra` (`panorama`,
`light`, or both, space separated), `idx` / `count` (position in the set, for the panorama),
`zx` / `zy` (zoom focus, percent), `chips`, `name`, `surface`, `ink`, images `shot2`, `icon`,
`art1`, `art2`. Empty images are hidden.

```html
<!doctype html>
<html><head><meta charset="utf-8">
<link href="https://fonts.googleapis.com/css2?family={{font}}:wght@500;800&display=swap" rel="stylesheet">
<style>
  html, body { margin: 0; width: 1080px; height: 1920px; overflow: hidden; }
  body { position: relative; font-family: "{{font}}", "Segoe UI", Roboto, system-ui, sans-serif; color: {{text}};
         background: linear-gradient(160deg, {{bg1}} 0%, {{bg2}} 100%); --idx: {{idx}}; --count: {{count}}; }
  body.light { background: {{surface}}; color: {{ink}}; }
  body.panorama { background: linear-gradient(100deg, {{bg1}} 0%, {{bg2}} 50%, {{bg1}} 100%);
         background-size: calc(var(--count) * 1080px) 1920px; background-position: calc(var(--idx) * -1080px) 0; }
  body.panorama .orb { display: block; }
  .orb { display: none; position: absolute; top: 1180px; width: 440px; height: 440px; border-radius: 50%;
         background: {{accent}}; opacity: .35; }
  .orb.r { left: 860px; } .orb.l { left: -220px; }          /* halves meet across neighbouring images */
  body::before { content: ""; position: absolute; inset: 0;
         background: radial-gradient(900px 700px at 20% 0%, rgba(255,255,255,.16), transparent 60%); }
  img[src=""] { display: none; }
  /* decorations sit in the side margins beside the device, never under the headline */
  .art { position: absolute; width: 220px; z-index: 0; filter: drop-shadow(0 18px 30px rgba(0,0,0,.25)); }
  .art.a1 { top: 760px; left: -70px; transform: rotate(-12deg); }
  .art.a2 { top: 1260px; right: -60px; transform: rotate(10deg); }
  .brand { display: none; position: absolute; top: 120px; left: 0; right: 0; text-align: center; }
  .brand img { width: 150px; height: 150px; border-radius: 34px; box-shadow: 0 16px 40px rgba(0,0,0,.3); }
  .brand div { margin-top: 18px; font-size: 44px; font-weight: 800; letter-spacing: -.5px; }
  .headline { position: absolute; top: 130px; left: 72px; right: 72px; height: 230px; text-align: center; z-index: 1;
         font-size: 84px; font-weight: 800; line-height: 1.08; letter-spacing: -1.5px;
         display: flex; align-items: flex-end; justify-content: center; }
  .headline em { font-style: normal; box-shadow: inset 0 -18px 0 {{accent}}; }
  .sub { position: absolute; top: 380px; left: 110px; right: 110px; height: 110px; text-align: center; z-index: 1;
         font-size: 38px; font-weight: 500; line-height: 1.3; opacity: .88; }
  .chips { position: absolute; top: 470px; left: 50%; transform: translateX(-50%); white-space: nowrap; z-index: 1;
         padding: 14px 30px; border-radius: 999px; background: rgba(255,255,255,.18); font-size: 30px; font-weight: 600; }
  .light .chips { background: rgba(0,0,0,.06); }
  .chips:empty { display: none; }
  .device { position: absolute; left: 50%; background: {{frame}}; box-shadow: 0 60px 120px rgba(0,0,0,.45);
         border-radius: 112px; padding: 20px; top: 540px; width: 860px; height: 1870px; margin-left: -430px; z-index: 2; }
  .screen { width: 100%; height: 100%; border-radius: 92px; overflow: hidden; background: #000; }
  .screen img { display: block; width: 100%; height: 100%; object-fit: cover; object-position: top center; }
  .device.back { display: none; }
  .callout { display: none; position: absolute; top: 880px; right: 40px; width: 460px; height: 460px; border-radius: 50%;
         border: 12px solid {{accent}}; box-shadow: 0 30px 80px rgba(0,0,0,.45); z-index: 3;
         background-size: 260%; background-position: {{zx}} {{zy}}; background-repeat: no-repeat; }
  /* layouts */
  .tilt .device { top: 600px; transform: rotate(-7deg); }
  .clean .device, .hero .device { top: 560px; width: 620px; height: 1348px; margin-left: -310px; border-radius: 84px; padding: 16px; }
  .clean .screen, .hero .screen { border-radius: 68px; }
  .hero .brand { display: block; }
  .hero .headline { top: 360px; height: 170px; font-size: 72px; }
  .hero .sub, .hero .chips { display: none; }
  .hero .device { top: 620px; }
  .float .device { top: 110px; width: 660px; height: 1430px; margin-left: -330px; border-radius: 88px; padding: 18px; }
  .float .screen { border-radius: 72px; }
  .float .headline { top: 1650px; height: 210px; align-items: flex-start; font-size: 72px; }
  .float .sub, .float .chips { display: none; }
  .duo .device { top: 620px; width: 640px; height: 1392px; margin-left: -150px; transform: rotate(5deg); border-radius: 86px; padding: 16px; }
  .duo .device.back { display: block; margin-left: -500px; top: 700px; transform: rotate(-7deg); z-index: 1; opacity: .96; }
  .duo .screen { border-radius: 70px; }
  .zoom .callout { display: block; }
</style></head>
<body class="{{layout}} {{extra}}">
  <div class="orb l"></div><div class="orb r"></div>
  <img class="art a1" src="{{art1}}" alt=""><img class="art a2" src="{{art2}}" alt="">
  <div class="brand"><img src="{{icon}}" alt=""><div>{{name}}</div></div>
  <div class="headline" data-check="headline"><span>{{headline}}</span></div>
  <div class="sub" data-check="sub">{{sub}}</div>
  <div class="chips" data-check="chips">{{chips}}</div>
  <div class="device back"><div class="screen"><img src="{{shot2}}" alt=""></div></div>
  <div class="device"><div class="screen"><img src="{{shot}}" alt=""></div></div>
  <div class="callout" style="background-image: url('{{shot}}')"></div>
</body></html>
```

### `docs/store/_tools/templates/strip.html`

`--style strip`. One page `count x 1080` wide, laid out by its own script from the `panels` JSON;
render.mjs cuts it with `parts`. Variables: `font`, `bg1`, `bg2`, `bg3`, `text`, `accent`, `frame`,
`ring`, `pill`, `pillink`, `herostyle`, `lang`, `dir`, `cjk` (JP/KR/TC/SC: which Noto CJK face backs
the display font), `panels`; images `shot0`…`shot7`, `icon`.

```html
<!doctype html>
<!--
  strip.html — the whole phone set as ONE canvas (count x 1080 by 1920), cut into 1080x1920
  screenshots by render.mjs "parts". Because it is one page, the background runs on from image to
  image and a device can straddle two images (the "span" look of the paid generators).

  Panels come from the data-panels attribute (JSON, HTML-escaped by render.mjs):
    [{ "kind": "hero",  "title": "...", "note": "New packs added", "stats": [{"v": "140", "l": "questions"}],
       "badges": ["..."], "icon": true },   // stats (number cards) or badges (laurels), note = pill under title
     { "kind": "span",  "top": "...", "pill": "...", "shot": "shot0" },      // device shared with the panel before
     { "kind": "phone", "top": "...", "pill": "...", "shot": "shot1", "tilt": 0, "fit": false },
     ...]
  "top" is the plain line, "pill" the keyword in the white sticker. Images shot0..shot7 and icon are
  hidden <img>s the script reads by id.
-->
<html><head><meta charset="utf-8">
<link href="https://fonts.googleapis.com/css2?family={{font}}:ital,wght@0,800;0,900;1,900&display=swap" rel="stylesheet">
<style>
  :root { --w: 1080px; --h: 1920px; --bg1: {{bg1}}; --bg2: {{bg2}}; --bg3: {{bg3}}; --text: {{text}};
          --accent: {{accent}}; --frame: {{frame}}; --ring: {{ring}}; --pill: {{pill}}; --pillink: {{pillink}}; }
  html, body { margin: 0; height: var(--h); overflow: hidden; }
  body { position: relative; color: var(--text);
         font-family: "{{font}}", "Noto Sans Thai", "Noto Sans Arabic", "Noto Sans CJK {{cjk}}", system-ui, sans-serif;
         /* one gradient across the whole strip: dark - bright - dark - bright ... */
         background: linear-gradient(100deg, var(--bg1) 0%, var(--bg2) 22%, var(--bg3) 45%, var(--bg2) 70%, var(--bg1) 100%); }
  img.src { display: none; }
  .blob { position: absolute; border-radius: 50%; filter: blur(70px); opacity: .30; pointer-events: none; }
  .panel { position: absolute; top: 0; width: var(--w); height: var(--h); }

  /* sticker caption: plain heavy line + keyword in a tilted white pill with a hard shadow */
  .cap { position: absolute; top: 110px; left: 60px; right: 60px; height: 380px; z-index: 5; text-align: center;
         display: flex; flex-direction: column; align-items: center; justify-content: flex-end; gap: 14px;
         padding: 0 0 26px; box-sizing: border-box; }  /* room for the tilted pill's corners + shadow */
  .cap .top { font-size: 96px; font-weight: 900; line-height: 1.02; letter-spacing: -2px; text-wrap: balance; white-space: pre-line;
         text-shadow: 0 6px 24px rgba(0,0,0,.25); }
  .cap .pill { display: inline-block; font-size: 92px; font-weight: 900; line-height: 1.1; letter-spacing: -2px;
         background: var(--pill); color: var(--pillink); padding: 6px 44px 14px; border-radius: 999px;
         transform: rotate(-3deg); box-shadow: 10px 12px 0 var(--accent); max-width: 920px; text-wrap: balance; }
  .cap .pill:empty, .cap .top:empty { display: none; }

  /* device: thin bright ring around a dark bezel */
  .device { position: absolute; z-index: 3; background: var(--frame); border-radius: 104px; padding: 18px;
         box-shadow: 0 0 0 7px var(--ring), 0 70px 140px rgba(0,0,0,.45); }
  .device .screen { width: 100%; height: 100%; border-radius: 88px; overflow: hidden; background: #000; position: relative; }
  .device img { display: block; width: 100%; height: 100%; object-fit: cover; object-position: top center; }
  .device .punch { position: absolute; top: 36px; left: 50%; width: 30px; height: 30px; margin-left: -15px;
         border-radius: 50%; background: var(--frame); z-index: 2; }

  /* hero: big left-aligned title, brand row, honest fact badges at the bottom-left */
  .hero .brand { position: absolute; top: 120px; left: 80px; display: flex; align-items: center; gap: 26px;
         font-size: 54px; font-weight: 900; letter-spacing: -1px; }
  .hero .brand img { width: 120px; height: 120px; border-radius: 30px; box-shadow: 0 14px 36px rgba(0,0,0,.3); }
  .hero .head { position: absolute; top: 300px; left: 80px; width: 520px; z-index: 5; display: flex; flex-direction: column; align-items: flex-start; gap: 40px; }
  .hero .title { width: 520px;
         font-size: 112px; font-weight: 900; font-style: {{herostyle}}; line-height: .98; letter-spacing: -3px;
         text-wrap: balance; text-shadow: 0 8px 30px rgba(0,0,0,.25); white-space: pre-line; word-break: keep-all; }  /* \n in the title = line break */
  .hero .badges { position: absolute; left: 60px; bottom: 120px; width: 600px; z-index: 5;
         display: flex; flex-direction: column; gap: 34px; align-items: flex-start; }
  .badge { display: flex; align-items: center; gap: 6px; }
  .badge svg { width: 70px; height: 150px; flex: none; }
  .badge svg.r { transform: scaleX(-1); }
  .badge span { font-size: 50px; font-weight: 900; line-height: 1.05; text-align: center; max-width: 400px; }
  /* stats: big number + small label, the honest alternative to award laurels */
  .hero .stats { position: absolute; left: 80px; bottom: 110px; width: 520px; z-index: 5; display: flex; flex-direction: column; gap: 22px; }
  .stat { display: flex; flex-wrap: wrap; align-items: baseline; column-gap: 22px; row-gap: 4px; padding: 22px 34px; border-radius: 36px;
         background: rgba(255,255,255,.13); box-shadow: inset 0 0 0 3px rgba(255,255,255,.22); }
  .stat b { font-size: 92px; font-weight: 900; line-height: 1; letter-spacing: -2px; color: var(--accent); white-space: nowrap; direction: ltr; unicode-bidi: isolate; }
  .stat span { font-size: 40px; font-weight: 800; line-height: 1.1; }
  .hero .note { display: inline-block; font-size: 40px; font-weight: 800; padding: 10px 30px 14px;
         border-radius: 999px; background: var(--pill); color: var(--pillink); transform: rotate(-2deg);
         box-shadow: 8px 9px 0 var(--accent); white-space: nowrap; }
  [dir="rtl"] .hero .stats { left: auto; right: 80px; }
  [dir="rtl"] .hero .head { left: auto; right: 80px; align-items: flex-end; }
  [dir="rtl"] .hero .title, [dir="rtl"] .hero .badges, [dir="rtl"] .hero .brand { left: auto; right: 80px; }
</style></head>
<body lang="{{lang}}" dir="{{dir}}" data-panels="{{panels}}">
  <img class="src" id="shot0" src="{{shot0}}"><img class="src" id="shot1" src="{{shot1}}">
  <img class="src" id="shot2" src="{{shot2}}"><img class="src" id="shot3" src="{{shot3}}">
  <img class="src" id="shot4" src="{{shot4}}"><img class="src" id="shot5" src="{{shot5}}">
  <img class="src" id="shot6" src="{{shot6}}"><img class="src" id="shot7" src="{{shot7}}">
  <img class="src" id="icon" src="{{icon}}">
<script>
  const W = 1080, H = 1920;
  const panels = JSON.parse(document.body.dataset.panels);
  document.body.style.width = (panels.length * W) + 'px';
  const src = id => (document.getElementById(id) || {}).src || '';
  const el = (tag, cls, html) => { const e = document.createElement(tag); if (cls) e.className = cls; if (html != null) e.textContent = html; return e; };
  const laurel = r => `<svg class="${r ? 'r' : ''}" viewBox="0 0 70 150" fill="currentColor"><path d="M58 146C22 126 6 92 10 50" stroke="currentColor" stroke-width="5" fill="none" stroke-linecap="round"/>${
    [[10,40,-30],[8,62,-20],[10,84,-10],[16,104,0],[26,122,12],[40,136,26]].map(([x,y,a]) => `<ellipse cx="${x+10}" cy="${y}" rx="8" ry="17" transform="rotate(${a} ${x+10} ${y})"/>`).join('')}</svg>`;

  // soft light blobs sitting on the seams, so the strip reads as one surface
  for (let i = 1; i < panels.length; i += 2) {
    const b = el('div', 'blob');
    Object.assign(b.style, { left: (i * W - 300) + 'px', top: (i % 4 === 1 ? 1150 : 250) + 'px', width: '600px', height: '600px', background: 'var(--accent)' });
    document.body.appendChild(b);
  }

  const device = (left, top, width, rot, shot) => {
    const d = el('div', 'device');
    Object.assign(d.style, { left: left + 'px', top: top + 'px', width: width + 'px', height: Math.round(width * 2400 / 1080 * 0.985) + 'px', transform: `rotate(${rot}deg)` });
    const s = el('div', 'screen'); s.appendChild(el('div', 'punch'));
    const im = el('img'); im.src = src(shot); s.appendChild(im); d.appendChild(s);
    document.body.appendChild(d);
  };

  // RTL stores show the carousel right-to-left: the hero is the rightmost image, so the strip is
  // laid out mirrored (render parts are mapped the same way by strip_jobs.py).
  const rtl = document.body.dir === 'rtl';
  panels.forEach((p, i) => {
    const x = (rtl ? panels.length - 1 - i : i) * W;
    const panel = el('div', 'panel ' + p.kind); panel.style.left = x + 'px';
    if (p.kind === 'hero') {
      const brand = el('div', 'brand');
      if (p.icon && src('icon')) { const ic = el('img'); ic.src = src('icon'); brand.appendChild(ic); }
      brand.appendChild(el('span', '', p.name || ''));
      panel.appendChild(brand);
      const head = el('div', 'head'); panel.appendChild(head);
      const t = el('div', 'title', p.title); t.dataset.check = 'hero-title'; head.appendChild(t);
      // A title written with \n keeps its lines; the font shrinks until the longest line fits.
      if (p.title.includes('\n')) t.style.whiteSpace = 'pre';   // keep the author's lines; the shrink loop below makes the longest fit
      t._fit = true;
      if (p.note) { const n = el('div', 'note', p.note); n.dataset.check = 'note'; head.appendChild(n); }
      if (p.stats) {
        const st = el('div', 'stats');
        p.stats.forEach(x => { const r = el('div', 'stat'); r.appendChild(el('b', '', x.v)); r.appendChild(el('span', '', x.l)); st.appendChild(r); });
        panel.appendChild(st);
      }
      const bs = el('div', 'badges');
      (p.badges || []).forEach(text => {
        const b = el('div', 'badge'); b.innerHTML = laurel(false); const s = el('span', '', text); s.dataset.check = 'badge'; b.appendChild(s);
        b.insertAdjacentHTML('beforeend', laurel(true)); bs.appendChild(b);
      });
      panel.appendChild(bs);
    } else {
      const cap = el('div', 'cap'); cap.dataset.check = 'caption-' + (i + 1);
      cap.appendChild(el('div', 'top', p.top || '')); cap.appendChild(el('div', 'pill', p.pill || ''));
      panel.appendChild(cap);
      if (p.kind === 'span') {
        // the device belongs to the previous (hero) panel's right half and this panel's left half
        if (rtl) device(x + W - 460, 560, 820, 9, p.shot);
        else device(x - 360, 560, 820, -9, p.shot);
      } else {
        // fit: the whole device on the canvas, for a screen whose point sits at the bottom of the capture
        if (p.fit) device(x + 235, 560, 610, p.tilt || 0, p.shot);
        else device(x + 130, 600, 820, p.tilt || 0, p.shot);
      }
    }
    document.body.appendChild(panel);
  });
  // shrink-to-fit pass for hero titles (after layout, once fonts are in)
  document.fonts.ready.then(() => document.querySelectorAll('.title').forEach(t => {
    let size = parseFloat(getComputedStyle(t).fontSize);
    const hero = t.closest('.panel'), head = t.parentElement;
    const below = hero.querySelector('.stats') || hero.querySelector('.badges');
    const room = below.getBoundingClientRect().top - 40;
    while (size > 64 && (t.scrollWidth > t.clientWidth + 2 || head.getBoundingClientRect().bottom > room)) {
      size -= 4; t.style.fontSize = size + 'px';
    }
  }));
</script>
</body></html>
```

### `docs/store/_tools/strip_jobs.py`

Builds the strip job (one per locale) from `theme.yml`, `captions.<locale>.yml` and
`raw/<locale>/index.json`. `STRIP_OUT=out-strip` writes somewhere other than `out/` for a trial run.

```python
#!/usr/bin/env python3
"""Build the strip.html jobs (one wide canvas per locale, cut into 1080x1920 images).

    python3 strip_jobs.py docs/store/<app> [locale ...] > docs/store/<app>/.jobs-strip.json

Reads theme.yml (bg1/bg2/bg3, text, accent, frame, ring, pill, pillink, font, name, icon) and each
captions.<locale>.yml `strip:` block:

    strip:
      hero: { title: "Full ITP English\ntest simulation", note: "New packs added",
          stats: [{ v: "140", l: "questions" }, { v: "115", l: "minutes" }, { v: "310–677", l: "score scale" }] }
      screens:
        home:   { top: "Full simulation", pill: "115 minutes" }   # first screen shares the hero's device
        result: { top: "Your score", pill: "by section" }

The first screen is drawn as a tilted device straddling the hero and panel 2; every other screen gets
its own panel. Output: out/<locale>/NN-<id>.jpg, hero first.
"""
import json
import os
import sys

import yaml

OUT = os.environ.get("STRIP_OUT", "out")
RTL = ("ar", "he", "fa", "ur")
CJK = {"ja": "JP", "ko": "KR", "zh-TW": "TC", "zh": "SC"}


def main(app_dir, locales):
    theme = yaml.safe_load(open(os.path.join(app_dir, "theme.yml")))
    if not locales:
        locales = sorted(d for d in os.listdir(os.path.join(app_dir, "raw")) if os.path.exists(os.path.join(app_dir, "raw", d, "index.json")))
    jobs = []
    for loc in locales:
        index = json.load(open(os.path.join(app_dir, "raw", loc, "index.json")))
        cap = yaml.safe_load(open(os.path.join(app_dir, f"captions.{loc}.yml")))
        strip = cap["strip"]
        screens = index["screens"]
        if len(screens) + 1 > 8:
            sys.exit(f"{loc}: {len(screens)} screens + hero > 8 (Play's maximum)")
        panels = [dict(kind="hero", name=theme.get("name", ""), icon=bool(theme.get("icon")), **strip["hero"])]
        images = {"icon": theme["icon"]} if theme.get("icon") else {}
        for i, s in enumerate(screens):
            c = strip["screens"].get(s["id"])
            if c is None:
                sys.exit(f"{loc}: no strip caption for screen {s['id']}")
            images[f"shot{i}"] = f"raw/{loc}/{s['file']}"
            panels.append(dict(kind="span" if i == 0 else "phone", shot=f"shot{i}", **c))
        names = ["00-hero"] + [os.path.splitext(s["file"])[0] for s in screens]
        ext = "png" if theme.get("output") == "png" else "jpg"
        jobs.append({
            "template": "../_tools/templates/strip.html",
            "width": 1080 * len(panels), "height": 1920,
            "format": "png" if ext == "png" else "jpeg", "quality": 95,
            # RTL: the strip is drawn mirrored (hero rightmost), so part k sits k panels from the right.
            "parts": [{"out": f"{OUT}/{loc}/{n}.{ext}", "x": 1080 * ((len(names) - 1 - k) if loc in RTL else k)}
                      for k, n in enumerate(names)],
            "vars": {
                "font": theme.get("strip_font", theme.get("font", "Inter")),
                "bg1": theme["bg1"], "bg2": theme["bg2"], "bg3": theme.get("bg3", theme["bg2"]),
                "text": theme.get("text", "#FFFFFF"), "accent": theme["accent"], "frame": theme.get("frame", "#0B0B0F"),
                "ring": theme.get("ring", "rgba(255,255,255,.35)"), "pill": theme.get("pill", "#FFFFFF"),
                "pillink": theme.get("pillink", "#111111"), "herostyle": theme.get("hero_style", "normal"),
                "lang": loc, "dir": "rtl" if loc in RTL else "ltr", "cjk": CJK.get(loc, "JP"),
                "panels": json.dumps(panels, ensure_ascii=False),
            },
            "images": images,
        })
    json.dump(jobs, sys.stdout, ensure_ascii=False, indent=1)


if __name__ == "__main__":
    main(sys.argv[1], sys.argv[2:])
```

### `docs/store/_tools/templates/contact.html`

One sheet per variant set so the owner compares sets at a glance: 2160x2040, up to eight
images `s1`…`s8` in a 4x2 grid of 540x960 tiles under a 120px label bar (`label`, `font`).

```html
<!doctype html>
<html><head><meta charset="utf-8"><style>
  html, body { margin: 0; width: 2160px; height: 2040px; overflow: hidden; background: #111; color: #fff;
         font-family: "{{font}}", system-ui, sans-serif; }
  .label { height: 120px; line-height: 120px; padding-left: 40px; font-size: 56px; font-weight: 800; }
  .grid { display: grid; grid-template-columns: repeat(4, 540px); grid-auto-rows: 960px; }
  .grid img { width: 540px; height: 960px; object-fit: cover; }
  img[src=""] { visibility: hidden; }
</style></head><body>
  <div class="label">{{label}}</div>
  <div class="grid"><img src="{{s1}}"><img src="{{s2}}"><img src="{{s3}}"><img src="{{s4}}">
  <img src="{{s5}}"><img src="{{s6}}"><img src="{{s7}}"><img src="{{s8}}"></div>
</body></html>
```

Templates are never overwritten, so a project made before `showcase.html` existed simply
gains the two new files; its `phone.html` stays as it is.

### `docs/store/_tools/templates/plain.html`

Undecorated: the capture fitted into 1080x1920 on a solid background, for stores or
listings that want the bare screen. Variables: `bg`, `shot`.

```html
<!doctype html>
<html><head><meta charset="utf-8"><style>
  html, body { margin: 0; width: 1080px; height: 1920px; overflow: hidden; background: {{bg}}; }
  img { display: block; width: 100%; height: 100%; object-fit: contain; }
</style></head><body><img src="{{shot}}" alt=""></body></html>
```

### `docs/store/_tools/templates/feature.html`

1024x500. Variables: `font`, `bg1`, `bg2`, `text`, `accent`, `frame`, `name`, `tagline`,
`icon` (image), `shot` (image).

```html
<!doctype html>
<html><head><meta charset="utf-8">
<link href="https://fonts.googleapis.com/css2?family={{font}}:wght@500;800&display=swap" rel="stylesheet">
<style>
  html, body { margin: 0; width: 1024px; height: 500px; overflow: hidden; }
  body { position: relative; font-family: "{{font}}", "Segoe UI", Roboto, system-ui, sans-serif; color: {{text}};
         background: linear-gradient(120deg, {{bg1}}, {{bg2}}); }
  .left { position: absolute; left: 64px; top: 0; bottom: 0; width: 540px; display: flex; flex-direction: column; justify-content: center; }
  .icon { width: 96px; height: 96px; border-radius: 22px; margin-bottom: 26px; box-shadow: 0 12px 30px rgba(0,0,0,.35); }
  .name { font-size: 64px; font-weight: 800; letter-spacing: -1.5px; line-height: 1; }
  .tagline { margin-top: 18px; font-size: 28px; font-weight: 500; opacity: .9; line-height: 1.3; }
  .tagline em { font-style: normal; box-shadow: inset 0 -10px 0 {{accent}}; }
  .device { position: absolute; right: 70px; top: 70px; width: 300px; height: 640px; background: {{frame}};
         border-radius: 44px; padding: 9px; transform: rotate(-8deg); box-shadow: 0 30px 60px rgba(0,0,0,.4); }
  .screen { width: 100%; height: 100%; border-radius: 36px; overflow: hidden; background: #000; }
  .screen img { display: block; width: 100%; height: 100%; object-fit: cover; object-position: top center; }
</style></head>
<body>
  <div class="left" data-check="left">
    <img class="icon" src="{{icon}}" alt="" onerror="this.style.display='none'">
    <div class="name">{{name}}</div>
    <div class="tagline">{{tagline}}</div>
  </div>
  <div class="device"><div class="screen"><img src="{{shot}}" alt=""></div></div>
</body></html>
```

### `docs/store/_tools/templates/tablet7.html`

1080x1920 portrait, same variables and headline block as `phone.html`. The 7" capture is
wider than a phone (about 0.625), so the device frame is wider and bleeds off the bottom:
`.device { left: 50%; top: 470px; width: 900px; height: 1440px; margin-left: -450px;
border-radius: 64px; padding: 24px; }`, `.screen img { width: 100%; height: auto; }`.

### `docs/store/_tools/templates/tablet.html`

2560x1440 landscape, headline on the left, framed screen on the right. Same variables as
`phone.html`. With `--vary`, every other screen gets `class="flip"` on `body` (headline right,
device left; mirror the two `left` values), and `tablet7.html` alternates the device between
bleeding off the bottom and floating with the headline underneath. Write it as a landscape adaptation of `phone.html`: `.device` 1100px wide by
1460px tall at `left: 1300px; top: 160px`, headline block at `left: 140px; top: 460px;
width: 1000px; font-size: 96px; text-align: left`.

## Step 5 — Build the job file and render

1. For each locale and each screen in `index.json` (respecting `--screen`), one job:
   ```json
   { "template": "../_tools/templates/phone.html", "width": 1080, "height": 1920,
     "out": "out/id/01-home.jpg", "format": "jpeg", "quality": 95,
     "vars": { "style": "bleed", "font": "Inter", "bg1": "#1E3A8A", "bg2": "#7C3AED", "text": "#FFFFFF",
               "accent": "#F59E0B", "frame": "#0B0B0F", "headline": "Semua pesanan dalam satu layar",
               "sub": "Pantau status tanpa buka-tutup aplikasi" },
     "images": { "shot": "raw/id/01-home.png" } }
   ```
   Paths are relative to the job file, which is written to `docs/store/<app>/.jobs.json`.
   Style `plain` uses `plain.html` with `bg` = `bg1`. `--tablet 10` uses `tablet.html` at
   2560x1440 from `raw-tablet/` into `out-tablet/`; `--tablet 7` uses `tablet7.html` at
   1080x1920 from `raw-tablet7/` into `out-tablet7/`.
   **With `--vary`** use `showcase.html` and assign layouts:
   - `layouts:` a list → use it in order.
   - `auto` → screen 1 `hero`; a screen whose caption has `zoom` → `zoom`, with `pair` →
     `duo` (`shot2` = that screen's capture); the rest cycle `bleed, float, tilt, clean`,
     skipping whatever the previous screen used. A caption's `layout:` always wins.
   - `extra`: `panorama` when `theme.yml` says so; `idx` = 0-based position, `count` = set size.
   - `art1`/`art2`: from `theme.yml` `art:`, rotating through the list, on at most every
     other screen so the decoration doesn't become wallpaper.

   **With `--variants N`** build N sets into `out/<locale>/variants/<A|B|C>/`:
   A = `auto` on the theme gradient; B = the cycle started one step later and `panorama` on;
   C = `extra: light` (`surface`/`ink`) with `clean`/`float` only. Then one `contact.html`
   job per set → `out/<locale>/variants/contact-<X>.jpg`, label `<App> — set <X>`.
   Nothing lands in `out/<locale>/` until `--pick`.
2. Feature graphic (unless `--screen` limits the run): `feature.html`, 1024x500,
   `out/<locale>/featureGraphic.<ext>`, `shot` = the first screen's capture, `tagline` from
   the captions file (fallback `theme.yml`).
3. **`--style strip`**: skip 1 and run
   `python3 -I docs/store/_tools/strip_jobs.py docs/store/<app> [locales] > docs/store/<app>/.jobs-strip.json`
   (needs PyYAML); output `out/<locale>/00-hero.jpg`, then `01-…` per capture. Look at every hero:
   the title must stay clear of the tilted device and the badges. The feature graphic is unchanged
   (step 2, in the regular job file).
4. `--dry-run`: stop here and print the job file path.
5. Run from the project root: `node docs/store/_tools/render.mjs docs/store/<app>/.jobs.json`.
   Any line starting with `OVERFLOW` names a text block that didn't fit: shorten that
   caption (or add a `<br>`-free shorter wording) and re-render that job.

## Step 6 — Verify and report

1. Open every output image with the Read tool. Check: headline fully visible, device
   content not hidden by the frame's rounded corners, colors match the theme, nothing
   looks squashed. Fix and re-render as needed; don't report an image you haven't looked at.
2. Write `docs/store/<app>/out/<locale>/index.json` listing the files in store order, the
   style, and the date.
3. Report a table: screen id, layout, headline, output file, status. With `--variants`,
   show the contact sheets (open each) and recommend one set with a one-line reason; with
   the owner's standing "pick the recommended option", `--pick` it straight away. Add the Play Console mapping:
   `out/<locale>/01-*.jpg ... 08-*.jpg` -> Phone screenshots, `featureGraphic` -> Feature
   graphic. Remind: the store accepts JPEG or 24-bit PNG without alpha, 320-3840 px per side; 2 phone
   screenshots is the publish minimum, 4 at 1080 px+ the threshold for Play's promotional
   surfaces, and large-screen listings need at least 4 tablet shots (`--tablet 7` and `--tablet 10`), only
   16:9 or 9:16.
4. Next step: `/play-store:listing <app>` writes the text and copies these images into the
   fastlane folder.

## Re-runs

Deterministic: same inputs, same images. Re-running overwrites `out/`. Edit `theme.yml`,
`captions.<locale>.yml`, or the templates and re-run to iterate on the look.
