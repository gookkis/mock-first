# Mock-First — GitHub Branding Guide

---

## 1. Repo name

**`mock-first`**

Fallbacks if it's taken: `mock-first-workflow`, `mockfirst-cc`.

Avoid a `claude-` prefix in the repo name — "Claude" belongs in the description and topics,
so the name stays short when spoken (`user/mock-first`).

---

## 2. Tagline

The core problem: in dev circles "mock" means a test double (gomock, Mockito, mock servers).
The tagline **must** correct that in its first sentence.

**Primary:**
> Agree on the screen before you write the code.

**Alternatives:**
> See it before you build it. A visual-first spec workflow for Claude Code.
> Mockup first, code second.

**Don't use:** anything containing "mocking", "test double", or "stub" — it reinforces the
misunderstanding.

---

## 3. Repo description (About)

350-character limit. Use this:

```
Visual-first spec workflow for Claude Code. PRD → HTML mockup → tasks → code,
with a /revise command that reports impact before anything changes.
Built for solo devs and small teams. (Not about test mocks — this is about UI mockups.)
```

The closing parenthetical reads awkwardly but saves a lot of confusion. Drop it once the
project is well known enough.

---

## 4. Topics

Fill all 20 slots — this is the main discovery path on GitHub.

```
claude-code, claude-code-plugin, claude-plugin, anthropic, ai-coding,
spec-driven-development, prd, mockup, wireframe, ui-mockup,
workflow, developer-tools, slash-commands, agentic-coding,
prototyping, product-development, android, jetpack-compose,
solo-developer, developer-workflow
```

---

## 5. Repo structure

```
mock-first/
├── .claude-plugin/
│   ├── plugin.json              # plugin manifest
│   └── marketplace.json         # so /plugin marketplace add works
├── commands/
│   ├── adopt.md                 # one-time bootstrap for a running project
│   ├── prd.md
│   ├── mockup.md
│   ├── break-task.md
│   ├── coding.md
│   ├── review.md                # read-only drift audit
│   └── revise.md
├── variants/
│   └── android-factory/         # multi-module variant: adopt, new-app, per-module docs
├── docs/
│   ├── CONCEPT.md               # the design paper
│   ├── COMPARISON.md            # vs Spec Kit, ShipSpec, etc — honestly
│   └── examples/
│       └── walkthrough.md       # one feature from nothing to a commit
├── assets/
│   ├── logo.svg
│   ├── lifecycle.svg            # flow diagram + loops
│   └── social-preview.png       # 1280×640
├── README.md
├── LICENSE                      # MIT
└── CHANGELOG.md
```

---

## 6. Logo

**Concept:** a wireframe becoming code.

A dashed rectangle (the mockup) on the left, an arrow, curly braces `{ }` on the right. Or
simpler: one wireframe frame with its bottom-right corner "folded" to reveal a line of code.

- Shape: square, rounded corners, thick strokes — it has to read at 32×32.
- Color: one accent only. Indigo (#6366F1) or teal (#14B8A6). Everything else monochrome.
- Ship a light and a dark variant for the README.

**Don't:** robot, brain, or lightning-bolt icons. That well is dry in AI tooling.

---

## 7. Social preview (1280×640)

Set this. A link shared to X/LinkedIn/Slack without one looks like an abandoned repo.

Contents: logo + name + tagline + one line of flow:
```
/prd  →  /mockup  →  /break-task  →  /coding
                 ↖ /revise ↙
```
Flat background. Don't paste a terminal screenshot — it's illegible at that size.

---

## 8. README skeleton

The order matters. People decide in the first 10 seconds.

```markdown
<p align="center"><img src="assets/logo.svg" width="88"></p>
<h1 align="center">Mock-First</h1>
<p align="center">Agree on the screen before you write the code.</p>
<p align="center">[badges]</p>

---

## The problem              <- 3 sentences, not a paragraph
   Text PRDs get misread. You find out after the code is written.

## How it works             <- the lifecycle diagram goes HERE, not further down

## Install                  <- a code block, above the fold
   /plugin marketplace add gookkis/mock-first
   /plugin install mock-first@mock-first

## 60-second walkthrough    <- one real feature, from /prd to a commit
   [screenshot of an HTML mockup — the one image the README must have]

## What makes it different
   - The mockup is a gate, not an option
   - /revise reports impact before changing anything
   - Nothing is deleted — superseded tasks are marked, not removed

## Not the same as...       <- short section, honest
   Spec Kit, ShipSpec, and others do PRD→tasks→code well.
   Mock-First adds the visual gate and change handling.
   If you don't need those, use them — they're more mature.

## Android multi-module variant
## Configuration
## FAQ                      <- question #1: "Is this about test mocks?" No.
## License
```

**Rule:** the install command must be visible without scrolling. Plenty of good repos bury
it under 2,000 words of philosophy and lose the reader.

---

## 9. Badges

Keep it sparse. Four at most:

```markdown
![Claude Code](https://img.shields.io/badge/Claude_Code-plugin-6366F1)
![License](https://img.shields.io/github/license/gookkis/mock-first)
![Stars](https://img.shields.io/github/stars/gookkis/mock-first)
![Version](https://img.shields.io/github/v/release/gookkis/mock-first)
```

Don't add build/coverage badges to a repo of markdown files — it reads as theater.

---

## 10. Language

Everything ships in **English** — the README, the docs, and the command files. The Claude
Code plugin audience is predominantly English-speaking, and the command files are prompts
the model reads, so a single language keeps their behavior consistent.

If you want a localized README later, add `README.<lang>.md` and link it from the first line:
`[English](README.md) · [Bahasa Indonesia](README.id.md)`. Keep `commands/` in English even
then — the commands already tell the model to answer in the user's own language.

---

## 11. What decides whether anyone looks at this repo

Ordered by real impact:

1. **One screenshot of an HTML mockup in the README.** That's the entire premise in a single
   image. Without it, Mock-First looks identical to five other spec-driven plugins.
2. **The lifecycle diagram** near the top, not at the bottom.
3. **A social preview image.**
4. **The "Not the same as..." section** — acknowledging competitors builds trust faster than
   claiming superiority.
5. **A real walkthrough**, not a `foo`/`bar` example.

Numbers 1 and 4 are the most commonly skipped, and they matter most.
