<p align="right">🌐 <b>English</b> · <a href="./README.zh-CN.md">简体中文</a></p>

<div align="center">

<img src="docs/assets/readme/actingcommand-icon.png" width="112" alt="ActingCommand icon">

**Chief Executive Officer & Chairman** — HS7097<br/>
**Chief Technology Officer & Chief Architect** — Claude Opus 5.5 · GPT‑6 Astra · Claude Fable 5.1 · Claude Fable 5 · GPT‑5.5<br/>
**Board Secretary & Chief Audit Officer** — Claude Opus 5.5 · Claude Fable 5.1 · Claude Fable 5 · Claude Opus 4.8<br/>
**Principal Engineer** — Claude Opus 5.5 · GPT‑6 Astra · GPT‑5.6 Sol · GPT‑5.5<br/>
**Director of Fleet Operations** — GPT‑6 Astra<br/>
**Fleet Supervisor** — GPT‑5.6 Sol Daybreak · DeepSeek V4.1 Flash 1017

</div>

**⚠️ Pre-release, debug phase: every release is a pre-release, and interfaces, configuration and file formats still change. Only the newest version is supported.**

# ActingCommand

- [ActingCommand-Runtime](https://github.com/HS7097/ActingCommand-Runtime) — resident Rust runtime, the core program, with its command line, tools and local MCP server
- [ActingCommand-UI](https://github.com/HS7097/ActingCommand-UI) — setup wizard and read-only console
- Standard packs for **Arknights**, **Azur Lane** and **Blue Archive** — made in separate repositories that are not public; the packs published so far are assets of this repository's [Releases](https://github.com/HS7097/ActingCommand/releases)

## This repository

This is the umbrella (portal) repository of the ActingCommand family. It holds no code: the code lives in the member repositories linked above. This repository carries the README, the program skill under `skills/`, and the versioned releases on its Releases page: the setup wizards and, unchanged, the release assets of the members for each version. `bundles/` keeps the Blue Archive bundle of the retired daily publishing for reference (no longer updated), and `docs/` holds the README images and older notes.

**What we can do:** Let an AI agent deploy our program, or install it yourself. Once the program skill is loaded in your harness and the MCP server is registered (see [Use with an AI agent](#use-with-an-ai-agent-mcp)), you can have the agent produce the materials that a program running in an Android emulator needs, and then run it again and again on a schedule.

**What the agent has to do:** Make some images and click regions.

**How we do it:** The Runtime is a resident Rust program that contains no game logic of its own. It connects to Android emulators over ADB (and the emulator vendor's interfaces), captures frames on a cadence, recognizes materials in each frame (template, color, OCR, neural network), clicks inside regions declared in advance, and writes every step, what it saw, what it did and how it went, as typed events into an append-only ledger. The ledger is the authoritative record: the scheduler decides the next run from it, the console only reads it, and when something goes wrong it is traced from the ledger alone. Everything game-specific, the images, the click regions, the task order and the data tables, lives in a per-game standard pack whose task packs the Runtime admits by their declared structure before it loads them; the Runtime only loads, checks and executes them. Switch the game by switching the pack; the program does not change.

**Join in:** If you have a better idea, or run into a problem while using it, open an issue in this repository or in the repository concerned.

## Status

- **Debug phase.** The main loop is: find a problem, fix it, check the result against what is expected, change again. Deployment and convenience come second.
- **Pre-release only.** Every release of every member is a pre-release. Only the newest version is supported; older series get no fixes.
- **Where the members stand.** The Runtime is in its 0.12 series; its newest release is v0.12.10. The UI's newest release is v0.11.4; there is no UI release for the 0.12 series, and the setup wizard and console of v0.11.4 do not work with a 0.12 Runtime (see [Install](#install)). The newest release of this repository is still v0.11.4, with Runtime v0.11.4 (see [Releases](#releases)).
- **What changed with Runtime 0.12.** A new ledger: a 0.12 Runtime starts on an empty state root and does not open a 0.11 ledger, and 0.11 programs cannot read a 0.12 one. A new pack format: the 0.11 standard packs do not load in 0.12. The command line and the watchdog answer with registered outcome codes and exit 0, 1 or 2 by the outcome's category, and MCP tool errors carry the same codes. Runtime v0.12.10 is the first release with ledger revision 4; going back from it to an earlier Runtime is not supported.
- **On real devices.** The project's own machine runs the Runtime of the 0.12 series around the clock on five MuMu instances. Routine batches run every day with the three standard packs; two more instances are connected for Girls' Frontline 2, whose standard pack is being made and is not published. The packs do not cover every task yet.
- **Version numbers.** X for major or incompatible changes, Y for new features or new coverage, Z for fixes (including features that serve a fix). Each member releases on its own; whether members fit together is decided by their declared interfaces, not by equal version numbers.

## What works today

The Runtime rows describe Runtime v0.12.10; the console and setup wizard rows describe UI v0.11.4.

| Part | What it does |
|---|---|
| Runtime (`actingcommand-actingd.exe`) | Resident process that accepts requests only on the loopback address (127.0.0.1 / [::1]) and has no network code; clients can close without affecting it. Scheduler: runs task packs by the approved catalog, on clock slots and on conditions such as a time read from the instance's facts, with one request queue and one worker per instance and leases and fences re-checked on every device touch; pause and resume (global or per instance, kept across restarts), priority offsets, resource targets and a disk-capacity gate. Every task that did not run says why. A recovery ladder whose triggers and rungs are configuration (return home, restart the app, restart the emulator) takes over an instance after a failed routine run; runs started by hand do not start it. A failure of one instance (its capture, its application control, an observation) affects only that instance. |
| Recognition and devices | Eight target kinds: template, color, click-only, OCR, neural network, color digest, composite (a combination of 2 to 8 other targets) and value test (a comparison of read values); OCR and NN run in process on ONNX Runtime, with models loaded on first use. MuMu: Nemu IPC capture and input, instance discovery and start/stop through MuMuManager, and an optional automatic start of an instance's emulator. A real Android phone on USB can be bound as an instance. The bundled adb 37.0.1. A press inside a box lands around the box's center and moves among a few candidate points. |
| Ledger | A single-writer SQLite ledger of sanitized typed events, each result with its registered outcome code; frames are stored by content digest. A frame retention cleaner removes frames that are no longer needed and moves the frames kept for errors and Lab into `<state root>\kept\<date>\`, where a person deletes them. |
| Command line and tools | `actingctl` (status and overview, events, runs and run evidence, "why did it not run", pause and resume, emulator control, settings, devices, request shutdown, watchdog, MCP); `actinglab` for recording `linear_steps` task packs from a live screen and checking packs, standard packs and resource repositories; `actingledger` for reading the ledger offline; `actingwatch.exe`, the watchdog launcher. |
| MCP server | `actingctl mcp-serve`: 38 tools for AI agents, see [Use with an AI agent](#use-with-an-ai-agent-mcp). |
| Console (`acui.exe`) | Native console that only reads the ledger: instance cards, timeline and details; six tabs that match the ledger's six views; reads offline from the state root or live through the Runtime. Its launcher starts the Runtime or asks it to close, and its Instance Configuration window edits the instances. Works with Runtime 0.11 only. |
| Setup wizard (`acsetup`) | Installs the releases of this repository: A/B installs and upgrades, verifying every file before anything changes; also a command line (see [Install](#install)). It is the only program in the family with network code. It refuses a 0.12 Runtime. |
| Standard packs | Arknights, Azur Lane and Blue Archive: each game's task packs plus the data the program and agents need. The packs published here (in v0.11.4) are 0.11 packs; packs for the 0.12 series are not published. The packs are made for 1280×720 frames; a task fails explicitly on a frame of another size. |

## Install

**Runtime 0.12 series.** No released setup wizard installs it: the setup wizard of UI v0.11.4 refuses a 0.12 Runtime with exit code 6 before it changes anything, because their declared interfaces do not fit. A 0.12 Runtime is laid out by hand on an empty state root: take the Runtime zip, the Tools zip and `SHA256SUMS` from a [Runtime release](https://github.com/HS7097/ActingCommand-Runtime/releases), check them, extract the Runtime zip into `<root>\runtime\` and the Tools zip into `<root>\tools\`, and write the private configuration `<root>\actingd.config.json` from the template. The Runtime's [INSTALL.md](https://github.com/HS7097/ActingCommand-Runtime/blob/main/distribution/windows/INSTALL.md) describes the configuration, the bundled adb, starting, inspecting and closing, and the watchdog. Within the 0.12 series an upgrade replaces `runtime\` and `tools\` and keeps the configuration and the ledger. No standard pack for the 0.12 series is published.

**Release v0.11.4 of this repository (0.11 series), with the setup wizard.** Requirements: Windows, and the MuMu Android emulator for the game instances, at 1280×720. The installation is per user; no administrator rights are needed.

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

**Command line:** acsetup also runs without its window:

```
acsetup --root <abs path> (--plan|--yes) [--conflicts new|old] [--associate <alias>=<bundle>/<server>] [--allow-downgrade] [--online|--from <folder>]
```

`--plan` lists every change and difference and changes nothing under the install root; `--yes` installs or upgrades. `--associate` can be repeated, once per instance alias. Exit codes: 0 done, 1 failed, 2 usage, 3 maintenance-binding differences without `--conflicts`, 4 a downgrade without `--allow-downgrade`, 5 a resource association to choose without `--associate`, 6 incompatible interfaces, that is, the interface revisions that the Runtime, the UI, acsetup and the standard packs declare do not fit; 2 to 6 stop before the installation changes. `acsetup --root <abs path> --resources <bundle zip> (--plan|--yes)` puts one standard pack into an existing A/B install and changes neither the programs nor the slot. The offline `acsetup-full-<tag>.exe` uses only the release it carries; the online `acsetup.exe` needs `--online` or `--from <folder>`. `acsetup --help` prints the whole usage. acsetup is a windowed program, so an interactive console prompt does not wait for it; the one-step scripts below wait for it and pass its exit code through.

**One-step scripts:** each release also carries two scripts, as release assets only, not as files of this repository:
- `install.ps1` downloads `acsetup-full-<tag>.exe` and its `.sha256` from the same release, checks the SHA-256, runs acsetup in command-line mode, prints its exit code and passes it through. The script's own exit codes are 10 (a download failed) and 11 (SHA-256 mismatch). Run `powershell -NoProfile -ExecutionPolicy Bypass -File install.ps1 -Root <install root> --plan` first, then the same with `--yes`.
- `install.sh` (Git Bash) fetches that release's `install.ps1`, checks it against the SHA-256 recorded inside install.sh and runs it with the same arguments, for example `bash install.sh -Root <install root> --plan`.

**Runtime watchdog:** from Runtime v0.11.3 the Runtime can be started again on its own when it ended without a formal close (a crash, a closed window, a reboot), and in the 0.12 series also when its last start ended in an error. Nothing enables it for you: run `<root>\runtime\actingctl.exe watchdog install --root <root>` once, from a normal (not elevated) PowerShell of the user the Runtime runs for. It registers a per-user task that runs `<root>\tools\actingwatch.exe` every minute. The watchdog never starts a Runtime that was closed formally, stays down when the last start ended fatally, and starts at most 3 times in 30 minutes. It watches an A/B installation, and from Runtime v0.12.1 also a root laid out by hand as above. `watchdog status --root <root>` reports without changing anything, `watchdog uninstall --root <root>` removes the task (run it before you remove an install), and the record is `<root>\watchdog\watchdog.log`. The Runtime's INSTALL.md, section "Runtime watchdog", has the details.

An AI agent can do all of this for you; the program skill tells it how. The complete description of the wizard, including upgrades, the offline edition and the install log, is in the UI repository's README, section [Setup wizard acsetup](https://github.com/HS7097/ActingCommand-UI#setup-wizard-acsetup).

## Use with an AI agent (MCP)

`actingctl mcp-serve` is a local MCP server on stdio (from Runtime v0.11.0). It offers tools only (no resources or prompts), talks only to the Runtime on the same machine, and passes the Runtime's answers and refusals through unchanged.

**Register it.** `mcp-config` prints the registration for your client and writes nothing:

```
<root>\runtime\actingctl.exe mcp-config --client claude [--tier observer,operator,author]
<root>\runtime\actingctl.exe mcp-config --client codex  [--tier observer,operator,author]
```

- Claude Code: run the printed command, `claude mcp add --scope user actingcommand -- "<root>\runtime\actingctl.exe" mcp-serve --tier …`.
- Codex: add the printed `[mcp_servers.actingcommand]` table to Codex's `config.toml`.

On an A/B install the printed program is the fixed entry `<root>\runtime\actingctl.exe`, so the registration keeps working after a slot switch. Start a new client session to load the server.

**Tiers.** `observer` is always on and is the default; besides reads it holds the settings the Runtime may change while running, handset device binding, the local capability-gap log and an instance's remedy choices. `operator` adds devices, applications and scheduling, `author` adds Lab recording. A tool outside the enabled tiers answers `tier_not_enabled`; to change the tiers, register again with another `--tier`. Runtime v0.12.10 has these 38 tools:

| Tier | Tools |
|---|---|
| observer (23) | `ac_overview`, `ac_events`, `ac_material`, `ac_observe`, `ac_get_run`, `ac_diagnose`, `ac_runs`, `ac_run_evidence`, `ac_resources_list`, `ac_targets_get`, `ac_config_set`, `ac_packs`, `ac_facts`, `ac_outcome_code`, `ac_devices`, `ac_device_bind`, `ac_device_unbind`, `ac_capability_gap`, `ac_remedy_set`, `ac_remedy_get`, `ac_schedule`, `ac_pack_check`, `ac_catalog_check` |
| operator (8) | `ac_run_pack`, `ac_stop_run`, `ac_pause`, `ac_resume`, `ac_emulator`, `ac_app`, `ac_ladder_drill`, `ac_targets_set` |
| author (7) | `ac_lab_observe`, `ac_lab_do`, `ac_record_start`, `ac_record_mark`, `ac_record_stop`, `ac_record_status`, `ac_binding_draft` |

`actingctl mcp-serve --list-tools --format markdown` prints the whole table with every tool's arguments. A tool error is `{class, outcome}`, and `mcp-serve --detail codes|full` sets how much of each outcome it carries.

**How it behaves.** Start with `ac_overview`. Long operations return a handle at once; poll it with `ac_get_run` and its `wait_s`. A run's handle is the Runtime's request id and survives restarts. After an uncertain result, ask `ac_get_run` first instead of sending the write again. Approvals, edits of the Runtime configuration file and restarts of the Runtime stay with the person; a setting the Runtime may change while running is set with `ac_config_set`, and every change is recorded.

**Program skill.** [`skills/actingcommand/SKILL.md`](skills/actingcommand/SKILL.md) is the agent's manual for these tools, with recipes for health checks, single runs, pack checks, diagnosis, resource targets and Lab recording; without the MCP server it falls back to the command-line manual `skills/actingcommand/references/cli-workshop.md`. To install it, copy or link the whole `skills/actingcommand` directory to `~/.claude/skills/actingcommand` (Claude Code) and `~/.agents/skills/actingcommand` (Codex). The skill describes Runtime v0.11.3 and its 22 tools; it does not cover Runtime v0.11.5 and later, including the 0.12 series. For a 0.12 Runtime, the tool descriptions that `mcp-serve --list-tools` prints are the reference.

**Known issues of the 0.11 series**, fixed in the 0.12 series:
1. With an instance alias that contains capital letters, `ac_pause` and `ac_resume` answer `client_action_invalid`, and the Lab tools' actions leave no trail. Use `actingctl pause` and `actingctl resume` instead. Fixed in v0.12.0.
2. `ac_lab_observe` (and `actinglab observe`) fails instead of truncating when its minimal output exceeds 2048 bytes. Fixed in v0.12.0.
3. After many Lab actions, `ac_overview` reports `incomplete` (`run_status_event_limit_exceeded`); the runs themselves are not affected. Fixed in v0.12.1.

## Releases

Each version is a release `vX.Y.Z` on this repository, marked as a pre-release. The members are decoupled: each member (Runtime, UI, each standard pack) releases on its own, from an exact commit, and only when it changes; the members keep working together as long as their interfaces stay compatible. Each Runtime and UI build declares, in its `BUILD-MANIFEST.json`, the interface revisions it reads and writes, and acsetup checks every combination it installs against these declarations before it changes anything. A release here carries each member's newest release with its assets unchanged, so the members' versions can differ from one another and from this release's; `MEMBERS.json` records each member's own tag.

The newest release here, **v0.11.4**, carries Runtime v0.11.4, UI v0.11.3, the Arknights standard pack v0.11.4, the Azur Lane standard pack v0.11.3 and the Blue Archive standard pack v0.11.4. Released since, and not in a release here: Runtime v0.11.5, v0.11.6 and the 0.12 series up to v0.12.10 on the [Runtime repository's Releases page](https://github.com/HS7097/ActingCommand-Runtime/releases), and UI v0.11.4 on the [UI repository's Releases page](https://github.com/HS7097/ActingCommand-UI/releases). The setup wizard therefore installs Runtime v0.11.4 and UI v0.11.3. A 0.12 Runtime does not fit the 0.11 UI and the 0.11 standard packs.

| Asset | Content |
|---|---|
| `actingcommand-runtime-<sha>.zip` | The Runtime: `actingcommand-actingd.exe`, `actingctl.exe`, the configuration template, INSTALL.md, RELEASE-NOTES.md |
| `actingcommand-tools-<sha>.zip` | The tools of the same build: `actinglab.exe`, `actingledger.exe`, `actingcommand-vision-provider-check.exe`, the watchdog launcher `actingwatch.exe`, and Google's Android platform-tools 37.0.1 (adb) under `platform-tools\`. Up to Runtime v0.11.5 it also carries the device-test probe `actingcommand-device-test.exe`, which v0.11.6 retires. OCR runs inside the Runtime, so no OCR provider ships |
| `acui-windows-<sha>.zip` | The console `acui.exe`, the setup wizard `acsetup.exe` and the fixed entry `acforward.exe` |
| `<game>-resources-<sha7>.zip` | A game's standard pack: all of its task packs, each checked before it loads, plus the data the program and agents need |
| `MEMBERS.json`, `SHA256SUMS` | The member commits and releases; the SHA-256 of every zip and of `MEMBERS.json` |
| `acsetup.exe`, `acsetup-full-<tag>.exe` | The online and offline setup wizards, each with its own `.sha256` |
| `install.ps1`, `install.sh` | The one-step install scripts (see Install) |

Before a release is published, every member zip is checked against its `BUILD-MANIFEST.json` (repository, commit, size and SHA-256 of every file), and the offline wizard is read back after it is assembled. This repository has no CI: releases are assembled and checked by hand.

The `build-r<Runtime>-u<UI>` pre-releases from the earlier daily publishing stay on the Releases page for reference; that publishing has been retired.

## Principles

- **The program core stays neutral.** The Runtime, its tools and the MCP server contain no game logic and no game information (no game names, pages, characters, stages, resource names, values or rules) in their code, contracts, default configuration, code tables or tests. Guard tests in the Runtime's CI scan for it.
- **Games come in through resource packs.** All game content, the images, recognition regions, click boxes, task order and data tables, lives only in the packs: task packs, gathered into one standard pack per game and checked before they load.
- **Fail loud.** Every error is visible: a clear message, a non-zero exit code, an Error or Warning in the ledger. A refusal is also a receipt, never silence; invalid configuration, a non-loopback address or missing evidence fail explicitly instead of degrading quietly, and "unknown" is never taken as "no".
- **The ledger is the authoritative record.** What the ledger says is what happened in the Runtime; a problem can be traced from the ledger alone.
- **Result codes.** Every result of the Runtime, its command line, the watchdog and the MCP server carries a code registered in one outcome catalog, kept by a guard in the Runtime's CI; each Runtime release carries a snapshot of the catalog. The Lab's own tools are still outside that guard.
- **Local only.** The Runtime listens only on loopback and has no network code.
- **Decoupled members.** The Runtime, the UI and each standard pack release on their own; whether they fit is decided by their declared interfaces.
