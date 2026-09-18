<div align="center">

<img src="docs/assets/readme/actingcommand-icon.png" width="112" alt="ActingCommand icon">

**Chief Executive Officer & Chairman** — HS7097<br/>
**Chief Technology Officer & Chief Architect** — GPT‑6 Astra<br/>
**Board Secretary & Chief Audit Officer** — Fable 5.1<br/>
**Principal Engineer** — GPT‑6 Astra<br/>
**Interviewing** — DeepSeek

</div>

**🌐 语言 / Language:** [简体中文](./README.md) · English

# ActingCommand

- [ActingCommand-Runtime](https://github.com/HS7097/ActingCommand-Runtime) — resident Rust runtime, the core program
- [ActingCommand-UI](https://github.com/HS7097/ActingCommand-UI) — read-only console
- [ActingCommand-Resources-Arknights](https://github.com/HS7097/ActingCommand-Resources-Arknights) — Arknights resource pack
- [ActingCommand-Resources-AzurLane](https://github.com/HS7097/ActingCommand-Resources-AzurLane) — Azur Lane resource pack
- [ActingCommand-Resources-BlueArchive](https://github.com/HS7097/ActingCommand-Resources-BlueArchive) — Blue Archive resource pack

## This repository

This is the umbrella (portal) repository of the ActingCommand family; no development happens here. Runtime and UI hang under it as submodule pointers:

| Directory | Points at |
|---|---|
| `ActingCommand-Runtime/` | Runtime repository, `main` |
| `ActingCommand-UI/` | UI repository, `main` |

The `sync-member-pointers` workflow, run by hand, moves the pointers to the latest commit of each `main` (main is protected by the same rulesets as Runtime and UI; only the bypass list can write it); a failed sync opens an issue labelled `sync-failure`. Resource packs are not distributed with this repository; each game ships from its own resource repository.

```
git clone --recurse-submodules https://github.com/HS7097/ActingCommand.git
```

## Installers and artifacts

Member build artifacts are synced daily to this repository's [Releases](https://github.com/HS7097/ActingCommand/releases) page: for each member the newest `main` commit that already has a successful build artifact, one pre-release per commit pair (`build-r<Runtime 7>-u<UI 7>`), independent of the pointers above:

| Asset | Source |
|---|---|
| `actingcommand-runtime-<sha>.zip` | Runtime "Windows exact-SHA build" runtime artifact: `actingcommand-actingd.exe`, `actingctl.exe`, config template, INSTALL.md, RELEASE-NOTES.md |
| `actingcommand-tools-<sha>.zip` | Tools artifact of the same build: `actinglab.exe`, `actingledger.exe`, vision-provider-check, device-test, `ac_fastdeploy_ppocr.dll` |
| `acui-windows-<sha>.zip` | UI build Windows artifact: `acui.exe`, LICENSE, README |
| `MEMBERS.json`, `SHA256SUMS` | Source runs and artifact ids; SHA-256 of every asset |

The `publish-artifacts` workflow fetches the source artifacts, re-checks every file against its BUILD-MANIFEST.json (SHA-256 and commit) and only then publishes. It runs daily at 03:47 UTC and can be run by hand; each commit pair is published once. Source artifacts are kept for 30 days; a pinned commit whose artifacts expired fails the workflow and opens an issue. Resource packs are not distributed here.
