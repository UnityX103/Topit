# Topit: pin a region or a whole window

[简体中文](README_zh.md) · [Downloads](https://github.com/UnityX103/Topit/releases) · [CI](https://github.com/UnityX103/Topit/actions/workflows/ci.yml)

Topit keeps selected content above other macOS windows. This fork of [lihaoyun6/Topit](https://github.com/lihaoyun6/Topit) adds **interactive region pinning**: crop part of a source window into an independent live floating panel. The original whole-window tools and settings remain available.

Use a region for a chat composer, progress indicator, or other small part of a window. Releases are currently prereleases; background interaction depends on the source application. See the limitations below.

## Features and their origins

| Capability | Origin | Entry point |
| --- | --- | --- |
| Pin whole windows, including multiple windows | Upstream | Topit! in the selector; Pin a Window in the menu bar |
| Pause/resume, opacity, unpin, close the original window | Upstream | Topit control menu in a pinned window's corner |
| Activation behavior, focus protection, auto-ducking, refresh settings | Upstream | Settings → Windows |
| Per-display selection, direct window picking, app filters, custom shortcuts, AppleScript | Upstream | Window selector, Settings, scripting commands |
| Crop part of a window into a live panel | Added in this fork | Pin a Region… or Command + Shift + T |
| Map pointer input and hand keyboard focus to the original field | Added in this fork | Click the corresponding location in the region |
| Move/resize a region, freeze/resume, copy snapshots, adjust opacity | Added in this fork | Hold Shift to edit the region |
| Double-Control auto-hide for regions | Added in this fork | Double-tap Control; Region Capture menu |
| Fixed-signature incremental updates, backups, CI and release verification | Added in this fork | Update scripts and [release guide](docs/RELEASING.md) |

**Whole-window and region panels have different interaction rules.** Upstream auto-ducking and focus protection apply to whole-window pinning. Double-Control auto-hide applies to region panels.

## Install and grant permissions

- Use macOS 13 or later. Universal releases contain both Apple Silicon and Intel architectures.
- Download `Topit-VERSION-macOS-universal.zip` from [this fork's Releases](https://github.com/UnityX103/Topit/releases), extract it, and place `Topit.app` in `/Applications`.
- Current releases use a fixed local signing certificate and are not Apple Developer ID notarized. First installation or a change of signing identity can require new permission grants.
- The upstream Homebrew formula installs the upstream application, without this fork's region features.

On first launch, use the in-app instructions to grant:

| Permission | Purpose |
| --- | --- |
| Screen Recording | Capture the selected window's image |
| Accessibility | Select/control windows and transfer input focus |

Click **Recheck Permissions** after granting access. If capture is still unavailable, quit Topit and reopen `/Applications/Topit.app`. Routine use does not require access to a signing keychain.

## Region pinning: added in this fork

### Create a region

1. Move the pointer over a source window and press **Command + Shift + T**.
2. Drag a rectangle in the selection overlay and release to create the region. **Esc** cancels. A click or a selection smaller than 8 points does not create a panel.
3. Alternatively, select a window thumbnail and click **Pin a Region…**. From the menu bar, pick the source window first, then crop it.

Only the selected area appears in the floating panel. Multiple regions can be created from different windows. Region capture is capped at 30 fps and also respects the configured maximum capture rate.

### Interact or edit

| Action | Effect |
| --- | --- |
| Ordinary click, drag, or scroll | Map pointer input to the corresponding source location |
| Click the original text field's location in the region | Attempt to give real keyboard focus to that field; the source app handles subsequent input |
| Hold Shift and drag | Move the region panel |
| Hold Shift and drag an edge | Resize the panel proportionally |
| Hold Shift and hover | Show Close Region, Freeze, and Copy Snapshot controls |
| Shift + right-click | Open freeze/resume, snapshot, Focus Source Window, close-region, and opacity options |

Release Shift to return to source interaction. Topit creates no proxy editor, mirrors no draft text, and synthesizes no duplicate keyboard events. Successful focus, typing, and input-method behavior still depend on the source application.

**Freeze** keeps the current image and stops capture and input for that region; resume to restart it. **Copy Snapshot** copies the visible crop. Closing a region keeps its source window open. **Focus Source Window** explicitly brings the original window forward. If the source closes, the region retains its last frame.

### Double-Control auto-hide

Quickly tap **Control** twice to enable auto-hide, then twice again to disable it. The same switch is available in the Region Capture and menu bar menus.

- Panels fade out as the pointer approaches and reappear when it leaves.
- **All region interaction stays disabled throughout this mode**, including clicks, typing, scrolling, and Shift editing. Reappearing does not restore interaction.
- Switching the mode off restores opacity and interaction. It applies to all region panels, including newly created ones, and defaults to off after restarting Topit.
- Control chords, long holds, and clicks or scrolling between taps do not count as a double tap.

## Whole-window pinning: inherited from upstream

### Select and pin a complete window

1. Open the window selector from Topit's main window or the menu bar.
2. Choose the appropriate display, select a thumbnail, then click **Topit!**.
3. Alternatively, use **Pin a Window** in the menu bar or **Select Window Directly** in the selector, then click the target window. Press Esc to cancel picking.

Repeat for additional windows. Activate a pinned whole window to interact with, move, or resize it normally. This mode uses the upstream window activation behavior rather than the region panel's Shift editing controls.

### Whole-window control menu

Move the pointer to the configured control corner and click the Topit icon. Change its corner in Settings.

| Upstream control | Behavior |
| --- | --- |
| Unpin | Remove the pinned overlay and keep the source window open |
| Pause / Resume | Temporarily unpin and stop capture, then restore whole-window pinning; this differs from freezing a region snapshot |
| Opacity | Adjust whole-window transparency with the slider; pause before resizing a window in translucent mode |
| Close | Close the original application's window as well as the overlay; use Unpin to keep the original open |

**Unpin all Windows** in the menu bar or Dock menu closes both whole-window overlays and region panels.

### Retained settings

Open Settings through the selector's gear button, the Topit application menu, or the menu bar.

| Settings page | Upstream behavior and usage |
| --- | --- |
| General | Launch at Login, Dock icon, and menu bar icon |
| Windows → Activate Pinned Window on | Hover by default, or Left Click; with click activation, activate the window before moving it |
| Windows → Prevent Focus Switching | Available with click activation; reduces focus changes when passing over a pinned window and requires double-click activation |
| Windows → Control Button Position | Choose one of the four corners for whole-window controls |
| Windows → Maximum Refresh Rate | 30, 60, 120 Hz, or No Limit; whole-window capture is also limited by the display, and regions remain capped at 30 fps |
| Windows → Ducking Between Pinned Windows | Hide other whole-window overlays that overlap the currently active window |
| Windows → Add Border for Translucent Window | Show a border around translucent whole-window overlays |
| App Filter | Toggle untitled windows; use + to hide an application's windows from the selector and − to remove a filter |

### Retained AppleScript commands

The upstream scripting interface remains available. For example, in macOS Script Editor:

```applescript
tell application "Topit"
    show selector
end tell
```

| Command | Purpose |
| --- | --- |
| `show selector` | Open the window selector |
| `select a window` | Start direct picking for whole-window pinning |
| `toggle frontmost` | Toggle whole-window pinning for the frontmost window |
| `toggle under mouse` | Toggle whole-window pinning for the window under the pointer |
| `unpin all windows` | Unpin all windows; this fork also closes region panels |

These commands do not add an interface for cropping regions. See [Scriptable.sdef](Topit/Scriptable/Scriptable.sdef) for the dictionary.

## Shortcuts

| Action | Current default / configuration |
| --- | --- |
| Crop a region | Command + Shift + T; configurable in Settings → Hotkey |
| Toggle region auto-hide | Double-tap Control; currently a fixed gesture |
| Edit a region panel | Hold Shift |
| Cancel an active selection | Esc |
| Upstream window picking, opening the selector, toggling under-pointer/frontmost windows, and unpinning all | Assign your own bindings in Settings → Hotkey; the current code supplies no default chords |

This fork migrates Command + Shift + T from whole-window pinning to region selection. Whole-window pinning remains available through buttons, menus, and custom shortcuts.

## Known limitations

- The source window must remain available. Complete occlusion, minimization, Spaces, or protected content can cause the source app to stop refreshing; this is not resolved for every application.
- Region input relies on private macOS window-focus interfaces. Compatibility depends on the OS and source app. The focus request does not explicitly raise the source, but the source application may still bring itself forward.
- Background pointer input uses Command-click semantics. Links and special controls may behave accordingly.
- Codex physical keyboard input, Chinese input methods, and window stacking have not completed installed-app acceptance. Passing automated checks does not establish compatibility with every text field.
- A retained frame after source closure or capture failure is not interactive live content. Restarting or updating Topit closes its panels; recreate them afterward.

## Updates and development

The upstream Sparkle updater is disabled in this fork. **Check for Updates…** shows a feature-build notice; get releases from this repository.

A `.topitupdate` package requires the **exact previous version, files, and signing identity**. It is not a general installer. Run `Scripts/topit_update.py apply PACKAGE_PATH --check` before applying it. Normal updates back up the previous installation, roll back on failure, and preserve preferences and permission records. A first signing-identity migration can still require new grants.

For example, the v0.1.22 incremental package targets the same-certificate 0.1.21/build31 installation. See the [release guide](docs/RELEASING.md) for installation, signing migration, and release maintenance.

For development, open `Topit.xcodeproj` in Xcode and select the `Topit` scheme. Run all automated checks with:

```bash
bash Scripts/test.sh
```

GitHub CI runs regressions on Intel and Apple Silicon and compiles a universal Release. Build products are only compiled; runtime interaction acceptance uses the app formally installed at `/Applications/Topit.app`.

## Attribution and license

This project is forked from [lihaoyun6/Topit](https://github.com/lihaoyun6/Topit). Whole-window pinning, its settings, and scripting support originate upstream. This fork has added region pinning and related changes since 2026-10-17. Shortcut recording uses [KeyboardShortcuts](https://github.com/sindresorhus/KeyboardShortcuts).

Original copyright notices are retained. The code remains licensed under [AGPL-3.0](LICENSE).
