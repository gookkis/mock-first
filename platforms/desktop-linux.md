# Platform profile — Desktop (Linux)

A Linux desktop app, where the desktop environment is not one thing: the window manager owns
the decorations, the toolkit decides the conventions, and the user may be on Wayland or X11,
GNOME or KDE, tiled or floating. Read this alongside `/mock-first:mockup`,
`/mock-first:break-task` or `/mock-first:coding` — those own the flow.

Built with Electron, Tauri, Flutter or Compose Multiplatform? This profile still applies.

## Pick the toolkit first

Linux has no single HIG, so **the toolkit is the decision that everything else follows** —
take it from the PRD's stack, and if the PRD doesn't say, ask once:

- **GTK4 / libadwaita** — the GNOME look: header bar with the window controls in it (CSD), no
  menu bar, a view switcher, preferences in their own window.
- **Qt / KDE Frameworks** — the Breeze look: a conventional menu bar, a sidebar, server-side
  decorations, and far more user-exposed configuration.

Don't mix the two in one screen. A GNOME header bar with a KDE menu bar underneath it is the
tell that nobody decided.

## Where things live (check the project shape FIRST)

1. **`docs/INDEX.md` exists** -> multi-app project, already mapped. Use its "Docs" column and
   "Scope prefix".
2. **Otherwise** -> several app binaries in one repo (multiple `meson.build` install targets,
   several `[[bin]]` in `Cargo.toml`, a monorepo of app packages) -> **multi-app**; one ->
   **single app**.

| Shape | Mockups | PRD | Tasks | Task numbers |
| --- | --- | --- | --- | --- |
| Single app | `docs/mockups/` | `docs/PRD.md` | `docs/TASKS.md` | `T01` |
| Multi-app | `docs/apps/<app>/mockups/` | `docs/apps/<app>/PRD.md` | `docs/apps/<app>/TASKS.md` | `<APP>-T01` |

When the PRD targets more than one platform, the file is `<screen>-desktop-linux.html`.

## Canvas

Build at a **1280x800 window content area**, and state the minimum window size the layout
survives. Then check it **narrow** — a tiling window manager can hand the app a third of the
screen without asking, and "our app assumes it gets half the monitor" is not a thing you get
to assume here.

Draw the decoration the toolkit implies: a GTK header bar with the close button top-right
inside the window, or a Qt window with a separate title bar drawn by the WM. Don't draw
macOS traffic lights.

## Design language

- Follow the chosen toolkit's guidelines, not a generic desktop look: GNOME HIG for
  libadwaita, KDE HIG for Qt.
- libadwaita: header bar with the primary action on the left and a hamburger on the right,
  boxed lists for settings, a preferences window, toast overlay for feedback, and the
  adaptive breakpoints that let the same window work narrow.
- Qt/KDE: menu bar, toolbar, sidebar, status bar, and a settings dialog that is expected to
  be thorough rather than minimal.
- Dark mode comes from the system preference through the XDG portal — read it, don't ship
  your own toggle as the only way.
- No system tray guarantee. GNOME has no tray by default, so an app whose only UI is a tray
  icon is invisible to a large share of users. If the PRD relies on one, say so now.

## Naming the component

Name the component the code will use:

```html
<!-- AdwApplicationWindow -->
<!-- AdwHeaderBar -->
<!-- AdwPreferencesWindow -->
<!-- GtkListView, virtualized -->
```

Qt stack -> `<!-- QMainWindow -->`, `<!-- QListView -->`. Electron or Tauri -> the web
component plus the native behaviour it fakes.

## States this platform always needs

Loading, empty, error with a retry, plus the Linux ones: **portal permission dialogs** (the
file chooser, screen capture and notifications all go through XDG portals in a sandbox, and
the user can refuse), **first run with no config directory yet**, and the window at a third
of the screen width.

## Task phases

```
Phase 0 — Build & packaging (multi-app only): new binary/target, dependencies
Phase 1 — Data: database / files under XDG dirs / network client
Phase 2 — Domain: models, services, interfaces
Phase 3 — Composition: dependency wiring
Phase 4 — Presentation: view models / controllers, state
Phase 5 — UI: widgets or .ui files matching the approved mockup
Phase 6 — Integration: actions, shortcuts, .desktop entry, MIME associations, portals
Phase 7 — Verification against the PRD's success criteria (manual, no automated tests)
```

## Verify

Read the repo for the real command — `meson compile -C build`, `cargo build`, `make`, or the
Flatpak builder. Then run it **on Wayland**, since that is the default on current GNOME and
KDE, and once on X11 if the app touches the clipboard, global shortcuts or screen capture,
which behave differently there.

Commit area = the module or binary: `feat(library): add sidebar filter (T07)`.

## Shipping it

No command set covers this here; what Linux asks for:

- **Flatpak on Flathub** as the default distribution path. It needs a manifest with the
  sandbox permissions spelled out, and Flathub additionally requires an **AppStream
  metainfo XML** file with a summary, description, screenshots and a release history, plus a
  `.desktop` entry and icons installed into the hicolor theme.
- AppImage, `.deb` or `.rpm` as secondary paths, each with its own dependency story — this is
  where "works on my distro" surfaces.
- A release that names the runtime version it was built against; a Flatpak runtime goes
  end-of-life and takes the app with it.

## Hidden work this platform always drags in

The `.desktop` entry and its MIME associations, XDG base directories (`~/.config`,
`~/.local/share`, `~/.cache`) instead of a folder in `$HOME`, fractional scaling at 125% and
150%, clipboard and screen-capture behaviour differing between Wayland and X11, library
version skew across distributions, translations via gettext, and accessibility through
AT-SPI.
