# Platform profile — Desktop (Windows)

A Windows desktop app, in a resizable window, on a system where DPI scaling and elevation are
real and unavoidable. Read this alongside `/mock-first:mockup`, `/mock-first:break-task` or
`/mock-first:coding` — those own the flow.

Built with Electron, Tauri, Flutter or Compose Multiplatform? This profile still applies: the
shell gives you a window, not Windows conventions. A cross-OS app ships one drawing per OS.

## Where things live (check the project shape FIRST)

1. **`docs/INDEX.md` exists** -> multi-app project, already mapped. Use its "Docs" column and
   "Scope prefix".
2. **Otherwise** -> more than one executable project in the `.sln` (or several app packages
   in a monorepo) -> **multi-app**; one -> **single app**.

| Shape | Mockups | PRD | Tasks | Task numbers |
| --- | --- | --- | --- | --- |
| Single app | `docs/mockups/` | `docs/PRD.md` | `docs/TASKS.md` | `T01` |
| Multi-app | `docs/apps/<app>/mockups/` | `docs/apps/<app>/PRD.md` | `docs/apps/<app>/TASKS.md` | `<APP>-T01` |

When the PRD targets more than one platform, the file is `<screen>-desktop-windows.html`.

## Canvas

Build at a **1280x800 window content area at 100% scale**, and state the minimum window size
the layout survives.

Then check it at **125% and 150%** — the default on most laptops shipped in the last five
years. This is the single most common Windows layout bug: fixed pixel heights that look right
on the developer's monitor and clip text on the user's. If the mockup uses a fixed height
anywhere, say so and why.

Draw the window chrome: title bar with the caption buttons (minimise, maximise, close)
**top-right**.

## Design language

- Fluent / WinUI 3. A Mac app ported across is obvious immediately — usually from the button
  order in dialogs or a missing maximise button.
- `NavigationView` down the left for navigation (it collapses to icons, then to a hamburger,
  as the window narrows), a command bar across the top for verbs.
- Mica or Acrylic as the window backdrop, light and dark following the system, with the
  user's accent colour respected.
- Content dialogs are modal to the window, and their buttons go **primary on the left**,
  Cancel to its right — the opposite of macOS. Don't copy the Mac order.
- The window will be **snapped** (Win+Left, or a quarter of the screen). The layout must hold
  at half-width without a horizontal scrollbar.
- `Ctrl`-based shortcuts, `Alt` access keys on menus, `F1` for help, right-click context
  menus.

## Naming the component

Name the WinUI component the code will use:

```html
<!-- NavigationView -->
<!-- CommandBar -->
<!-- ContentDialog -->
<!-- ListView, virtualized -->
```

WPF or WinForms stack -> name that equivalent. Electron or Tauri -> name the web component
and the native behaviour it has to fake.

## States this platform always needs

Loading, empty, error with a retry, plus the Windows ones: **an action that needs
elevation** (the UAC shield on the control, and what happens when the user declines),
**per-monitor DPI change** (the window dragged to a second monitor at a different scale), and
first-run before any configuration exists.

## Task phases

```
Phase 0 — Project & packaging (multi-app only): new project, references, build config
Phase 1 — Data: database / file storage / network client
Phase 2 — Domain: models, services, interfaces
Phase 3 — DI: the container and its registrations
Phase 4 — Presentation: view models, state, commands
Phase 5 — UI: XAML views matching the approved mockup
Phase 6 — Integration: navigation, window management, app lifecycle, file associations
Phase 7 — Verification against the PRD's success criteria (manual, no automated tests)
```

## Verify

```
dotnet build <project>
```

or `msbuild` for an older solution — read the repo rather than guessing. An Electron or Tauri
app uses its own build. Then run it and check at 100% **and** 150% scaling.

Commit area = the project: `feat(library): add sidebar filter (T07)`.

## Shipping it

No command set covers this here; what Windows asks for:

- An **MSIX package** for the Microsoft Store or for sideloading, or a classic installer
  (WiX, NSIS, Squirrel) for direct download.
- **Code signing.** An unsigned installer triggers a SmartScreen warning that most users read
  as a virus alert, and a new certificate has no reputation until enough installs accumulate.
- An update path, decided before release rather than after.
- If it touches `Program Files`, the registry outside `HKCU`, or a service: an elevation
  story, declared in the manifest.

## Hidden work this platform always drags in

Per-monitor DPI awareness declared in the manifest, `%APPDATA%` versus `%LOCALAPPDATA%` for
roaming versus machine-local data, file paths with spaces and paths over 260 characters,
antivirus false positives on a freshly signed binary, window position and state persisted
across launches, taskbar integration (jump list, progress, notifications), and keyboard
navigation with `Tab` order that matches the visual order.
