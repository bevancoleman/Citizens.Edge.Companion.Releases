# Configuration reference

## Data folder

Everything the Companion stores lives in one data folder. Override it with `--data-dir <dir>`.

| OS | Default |
|---|---|
| Linux | `$XDG_DATA_HOME/citizens-edge-companion`, else `~/.local/share/citizens-edge-companion` |
| Windows (experimental) | `%LOCALAPPDATA%\CitizensEdgeCompanion` (an existing `%APPDATA%\CitizensEdgeCompanion` from an older build is kept and used) |

Contents: `companion.db` (SQLite store), `token` (pairing token), `captures/`, `logs/` (`companion.log` plus rotated copies and `crash-*.json`), and the optional config files below. Do not post this folder publicly.

## Ports and binding

The Companion listens on `127.0.0.1` only (WebSocket and HTTP on one port, default `47823`; change with `--port`). Browser connections must present the pairing token and come from an allowed origin; add more with `--allow-origin <url>` (repeatable). `ce-overlay` has its own local control port (default `47824`, `control_port` in `overlay.toml`). The CLI talks to a running Companion over a local control socket: `$XDG_RUNTIME_DIR/citizens-edge-companion.sock`, else `companion.sock` in the data folder.

## Pairing token

The first start creates a random token in `<data dir>/token` (file mode 0600). The web page and `ce-overlay` read it to prove they are allowed to talk to your Companion. Treat it like a password: do not share it, paste it in issues or commit it. It is redacted from logs. To rotate it, stop the Companion, delete the `token` file and start again; paired pages will need to pair again.

## Game folder discovery

The Companion needs the Star Citizen `LIVE` folder (it contains `Game.log` and `logbackups/`). Order:

1. `--sc-dir <dir>` (alias `--log`)
2. `CE_SC_DIR` environment variable
3. The first existing of these (Linux), each followed by `drive_c/Program Files/Roberts Space Industries/StarCitizen/LIVE`: `/mnt/games/star-citizen/star-citizen`, `~/Games/star-citizen`, `~/Games/star-citizen/star-citizen`, `~/Games/star-citizen-live`, `~/.wine`. On Windows: the RSI Launcher's library setting, then `C:\Program Files\Roberts Space Industries\StarCitizen` (and the same path on other drives), channels LIVE, PTU, EPTU, HOTFIX.

An explicit value is used as given, without checking it exists. If nothing is found, start fails and lists what it tried.

## Environment variables

| Variable | Values / default | Effect |
|---|---|---|
| `CE_SC_DIR` | folder | Star Citizen `LIVE` folder (see above). |
| `CE_ENV` | `production` (default), `staging`, `local` | Which site the web link and QR code open; same as `--env`. |
| `CE_NO_ALT_SCREEN` | `1` | Same as `--no-alt-screen`. |
| `CE_OCR_THREADS` | integer 1-64 | Threads for the OCR worker (default 2 while the game has focus, else 4); same as `--ocr-threads`. |
| `CE_OCR_MODELS_DIR` | folder | Load OCR models from this folder instead of the packaged ones. |
| `CE_FILE_LOG` | `off` | Stop writing `logs/companion.log`. |
| `CE_LOG_MAX_MB` | number, default `5` | Rotate the log at this size. |
| `CE_LOG_FILES` | integer, default `5` | Number of log files kept. |
| `CE_CHROME_CROP` | `off`, `0`, `false` | Disable the OCR step that crops window chrome. On by default. |

### Update

| Variable | Default | Effect |
|---|---|---|
| `CE_UPDATE_URL` | the project's public release feed | Directory URL holding `releases.<channel>.json`, or the feed file itself (ends in `.json`). |
| `CE_UPDATE_CHANNEL` | `linux-x64`, `linux-arm64`, `win-x64` ... by platform | Feed channel name. |
| `CE_AUTO_UPDATE` | on | `off`, `0` or `false` stops a supervised daemon (`daemon --supervise`) from updating itself. It checks every 6 h, only while the game is not running and the capture queue is idle, and only for installer-based installs. |

## Config files in the data folder

All are optional; a missing file means defaults. Edit while the Companion is stopped unless noted.

### ocr-settings.json

```json
{ "ocr": { "geometryReader": true } }
```

`ocr.geometryReader`: `true` (default; also used when unset), `false`, or `"auto"` (on only where the location profile says so). Read for every capture, no restart needed. `backfill --captures --geometry on|off` overrides it for one run.

### capture-size.json

```json
{ "maxHeight": 1440, "maxPixels": 5000000 }
```

Caps the size of saved captures. `maxHeight` (integer, default 1440, ignored below 240) and `maxPixels` (integer, default 5000000, ignored below 100000). Either key may be omitted.

### storage-policy.json

Normally written by `ce-companion storage configure`; see below.

### capture-privacy.json

```json
{ "approvedOverlays": [], "classifier": { "model": "models/capture-privacy.onnx" } }
```

Optional local safety settings for window capture. `approvedOverlays`: list of `{ "resourceClass": "...", "processId": 123 }` entries allowing a specific overlay window that sits above the game (both must match the current process; process ids change every session, so keep the list empty unless needed). `classifier.model`: path to an optional local ONNX model that vetoes captures that do not look like the game; if set and it cannot load, capture refuses rather than skipping the check. Leave unset unless you know you need it. The window gate itself cannot be turned off, and only the game window is ever captured.

### overlay.toml

Settings for `ce-overlay`. Invalid files fall back to defaults and the reason shows on the overlay's status strip. Everything is optional:

```toml
local_label = "LOC"          # name of the local clock
control_port = 47824         # ce-overlay command socket (localhost)
alert_hold_secs = 20         # wary alert on-screen time
alert_solid_secs = 20        # panel stays solid after a hostile alert
alert_hostile_hold_secs = 300
hover_solid_secs = 5         # solid after mouse hover; 0 = off
alert_wakes = true           # a new alert re-shows a hidden overlay
global_hotkeys = true        # false: register no keys
hints = "always"             # always | auto | off
hint_secs = 10
timer_ready_window_secs = 600
timer_warn_secs = 10         # 0 = no early pulse
timer_wakes = true

[link]
host = "127.0.0.1"
port = 47823                 # the Companion's port
# token_file = "~/path/to/token"   # default: <data dir>/token

[window]
width = 320.0
calm_alpha = 64              # opacity 0-255, calm
alert_alpha = 235            # opacity 0-255, during an alert
# x = 100.0                  # start position (not on Wayland)
# y = 100.0

[[clocks]]                   # pinned clocks, repeatable
label = "EST"
zone = "America/New_York"    # IANA zone

[hotkeys]                    # command = trigger; "" turns a key off
toggle = "CTRL+SHIFT+ALT+F1" # XDG shortcut syntax

[duration_secs]              # CZ timer lengths in seconds, by kind
door = 45

[[location]]                 # CZ sub-panels; empty = built-in list
name = "CZ"
icon = "▣"
timers = ["door"]
places = ["some place name"] # suggest this panel when your location contains a word

[exec_hangar]                # exec hangar cycle; absent = unverified placeholder
epoch = "2026-01-01T00:00:00Z"
phases = [{ name = "RED", secs = 3600 }, { name = "GREEN", secs = 900 }]
```

The built-in CZ timer lengths and hangar schedule are placeholders (the panel says so); measure them in game and set `duration_secs` / `exec_hangar`. A `[[location]]` may instead use `kind = "hangar"` for the computed hangar schedule. Hotkey commands: `toggle`, `show`, `hide`, `move`, `compact`, `scan`, `dismiss`, `hint`, `mode`, `panel`, `slot1` to `slot9`, `quit`.

## Storage budget and cleanup

Captures are kept under a budget. Defaults, in `storage-policy.json` (camelCase keys):

| Key | Default | Meaning |
|---|---|---|
| `enabled` | `false` | Scheduled conversion and expiry. The size cap is always enforced. |
| `maxBytes` | `10000000000` (10 GB) | Total capture budget. |
| `protectedBytes` | `2147483648` (2 GiB) | Part of the budget reserved for protected captures; must fit in `maxBytes`. |
| `previewAfterDays` | `7` | Originals (PNG) are replaced by a reduced preview after this many days. |
| `deleteAfterDays` | `90` | Captures expire after this many days (1-3650, greater than `previewAfterDays`). |
| `ignoredDeleteAfterDays` | `7` | Ignored captures expire after this many days. |
| `previewMaxWidth` | `1280` | Preview width in pixels (320-7680). |
| `previewQuality` | `80` | Preview quality (1-100). |

An invalid file is reported as an error rather than silently replaced by defaults. Preview what cleanup would do with `ce-companion storage cleanup`, then apply it with `--apply`. Cleanup is held while the game is running. Protect individual captures with `storage protect`.
