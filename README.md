<p align="right">🌐 <b>English</b> · <a href="./README.zh-CN.md">简体中文</a></p>

<div align="center">

<img src="docs/assets/readme/actingcommand-icon.png" width="112" alt="ActingCommand icon">

**Chief Executive Officer & Chairman** — HS7097<br/>
**Chief Technology Officer & Chief Architect** — Claude Opus 5.5 · GPT‑6 Astra · Claude Fable 5 · GPT‑5.6 Sol<br/>
**Board Secretary & Chief Audit Officer** — Claude Opus 5.5 · Claude Fable 5.1<br/>
**Principal Engineer** — Claude Opus 5.5 · GPT‑6 Astra · GPT‑5.6 Sol<br/>
**Interviewing** — DeepSeek

</div>

**⚠️ Pre-release, debug phase: every release is a pre-release, and interfaces, configuration and file formats will still change. Only the newest version is supported. The 0.12 series brings breaking changes (see [Roadmap](#roadmap-planned)).**

# ActingCommand

- [ActingCommand-Runtime](https://github.com/HS7097/ActingCommand-Runtime) — resident Rust runtime, the core program, with its command line, tools and local MCP server
- [ActingCommand-UI](https://github.com/HS7097/ActingCommand-UI) — setup wizard and read-only console
- Standard packs for **Arknights**, **Azur Lane** and **Blue Archive** — published as assets of this repository's [Releases](https://github.com/HS7097/ActingCommand/releases); they are developed in separate repositories that are not public

## This repository

This is the umbrella (portal) repository of the ActingCommand family. It holds no code: the code lives in the member repositories linked above. This repository carries the README, the program skill under `skills/`, and the versioned releases on its Releases page: the setup wizards and, unchanged, the release assets of the members for each version. `bundles/` keeps the Blue Archive bundle of the retired daily publishing for reference (no longer updated), and `docs/` holds the README images and older notes.

**What we can do:** Let an AI agent deploy our program, or install it yourself with the setup wizard published with every release (online `acsetup.exe`, offline `acsetup-full-<tag>.exe`). Once the program skill is loaded in your harness and the MCP server is registered (see [Use with an AI agent](#use-with-an-ai-agent-mcp)), you can have the agent produce the materials that a program running in an Android emulator needs, and then run it again and again on a schedule.

**What the agent has to do:** Make some images and click regions.

**How we do it:** The Runtime is a resident Rust program that contains no game logic of its own. It connects to Android emulators over ADB (and the emulator vendor's interfaces), captures frames on a cadence, recognizes materials in each frame (template, color, OCR, neural network), clicks inside regions declared in advance, and writes every step, what it saw, what it did and how it went, as typed events into an append-only ledger. The ledger is the authoritative record: the scheduler decides the next run from it, the console only reads it, and when something goes wrong it is traced from the ledger alone. Everything game-specific, the images, the click regions, the task order and the data tables, lives in a per-game standard pack whose task packs are sealed by the digest of their content; the Runtime only loads, verifies and executes them. Switch the game by switching the pack; the program does not change.

**Join in:** If you have a better idea, or run into a problem while using it, open an issue in this repository or in the repository concerned.

## Status

- **Debug phase.** The main loop is: find a problem, fix it, check the result against what is expected, change again. Deployment and convenience come second.
- **Pre-release only.** Every release of every member is a pre-release. Only the newest version is supported; older series get no fixes.
- **Breaking changes ahead.** From 0.12.0 the Runtime uses a new ledger format and does not carry a 0.11 state root over, the command line's output and exit codes change, the MCP tiers are removed and Lab is no longer installed by default (see [Roadmap](#roadmap-planned)).
- **On real devices.** Since early October the Runtime runs routine batches every day on real MuMu instances under its approved catalog, with all three standard packs. The packs do not cover every task yet, and multi-day unattended runs are still being validated.
- **Version numbers.** X for major or incompatible changes, Y for new features or new coverage, Z for fixes (including features that serve a fix). Each member releases on its own; whether members fit together is decided by their declared interfaces, not by equal version numbers.

## What works today

| Part | What it does |
|---|---|
| Runtime (`actingcommand-actingd.exe`) | Resident process that accepts requests only on the loopback address (127.0.0.1 / [::1]) and has no network code; clients can close without affecting it. Scheduler: runs task packs by the approved catalog and its clock slots, with leases and fences re-checked on every device touch, pause and resume (global or per instance, kept across restarts from v0.11.3), priority offsets, resource targets and a disk-capacity gate; one request queue per instance from v0.11.5 and one worker per instance from v0.11.6. A failed routine run goes to the three-rung recovery ladder: return home, restart the app, restart the emulator (runs started by hand do not). From v0.11.6, after a restart every unfinished run is settled exactly once, by its terminal state or as interrupted, and a run that cannot be settled at start pauses only its instance and reports an Error. |
| Recognition and devices | Template, color, click-only, OCR and neural-network targets, plus color digests; OCR and NN run in process on ONNX Runtime, with models loaded on first use. MuMu: Nemu IPC capture and input, instance discovery and start/stop through MuMuManager; the bundled adb 37.0.1. |
| Ledger | A single-writer SQLite ledger of sanitized typed events; frames are stored by content digest. From v0.11.6 a frame retention cleaner removes frames that are no longer needed and moves the frames kept for errors and Lab into `<state root>\kept\<date>\`, where a person deletes them. |
| Command line and tools | `actingctl` (status, pause and resume, emulator control, request shutdown, watchdog, MCP); `actinglab` for recording `linear_steps` task packs from a live screen and checking packs and catalogs; `actingledger` for reading the ledger offline; `actingwatch.exe`, the watchdog launcher. |
| MCP server | `actingctl mcp-serve` (from v0.11.0): 22 tools for AI agents, see [Use with an AI agent](#use-with-an-ai-agent-mcp). |
| Console (`acui.exe`) | Native console that only reads the ledger: instance cards, timeline and details; six tabs that match the ledger's six views; reads offline from the state root or live through the Runtime. Its launcher starts the Runtime or asks it to close, and its Instance Configuration window edits the instances. |
| Setup wizard (`acsetup`) | A/B installs and upgrades, verifying every file before anything changes; also a command line (see [Install](#install)). It is the only program in the family with network code. |
| Standard packs | Arknights, Azur Lane and Blue Archive: each game's task packs plus the data the program and agents need. The packs of today are made for 1280×720 frames; a task fails explicitly on a frame of another size. |

## Install

Requirements: Windows, and the MuMu Android emulator for the game instances, at 1280×720. The installation is per user; no administrator rights are needed.

1. From the [Releases](https://github.com/HS7097/ActingCommand/releases) page, take the newest release `vX.Y.Z` and download **one** setup wizard:
   - `acsetup.exe` (online): downloads the rest of the newest release itself;
   - `acsetup-full-<tag>.exe` (offline): carries the whole release.

   Each has a `.sha256` file next to it if you want to check the download.
2. Run it. The wizard has five steps:
   1. **Location**: the install root, by default `%LOCALAPPDATA%\Programs\ActingCommand`. If an installation is already there, this run is an upgrade.
   2. **Install**: fetches (or extracts) the release, checks every file against `SHA256SUMS` and the `BUILD-MANIFEST.json` inside each zip, and stops if anything differs. Only then does it lay out the program core (`runtime\` and `ui\`) in the program slot `A\`, and at the install root the fixed entries in `runtime\` and `ui\`, which always start the slot that `install\active.json` selects. A slot holds only the program core: the tools are in `tools\` at the install root, the vision models and ONNX Runtime in `vision\` (models in `vision\models\<model_ref>\`, ONNX Runtime in `vision\ort\`), and the resource packages (`packages\`) and the state (`state\`) stay at the install root as well; switching the slot leaves all of them as they are. The release does not carry vision models or ONNX Runtime: until they are in `vision\`, a target that needs OCR or a neural network fails explicitly.
   3. **Options**: writes the Runtime configuration `actingd.config.json` and the console settings. Start at boot, a Start menu shortcut and a desktop shortcut are optional.
   4. **Instances** (optional): finds MuMu and its instances, and offers the standard packs carried by the release. You can skip it and do it later with the console's Instance Configuration button.
   5. **Finish**: a summary of what was installed and where the install log is.
3. To upgrade, run the wizard of a newer release on the same install root. It prepares the new version in the other slot (`B\` or `A\`) before it switches. Your configuration and state are kept, and the version it replaces stays in its slot, where `ui\acsetup.exe --rollback` selects it again. When the installed Runtime is v0.11.1, v0.11.2 or v0.11.3, close it formally before the upgrade (`<root>\runtime\actingctl.exe request-shutdown --state-root <state root> --wait 60`; if it is refused as busy, retry once it is idle), then run the wizard, then start the Runtime if it is not running. An install from before v0.11.1 has its programs and configuration moved to `install\initial-backup-<generation>\` on its first upgrade.

**Rollback limits.** Do not switch `install\active.json` back by hand; use `--rollback`. These rollbacks are not possible:
- From v0.11.3 the Runtime writes the ledger at interface revision 2, which v0.11.2 and earlier cannot read; `--rollback` to such a slot stops with exit code 1 and changes nothing.
- On a large state root the upgrade from v0.11.1–v0.11.3 to v0.11.4 or later is one-way in practice: the rollback runs the old version's ledger check, which fails on such roots and leaves the installation unchanged.
- Once the frame retention cleaner of Runtime v0.11.6 has run, v0.11.5 and earlier refuse that state root. The cleaner is on when the configuration does not mention it; for the first start after the upgrade to v0.11.6, consider turning it off (`"frame_retention_enabled": false`) and on again once everything looks right. The Runtime's RELEASE-NOTES.md describes the setting and how a configuration change reaches an A/B install.

**Command line:** acsetup also runs without its window:

```
acsetup --root <abs path> (--plan|--yes) [--conflicts new|old] [--associate <alias>=<bundle>/<server>] [--allow-downgrade] [--online|--from <folder>]
```

`--plan` lists every change and difference and changes nothing under the install root; `--yes` installs or upgrades. `--associate` can be repeated, once per instance alias. Exit codes: 0 done, 1 failed, 2 usage, 3 maintenance-binding differences without `--conflicts`, 4 a downgrade without `--allow-downgrade`, 5 a resource association to choose without `--associate`, 6 (from v0.11.3) incompatible interfaces, that is, the interface revisions that the Runtime, the UI, acsetup and the standard packs declare do not fit; 2 to 6 stop before the installation changes. From v0.11.3, `acsetup --root <abs path> --resources <bundle zip> (--plan|--yes)` puts one standard pack into an existing A/B install and changes neither the programs nor the slot. The offline `acsetup-full-<tag>.exe` uses only the release it carries; the online `acsetup.exe` needs `--online` or `--from <folder>`. `acsetup --help` prints the whole usage. acsetup is a windowed program, so an interactive console prompt does not wait for it; the one-step scripts below wait for it and pass its exit code through.

**One-step scripts:** from v0.11.2 each release also carries two scripts, as release assets only, not as files of this repository:
- `install.ps1` downloads `acsetup-full-<tag>.exe` and its `.sha256` from the same release, checks the SHA-256, runs acsetup in command-line mode, prints its exit code and passes it through. The script's own exit codes are 10 (a download failed) and 11 (SHA-256 mismatch). Run `powershell -NoProfile -ExecutionPolicy Bypass -File install.ps1 -Root C:\AC --plan` first, then the same with `--yes`.
- `install.sh` (Git Bash) fetches that release's `install.ps1`, checks it against the SHA-256 recorded inside install.sh and runs it with the same arguments, for example `bash install.sh -Root C:/AC --plan`.

**Runtime watchdog:** from v0.11.3 the Runtime can be started again on its own when it ended without a formal close (a crash, a closed window, a reboot). The installer does not enable it: run `<root>\runtime\actingctl.exe watchdog install --root <root>` once, from a normal (not elevated) PowerShell of the user the Runtime runs for. It registers a per-user task that runs `<root>\tools\actingwatch.exe` every minute. The watchdog never starts a Runtime that was closed formally, stays down when the last Runtime log ends in a FATAL line, and starts at most 3 times in 30 minutes. From Runtime v0.11.6 it also starts the Runtime again when the held start of an in-place upgrade failed or timed out; a close accepted during that held start stays down. `watchdog status --root <root>` reports without changing anything, `watchdog uninstall --root <root>` removes the task (run it before you remove an install), and the record is `<root>\watchdog\watchdog.log`. The Runtime's INSTALL.md, section "Runtime watchdog", has the details.

An AI agent can do all of this for you; the program skill tells it how. The complete description of the wizard, including upgrades, the offline edition and the install log, is in the UI repository's README, section [Setup wizard acsetup](https://github.com/HS7097/ActingCommand-UI#setup-wizard-acsetup).

## Use with an AI agent (MCP)

From Runtime v0.11.0, `actingctl mcp-serve` is a local MCP server on stdio. It offers tools only (no resources or prompts), talks only to the Runtime on the same machine, and passes the Runtime's answers and refusals through unchanged.

**Register it.** `mcp-config` prints the registration for your client and writes nothing:

```
<root>\runtime\actingctl.exe mcp-config --client claude [--tier observer,operator,author]
<root>\runtime\actingctl.exe mcp-config --client codex  [--tier observer,operator,author]
```

- Claude Code: run the printed command, `claude mcp add --scope user actingcommand -- "<root>\runtime\actingctl.exe" mcp-serve --tier …`.
- Codex: add the printed `[mcp_servers.actingcommand]` table to Codex's `config.toml`.

From v0.11.2, on an A/B install the printed program is the fixed entry `<root>\runtime\actingctl.exe`, so the registration keeps working after a slot switch. Start a new client session to load the server.

**Tiers (0.11 series).** `observer` reads and is always on; it is the default. `operator` adds devices and scheduling, `author` adds Lab recording. A tool outside the enabled tiers answers `tier_not_enabled`; to change the tiers, register again with another `--tier`. The 0.12 series is planned to remove the tiers.

| Tier | Tools |
|---|---|
| observer | `ac_overview`, `ac_events`, `ac_material`, `ac_get_run`, `ac_diagnose`, `ac_resources_list`, `ac_targets_get`, `ac_pack_check`, `ac_catalog_check` |
| operator | `ac_run_pack`, `ac_stop_run`, `ac_pause`, `ac_resume`, `ac_emulator`, `ac_targets_set` |
| author | `ac_lab_observe`, `ac_lab_do`, `ac_record_start`, `ac_record_mark`, `ac_record_stop`, `ac_record_status`, `ac_binding_draft` |

`actingctl mcp-serve --list-tools --format markdown` prints the whole table with every tool's arguments.

**How it behaves.** Long operations return a handle at once; poll it with `ac_get_run` and its `wait_s`. A run's handle is the Runtime's request id and survives restarts. After an uncertain result, ask `ac_get_run` first instead of sending the write again. Approvals, edits of the Runtime configuration and restarts of the Runtime stay with the person.

**Program skill.** [`skills/actingcommand/SKILL.md`](skills/actingcommand/SKILL.md) is the agent's manual for these tools, with recipes for health checks, single runs, pack checks, diagnosis, resource targets and Lab recording; without the MCP server it falls back to the command-line manual `skills/actingcommand/references/cli-workshop.md`. To install it, copy or link the whole `skills/actingcommand` directory to `~/.claude/skills/actingcommand` (Claude Code) and `~/.agents/skills/actingcommand` (Codex). The skill describes Runtime v0.11.3 and does not yet cover the changes of v0.11.5 and v0.11.6; a rewrite for the 0.12 series is planned.

**Known issues** (confirmed on Runtime v0.11.6, fixes planned for the 0.12 series):
1. With an instance alias that contains capital letters, `ac_pause` and `ac_resume` answer `client_action_invalid`, and the Lab tools' actions leave no trail. Use `actingctl pause` and `actingctl resume` instead.
2. `ac_lab_observe` (and `actinglab observe`) fails instead of truncating when its minimal output exceeds 2048 bytes.
3. After many Lab actions, `ac_overview` reports `incomplete` (`run_status_event_limit_exceeded`); the runs themselves are not affected.

## Releases

Each version is a release `vX.Y.Z` on this repository, marked as a pre-release. The members are decoupled: each member (Runtime, UI, each standard pack) releases on its own, from an exact commit, and only when it changes; the members keep working together as long as their interfaces stay compatible. From v0.11.3 each Runtime and UI build declares, in its `BUILD-MANIFEST.json`, the interface revisions it reads and writes, and acsetup checks every combination it installs against these declarations before it changes anything. A release here carries each member's newest release with its assets unchanged, so the members' versions can differ from one another and from this release's; `MEMBERS.json` records each member's own tag.

The newest release here, **v0.11.4**, carries Runtime v0.11.4, UI v0.11.3, the Arknights standard pack v0.11.4, the Azur Lane standard pack v0.11.3 and the Blue Archive standard pack v0.11.4. Runtime v0.11.5 and v0.11.6 have been released since, on the [Runtime repository's Releases page](https://github.com/HS7097/ActingCommand-Runtime/releases), and are not yet in a release here, so the setup wizard installs Runtime v0.11.4 for now.

| Asset | Content |
|---|---|
| `actingcommand-runtime-<sha>.zip` | The Runtime: `actingcommand-actingd.exe`, `actingctl.exe`, the configuration template, INSTALL.md, RELEASE-NOTES.md |
| `actingcommand-tools-<sha>.zip` | The tools of the same build: `actinglab.exe`, `actingledger.exe`, `actingcommand-vision-provider-check.exe`, from v0.11.3 the watchdog launcher `actingwatch.exe`, and Google's Android platform-tools 37.0.1 (adb) under `platform-tools\`. Up to Runtime v0.11.5 it also carries the device-test probe `actingcommand-device-test.exe`, which v0.11.6 retires. From v0.11.2 it no longer carries the OCR provider: OCR runs inside the Runtime |
| `acui-windows-<sha>.zip` | The console `acui.exe`, the setup wizard `acsetup.exe` and the fixed entry `acforward.exe` |
| `<game>-resources-<sha7>.zip` | A game's standard pack: all of its task packs, each identified by the digest of its content and checked before it loads, plus the data the program and agents need |
| `MEMBERS.json`, `SHA256SUMS` | The member commits and releases; the SHA-256 of every zip and of `MEMBERS.json` |
| `acsetup.exe`, `acsetup-full-<tag>.exe` | The online and offline setup wizards, each with its own `.sha256` |
| `install.ps1`, `install.sh` | From v0.11.2: the one-step install scripts (see Install) |

Before a release is published, every member zip is checked against its `BUILD-MANIFEST.json` (repository, commit, size and SHA-256 of every file), and the offline wizard is read back after it is assembled. This repository has no CI: releases are assembled and checked by hand.

The `build-r<Runtime>-u<UI>` pre-releases from the earlier daily publishing stay on the Releases page for reference; that publishing has been retired.

## Principles

- **The program core stays neutral.** The Runtime, its tools and the MCP server contain no game logic and no game information (no game names, pages, characters, stages, resource names, values or rules) in their code, contracts, default configuration, code tables or tests. Guard tests in the Runtime's CI scan for it.
- **Games come in through resource packs.** All game content, the images, recognition regions, click boxes, task order and data tables, lives only in the packs: task packs, gathered into one standard pack per game, sealed by digest and checked before they load.
- **Fail loud.** Every error is visible: a clear message, a non-zero exit code, an Error or Warning in the ledger. A refusal is also a receipt, never silence; invalid configuration, a non-loopback address or missing evidence fail explicitly instead of degrading quietly, and "unknown" is never taken as "no".
- **The ledger is the authoritative record.** What the ledger says is what happened in the Runtime; a problem can be traced from the ledger alone.
- **Result codes.** Every outcome has a code registered in the outcome catalog (catalog and CI guard from v0.11.5); the codes are being unified in the 0.12 series.
- **Local only.** The Runtime listens only on loopback and has no network code.
- **Decoupled members.** The Runtime, the UI and each standard pack release on their own; whether they fit is decided by their declared interfaces.

## Roadmap (planned)

Nothing in this section is released yet. The order inside a series may still change.

**0.12 series** (mainly the Runtime, partly with the next UI release):

| Topic | Planned |
|---|---|
| Unified result codes | One registered outcome catalog shared by every program, each code with a class; one result line per command, exit codes reduced to 0/1/2; errors explained by code, in Chinese and English. |
| Ledger as observer | Live state moves into two in-memory stores (the scheduler workbench and the instance base information store); the ledger records everything through probes, silent gates on the paths the data must take, and stays the only authoritative record. A failed write is retried, then dumped with an explicit stop and imported at the next start; status queries no longer write to the ledger, and the start no longer replays the whole ledger. |
| New ledger | From 0.12.0 a new ledger format (revision 3) that starts empty; a 0.11 state root is not carried over (the old installation is kept as an archive, not deleted). |
| Unified interface | One gate with three front ends, the command line, MCP and the UI, matching one to one. The general interface is always there (queries, pause and resume, monitoring, priority offsets, schedulable-pack state, request shutdown, an upgrade mode for installers, instance and resource targets). Each request records only which front end it came from; there are no per-caller permissions, and the MCP tiers are removed. |
| Lab as a detachable debug module | Not installed by default. A new installer page has two independent options, and either one installs Lab: **authoring** (recording and pack-making tools for people who make resources with an agent) and **debugging** (direct control of the Runtime's internal actions for advanced users). Without Lab its channel stays but cannot be called. |
| Configurable recovery ladder | Triggers (by result-code class), counts and time windows, rung order, per-rung time limits, cool-downs and suppression all move into configuration, with defaults in the release; changed in the file for the next start, or at once with one command from the command line, MCP or UI, recorded in the ledger. New triggers: repeated stability warnings within a short time, and behavior that does not fit the pack's logic. |
| Performance pacing | When the host stalls, the Runtime keeps dispatching but stretches the intervals between and within steps; it measures CPU, GPU and responsiveness continuously and pauses scheduling only while responsiveness keeps falling, resuming on its own. Measurement only at first; the control follows once real data has been looked at. |
| Scheduling rules | An instance runs every task of the standard pack it has, by rule, and **every task that did not run says why**, in status and MCP. Time blocks by blacklist with a gradual quiet-down before one; task packs can be suspended; locked features are declared as data. |
| Data refresh | Unknown or stale instance data no longer fails a whole round; an ordinary reading pack opens the right page and refreshes it. |
| Selection and shared data tables | Shared data tables in standard packs (referenced by digest, read by key), several choices in one step, candidate layouts and tracking across frames; the Runtime provides only the generic mechanism, the tables' content stays in the packs. |
| Per-instance goals | One goal per instance with one entry and two ways to write it, together or alone: **task weights** (run a task more for a particular yield) and **special goals** (for example "save up to X" of a resource); written through the general interface and recorded in the ledger; goals take precedence over the standard pack's defaults. |
| MCP additions | Everyday reads (filtered and long-polled events, run lists, instance data, frame export, one run's evidence, diagnosis of all instances, a sectioned overview), code lookup, pack lists, "why did it not run / when is it due next", asking the scheduler for one run, reading and writing instance goals; an agent inbox and briefings; the known issues above fixed. The command line is the reference and MCP maps to it one to one. |
| Automatic emulator start | One configuration key, off by default. |
| Resolution support (end of the 0.12 series) | Any 16:9 landscape or 9:16 portrait frame converted to the pack's declared base coordinate system (1280×720 today), with the short side at least 720; other ratios refused with a reason. |
| Next UI release | Installer page for the two Lab options, errors explained by code (Chinese and English), reading the new ledger, machine-readable installer output (`--json`); planned as a major version step of the UI. |

**0.13 series:**

| Topic | Planned |
|---|---|
| Battle layer | Generic building blocks for battles: reading results, executing grid moves, and a public data format; every game's battle content stays in its pack. 0.13.0 lays the shared base; its first user is automatic battle in the Arknights standard pack. |
| Generic spatial components (clean-room) | Multi-finger hold gestures with sensitivity and latency calibration, minimap localization, path finding on route maps, closed-loop walking and camera turning. Written clean-room from public principles, with no third-party code, maps or models and no unpacking; in-process backends alongside color matching, color digests and OCR. Maps, route maps and control layouts are pack data; localization and walking come during 0.13.x. |
| MaaFramework pipeline import/export | A package view that converts our task packs to and from MaaFramework-style pipeline JSON; a converted pack is saved back only when it passes the pack check, otherwise it reports the error and writes nothing. It follows the newest format and reports format changes. Editing happens in our own UI (a node-graph editor in the next authoring workbench); no third-party editor ships with the release. |

**Breaking changes during the 0.12 series:** the new ledger, with no 0.11 state root carried over; changed command-line output and exit codes; no more MCP tiers; Lab not installed by default. The current setup wizard (UI v0.11.3) is expected not to install a 0.12.0 Runtime; that comes with the next UI release.
