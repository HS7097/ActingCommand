**🌐 Language / 语言:** English · [简体中文](./README.zh-CN.md)

<div align="center">

<img src="docs/assets/readme/actingcommand-icon.png" width="112" alt="ActingCommand icon">

**Chief Executive Officer & Chairman** — HS7097<br/>
**Chief Technology Officer & Chief Architect** — GPT‑6 Astra<br/>
**Board Secretary & Chief Audit Officer** — Fable 5.1<br/>
**Principal Engineer** — GPT‑6 Astra<br/>
**Interviewing** — DeepSeek

</div>

**⚠️ This program is still iterating rapidly; expect it to be complete within 2–5 weeks.**

# ActingCommand

- [ActingCommand-Runtime](https://github.com/HS7097/ActingCommand-Runtime) — resident Rust runtime, the core program
- [ActingCommand-UI](https://github.com/HS7097/ActingCommand-UI) — read-only console
- [ActingCommand-Resources-Arknights](https://github.com/HS7097/ActingCommand-Resources-Arknights) — Arknights resource pack
- [ActingCommand-Resources-AzurLane](https://github.com/HS7097/ActingCommand-Resources-AzurLane) — Azur Lane resource pack
- [ActingCommand-Resources-BlueArchive](https://github.com/HS7097/ActingCommand-Resources-BlueArchive) — Blue Archive resource pack

## This repository

This is the umbrella (portal) repository of the ActingCommand family. It holds no code: the code lives in the member repositories linked above, and each link goes to that repository's home page. This repository carries only the README and the installers and artifacts on its Releases page. Resource packs are not distributed here; each game ships from its own resource repository.

**What we can do:** Let an AI agent deploy our program (or a person install it; the human-friendly setup wizard is still being built). Once the matching skill is loaded in your harness, you can have the agent produce the materials that a program running in an Android emulator needs, and then run it again and again on a schedule.

**What the agent has to do:** Make some images and click regions.

**How we do it:** The Runtime is a resident Rust program that contains no game logic of its own. It connects to Android emulators over ADB (and the emulator vendor's interfaces), captures frames on a cadence, recognizes materials in each frame (template, color, OCR, neural network), clicks inside regions declared in advance, and writes every step, what it saw, what it did and how it went, as typed events into an append-only ledger. The ledger is the single source of truth: the scheduler decides the next run from it, the console only reads it, and when something goes wrong it is traced from the ledger alone. Everything game-specific, the images, the click regions and the task order, lives in a per-game resource pack sealed by hash; the Runtime only loads, verifies and executes it. Switch the game by switching the pack; the program does not change.

**Join in:** If you have a better idea, or run into a problem while using it, open an issue in this repository, or open a pull request directly in the repository concerned.

## Installers and artifacts

Member build artifacts are synced daily to this repository's [Releases](https://github.com/HS7097/ActingCommand/releases) page: for each member the newest `main` commit that already has a successful build artifact, one pre-release per commit pair (`build-r<Runtime 7>-u<UI 7>`):

| Asset | Source |
|---|---|
| `actingcommand-runtime-<sha>.zip` | Runtime "Windows exact-SHA build" runtime artifact: `actingcommand-actingd.exe`, `actingctl.exe`, config template, INSTALL.md, RELEASE-NOTES.md |
| `actingcommand-tools-<sha>.zip` | Tools artifact of the same build: `actinglab.exe`, `actingledger.exe`, vision-provider-check, device-test, `ac_fastdeploy_ppocr.dll` |
| `acui-windows-<sha>.zip` | UI build Windows artifact: `acui.exe`, `acsetup.exe` (the setup wizard), LICENSE, README |
| `MEMBERS.json`, `SHA256SUMS` | Source runs and artifact ids; SHA-256 of every asset |

The `publish-artifacts` workflow fetches the source artifacts, re-checks every file against its BUILD-MANIFEST.json (SHA-256 and commit) and only then publishes. It runs daily at 03:47 UTC and can be run by hand; each commit pair is published once. Source artifacts are kept for 30 days; a pinned commit whose artifacts expired fails the workflow and opens an issue. Resource packs are not distributed here.
