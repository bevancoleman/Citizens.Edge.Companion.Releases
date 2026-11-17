# Citizen's Edge Companion: downloads

This repository holds **only the built downloads and update feed** for the Citizen's Edge Companion. It contains no source code: the project's source repositories are private, so this is the one public place to get the files.

## What is the Companion?

A small program that runs on your gaming PC next to Star Citizen. It:

- reads the game's `Game.log` and keeps a local database of what happened;
- on a hotkey, captures the game window and reads it with OCR (for example commodity terminals);
- streams that live to the Citizen's Edge **/live** page in your browser over a local connection.

Principles:

- **Read-only towards the game.** It reads the log and captures the game window. It never writes into the game's install and never hooks the game's input or memory (Easy Anti-Cheat).
- **Local by default.** It listens on `127.0.0.1` only, protected by a pairing token and an origin check. Nothing is shared unless you turn it on.
- **Your data stays on your PC**, in a local data folder.

## Which download do I want?

Releases here come in two channels:

| Channel | Where | For |
|---|---|---|
| **Production** | Releases named `vX.Y.Z` (the latest one is "Latest") | Everyone. *There is no production release yet.* |
| **Staging** | The pre-release named `staging` | Testers. It is rebuilt from the newest development code on every change, so it can be broken. |

| Platform | Status |
|---|---|
| Linux x64 | Works. Recommended: the **AppImage** (single file, self-updating). A tarball is still published for now. |
| Windows x64 | **Experimental: written, but not yet run on a real Windows machine.** The installer, window capture, hotkeys, overlay and self-update are all built, so expect rough edges and please report them. Installers are unsigned, so SmartScreen will warn. Run Star Citizen in **borderless or windowed** mode: exclusive fullscreen cannot be captured. |

## Linux: install the AppImage

The AppImage is one file: download it, make it executable and run it. It can update itself the same way the Windows installer does (see Updating below).

```sh
base=https://github.com/bevancoleman/Citizens.Edge.Companion.Releases/releases/download/staging   # or .../download/vX.Y.Z
curl -LO "$base/CitizensEdgeCompanion-linux-x64.AppImage"
chmod +x CitizensEdgeCompanion-linux-x64.AppImage
./CitizensEdgeCompanion-linux-x64.AppImage            # start the Companion and its live view
./CitizensEdgeCompanion-linux-x64.AppImage overlay    # the in-game overlay (also: toggle, hide, nudge left, ...)
```

`SHA256SUMS-linux-x64.txt` lists the checksum (`sha256sum CitizensEdgeCompanion-linux-x64.AppImage`). Everything below that says `./ce-companion ...` works the same as `./CitizensEdgeCompanion-linux-x64.AppImage ...`. Putting the file somewhere permanent, such as `~/Applications`, is a good idea, because self-update replaces it in place.

## Linux: the tarball (alternative)

The tarball is kept for packagers and for anyone who prefers plain binaries; it does **not** update itself. It will be dropped once the AppImage has proved itself.

Download the files from the [`staging` pre-release](https://github.com/bevancoleman/Citizens.Edge.Companion.Releases/releases/tag/staging) (or a `vX.Y.Z` release once there is one), then verify and unpack:

```sh
# 1. download (staging shown; for a version use .../releases/download/vX.Y.Z/...)
base=https://github.com/bevancoleman/Citizens.Edge.Companion.Releases/releases/download/staging
curl -LO "$base/ce-companion-linux-x64.tar.gz"
curl -LO "$base/ce-companion-linux-x64.tar.gz.sha256"

# 2. verify the checksum (prints "ce-companion-linux-x64.tar.gz: OK")
sha256sum -c ce-companion-linux-x64.tar.gz.sha256

# 3. unpack
tar -xzf ce-companion-linux-x64.tar.gz
cd ce-companion-linux-x64

# 4. see which build you have
cat BUILD-INFO.json        # version and the exact commit it was built from
```

The folder contains the `ce-companion` program, the `ce-overlay` in-game HUD a `models/` folder (the OCR models, keep it next to the program) and `BUILD-INFO.json`.

## Run it

```sh
./ce-companion              # start the Companion and its live view
./ce-companion start --sc-dir "/path/to/StarCitizen/LIVE"   # if it cannot find your LIVE folder
./ce-companion daemon       # headless, for a service or autostart
./ce-companion status       # what the running Companion is doing
./ce-companion --help       # all commands
```

Then open the Citizen's Edge **/live** page in your browser; it connects to the Companion on your own machine.

Your data lives in `~/.local/share/citizens-edge-companion/` on Linux (`%LOCALAPPDATA%\CitizensEdgeCompanion` on Windows). It includes your pairing token, so do not share that folder.

## The interfaces

The Companion has no window of its own. You look at what it sees in three places, all talking to the program on your own PC (never to third-party sites):

| Interface | What it is | In these downloads? |
|---|---|---|
| **/live page** | The Citizen's Edge web page, in your browser. The fullest view: log events, captures, contacts, timers. | Yes: the page is hosted by Citizen's Edge, it connects to your local Companion. |
| **Terminal view** | `ce-companion` itself shows a live view in the terminal; `ce-companion client` opens the same view (with a command prompt) against a Companion that is already running. | Yes, part of `ce-companion`. |
| **In-game overlay** | `ce-overlay`: a small heads-up display drawn over the game. Status strip, contact alerts, and a context box with tabs (trade, mining, contested-zone timers, and so on). It never takes keyboard focus from the game; you drive it with global hotkeys. | **Linux: yes** (`overlay` in the AppImage, or `ce-overlay` in the tarball). **Windows: included but untested** (`ce-companion overlay`; a click-through window that never takes focus). Linux (Wayland, KDE) is the supported platform. |

The overlay and the terminal view show the same data as `/live`, with more detail in places. They are controlled with commands such as `ce-overlay toggle`, `show`, `hide`, `compact` and `quit`, plus the hotkeys it registers (the numeric keypad, with the Windows key to move it). Run `ce-overlay --demo` to preview it with sample data and no connection. On KDE you can also move it without the mouse (handy while the game holds the pointer): `ce-overlay nudge up|down|left|right` and `ce-overlay snap`, which have default keys (shown in the overlay's key hints).

## Windows: install (experimental)

Download `CitizensEdgeCompanion-win-x64-Setup.exe` from the release and run it. Windows may show a SmartScreen warning because it is not signed yet. Alongside it you will find `SHA256SUMS-win-x64.txt` to verify the download:

```powershell
Get-FileHash .\CitizensEdgeCompanion-win-x64-Setup.exe -Algorithm SHA256
# compare with the line in SHA256SUMS-win-x64.txt
```

Uninstalling (Settings > Apps) removes the start-at-login entry but never deletes your data folder. A portable zip is also published; it does not get a Start-menu entry.

## Updating

- **Linux tarball:** download the new release and replace the folder. Keep your data folder; it is separate.
- **Linux AppImage and Windows installer (both Velopack):** the Companion can update itself. Check by hand with:

  ```sh
  ce-companion update --check      # is there a newer version?
  ce-companion update --download   # fetch it
  ce-companion update --apply      # install it and restart
  ```

  A running background Companion also does this on its own every few hours, but only while the game is **not** running and nothing is being processed. Turn that off with the environment variable `CE_AUTO_UPDATE=off`.
- The update feed is this repository's latest release. To follow the staging channel instead, or to point at a different feed:

  ```sh
  CE_UPDATE_URL=https://github.com/bevancoleman/Citizens.Edge.Companion.Releases/releases/download/staging \
    ce-companion update --check
  ```

  Other settings: `CE_UPDATE_CHANNEL` (`win-x64` or `linux-x64`).
- Builds that were not installed by the installer or AppImage (the tarball, a build from source) report "updates unavailable" instead of updating themselves.

## What is in each release

| File | What |
|---|---|
| `CitizensEdgeCompanion-linux-x64.AppImage` | Linux, self-updating (recommended) |
| `ce-companion-linux-x64.tar.gz` + `.sha256` | Linux programs (`ce-companion`, `ce-overlay`), models and `BUILD-INFO.json`; no self-update |
| `CitizensEdgeCompanion-win-x64-Setup.exe` | Windows installer |
| `CitizensEdgeCompanion-win-x64-Portable.zip` | Windows portable zip |
| `CitizensEdgeCompanion-<version>-<platform>-full.nupkg`, `releases.<platform>.json`, `RELEASES-<platform>`, `assets.<platform>.json` | Update feed files the updater reads (`<platform>` is `linux-x64` or `win-x64`). You do not need to download these by hand. |
| `SHA256SUMS-<platform>.txt`, `BUILD-INFO-<platform>.json` | Checksums and build info |

Releases are built automatically from a single reviewed commit: the production release for a version is the exact build that was tested as staging, promoted without rebuilding.

## Problems

Open an issue in this repository. Please include the output of `ce-companion status` and the contents of `BUILD-INFO.json`, but **not** your data folder or pairing token.

## Reference

- [Command-line reference](docs/commands.md): every `ce-companion` and `ce-overlay` command and switch.
- [Configuration reference](docs/configuration.md): data folder, environment variables, config files, updates and storage.
