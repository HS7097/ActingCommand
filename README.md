<p align="right">🌐 <b>English</b> · <a href="./README.zh-CN.md">简体中文</a></p>

<div align="center">

<img src="docs/assets/readme/actingcommand-icon.png" width="112" alt="ActingCommand icon">

**Chief Executive Officer & Chairman** — HS7097<br/>
**Chief Technology Officer & Chief Architect** — Claude Opus 5.5 · GPT‑6 Astra · Claude Fable 5 · GPT‑5.6 Sol<br/>
**Board Secretary & Chief Audit Officer** — Claude Opus 5.5 · Claude Fable 5.1<br/>
**Principal Engineer** — Claude Opus 5.5 · GPT‑6 Astra · GPT‑5.6 Sol<br/>
**Interviewing** — DeepSeek

</div>

**⚠️ The main-line features are complete and have run end to end on a real emulator instance; multi-day validation and clean-up are still under way, and interfaces may still change.**

# ActingCommand

- [ActingCommand-Runtime](https://github.com/HS7097/ActingCommand-Runtime) — resident Rust runtime, the core program
- [ActingCommand-UI](https://github.com/HS7097/ActingCommand-UI) — setup wizard and read-only console
- [ActingCommand-Resources-Arknights](https://github.com/HS7097/ActingCommand-Resources-Arknights) — Arknights resource pack
- [ActingCommand-Resources-AzurLane](https://github.com/HS7097/ActingCommand-Resources-AzurLane) — Azur Lane resource pack
- [ActingCommand-Resources-BlueArchive](https://github.com/HS7097/ActingCommand-Resources-BlueArchive) — Blue Archive resource pack

## This repository

This is the umbrella (portal) repository of the ActingCommand family. It holds no code: the code lives in the member repositories linked above, and each link goes to that repository's home page. This repository carries only the README, the program skill under `skills/`, and the versioned releases on its Releases page: the setup wizards and, unchanged, the release assets of the member repositories for each version.

**What we can do:** Let an AI agent deploy our program, or install it yourself with the setup wizard published with every release (online `acsetup.exe`, offline `acsetup-full-<tag>.exe`). Once the matching skill is loaded in your harness, you can have the agent produce the materials that a program running in an Android emulator needs, and then run it again and again on a schedule.

**Program skill:** [`skills/actingcommand/SKILL.md`](skills/actingcommand/SKILL.md) is the manual for an AI agent that operates an installed ActingCommand through its local MCP server `actingctl mcp-serve` (Runtime v0.11.0), with the command-line manual as the fallback. To install it, copy or link the whole `skills/actingcommand` directory to `~/.claude/skills/actingcommand` (Claude Code) and `~/.agents/skills/actingcommand` (Codex).

**What the agent has to do:** Make some images and click regions.

**How we do it:** The Runtime is a resident Rust program that contains no game logic of its own. It connects to Android emulators over ADB (and the emulator vendor's interfaces), captures frames on a cadence, recognizes materials in each frame (template, color, OCR, neural network), clicks inside regions declared in advance, and writes every step, what it saw, what it did and how it went, as typed events into an append-only ledger. The ledger is the single source of truth: the scheduler decides the next run from it, the console only reads it, and when something goes wrong it is traced from the ledger alone. Everything game-specific, the images, the click regions and the task order, lives in a per-game resource pack sealed by hash; the Runtime only loads, verifies and executes it. Switch the game by switching the pack; the program does not change.

**Join in:** If you have a better idea, or run into a problem while using it, open an issue in this repository, or open a pull request directly in the repository concerned.

## Install

Requirements: Windows, and the MuMu Android emulator for the game instances. The installation is per user; no administrator rights are needed.

1. From the [Releases](https://github.com/HS7097/ActingCommand/releases) page, take the newest release `vX.Y.Z` and download **one** setup wizard:
   - `acsetup.exe` (online): downloads the rest of that release itself;
   - `acsetup-full-<tag>.exe` (offline): carries the whole release.

   Each has a `.sha256` file next to it if you want to check the download.
2. Run it. The wizard has five steps:
   1. **Location**: the install root, by default `%LOCALAPPDATA%\Programs\ActingCommand`. If an installation is already there, this run is an upgrade.
   2. **Install**: fetches (or extracts) the release, checks every file against `SHA256SUMS` and the `BUILD-MANIFEST.json` inside each zip, and stops if anything differs. Only then does it lay out the program core (`runtime\` and `ui\`) in the program slot `A\`, and at the install root the fixed entries in `runtime\` and `ui\`, which always start the slot that `install\active.json` selects. A slot holds only the program core: the tools are in `tools\` at the install root, the vision models and ONNX Runtime in `vision\` (models in `vision\models\<model_ref>\`, ONNX Runtime in `vision\ort\`), and the resource packages (`packages\`) and the state (`state\`) stay at the install root as well; switching the slot leaves all of them as they are.
   3. **Options**: writes the Runtime configuration `actingd.config.json` and the console settings. Start at boot, a Start menu shortcut and a desktop shortcut are optional.
   4. **Instances** (optional): finds MuMu and its instances, and offers the game resource packages carried by the release. You can skip it and do it later with the console's Instance Configuration button.
   5. **Finish**: a summary of what was installed and where the install log is.
3. To upgrade, run the wizard of a newer release on the same install root. It prepares the new version in the other slot (`B\` or `A\`) before it switches. Your configuration and state are kept, and the version it replaces stays in its slot, where `ui\acsetup.exe --rollback` selects it again. An install from before v0.11.1 has its programs and configuration moved to `install\initial-backup-<generation>\` on its first upgrade.

**Command line:** acsetup also runs without its window:

```
acsetup --root <abs path> (--plan|--yes) [--conflicts new|old] [--associate <alias>=<bundle>/<server>] [--allow-downgrade] [--online|--from <folder>]
```

`--plan` lists every change and difference and changes nothing under the install root; `--yes` installs or upgrades. `--associate` can be repeated, once per instance alias. Exit codes: 0 done, 1 failed, 2 usage, 3 maintenance-binding differences without `--conflicts`, 4 a downgrade without `--allow-downgrade`, 5 a resource association to choose without `--associate`; 2, 3, 4 and 5 stop before the installation changes. The offline `acsetup-full-<tag>.exe` uses only the release it carries; the online `acsetup.exe` needs `--online` or `--from <folder>`. `acsetup --help` prints the whole usage. acsetup is a windowed program, so an interactive console prompt does not wait for it; the one-step scripts below wait for it and pass its exit code through.

**One-step scripts:** from v0.11.2 each release also carries two scripts, as release assets only, not as files of this repository:
- `install.ps1` downloads `acsetup-full-<tag>.exe` and its `.sha256` from the same release, checks the SHA-256, runs acsetup in command-line mode, prints its exit code and passes it through. The script's own exit codes are 10 (a download failed) and 11 (SHA-256 mismatch). Run `powershell -NoProfile -ExecutionPolicy Bypass -File install.ps1 -Root F:\AC --plan` first, then the same with `--yes`.
- `install.sh` (Git Bash) fetches that release's `install.ps1`, checks it against the SHA-256 recorded inside install.sh and runs it with the same arguments, for example `bash install.sh -Root F:/AC --plan`.

An AI agent can do all of this for you; the program skill above tells it how. The complete description of the wizard, including upgrades, the offline edition and the install log, is in the UI repository's README, section [Setup wizard acsetup](https://github.com/HS7097/ActingCommand-UI#setup-wizard-acsetup).

## Releases

Each version is a release `vX.Y.Z` on this repository (at present previews, marked as pre-releases). The components are decoupled: each component repository (Runtime, UI, each resource repository) releases on its own, from an exact commit by its own release workflow, and only when it changes; the components keep working together as long as their interfaces stay compatible. A release here carries each member's newest release with its assets unchanged, so the members' versions can differ from one another and from this release's; `MEMBERS.json` records each member's own tag. v0.11.2, for example, carries Runtime v0.11.2, UI v0.11.2, Azur Lane resources v0.11.2, Arknights resources v0.11.1 and Blue Archive resources v0.11.1:

| Asset | Content |
|---|---|
| `actingcommand-runtime-<sha>.zip` | The Runtime: `actingcommand-actingd.exe`, `actingctl.exe`, the configuration template, INSTALL.md, RELEASE-NOTES.md |
| `actingcommand-tools-<sha>.zip` | The tools of the same build: `actinglab.exe`, `actingledger.exe`, two check programs and adb. From v0.11.2 it no longer carries the OCR provider: OCR runs inside the Runtime |
| `acui-windows-<sha>.zip` | The console `acui.exe` and the setup wizard |
| `<game>-resources-<sha7>.zip` | A game's standard resource package from its resource repository |
| `MEMBERS.json`, `SHA256SUMS` | The member commits and releases; the SHA-256 of every zip and of `MEMBERS.json` |
| `acsetup.exe`, `acsetup-full-<tag>.exe` | The online and offline setup wizards, each with its own `.sha256` |
| `install.ps1`, `install.sh` | From v0.11.2: the one-step install scripts (see Install) |

Before a release is published, every member zip is checked against its `BUILD-MANIFEST.json` (repository, commit, size and SHA-256 of every file), and the offline wizard is read back after it is assembled. This repository has no CI: releases are assembled and checked by hand.

The `build-r<Runtime>-u<UI>` pre-releases from the earlier daily publishing stay on the Releases page for reference; that publishing has been retired.
