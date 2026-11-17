# Command reference

`ce-companion` is one program with sub-commands. Run `ce-companion help <command>` for the built-in usage of any of them. Commands that ask a running Companion (status, capture, captures, ...) talk to it over a local control socket; some also work without one by reading the data folder.

Global options accepted by every command: `--data-dir <dir>` (data folder, see [configuration](configuration.md)), `--json` (structured output where supported), `--log-format text|json|full` (`text` on a terminal, `json` when piped, `full` = text with dates), `--utc` (UTC timestamps), `-h`/`--help`. `ce-companion --version` prints the version.

Windows is **experimental**: every command is built for it, but none has been run on a real Windows machine yet, so expect problems and report them.

## Running

| Command | What it does |
|---|---|
| `ce-companion` / `ce-companion start` | The Companion plus, on a terminal, its live view, in one process. Backfills, tails `Game.log`, serves the live feed on `127.0.0.1`. |
| `ce-companion daemon` | Headless Companion for a service or autostart entry. |
| `ce-companion client` (alias `tui`) | Live view of a Companion that is already running. |

Options for `start` and `daemon`:

| Flag | Meaning |
|---|---|
| `--sc-dir <dir>` | Star Citizen `LIVE` folder (alias `--log`). See [game folder discovery](configuration.md#game-folder-discovery). |
| `--port <n>` | WebSocket port. Default `47823`. |
| `--allow-origin <url>` | Extra allowed browser origin. Repeatable. |
| `--no-capture` | Disable screen capture and OCR. |
| `--no-hotkey` | Do not register the global capture shortcut; bind `ce-companion capture` in your desktop's shortcut settings instead. |
| `--ocr-threads <n>` | onnxruntime threads for the low-priority OCR worker. Default: 2 while Star Citizen has focus, 4 otherwise; a number fixes it. |
| `--ocr-relief auto\|off\|<1-10000>` | Give OCR workers a low CPU weight while the game runs (Linux, needs `systemd-run --user`; otherwise ignored). `auto` is the default. |
| `--env production\|staging\|local` | Which site the printed web link and QR code open (or `CE_ENV`). Default `production`. |
| `--web-url <url>` | The `/live` page the link opens, for example `http://localhost:3000/live/`. |
| `--verbose` | Report every `Game.log` batch. |
| `--data-dir <dir>` | Data folder. |
| `--full-desktop` | Rejected: only the game window is ever captured. |

`start` only: `--no-ui` (plain log output, also used when piped), `--no-alt-screen` (live view on the normal screen so terminal scrollback works; or `CE_NO_ALT_SCREEN=1`), `--no-qr` (do not print the QR code).
`daemon` only: `--supervise` restarts it after a crash (Linux), forwards SIGINT/SIGTERM and keeps a crash log.

```sh
ce-companion start --sc-dir "~/Games/star-citizen/drive_c/Program Files/Roberts Space Industries/StarCitizen/LIVE"
ce-companion daemon --supervise --no-hotkey
ce-companion client
```

## Status and inspection

| Command | What it does |
|---|---|
| `status [--json] [--port n] [--env e] [--web-url u] [--no-qr]` | Store location and event counts; with a Companion running also tailing state, ship, location, shard, capture queue and OCR timing, web clients and uptime. |
| `logs [--limit n] [--errors] [--crashes] [--since 1h\|30m\|2d\|<ISO date>] [--utc] [--json]` | Path of the rotating log file and its last lines (default 50). `--errors`: warnings, errors and crash records only. `--crashes`: list crash records. |
| `models [--json]` | Where the OCR models come from in this build, and the state of the optional privacy classifier. |
| `combat [--limit n] [--json]` | Combat timeline: sessions from leaving to entering armistice, newest first. |
| `combat auto-capture [off \| <trigger>]` | Show or set when a screenshot is taken after a combat event. Off by default; at most one per 20 s and 10 per hour. |
| `notify` | Show notification settings. |
| `notify test [--kind hostile\|kos\|wary]` | Send a test alert through the enabled channels. |
| `notify set <setting> <value>` | Settings: `audio`, `mute`, `voice` (on/off), `volume` (0-100), `cooldown` (minutes), `desktop` (pop-ups, default off), `quiet` (on/off), `quiet-start`/`quiet-end` (HH:MM). |

```sh
ce-companion status --json
ce-companion logs --errors --since 1h
```

## Captures

| Command | What it does |
|---|---|
| `capture` | Take a screenshot of the game window and OCR it (on the running Companion if there is one). |
| `capture import <file>` | Import an existing screenshot and OCR it. |
| `captures [--status s] [--tag t] [--limit n] [--json]` | List captures. `--status`: `processed`, `unrecognised`, `failed`, `ignored`, `processing` (repeatable). `--tag`: `evidence`, `report-example`, `favourite` (repeatable). |
| `captures --id <id>` | One capture, by full id or short id (`C-YYMMDD-HHMMSS-mmm`). `--full` reindexes every folder. |
| `ocr-report [--since <date>] [--json]` | OCR recall by screen type, from captures you annotated. |
| `ocr-report --quality [--truth <file>\|confirmed] [--baseline <file>] [--since <date>] [--now]` | Commodity field quality and the change from a saved `--json` report. |
| `annotations export [--out <dir>] [--since <date>]` | Annotated captures as a `references.md` table (stdout, or `<dir>/references.md`). |

```sh
ce-companion capture
ce-companion captures --status failed --limit 10
```

## Re-reading logs and captures

These are heavy and exclusive. While Star Citizen is running they are held until the game exits; `--now` overrides that.

| Command | What it does |
|---|---|
| `backfill [--full] [--now]` | Ingest new `logbackups/*.log` and `Game.log`. `--full` re-parses every log. |
| `backfill --captures [--kind <kind>] [--only-missing] [--geometry on\|off] [--deep] [--yes]` | Re-run OCR on saved captures. `--kind` limits to one kind (for example `commodity-terminal`); `--only-missing` skips captures already read by the current reader; `--geometry` forces the geometry reader on or off for this run; `--deep` also reads kiosk rows with the deep model (slow, asks first). |
| `reevaluate [--kind <kind>] [--dry-run] [--yes] [--json]` | After an upgrade: re-parse logs, re-run OCR, re-check confirmed values, remove stale derived data. `--dry-run` changes nothing. Captures and images are never touched. |

```sh
ce-companion backfill --full
ce-companion reevaluate --dry-run
```

## Storage

| Command | What it does |
|---|---|
| `storage` | Show capture-storage status and policy. |
| `storage configure [--enable\|--disable] [--max-bytes n] [--protected-bytes n]` | Change the policy. `--enable` turns on scheduled cleanup. |
| `storage cleanup [--apply]` | Preview which old captures would be reduced or expired; `--apply` does it. Held while the game runs. |
| `storage protect <capture-id> --enable\|--disable` | Protect a capture from expiry. |
| `storage register --manifest <json> [--reason text] [--apply]` | Protect a list of capture ids from a JSON file. |

See [storage budget and cleanup](configuration.md#storage-budget-and-cleanup) for defaults.

```sh
ce-companion storage configure --enable --max-bytes 20000000000
ce-companion storage cleanup
```

## Truth review (commodity OCR)

`truth` lets you review and confirm what the OCR read from commodity terminals: `truth queue [--limit n] [--json]` (doubtful rows first), `truth confirm <id> [--price n] [--quantity n] [--status text] [--name text]`, `truth confirm --all-agreed [--yes]`, `truth unconfirm <id>`, `truth report <id> needs-reprocessing|bad-crop|unreadable [--note text]`, `truth reports`, `truth resolve <id> --note text`, `truth stats`. Further `truth` sub-commands are developer tools.

```sh
ce-companion truth queue --limit 20
```

## Update

| Command | What it does |
|---|---|
| `update --check` | Compare with the newest release. |
| `update --download` | Download and stage the newest release. |
| `update --apply` | Apply the staged release and restart (refused while a Companion runs for the data folder). |

Updating works for installs made with the installer; the tarball and development builds report "updates unavailable". Variables: `CE_UPDATE_URL`, `CE_UPDATE_CHANNEL`, `CE_AUTO_UPDATE` (see [configuration](configuration.md#update)).

## Shell completions and help

```sh
ce-companion completions bash   # also: zsh, fish, powershell
ce-companion help status
```

## ce-overlay

`ce-overlay` is a separate small always-on-top panel (clocks, timers, alerts) that reads the Companion's feed. There is no settings screen; it is configured by `overlay.toml` (see [configuration](configuration.md#overlaytoml)).

```
ce-overlay [--data-dir <dir>] [--window | --layer] [--move] [--solid] [--demo] [--app-id <id>]
ce-overlay <command>
```

| Flag | Meaning |
|---|---|
| `--data-dir <dir>` | Companion data folder (where `overlay.toml` and the pairing token live). |
| `--window` / `--layer` | Force a normal window or a layer-shell surface (not both). |
| `--move` | Start in move mode. |
| `--solid` | Start with the panel fully opaque. |
| `--demo` | Run with demo data. |
| `--app-id <id>` | Window application id. Default `ce-overlay`. |
| `-h`, `--help` | Usage. |

With no command, `ce-overlay` starts the panel (one instance per control port). With a command, it sends it to the running overlay and exits (`ce-overlay is not running` and exit 1 if none): `toggle`, `show`, `hide`, `move`, `compact`, `scan`, `dismiss`, `hint`, `mode`, `panel`, `slot1` to `slot9`, `quit`.

Default global keys (Linux, via the desktop's GlobalShortcuts portal; set `global_hotkeys = false` to leave all keys alone): Meta+Keypad 0 (hint), Keypad Enter (dismiss), `.` (compact), `*` (mode), `/` (panel), and keypad 1 to 9 (slots). Show/hide and `scan` have no default key; `ce-overlay toggle` works from any shortcut tool.

```sh
ce-overlay --demo
ce-overlay toggle
```
