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
| Linux x64 | Works. Tarball. |
| Windows x64 | **Experimental, untested on real machines yet.** An installer is published, but several Windows features (global hotkey, window capture, the control channel) are not implemented, so it does not do the main job yet. Installers are unsigned, so SmartScreen will warn. |

## Linux: install from the tarball

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

The folder contains the `ce-companion` program, a `models/` folder (the OCR models, keep it next to the program) and `BUILD-INFO.json`.

## Run it

```sh
./ce-companion              # start the Companion and its live view
./ce-companion start --sc-dir "/path/to/StarCitizen/LIVE"   # if it cannot find your LIVE folder
./ce-companion daemon       # headless, for a service or autostart
./ce-companion status       # what the running Companion is doing
./ce-companion --help       # all commands
```

Then open the Citizen's Edge **/live** page in your browser; it connects to the Companion on your own machine.

Your data lives in `~/.local/share/citizens-edge-companion/` on Linux (`%APPDATA%\CitizensEdgeCompanion` on Windows). It includes your pairing token, so do not share that folder.

## Windows: install (experimental)

Download `CitizensEdgeCompanion-win-x64-Setup.exe` from the release and run it. Windows may show a SmartScreen warning because it is not signed yet. Alongside it you will find `SHA256SUMS-win-x64.txt` to verify the download:

```powershell
Get-FileHash .\CitizensEdgeCompanion-win-x64-Setup.exe -Algorithm SHA256
# compare with the line in SHA256SUMS-win-x64.txt
```

Uninstalling never deletes your data folder.

## Updating

- **Linux tarball:** download the new release and replace the folder. Keep your data folder; it is separate.
- **Windows installer (Velopack):** the Companion can update itself. Check by hand with:

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
- Builds that were not installed by the installer (the tarball, a build from source) report "updates unavailable" instead of updating themselves.

## What is in each release

| File | What |
|---|---|
| `ce-companion-linux-x64.tar.gz` + `.sha256` | Linux program, models and `BUILD-INFO.json` |
| `CitizensEdgeCompanion-win-x64-Setup.exe` | Windows installer |
| `CitizensEdgeCompanion-win-x64-Portable.zip` | Windows portable zip |
| `CitizensEdgeCompanion-<version>-win-x64-full.nupkg`, `releases.win-x64.json`, `RELEASES-win-x64`, `assets.win-x64.json` | Update feed files the installer's updater reads. You do not need to download these by hand. |
| `SHA256SUMS-win-x64.txt`, `BUILD-INFO-win-x64.json` | Checksums and build info for the Windows files |

Releases are built automatically from a single reviewed commit: the production release for a version is the exact build that was tested as staging, promoted without rebuilding.

## Problems

Open an issue in this repository. Please include the output of `ce-companion status` and the contents of `BUILD-INFO.json`, but **not** your data folder or pairing token.
