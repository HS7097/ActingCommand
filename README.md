<div align="center">

<img src="docs/assets/readme/actingcommand-icon.png" width="112" alt="ActingCommand 图标">

**首席执行官 兼 董事长** — HS7097<br/>
**首席技术官 兼 首席架构师** — GPT‑6 Astra<br/>
**董事会秘书 兼 首席审计官** — Fable 5.1<br/>
**首席技术工程师** — GPT‑6 Astra<br/>
**正在面试** — DeepSeek

</div>

**🌐 语言 / Language:** 简体中文 · [English](./README.en.md)

# ActingCommand

- [ActingCommand-Runtime](https://github.com/HS7097/ActingCommand-Runtime) — Rust 常驻运行时，核心程序
- [ActingCommand-UI](https://github.com/HS7097/ActingCommand-UI) — 只读监控台
- [ActingCommand-Resources-Arknights](https://github.com/HS7097/ActingCommand-Resources-Arknights) — Arknights 资源包
- [ActingCommand-Resources-AzurLane](https://github.com/HS7097/ActingCommand-Resources-AzurLane) — Azur Lane 资源包
- [ActingCommand-Resources-BlueArchive](https://github.com/HS7097/ActingCommand-Resources-BlueArchive) — Blue Archive 资源包

## 本仓

本仓是 ActingCommand 项目族的伞仓（门面页），不承载代码；代码在上面各成员仓，链接直达仓库主页。本仓只放 README 与 Releases 里的安装包与造物。资源包不随本仓分发，按游戏各自的资源仓分发。

## 安装包与造物

成员仓的构建产物每日自动同步到本仓的 [Releases](https://github.com/HS7097/ActingCommand/releases)：取各成员仓 `main` 上最新一个已有成功构建产物的提交，每个发布对应一对提交（`build-r<Runtime 前 7 位>-u<UI 前 7 位>`），标为预发布候选：

| 资产 | 来源 |
|---|---|
| `actingcommand-runtime-<sha>.zip` | Runtime 仓"Windows exact-SHA build"的 runtime 产物：`actingcommand-actingd.exe`、`actingctl.exe`、配置模板、INSTALL.md、RELEASE-NOTES.md |
| `actingcommand-tools-<sha>.zip` | 同一构建的 tools 产物：`actinglab.exe`、`actingledger.exe`、vision-provider-check、device-test、`ac_fastdeploy_ppocr.dll` |
| `acui-windows-<sha>.zip` | UI 仓 build 的 Windows 产物：`acui.exe`、LICENSE、README |
| `MEMBERS.json`、`SHA256SUMS` | 来源运行与产物 ID；全部资产的 SHA-256 |

发布由工作流 `publish-artifacts` 完成：取来源仓的产物，逐文件核对 BUILD-MANIFEST.json 里的 SHA-256 与提交号后再发布；每日 03:47 UTC 自动运行，也可手动运行；同一对提交只发一次。来源仓产物保留 30 天，超期未发布的提交会在工作流里报错并开 issue。资源包不随本仓分发。
