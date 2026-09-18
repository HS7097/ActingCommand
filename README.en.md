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

This is the umbrella (portal) repository of the ActingCommand family. It holds no code: the code lives in the member repositories linked above, and each link goes to that repository's home page. This repository carries only the README and the installers and artifacts on its Releases page. Resource packs are not distributed here; each game ships from its own resource repository.

## Installers and artifacts

Member build artifacts are synced daily to this repository's [Releases](https://github.com/HS7097/ActingCommand/releases) page: for each member the newest `main` commit that already has a successful build artifact, one pre-release per commit pair (`build-r<Runtime 7>-u<UI 7>`):

| Asset | Source |
|---|---|
| `actingcommand-runtime-<sha>.zip` | Runtime "Windows exact-SHA build" runtime artifact: `actingcommand-actingd.exe`, `actingctl.exe`, config template, INSTALL.md, RELEASE-NOTES.md |
| `actingcommand-tools-<sha>.zip` | Tools artifact of the same build: `actinglab.exe`, `actingledger.exe`, vision-provider-check, device-test, `ac_fastdeploy_ppocr.dll` |
| `acui-windows-<sha>.zip` | UI build Windows artifact: `acui.exe`, `acsetup.exe` (the setup wizard), LICENSE, README |
| `MEMBERS.json`, `SHA256SUMS` | Source runs and artifact ids; SHA-256 of every asset |

The `publish-artifacts` workflow fetches the source artifacts, re-checks every file against its BUILD-MANIFEST.json (SHA-256 and commit) and only then publishes. It runs daily at 03:47 UTC and can be run by hand; each commit pair is published once. Source artifacts are kept for 30 days; a pinned commit whose artifacts expired fails the workflow and opens an issue. Resource packs are not distributed here.
