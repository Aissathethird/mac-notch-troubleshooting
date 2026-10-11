# mac-notch-troubleshooting

Fixes and guides for MacBook notch apps and the free utilities around them.

Most "notch app not working" searches land on raw GitHub issues. This repo collects the fixes in one place with the exact commands, plus the two utilities people most often want alongside a notch app: keeping the Mac awake and locking the keyboard.

Maintained by Karim, the developer of [Korinotch](https://korinotch.com/). Everything else here is vendor-neutral; PRs that correct or extend any section are welcome.

Applies to: MacBook Pro 14"/16" (2021+), MacBook Air M2/M3/M4 (2022+). macOS Sonoma (14), Sequoia (15), Tahoe (26).

---

## Boring Notch not opening

"Not opening" is actually three different failures. Work out which one you have before trying fixes.

### 1. The app will not launch at all (Gatekeeper)

Symptom: "Apple could not verify whether 'boringNotch' is free of malware". Boring Notch is open source (GPL-3.0) but unsigned, so macOS quarantines it on first launch.

Fix A, GUI: System Settings > Privacy & Security > **Open Anyway**, then launch again.

Fix B, Terminal (works on managed Macs where Open Anyway is hidden):

```bash
xattr -dr com.apple.quarantine "/Applications/Boring Notch.app"
```

The bundle name has varied between releases (older builds used `boringNotch.app`); check `ls /Applications | grep -i notch`.

Fix C, Homebrew, which skips the quarantine flag:

```bash
brew install --cask TheBoredTeam/boring-notch/boring-notch
```

Requires macOS 14 or later, Apple Silicon or Intel.

### 2. The app is running but the notch is invisible after sleep

Symptom: you closed the lid, and this morning the notch area is empty. The app does not appear in Force Quit because it has no Dock presence. This is [issue #336](https://github.com/TheBoredTeam/boring.notch/issues/336), open since January 2025, reported on macOS 15.3 through 26.x. Triggers: lid close, external display plug-in, user switching.

Fix: click the menu-bar sparkle icon and choose the restart-widget option. If that icon is gone too, kill the process and relaunch:

```bash
pkill -x boringNotch || pkill -if "boring notch"
open -a "Boring Notch"
```

There is no permanent upstream fix yet; quitting the app before closing the lid avoids it.

### 3. The notch shows but Now Playing is empty

Symptom: album art and controls never appear while music plays. macOS 15.4 changed the private MediaRemote framework and broke Now Playing for most third-party apps ([#417](https://github.com/TheBoredTeam/boring.notch/issues/417)); Boring Notch fixed it via an adapter, then Apple Music broke again on macOS 26 ([#779](https://github.com/TheBoredTeam/boring.notch/issues/779)).

Fix: update to the latest release. If stuck on an old build, uninstall fully (quit from the menu-bar icon, remove from System Settings > General > Login Items, trash it or use Pearcleaner) and reinstall via Homebrew. Re-grant Accessibility and Screen Recording afterwards; macOS updates often reset them.

Related: [Boring Notch alternatives](https://korinotch.com/blog/boring-notch-alternatives.html) and [why some notch apps drain battery](https://korinotch.com/blog/notch-app-battery-drain.html).

---

## Keep your Mac awake for Claude Code / long agent runs

A fact most guides skip: **Claude Code already spawns `caffeinate -i -t 300` itself** during an active session and respawns it every 300 seconds ([anthropics/claude-code #21432](https://github.com/anthropics/claude-code/issues/21432)). `killall caffeinate` does nothing lasting, and wrapping it in your own `caffeinate -i claude` is redundant for idle sleep. The two real problems:

1. **Display sleep vs system sleep.** `caffeinate -i` prevents idle *system* sleep; the display may still turn off, which is fine. Use `caffeinate -di` to keep the screen on, or `caffeinate -t 3600` for a fixed hour.
2. **Clamshell (lid closed).** No `caffeinate` flag survives closing the lid on battery. You need either an external display plus power (clamshell mode), or to disable sleep at the power-management level:

```bash
sudo pmset -a disablesleep 1   # lid-closed sleep off
# ... run your agent ...
sudo pmset -a disablesleep 0   # restore. People forget this line.
```

Warning: with `disablesleep 1` the Mac stays fully awake in a backpack, gets hot and drains the battery. Always restore to `0`; verify with `pmset -g live | grep SleepDisabled`. To see what is holding the Mac awake: `pmset -g assertions`.

For a one-click toggle, the free [Keep Awake mini app](https://korinotch.com/apps/keep-awake.html) sits in the menu bar, supports timed sessions, and handles lid-closed on power with an external display. Write-up: [Keep your Mac awake from the notch](https://korinotch.com/blog/keep-awake-mac-notch.html).

---

## Lock the keyboard for cleaning

There is no built-in keyboard lock on a Mac. Lock Screen (Control+Command+Q) still accepts keystrokes into the password field, and sleep wakes on the first key press. Apple's own advice is to shut down before cleaning.

Why no Mac App Store app can do this properly: blocking keystrokes requires a `CGEventTap` that intercepts HID events system-wide, which needs the Accessibility permission. That is outside the App Store sandbox, so App Store "keyboard cleaners" are limited to a short countdown or an overlay that still leaks modifier keys. Apple rejected KeyboardCleanTool from the store in 2011.

What works, all outside the App Store:

- **KeyboardCleanTool** (folivora.ai, free, macOS 13+): locks keyboard, optionally trackpad and mouse. Grant Accessibility, then quit and reopen once.
- **KeyboardBlocker** (open source, MIT, macOS 11+): hold Escape three seconds to unlock.
- **Korinotch Keyboard Lock** (free mini app, macOS 14+): one click, full-screen overlay so you can see it is locked, hotkey to unlock. Also for kids watching a video or a cat on the desk. [korinotch.com/apps/keyboard-lock.html](https://korinotch.com/apps/keyboard-lock.html)

No app can block the power/Touch ID button; keep your finger off the top-right key. After removing a keyboard-lock app, revoke its Accessibility entry:

```bash
open "x-apple.systempreferences:com.apple.preference.security?Privacy_Accessibility"
```

Guide: [How to lock a Mac keyboard](https://korinotch.com/blog/keyboard-lock-mac.html).

---

## Notch file tray / shelf options

A notch file tray (or shelf) is a temporary staging area: drag a file onto the notch, move to another folder or app, drag it back out. Files are held by reference, not copied.

Which notch apps have one, as of October 2026:

- **Alcove**: no file tray. Requested in [alcove-releases #486](https://github.com/henrikruscon/alcove-releases/issues/486), marked planned and a duplicate of #83. Check the current changelog before buying for this reason.
- **Boring Notch**: yes, with AirDrop, built on NotchDrop's code.
- **NotchNook**: yes, mature shelf with AirDrop.
- **DynamicLake**: yes.
- **Korinotch**: yes, dropzone tray plus clipboard history. The shelf alone is also the free [File Shelf mini app](https://korinotch.com/apps/file-shelf.html), which runs next to any other notch app, including Alcove.

More: [File tray and dropzone on the notch](https://korinotch.com/blog/file-tray-dropzone.html).

---

## Comparison table of notch apps

Prices change; "paid" means one-time or subscription, check each vendor's site. Details verified October 2026.

| App | Price | License / signing | Min macOS | Media HUD | File tray | Keep awake | Keyboard lock | Notifications in notch |
|---|---|---|---|---|---|---|---|---|
| Boring Notch | Free | GPL-3.0, unsigned | 14 | Yes | Yes (AirDrop) | No | No | No |
| NotchNook | Paid (trial, also on Setapp) | Proprietary, signed | 14 | Yes | Yes | No | No | Limited |
| Alcove | Paid (72h trial) | Proprietary, signed | 15 | Yes | No (planned) | No | No | Own alerts only |
| DynamicLake | Paid | Proprietary, signed | 14 | Yes | Yes | No | No | Limited |
| Korinotch | Free 14 days, then $11.50 lifetime or $1.50/mo | Proprietary, signed and notarized | 14 | Yes | Yes + clipboard | Yes | Yes | Yes (system, WhatsApp, Gmail) |

Facts other comparison pages get wrong: Boring Notch is GPL-3.0, not MIT. Alcove needs macOS 15, not 14. On macOS 26 Tahoe, system notification banners are delivered differently, so banner-in-notch features in any app have limits and may need Accessibility re-granted after upgrading.

See also: [Korinotch vs Alcove](https://korinotch.com/blog/korinotch-vs-alcove.html).

---

## Free tools

Korinotch ships eleven menu-bar utilities as free standalone mini apps (free with a Korinotch account sign-in, no purchase). Catalog: [korinotch.com/apps/](https://korinotch.com/apps/)

- [Keep Awake](https://korinotch.com/apps/keep-awake.html): one-click or timed, lid-closed on power + external display.
- [Keyboard Lock](https://korinotch.com/apps/keyboard-lock.html): overlay lock, hotkey to unlock.
- [File Shelf](https://korinotch.com/apps/file-shelf.html): Dropover-style staging on the notch.
- [Clipboard Mini](https://korinotch.com/apps/clipboard-mini.html): recent copies one click away.
- [Screen OCR](https://korinotch.com/apps/screen-ocr.html): draw a box, copy text from images, video, PDFs.
- [Color Picker](https://korinotch.com/apps/color-picker.html): any pixel to HEX.
- [Usage](https://korinotch.com/apps/usage.html): CPU, memory, network in the menu bar.
- [Break Reminder](https://korinotch.com/apps/break-reminder.html): stand up, rest your eyes on a schedule.
- [App Cleaner](https://korinotch.com/apps/app-cleaner.html): uninstall apps with their leftovers.
- [Format Converter](https://korinotch.com/apps/format-converter.html): drop a file on the notch to convert it.
- [HUD Notch](https://korinotch.com/apps/hud-notch.html): small volume HUD near the notch.

Overview: [Free Korinotch mini apps guide](https://korinotch.com/blog/free-korinotch-mini-apps-guide.html).

---

## Contributing

Missing fix, changed command, outdated price? Open a PR or issue. Keep entries factual, dated, and linked to the upstream issue where one exists. Text is CC BY 4.0 (see LICENSE-note.txt). Korinotch support: support@korinotch.com.
