# Platform profile — Desktop (macOS)

A Mac app, in a resizable window, on a system whose conventions users know by heart. Read
this alongside `/mock-first:mockup`, `/mock-first:break-task` or `/mock-first:coding` — those
own the flow.

Built with Electron, Tauri, Flutter or Compose Multiplatform? This profile still applies:
those shells give you a window on macOS, they do not give you macOS behaviour. The menu bar,
the shortcuts and the window semantics below are still what a Mac user expects, and a
cross-OS app ships one drawing per OS.

## Where things live (check the project shape FIRST)

1. **`docs/INDEX.md` exists** -> multi-app project, already mapped. Use its "Docs" column and
   "Scope prefix".
2. **Otherwise** -> more than one app target in the `.xcodeproj`/`.xcworkspace`, or several
   app packages in a monorepo -> **multi-app**; one app -> **single app**.

| Shape | Mockups | PRD | Tasks | Task numbers |
| --- | --- | --- | --- | --- |
| Single app | `docs/mockups/` | `docs/PRD.md` | `docs/TASKS.md` | `T01` |
| Multi-app | `docs/apps/<app>/mockups/` | `docs/apps/<app>/PRD.md` | `docs/apps/<app>/TASKS.md` | `<APP>-T01` |

When the PRD targets more than one platform, the file is `<screen>-desktop-macos.html`.

## Canvas

Build at a **1440x900 window content area**, and state the **minimum window size** the layout
survives — a desktop window is resizable, and a mockup that only works at one width hides the
real problem. Draw the window chrome: traffic lights top-left, a unified toolbar with the
title inline.

The **menu bar is not in the window** — it belongs to the system, at the top of the screen.
Don't draw an in-window menu bar, and don't put actions there that a Mac user expects in the
menu bar.

## Design language

- Apple HIG. A Windows app ported across is instantly obvious, and the giveaway is usually
  the menu bar, the button order, or a settings screen embedded in the main window.
- Sidebar (source list) on the left for navigation, translucent material, with the content
  area on the right. Toolbar at the top for the verbs that act on what's selected.
- **Settings is its own window**, opened with `⌘,` and titled "Settings" (not "Preferences",
  since Ventura) — never a tab inside the main window.
- Sheets slide out of the window they belong to. A confirm dialog's buttons go
  **Cancel on the left, the default action on the right**.
- Full keyboard coverage with `⌘`-based shortcuts, and `⌘` for the platform's own verbs
  (`⌘N`, `⌘W`, `⌘Q`, `⌘,`). Right-click context menus, drag and drop, and multiple windows or
  tabs where the content supports it.
- Dark mode and the system accent colour are not optional.

## Naming the component

Name the SwiftUI component the code will use:

```html
<!-- NavigationSplitView -->
<!-- Table(selection:) -->
<!-- .toolbar { ToolbarItem(placement: .primaryAction) } -->
<!-- Settings scene -->
```

AppKit stack -> name the AppKit equivalent (`<!-- NSSplitViewController -->`). Electron or
Tauri -> name the web component, and note the native behaviour it has to fake
(`<!-- <Sidebar> — vibrancy via backdrop-filter -->`).

## States this platform always needs

Loading, empty, error with a retry, plus the macOS ones: **all windows closed while the app
keeps running** (the Dock icon is still there — clicking it must reopen something),
**permission denied** for the sandbox prompts (Files & Folders, Full Disk Access,
Accessibility) including what the screen shows after a refusal, and dark mode.

## Task phases

```
Phase 0 — Target & package (multi-app only): new target or SPM package, dependencies
Phase 1 — Data: SwiftData / Core Data / files / network client
Phase 2 — Domain: models, use cases, repository protocols
Phase 3 — Composition: environment objects, dependency wiring
Phase 4 — Presentation: @Observable view models, state, actions
Phase 5 — UI: SwiftUI views matching the approved mockup
Phase 6 — Integration: menu bar commands, window management, app lifecycle, deep links
Phase 7 — Verification against the PRD's success criteria (manual, no automated tests)
```

Phase 6 is the one people skip on macOS and pay for later — the menu bar is part of the app,
not decoration.

## Verify

```
xcodebuild -scheme <scheme> -destination 'platform=macOS' build
```

Read the scheme from the project rather than guessing it. An Electron or Tauri app uses its
own build (`npm run build`, `cargo tauri build`) — read `package.json` or `tauri.conf.json`.

Commit area = the target or package: `feat(library): add sidebar filter (T07)`.

## Shipping it

No command set covers this here; what macOS asks for:

- A **Developer ID** signed build, **notarized** through `notarytool` and stapled — an
  unnotarized app is blocked by Gatekeeper on first launch with a message users read as
  "this app is malware".
- A `.dmg` or `.pkg` for direct distribution, plus an update path (Sparkle is the usual one);
  or the Mac App Store, which additionally requires the App Sandbox and its own review.
- A **universal binary** (arm64 + x86_64) unless the PRD says Apple Silicon only.
- Hardened runtime, and an entitlement for every capability the app actually uses.

## Hidden work this platform always drags in

The menu items macOS expects to exist whether or not the PRD mentions them (About, Settings,
Services, Hide, the Window and Help menus), `⌘W` closing a window versus `⌘Q` quitting the
app, window frame and state restoration between launches, sandbox file access surviving a
relaunch (security-scoped bookmarks), Retina assets, VoiceOver labels, and full-screen mode.
