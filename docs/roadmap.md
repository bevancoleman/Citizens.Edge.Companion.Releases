# Companion roadmap: what we are looking at adding

This is a list of ideas **under consideration** for the Citizen's Edge Companion. It is not a promise or a schedule. Nothing here is built yet unless it says so, and items may change, move or be dropped as we learn what is practical and what people actually want.

**Want something on this list sooner, or something that is not here?** Open an issue in this repository with the title starting `Feature:`. Adding a 👍 to an existing request helps us see what matters most. Bugs and OCR reports go in issues too (see the [main page](../README.md#problems)).

## Ground rules for every feature

These come from the Companion's principles and apply to everything below:

- **Read-only towards the game.** Features read `Game.log` and capture the game window. They do not hook the game, read its memory, send it input, or change its files.
- **Local by default.** Your data stays on your PC. Anything shared with other people (for example your org) is off until you turn it on, and is per feature.
- **Game updates can break things.** Features that read the log depend on the log's format, which Cloud Imperium can change in any patch. When that happens a log-based feature may stop working until we update it, and we will say so rather than show wrong data.

## How to read the tables

| Column | Meaning |
|---|---|
| **Reads** | Where the data comes from: **Log** (`Game.log`), **Screen** (capture + OCR on your hotkey), **Online** (needs the Citizen's Edge site or an org account), **None** (built-in reference data) |
| **Platforms** | Where we expect it to work. Linux (KDE/Wayland) is the supported platform today; Windows is experimental |

---

## Blueprints and reputation

Crafting blueprints are earned through missions and the list is long, so this is the area we are most interested in.

| Idea | What it would do | Reads | Notes |
|---|---|---|---|
| **Blueprint tracker** ([#4](https://github.com/bevancoleman/Citizens.Edge.Companion.Releases/issues/4)) | Shows which blueprints you own, which you are missing (with the missions that unlock them and a link out to a community mission database), and a watch list of ones you want. Search and filter by type, class, maker or grade. Alerts you the moment a new one drops. | Log | The game writes a line to `Game.log` when you receive a blueprint. We would also read the older log files the game keeps, to fill in your collection on first run. The game deletes old logs, so we would keep our own record. |
| **Set and progress views** | Armour as sets ("one piece from complete"), ship weapons by size ladder, components by type and size. | Log | Builds on the blueprint tracker. |
| **Reputation tracking** | Shows your standing with the factions and contacts you work for, as a number with progress to the next rank. The in-game screen only shows a bar. | Log, Screen | Estimated from contract results in the log, and corrected by reading the in-game reputation screen when you press the scan key. We are not yet sure how complete the log information is, so early versions may be approximate. |
| **Find a grind partner** | For a blueprint you want, shows which people in your org have the reputation to run the missions that award it, and who already owns it. | Online | Org members only, and only for people who chose to share their blueprints and reputation. Never shared outside your org. |
| **Org blueprint registry** | Who in your org can craft what, and who is missing what. | Online | Opt-in per person. Never shared outside your org. |
| **Grind route planner** | Pick the blueprints you want and your current reputation, and get a step-by-step list of contracts to run. | None, Log | Uses your tracked reputation if you have it on, otherwise you enter it. |
| **Mission details in the overlay** | In the overlay, show what a mission's blueprint pool is, the chance, the reputation needed and how many of its blueprints you already own. | Log | We are deliberately showing this in the overlay and **not** editing the game's own text files. Some community tools do edit them, but that would break our rule of never changing the game's files. |
| **Crafting calculator** | Pick a blueprint, set the material quality, see the recipe and the resulting changes. | None | Reference data needs updating every patch. |
| **Org work orders** | Queue items to craft with quantities and share the list with your org. | Online | Builds on the crafting calculator. |

## Trading and hauling

| Idea | What it would do | Reads | Notes |
|---|---|---|---|
| **Price age on every price** | Shows how old each price is, based on when a player last reported it, not when it was fetched. | Online | You can already see "last updated" on the trade panel. This would make it consistent everywhere. |
| **Trade route planner** | Best buy and sell pairs for your cargo size, in the current system or globally. | Online | Uses community price data, with a newer scan from your org taking priority. |
| **Hauling mission import** | Press the scan key on the mission list to read hauling missions, then plan an efficient stop and delivery order. | Screen | Uses the same hotkey capture as commodity terminals. |

## Mining

| Idea | What it would do | Reads | Notes |
|---|---|---|---|
| **Signature reader** | Reads the signature number on your HUD and tells you the likely mineral, multiplier and rarity. | Screen | Will need a setup step to mark where the number appears on your screen. Fast camera movement can cause missed reads. |
| **Mining solver estimate** ([#6](https://github.com/bevancoleman/Citizens.Edge.Companion.Releases/issues/6)) | For the rock you just scanned, which head and modules to use with your setup, and whether you need more than one player (for example in a MOLE). | Screen | Needs your mining setup entered once in the preferences screen. |
| **Refined yield estimate** ([#7](https://github.com/bevancoleman/Citizens.Edge.Companion.Releases/issues/7)) | Shows the SCU you can expect after refining, in brackets, on each scan. | Screen | An estimate, marked `~`: yield varies by refinery. |
| **Spoken scan readout** ([#8](https://github.com/bevancoleman/Citizens.Edge.Companion.Releases/issues/8)) | Optionally says the scan result aloud, for example "titanium, five". | Screen | Off by default. |

## Contacts and friend-or-foe

| Idea | What it would do | Reads | Notes |
|---|---|---|---|
| **Nameplate detection** | A second way to spot people on your contacts list: reading nameplates in the world, in addition to the radar scan. | Screen | Needs the game in borderless or windowed mode, and clear text. Nameplates only appear at short range. |
| **Unified contact display** ([#10](https://github.com/bevancoleman/Citizens.Edge.Companion.Releases/issues/10)) | One panel with the latest contacts and their state (hostile, friendly, neutral, or unknown if not yet looked up), with events second. Alerts only for the last few hostile contacts scanned where you are. | None | Overlay layout change. |

## Overlay

| Idea | What it would do | Reads | Notes |
|---|---|---|---|
| **Pinned items** | Keep chosen things (a watched blueprint, an ore, a contact) visible in a small always-on HUD that does not take clicks. | None | |
| **Sound alerts** | Optional short sounds for important events (hostile contact nearby, blueprint drop). | None | Off by default. |
| **Look and feel options** | Adjustable transparency, hide automatically when the game is not in focus, remember position, quick click-through switch. | None | Some of this exists already on Linux. |
| **Notes and timers** | A scratch area for notes and ad-hoc timers. | None | |
| **Choose your panels** | Turn individual overlay panels on or off so you only see what you use. | None | |
| **Last-scan status line** ([#12](https://github.com/bevancoleman/Citizens.Edge.Companion.Releases/issues/12)) | One line at the top showing the last scan and how long ago, instead of a feed entry for every scan. Failures still show in the feed. | None | |
| **Background-pause indicator** ([#9](https://github.com/bevancoleman/Citizens.Edge.Companion.Releases/issues/9)) | Shows when background reading is paused (combat, a hostile contact marked, or heavy PC load), why, and until when. | None | Also in the terminal view and `status`. |

## From the game log

| Idea | What it would do | Reads | Notes |
|---|---|---|---|
| **Shared mission helper** | Spots a mission shared by a party member and helps you accept it before the in-game prompt vanishes. | Log | We would only assist, not press anything for you: the Companion never sends input to the game. |
| **Crash and disconnect alerts** | Tells you when the game has crashed or you were disconnected, and whether it looks like your PC or the servers. | Log | |
| **Career and session stats** | Playtime, ships flown, missions, spending and travel, by patch. | Log | Some figures the game no longer records in the log, such as kills and wallet balance, so those would show as unavailable rather than zero. |
| **Log backup and move between PCs** | Keeps your history even after the game deletes old logs, and lets you export and import it. | Log | |

## The Companion itself

| Idea | What it would do | Reads | Notes |
|---|---|---|---|
| **Preferences screen** ([#5](https://github.com/bevancoleman/Citizens.Edge.Companion.Releases/issues/5)) | Set general options, feature switches, trading preferences and your mining ship from the `/live` page; the Companion stores them. | None | The overlay stays display-only. |
| **Use it from phones, tablets and other PCs** ([#3](https://github.com/bevancoleman/Citizens.Edge.Companion.Releases/issues/3)) | Open the `/live` page on another device and pair it with a short code, directly over your home network where possible. | None | Off by default, view-only to start, screenshots never leave your PC. |
| **Documented, versioned API** ([#2](https://github.com/bevancoleman/Citizens.Edge.Companion.Releases/issues/2)) | A readable, versioned reference so other tools can build on the Companion. | None | The interface is not public yet, so it can be tidied before the first version. |
| **Memory use display** ([#11](https://github.com/bevancoleman/Citizens.Edge.Companion.Releases/issues/11)) | Shows memory used by the Companion and the game, and free memory, in the terminal view and `status`. | None | Read-only: normal OS process figures only. |
| **Full user documentation** ([#13](https://github.com/bevancoleman/Citizens.Edge.Companion.Releases/issues/13)) | Install, first run, every command and setting, for all the apps. | None | Where to host it is undecided. |

## Accounts and sharing

| Idea | What it would do | Reads | Notes |
|---|---|---|---|
| **Works without an account** | Everything personal (blueprints, stats, timers) works with no sign-in. Sign-in is only for org features. | None | This is how it works today for the features that exist; the aim is to keep it that way. |
| **Hangar sync** | Keep your hangar the same between browsers and the Companion. | Online | Today your hangar is stored in the browser only. We have not yet decided on the source of the data. |
| **Org shared sheets** | Shared tables for your org, such as stock or loadouts. | Online | |

---

## Not planned

- Anything that reads the game's memory, injects code, or sends input or automates gameplay.
- Editing the game's own files or text.
- Showing a person's private data to anyone outside their org.
