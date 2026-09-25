<p align="right">🌐 <a href="./README.md">English</a> · <b>简体中文</b></p>

<div align="center">

<img src="docs/assets/readme/actingcommand-icon.png" width="112" alt="ActingCommand 图标">

**首席执行官 兼 董事长** — HS7097<br/>
**首席技术官 兼 首席架构师** — GPT‑6 Astra<br/>
**董事会秘书 兼 首席审计官** — Fable 5.1<br/>
**首席技术工程师** — GPT‑6 Astra<br/>
**正在面试** — DeepSeek

</div>

**⚠️ 本程序仍在快速迭代，预计 2–5 星期内完成。**

# ActingCommand

- [ActingCommand-Runtime](https://github.com/HS7097/ActingCommand-Runtime) — Rust 常驻运行时，核心程序
- [ActingCommand-UI](https://github.com/HS7097/ActingCommand-UI) — 只读监控台
- [ActingCommand-Resources-Arknights](https://github.com/HS7097/ActingCommand-Resources-Arknights) — Arknights 资源包
- [ActingCommand-Resources-AzurLane](https://github.com/HS7097/ActingCommand-Resources-AzurLane) — Azur Lane 资源包
- [ActingCommand-Resources-BlueArchive](https://github.com/HS7097/ActingCommand-Resources-BlueArchive) — Blue Archive 资源包

## 本仓

本仓是 ActingCommand 项目族的伞仓（门面页），不承载代码；代码在上面各成员仓，链接直达仓库主页。本仓只放 README、`bundles/` 目录下的资源包，以及 Releases 里的安装包与造物。每日发布为每个资源仓转发一个资源包（bundle）：默认取本仓 `bundles/` 目录携带的资源包（当前做法）；若配置了读取令牌，则改取该资源仓最新发布的资源包。

**我们能做什么：** 让智能体部署（或人类安装——人类友好的安装界面正在制作）我们的程序；在 Harness 里加载对应的 skill 之后，你就可以让智能体为运行在安卓模拟器上的程序制作所需的素材，然后定期重复运行。

**智能体需要做什么：** 制作一些图片和点击区域。

**我们怎么做的：** Runtime 是一个常驻的 Rust 程序，本身不含任何游戏逻辑。它通过 ADB（以及模拟器厂商提供的接口）连上安卓模拟器，按节拍截帧，在帧上识别素材（模板、颜色、OCR、神经网络），在事先声明的区域内点击，并把每一步——看到了什么、做了什么、结果如何——作为类型化事件写进一本只追加的账本。账本是唯一的事实来源：调度器按它决定下一次运行，监控台只读它，出了问题也只从它溯源。游戏相关的一切——图片、点击区域、任务顺序——都放在按游戏分开的资源包里，由哈希封印；Runtime 只装载、校验、执行。换游戏换资源包，程序不动。

**欢迎参与：** 如果您有更好的想法，或者在使用中遇到了什么问题，可以直接在本仓创建 issue，或是在对应仓直接创建 PR。

## 安装包与造物

成员仓的构建产物每日自动同步到本仓的 [Releases](https://github.com/HS7097/ActingCommand/releases)：取各成员仓 `main` 上最新一个已有成功构建产物的提交，每个发布对应一对提交（`build-r<Runtime 前 7 位>-u<UI 前 7 位>`），标为预发布候选：

| 资产 | 来源 |
|---|---|
| `actingcommand-runtime-<sha>.zip` | Runtime 仓"Windows exact-SHA build"的 runtime 产物：`actingcommand-actingd.exe`、`actingctl.exe`、配置模板、INSTALL.md、RELEASE-NOTES.md |
| `actingcommand-tools-<sha>.zip` | 同一构建的 tools 产物：`actinglab.exe`、`actingledger.exe`、vision-provider-check、device-test、`ac_fastdeploy_ppocr.dll` |
| `acui-windows-<sha>.zip` | UI 仓 build 的 Windows 产物：`acui.exe`、`acsetup.exe`（安装引导程序）、LICENSE、README |
| `acsetup.exe` | 在线安装引导程序（即 UI 产物里的 `acsetup.exe` 单独一份）：其余文件从本发布件下载；旁有 `acsetup.exe.sha256`；不列入 SHA256SUMS |
| `acsetup-full-<tag>.exe` | 离线安装引导程序，内嵌整个发布件：全部 zip、MEMBERS.json、SHA256SUMS；旁有 `acsetup-full-<tag>.exe.sha256`；不列入 SHA256SUMS |
| `<game>-bundle-<sha7>.zip` | 各资源仓最新的资源包（其最新的 `bundle-*` 发布），有则转发；在读取令牌就位之前，则为本仓 `bundles/` 目录里携带的资源包：`applications.json`、`bundle.json`、`packs/`（封印后的资源包） |
| `MEMBERS.json`、`SHA256SUMS` | 来源运行与产物 ID；全部 zip 与 MEMBERS.json 的 SHA-256 |

发布由工作流 `publish-artifacts` 完成：取来源仓的产物，逐文件核对 BUILD-MANIFEST.json 里的 SHA-256 与提交号后再发布；每日 03:47 UTC 自动运行，也可手动运行；同一对提交只发一次。来源仓产物保留 30 天，超期未发布的提交会在工作流里报错并开 issue。各资源仓最新的资源包在可用时（在读取令牌就位之前，则为本仓 `bundles/` 目录里携带的资源包）按其 `.sha256` 校验后，随每日发布一并转发。
