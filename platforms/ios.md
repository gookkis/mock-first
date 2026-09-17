# Platform profile — iOS

A native iOS app. The mockup stays static HTML (fast to write, opens straight in a browser)
— not SwiftUI — but it is built so the translation to SwiftUI is direct. Read this alongside
`/mock-first:mockup`, `/mock-first:break-task` or `/mock-first:coding` — those own the flow.

## Where things live (check the project shape FIRST)

1. **`docs/INDEX.md` exists** -> multi-target project, already mapped. Use the scope's
   "Docs" column and its "Scope prefix".
2. **No `INDEX.md`** -> look at what the repo has:
   - more than one app target in the `.xcodeproj` / `.xcworkspace`, or local SPM packages
     under `Packages/` that several targets share -> **multi-app**;
   - one app target -> **single app**.
3. **Nothing generated yet** -> single app, and say so in one line.

| Shape | Mockups | PRD | Tasks | Task numbers |
| --- | --- | --- | --- | --- |
| Single app | `docs/mockups/` | `docs/PRD.md` | `docs/TASKS.md` | `T01` |
| Multi-app | `docs/apps/<app>/mockups/` | `docs/apps/<app>/PRD.md` | `docs/apps/<app>/TASKS.md` | `<APP>-T01` |

## Canvas

Build at a **393x852 viewport** (iPhone 15/16 class). Leave the safe areas visible in the
mockup — 59px at the top under the Dynamic Island, 34px at the bottom for the home
indicator — so nothing important is designed into a space the system owns.

## Design language

Follow the HIG; an Android screen ported across is immediately obvious to an iOS user.

- Navigation stack with a large title that collapses to inline on scroll. The Back button
  carries the **previous screen's title** — never invent a label for it.
- Tab bar at the bottom for 3-5 destinations. Not a drawer.
- Inset grouped lists for settings and detail, with swipe actions for row-level verbs and a
  context menu (long press) for the rest.
- Sheets with detents for a secondary task; a full-screen cover only when the task owns the
  session. A modal pairs **Cancel** (left) with **Done/Save** (right).
- System typography and Dynamic Type — no fixed 14px body text. SF Symbols for icons, named
  in the mockup.
- Swipe-from-edge to go back must keep working; don't design a screen that traps it.

## Naming the component

Name the SwiftUI component the code will use, in an HTML comment:

```html
<!-- NavigationStack -->
<!-- List(.insetGrouped) -->
<!-- .sheet(detents: [.medium]) -->
<!-- SF Symbol: square.and.arrow.up -->
```

If the PRD's stack says UIKit, name the UIKit equivalent instead
(`<!-- UITableView, grouped -->`).

## States this platform always needs

Loading, empty, error with a retry, plus: **permission denied** for anything behind a
usage-description prompt (camera, photos, location, notifications) and what the screen shows
after the user has said no once, and **dark mode** — iOS users switch it system-wide.

## Task phases

```
Phase 0 — Target & package (multi-app only): new target or SPM package, dependencies
Phase 1 — Data: SwiftData / Core Data / network client
Phase 2 — Domain: models, use cases, repository protocols
Phase 3 — Composition: environment objects, dependency wiring
Phase 4 — Presentation: @Observable view models, state, actions
Phase 5 — UI: SwiftUI views matching the approved mockup
Phase 6 — Integration: navigation, deep links, app entry point
Phase 7 — Verification against the PRD's success criteria (manual, no automated tests)
```

Skip phases that don't apply.

## Verify

```
xcodebuild -scheme <scheme> -destination 'generic/platform=iOS' build
```

For an SPM package on its own, `swift build` is faster. Read the scheme name from the
project rather than guessing it. Then run it in the simulator and check against the
approved mockup.

Commit area = the target or package: `feat(settings): add export row (T07)`.

## Shipping it

There is no App Store command set in this repo — say so plainly rather than pointing the user
at the Play Store commands, which drive ADB and write `fastlane/metadata/android/` and do not
transfer. What iOS asks for at that point, so nothing is discovered on submission day:

- Screenshots at the sizes App Store Connect still requires (6.9" and 6.5" displays), per
  locale, taken from the simulator.
- App Privacy "nutrition labels" — the same questions `/portfolio:privacy` answers for Play,
  asked differently. That command's read of the code (permissions, SDKs, accounts, local
  data) is still the right input; the form is not.
- An archive signed with a distribution certificate and provisioning profile, uploaded via
  Xcode or `xcrun altool`.
- Export compliance, age rating, and a support URL.

## Hidden work this platform always drags in

`Info.plist` usage descriptions for every permission (a missing one crashes on first use),
capabilities and entitlements, App Transport Security for non-HTTPS endpoints, Dynamic Type
at the largest size, VoiceOver labels, keyboard avoidance for a field near the bottom, and
state restoration after the app is jetsammed in the background.
