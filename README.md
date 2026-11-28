<p align="right">🌐 <b>English</b> · <a href="./README.zh-CN.md">简体中文</a></p>

<div align="center">

<img src="docs/assets/readme/actingcommand-icon.png" width="112" alt="ActingCommand icon">

<h1>ActingCommand</h1>

<p><b>Local-First Multi-Game Automation, Deterministic Visual Execution & AI-Assisted Operations System</b></p>

**Chief Executive Officer & Chairman** — HS7097<br/>
**Chief Technology Officer & Chief Architect** — Claude Opus 5.5 · GPT‑6 Astra · Claude Fable 5.2 · Claude Fable 5.1 · Claude Fable 5 · GPT‑5.5<br/>
**Board Secretary & Chief Audit Officer** — Claude Opus 5.5 · Claude Fable 5.2 · Claude Fable 5.1 · Claude Fable 5 · Claude Opus 4.8<br/>
**Principal Engineer** — GPT‑6 Astra · Claude Opus 5.5 · GPT‑5.6 Sol · GPT‑5.5<br/>
**Director of Fleet Operations** — GPT‑6 Astra<br/>
**Fleet Supervisor** — GPT‑6 Sol Daybreak · GPT‑5.6 Sol Daybreak · DeepSeek V4.1 Flash 1017<br/>
**Business Operations Lead** — Gemini‑4 Carbon · Gemini‑4 Argon

</div>

> [!IMPORTANT]
> **Active `0.12` Convergence Preview Series (Road to `v1.00`)**: Interfaces, configuration schemas, and pack formats are actively converging through live-device validation. Only the newest release is supported; older series receive no backported fixes.

---

**ActingCommand** is a local multi-game automation and orchestration system for Windows: everyday routine runs are **executed 100% deterministically by a resident Rust daemon on your machine (zero LLM token cost, zero network code)**, scheduling multiple Android emulators or physical phones by net benefit and resource redlines—while natively providing an authoring workshop and a local MCP server so AI agents can help author task packs, inspect the authoritative ledger, and diagnose anomalies on demand.

---

## 1. What It Can Do Today

All capabilities below are shipped and working in the latest member releases ([`Runtime v0.12.83`](https://github.com/HS7097/ActingCommand-Runtime/releases/tag/v0.12.83) · [`UI v0.12.33`](https://github.com/HS7097/ActingCommand-UI/releases/tag/v0.12.33)):

- **Unattended Local Resident Daemon (`actingd`)**
  - Runs independently in the background; closing consoles or CLI clients never interrupts active schedules, and the per-user `actingwatch` watchdog can automatically restart the daemon after crashes or reboots.
  - Native **MuMu Emulator** integration (high-speed Nemu IPC frame capture and touch input, plus automatic instance discovery and emulator start/stop via MuMuManager) alongside **USB Android handsets** (ships with official `adb 37.0.1`), with live hot-switching of touch and capture backends.
  - Strict per-instance fault isolation: a failure on one device affects only that instance; idle instances automatically have their game closed (or emulator stopped) to save memory and restarted ahead of the next scheduled run.
- **Net-Benefit & Dual-Waterline Smart Scheduling**
  - Dispatches tasks on both clock slots and live instance facts read into the ledger (such as stamina recovery timestamps or facility cooldowns).
  - Ranks business tasks dynamically by **`net benefit ÷ expected run time`**, refusing to dispatch tasks whose benefit is unknown or non-positive; resource pools enforce **stop lines** and **hard redlines** so reserves saved for banners or events are never drained.
  - Supports persistent global or per-instance pause/resume, priority offsets, and disk-capacity gates; an account can set targets on its resource pools, and a target's gap counts toward the benefit of the tasks that fill it—and **every task that did not run states an explicit reason**.
- **Hybrid Deterministic Vision + Graded Recovery Ladder**
  - Eight recognition target kinds: template matching, color, color digest, click-only, OCR, ONNX neural network, composite (2–8 targets), and value comparison, plus multi-match anchor search with masks and several scales; ONNX models load in-process on first use, OCR supports DirectML GPU acceleration, and clicks rotate naturally among central candidate points.
  - Failed scheduled runs automatically hand the instance over to a configurable **recovery ladder** (return home → restart game app → restart emulator); manual debug runs never trigger the ladder accidentally. When the ladder is exhausted, an instance is isolated awaiting a person, or a run fails with a listed outcome code, configured rules can wake a registered agent session (Claude Code / Codex / DSH), and a daily summary can be sent at a set time.
- **Single-Writer Authoritative Ledger & Native Console (`acui`)**
  - Every step—what was seen, where it clicked, and which registered outcome code returned—is sanitized into an append-only SQLite ledger; frames are deduplicated by content digest, expired frames are cleaned automatically, and error/Lab evidence frames are archived under `kept/`.
  - The native desktop console `acui` inspects all six ledger views live (via the Runtime) or offline (directly from the state root), displays account and watchdog status, starts/stops the Runtime, and hot-switches device backends; its home page has tabs for the live view, the daily tasks, and goals, pauses/resumes scheduling for all instances or a single one, and sets an account's goals on the goals tab.
- **Game-Neutral Economy Modeler (`acmodel`)**
  - Standalone offline Python 3.13 modeling and solving tool featuring LP/MILP economy optimization, exact gacha pity chains, Wilson confidence bounds, and sensitivity derivation graphs, exporting prices, redline thresholds, and task scores directly into schedule bindings.
- **Authoring Workshop & AI Agent Integration (`actinglab` + Local MCP Server `actinglab-mcp`)**
  - `actinglab` is the single entry point for developers and agents: one operation table (169 operations today) drives both the command line and MCP. It records `linear_steps` task packs from live screens in stages (including choosing an item by its name and reading scrolling lists), checks and trial-runs standard packs, hot-places standard packs, adds instances, provides everyday short inspection/control commands, and deploys or rolls back in one step (`deploy` / `rollback`).
  - `actinglab-mcp` exposes every operation of that table as one tool over local stdio (**169 tools** on Windows, no tiers) so AI agents can check fleet health, inspect single frames, record new packs, and trace failures safely; the first-generation `actingctl mcp-serve` (46 tier-gated tools) keeps shipping with the Runtime until it is removed.

---

## 2. What Is Actively Being Built

As the `0.12.z` series converges toward **`v1.00`** (ending preview), the team is actively building:

- **Full `0.12` Standard Pack Coverage Across Four Flagship Games**: **Arknights**, **Azur Lane**, **Blue Archive**, and **Girls' Frontline 2: Exilium** run continuously on the project's 6-instance testbed while full daily, progression-farming, recurring-event packs, and scheduler benefit tables are being completed.
- **Farming Driven by Account Goals and Their Gaps**: Connecting per-game character/gear valuation engines to account goals so that what is missing decides what gets farmed, alongside more inventory and progress readings taken from the game screens so the scheduler judges by the account's live state.
- **Plug-and-Play Handsets & an iOS Backend**: Scheduling Android phones at their native resolution (including screens wider than 16:9, with nothing changed on the phone for the program's sake), with an iOS device backend in development.
- **Multi-Game Rotation Within a Single Device / Emulator Instance**: Moving beyond one-game-per-instance so a single phone or emulator instance can host multiple games (including Android multi-user app clones), queuing tasks by cross-game urgency in one foreground queue and cleanly stopping background apps on switch.
- **Player Control Loop in `acui`**: Adding visual instance setup (done through `actinglab instance add` today) and routine toggles to `acui`, and making the goals tab show account-level goals only, with daily tasks listed by their business names; with the command line and MCP on the `actinglab` side already unified on one operation table, this forms two clean entry points (**`acui` for everyday players, `actinglab` for developers and AI agents**).
- **Return of Bundled Releases**: The `acsetup` wizard now ships with the console release (installing offline from an install set); once `0.12` interfaces and ledger schemas lock for `v1.00`, full install-set releases on this umbrella repository and one-click installation/upgrades return.

---

## 3. How It Works

```mermaid
flowchart LR
    Pack["Standard Packs (1280x720 Assets / Click Boxes / Schedules / Values)"] --> RT["Rust Daemon actingd (Zero Game Code / Benefit Scheduler / ONNX Vision)"]
    RT <--> Dev["MuMu Emulators / USB Android Phones"]
    RT --> Ledger[("SQLite Authoritative Ledger (Typed Events / Outcome Codes / Evidence Frames)")]
    Ledger --> UI["Native Console acui"]
    Ledger <--> Lab["actinglab & Local MCP Server actinglab-mcp (One Operation Table / 169 AI Agent Tools)"]
```

1. **Strictly Neutral Core—Switch Games by Switching Packs**: The source code, contracts, configuration templates, outcome catalogs, and tests of the Runtime, UI, CLI, and MCP server contain **zero game logic or game-specific strings** (enforced by CI guard scans). Every image, click region, state transition, and value table lives inside per-game standard packs validated before admission.
2. **The Ledger Is the Single Source of Truth**: The scheduler decides what runs next from ledger facts, the console reads ledger views exclusively, and any incident can be reconstructed from the ledger and its evidence frames alone.
3. **Fail Loud**: Every operation returns a registered outcome code and deterministic exit status; invalid configuration, a frame that cannot be mapped to the pack's base size (all current standard packs target `1280×720`; larger frames of the same aspect ratio are scaled down to it), or missing evidence fails explicitly with preserved diagnostics rather than clicking blindly or degrading silently.
4. **Local Loopback Only**: `actingd` listens strictly on `127.0.0.1` / `[::1]` and contains no networking code: the Runtime itself sends nothing off your machine. What a connected AI agent reads through MCP goes wherever that agent sends it.
5. **Decoupled, Interface-Verified Releases**: Runtime, UI, and each standard pack release independently when changed; compatibility is verified against declared interface revisions in `BUILD-MANIFEST.json`, not by matching version strings.

---

## 4. What You Need to Do

### System Prerequisites
- **OS**: Windows (developed and tested on Windows 11; installs per user; no administrator privileges required).
- **Target Device**: **MuMu Android Emulator** (recommended resolution **`1280×720`**; larger resolutions of the same aspect ratio such as `1920×1080` are scaled down to the base size automatically, while recording task packs requires `1280×720`), or an Android phone connected via USB debugging.
- **Optional Runtimes**: ONNX Runtime + vision models under `vision\` for OCR / neural-network targets; Python 3.13 for the standalone `acmodel` economy solver.

### Choose Your Onboarding Path

- **For Everyday Players (Looking for Turnkey Daily Farming)**
  - The `0.12` series introduced a new ledger and pack format; **`0.12` standard packs are not yet publicly distributed**. The graphical `acsetup` wizard ships with the console release (since UI `v0.12.17`), but it installs offline from an "install set" holding the Runtime, the tools, the console, and each game's standard pack, and no install set is published yet. Watch this repository for the upcoming `v1.00` milestone release when install sets and standard packs reconverge.
  - *(Legacy note: Release [`v0.11.4`](https://github.com/HS7097/ActingCommand/releases) on this repository still carries the older `0.11` setup wizard and `0.11` packs for Arknights, Azur Lane, and Blue Archive as a historical reference; `0.11` is incompatible with `0.12` and is no longer maintained.)*

- **For Developers & Pack Authors (Running the Latest `0.12` Core + AI Tooling Today)**
  1. **Lay Out a `0.12` Root Once**: Download the newest `actingcommand-runtime-<sha>.zip`, `actingcommand-tools-<sha>.zip`, and `SHA256SUMS` from [`ActingCommand-Runtime` Releases](https://github.com/HS7097/ActingCommand-Runtime/releases), verify checksums, extract into `<root>\runtime\` and `<root>\tools\`, and create `<root>\actingd.config.json` from the template (see Runtime's [`INSTALL.md`](https://github.com/HS7097/ActingCommand-Runtime/blob/v0.12.83/distribution/windows/INSTALL.md) for DirectML GPU OCR, bundled `adb`, startup, and shutdown); alternatively, `actinglab.exe new <new root> --build <verified release directory>` from Tools builds a fresh root with its first configuration and an empty state root in one call (it starts nothing). Optionally grab `acui-windows-<sha>.zip` from [`ActingCommand-UI` Releases](https://github.com/HS7097/ActingCommand-UI/releases) for the `acui.exe` console.
  2. **Adopt Managed Updates & Watchdog**: Run `<root>\tools\actinglab.exe adopt` once to take the root under the installer; from then on `actinglab.exe deploy <tag or commit> --dir <downloaded release directory>` stages, plans, stops, applies, starts, and runs the post-start checks in one step, and `actinglab.exe rollback` returns to an earlier kept build (add `--plan` to either to print the plan only), keeping your config and ledger in place; `actinglab.exe up` / `down` start and formally stop the Runtime. Run `<root>\tools\actinglab.exe watchdog install` once from a non-elevated PowerShell to enable automatic crash recovery. (The first-generation `actingctl install …` commands stay available until they are removed.)
  3. **(Optional) Connect an AI Agent via Local MCP**: Print the registration snippet of `actinglab-mcp` (writes no client file) and add it to your agent harness (`claude` / `codex` / `antigravity` / `dsh`), then start a fresh client session:
     ```powershell
     <root>\tools\actinglab.exe mcp config --client claude
     <root>\tools\actinglab.exe mcp config --client codex
     ```

<details>
<summary><b>Expand: The Two MCP Server Generations, Tool Lists & Agent Conventions (Runtime v0.12.83)</b></summary>

<br/>

- **Second-Generation Server `actinglab-mcp` (the main entry today)**: No tiers; every operation of the Lab's operation table is one tool, as a rule named `ac_` plus the command-line words (spaces and `-` become `_`, so `emu start` is `ac_emu_start`), 169 of them on Windows—the server's `tools/list` is authoritative. Call `ac_status` first; a call that takes longer than about 25 seconds returns a handle to follow with `ac_run_show` (`wait_s`); after an uncertain write, read `ac_status` or `ac_run_show` first and never resend it with new arguments; starting, stopping or restarting the Runtime, replacing its configuration, deploying, and rolling back are called only when a person asked for that operation.
- **First-Generation Server `actingctl mcp-serve` (available until it is removed)**: The tiers, conventions, and tool table below describe it; register it with `<root>\runtime\actingctl.exe mcp-config --client <client> --entry --root <root> --tier <tiers>`.
- **Permission Tiers**: `observer` is enabled by default (reads, safe live settings, performance presets, handset binding, capability-gap logs, remedy choices, and routine health); `operator` adds device/app/schedule controls and evidence read confirmations (also required when `ac_config_set` switches touch/capture backends); `author` adds live Lab observation and pack recording.
- **Operating Conventions**: Call `ac_overview` first; long operations immediately return a restart-safe handle polled via `ac_get_run` (`wait_s`)—always query `ac_get_run` before retrying an uncertain write. Configuration file edits, catalog approvals, and Runtime restarts stay with the human operator.
- **Skill Reference**: [`skills/actingcommand/SKILL.md`](skills/actingcommand/SKILL.md) describes Runtime `0.12.70` and the Lab's MCP server `actinglab-mcp` (it differs from the first-generation server this section still describes: no tiers, other tool names); [`references/pack-making.md`](skills/actingcommand/references/pack-making.md) is the way from a recording to a placed package, and [`references/tools.md`](skills/actingcommand/references/tools.md) lists every tool. At run time the server's `tools/list` is authoritative.

| Tier | Tools |
|---|---|
| `observer` (25) | `ac_overview`, `ac_events`, `ac_material`, `ac_observe`, `ac_get_run`, `ac_diagnose`, `ac_runs`, `ac_run_evidence`, `ac_resources_list`, `ac_targets_get`, `ac_config_set`, `ac_perf_preset`, `ac_packs`, `ac_facts`, `ac_outcome_code`, `ac_devices`, `ac_device_bind`, `ac_device_unbind`, `ac_capability_gap`, `ac_remedy_set`, `ac_remedy_get`, `ac_schedule`, `ac_routine_summary`, `ac_pack_check`, `ac_catalog_check` |
| `operator` (9) | `ac_run_pack`, `ac_stop_run`, `ac_pause`, `ac_resume`, `ac_emulator`, `ac_app`, `ac_ladder_drill`, `ac_evidence_confirm_read`, `ac_targets_set` |
| `author` (12) | `ac_lab_observe`, `ac_lab_do`, `ac_record_start`, `ac_record_mark`, `ac_record_stop`, `ac_record_status`, `ac_record_candidates`, `ac_record_amend`, `ac_record_step`, `ac_record_archive`, `ac_record_reopen`, `ac_binding_draft` |

</details>

---

## 5. Repository Map

ActingCommand is organized as a decoupled repository family combining **open-source game-neutral infrastructure** with **isolated private asset and valuation repositories**, keeping proprietary game screenshots, coordinates, and economy tables strictly separated from the neutral engine:

| Repository | Visibility | Role & Scope | Current Status |
|---|---|---|---|
| [**ActingCommand**](https://github.com/HS7097/ActingCommand) *(this repo)* | 🔓 Public | **Umbrella Portal**: Project overview, license policy, `skills/` agent manual, and cross-member milestone Releases | Docs active (bundled Release stays at legacy `v0.11.4` until `v1.00`) |
| [**ActingCommand-Runtime**](https://github.com/HS7097/ActingCommand-Runtime) | 🔓 Public | **Core Runtime**: Resident `actingd` daemon, scheduler, vision engine, SQLite ledger, `actingctl` / `actinglab` / `actingledger` / `actingwatch`, and the local MCP server (`actinglab-mcp`) | Active (`v0.12` series, latest [`v0.12.83`](https://github.com/HS7097/ActingCommand-Runtime/releases/tag/v0.12.83)) |
| [**ActingCommand-UI**](https://github.com/HS7097/ActingCommand-UI) | 🔓 Public | **Desktop UI & Neutral Modeler**: Native `acui` console, `acsetup` wizard, and game-neutral `acmodel` economy solver | Active (`v0.12` series, latest [`v0.12.33`](https://github.com/HS7097/ActingCommand-UI/releases/tag/v0.12.33)) |
| **ActingCommand-Models** | 🔒 Private | **Game Valuation & Economy Engines**: Per-game character valuation, progression calculators, and adapters hosted by `acmodel` | Active internal integration |
| **ActingCommand-Resources-Arknights** | 🔒 Private | **Arknights Standard Pack**: `1280×720` visual anchors, click boxes, task packs, and schedule value tables | `0.12` pack completion & live testing |
| **ActingCommand-Resources-AzurLane** | 🔒 Private | **Azur Lane Standard Pack**: `1280×720` visual anchors, click boxes, task packs, and schedule value tables | `0.12` pack completion & live testing |
| **ActingCommand-Resources-BlueArchive** | 🔒 Private | **Blue Archive Standard Pack**: `1280×720` visual anchors, click boxes, task packs, and schedule value tables | `0.12` pack completion & live testing |
| **ActingCommand-Resources-GirlsFrontline2** | 🔒 Private | **Girls' Frontline 2: Exilium Standard Pack**: `1280×720` visual anchors, click boxes, task packs, and schedule value tables | `0.12` pack completion & live testing |
| **ActingCommand-Resources-Nikke** | 🔒 Private | **NIKKE Resource & Economy Data**: Cost graphs, economy rules, and fixed-reference capability views | Data modeling research |
| **ActingCommand-Workflow** | 🔒 Private | **Governance & Architecture**: Cross-repo contracts, issue tracking, outcome-code audits, and multi-agent workflows | Internal daily operations |

*(Note: `bundles/` and `docs/` in this umbrella repository hold archived materials from earlier milestones for reference only and are no longer updated.)*

---

## Contributing & License

- **Feedback & Issues**: Have suggestions or run into an issue while running `0.12` or authoring packs? Open an [Issue](https://github.com/HS7097/ActingCommand/issues) in this repository or the relevant public member repository.
- **License**: This repository and ActingCommand-Runtime are released under [`AGPL-3.0-only`](LICENSE); ActingCommand-UI is `GPL-3.0-only`. No source code of other automation projects is in the project's repositories; the Arknights standard pack contains interface template images from MaaAssistantArknights (`AGPL-3.0-only`). See [`LICENSE_POLICY.md`](LICENSE_POLICY.md) and [`NOTICE.md`](NOTICE.md) for the upstream license matrix, third-party components and the compliance rules.
