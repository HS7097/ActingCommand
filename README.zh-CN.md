<p align="right">🌐 <a href="./README.md">English</a> · <b>简体中文</b></p>

<div align="center">

<img src="docs/assets/readme/actingcommand-icon.png" width="112" alt="ActingCommand 图标">

**首席执行官 兼 董事长** — HS7097<br/>
**首席技术官 兼 首席架构师** — Claude Opus 5.5 · GPT‑6 Astra · Claude Fable 5 · GPT‑5.6 Sol<br/>
**董事会秘书 兼 首席审计官** — Claude Opus 5.5 · Claude Fable 5.1<br/>
**首席技术工程师** — Claude Opus 5.5 · GPT‑6 Astra · GPT‑5.6 Sol<br/>
**正在面试** — DeepSeek

</div>

**⚠️ 主线功能已完成，并已在真实模拟器实例上端到端跑通；多日长跑验证与收尾仍在进行，接口仍可能调整。**

# ActingCommand

- [ActingCommand-Runtime](https://github.com/HS7097/ActingCommand-Runtime) — Rust 常驻运行时，核心程序
- [ActingCommand-UI](https://github.com/HS7097/ActingCommand-UI) — 安装向导与只读监控台
- [ActingCommand-Resources-Arknights](https://github.com/HS7097/ActingCommand-Resources-Arknights) — Arknights 资源包
- [ActingCommand-Resources-AzurLane](https://github.com/HS7097/ActingCommand-Resources-AzurLane) — Azur Lane 资源包
- [ActingCommand-Resources-BlueArchive](https://github.com/HS7097/ActingCommand-Resources-BlueArchive) — Blue Archive 资源包

## 本仓

本仓是 ActingCommand 项目族的伞仓（门面页），不承载代码；代码在上面各成员仓，链接直达仓库主页。本仓只放 README、`skills/` 下的程序 skill，以及 Releases 里按版本号发布的版本：每个版本包含安装向导，以及各成员仓为该版本发布的原样资产。

**我们能做什么：** 让智能体部署我们的程序，或由人使用随每个版本一并发布的安装向导（在线 `acsetup.exe`、离线 `acsetup-full-<tag>.exe`）自行安装；在 Harness 里加载对应的 skill 之后，你就可以让智能体为运行在安卓模拟器上的程序制作所需的素材，然后定期重复运行。

**程序 skill：** [`skills/actingcommand/SKILL.md`](skills/actingcommand/SKILL.md) 是给智能体用的车间手册，讲怎样用命令行操作已安装的 ActingCommand（CLI 版 v0，对应 Runtime v0.9.1 与 UI v0.9.0；第 7 节讲 Runtime v0.10.0 的 Lab 录制与挂起任务报告）。

**智能体需要做什么：** 制作一些图片和点击区域。

**我们怎么做的：** Runtime 是一个常驻的 Rust 程序，本身不含任何游戏逻辑。它通过 ADB（以及模拟器厂商提供的接口）连上安卓模拟器，按节拍截帧，在帧上识别素材（模板、颜色、OCR、神经网络），在事先声明的区域内点击，并把每一步——看到了什么、做了什么、结果如何——作为类型化事件写进一本只追加的账本。账本是唯一的事实来源：调度器按它决定下一次运行，监控台只读它，出了问题也只从它溯源。游戏相关的一切——图片、点击区域、任务顺序——都放在按游戏分开的资源包里，由哈希封印；Runtime 只装载、校验、执行。换游戏换资源包，程序不动。

**欢迎参与：** 如果您有更好的想法，或者在使用中遇到了什么问题，可以直接在本仓创建 issue，或是在对应仓直接创建 PR。

## 安装

需要 Windows；游戏实例运行在 MuMu 安卓模拟器上。按当前用户安装，不需要管理员权限。

1. 在 [Releases](https://github.com/HS7097/ActingCommand/releases) 页面找到最新的版本 `vX.Y.Z`，**只下载一个**安装向导：
   - `acsetup.exe`（在线版）：自己去下载该版本的其余文件；
   - `acsetup-full-<tag>.exe`（离线版）：已带上整个版本。

   两者旁边各有一个 `.sha256` 文件，可以用来校验下载。
2. 运行安装向导，共五步：
   1. **位置**：选择安装根目录，默认 `%LOCALAPPDATA%\Programs\ActingCommand`。该目录里已有安装时，这次运行就是升级。
   2. **安装**：下载（或解出）该版本，逐个文件对照 `SHA256SUMS` 和每个 zip 里的 `BUILD-MANIFEST.json` 核对，有任何不符就停下；全部核对通过后，才铺设 `runtime\`、`ui\`、`tools\`。
   3. **选项**：写好 Runtime 配置 `actingd.config.json` 和监控台设置。开机启动、开始菜单快捷方式、桌面快捷方式都是可选项。
   4. **实例**（可跳过）：找到 MuMu 及其实例，并列出该版本附带的游戏资源包。跳过的话，以后可以用监控台顶栏的“实例配置”按钮补上。
   5. **完成**：汇总装了什么，以及安装日志在哪里。
3. 升级时，在同一个安装根目录上运行更新版本的安装向导即可。原有的配置和状态都会保留，被替换的版本放在 `previous\` 里。

这些也可以交给智能体来做，上面的程序 skill 写了怎么做。安装向导的完整说明（包括升级、离线版、安装日志）见 UI 仓 README 的 [Setup wizard acsetup](https://github.com/HS7097/ActingCommand-UI#setup-wizard-acsetup) 一节。

## 版本发布

每个版本在本仓发布为 `vX.Y.Z`（目前都是预览版，标记为 pre-release）。各成员先由自己仓库的发布工作流从确切的提交发布，本仓的发布原样带上这些资产：

| 资产 | 内容 |
|---|---|
| `actingcommand-runtime-<sha>.zip` | Runtime：`actingcommand-actingd.exe`、`actingctl.exe`、配置模板、INSTALL.md、RELEASE-NOTES.md |
| `actingcommand-tools-<sha>.zip` | 同一次构建的工具：`actinglab.exe`、`actingledger.exe` 和 OCR 提供者 |
| `acui-windows-<sha>.zip` | 监控台 `acui.exe` 和安装向导 |
| `<game>-resources-<sha7>.zip` | 各游戏资源仓发布的标准资源包 |
| `MEMBERS.json`、`SHA256SUMS` | 各成员的提交与发布；每个 zip 以及 `MEMBERS.json` 的 SHA-256 |
| `acsetup.exe`、`acsetup-full-<tag>.exe` | 在线版、离线版安装向导，各带一个 `.sha256` |

发布之前，每个成员 zip 都会对照它自带的 `BUILD-MANIFEST.json`（仓库、提交、每个文件的大小和 SHA-256）核对一遍；离线版安装向导组装好后还会回读核对。本仓没有 CI，版本由人工组装和核对。

以前每日发布留下的 `build-r<Runtime>-u<UI>` 预发布仍保留在 Releases 页面上供参考；每日发布已经停用。
