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

**程序 skill：** [`skills/actingcommand/SKILL.md`](skills/actingcommand/SKILL.md) 是给智能体用的手册，讲怎样通过本地 MCP 服务 `actingctl mcp-serve`（Runtime v0.11.0）操作已安装的 ActingCommand，命令行手册作为兜底。安装时把整个 `skills/actingcommand` 目录复制或链接到 `~/.claude/skills/actingcommand`（Claude Code）和 `~/.agents/skills/actingcommand`（Codex）。

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
   2. **安装**：下载（或解出）该版本，逐个文件对照 `SHA256SUMS` 和每个 zip 里的 `BUILD-MANIFEST.json` 核对，有任何不符就停下；全部核对通过后，才把程序核心（`runtime\` 和 `ui\`）铺进程序槽 `A\`，并在安装根目录的 `runtime\`、`ui\` 里放好固定入口，固定入口总是启动 `install\active.json` 选中的槽。槽里只有程序核心：工具在安装根目录的 `tools\` 里，视觉模型和 ONNX Runtime 在 `vision\` 里（模型在 `vision\models\<model_ref>\`，ONNX Runtime 在 `vision\ort\`），资源包（`packages\`）和状态（`state\`）也都留在安装根目录；切换槽位时它们都保持原样。
   3. **选项**：写好 Runtime 配置 `actingd.config.json` 和监控台设置。开机启动、开始菜单快捷方式、桌面快捷方式都是可选项。
   4. **实例**（可跳过）：找到 MuMu 及其实例，并列出该版本附带的游戏资源包。跳过的话，以后可以用监控台顶栏的“实例配置”按钮补上。
   5. **完成**：汇总装了什么，以及安装日志在哪里。
3. 升级时，在同一个安装根目录上运行更新版本的安装向导即可。它先在另一个槽（`B\` 或 `A\`）里准备好新版本，再切换过去。原有的配置和状态都会保留，被替换的版本留在它的槽里，可以用 `ui\acsetup.exe --rollback` 切回。Runtime 不能从 v0.11.3 或更新的版本回滚到 v0.11.2 或更早的版本：从 v0.11.3 起它按接口修订 2 写账本，更早的 Runtime 读不了，所以 `--rollback` 到这样的槽会以退出码 1 停下，不做任何改动；也不要手工把 `install\active.json` 改回去。v0.11.1 之前的安装在第一次升级时，原有程序和配置会移到 `install\initial-backup-<generation>\`。

**命令行：** acsetup 也可以不开窗口运行：

```
acsetup --root <abs path> (--plan|--yes) [--conflicts new|old] [--associate <alias>=<bundle>/<server>] [--allow-downgrade] [--online|--from <folder>]
```

`--plan` 列出全部改动和差异，安装根目录下不改任何文件；`--yes` 执行安装或升级。`--associate` 可以重复，每个实例别名一次。退出码：0 完成，1 失败，2 用法错误，3 维护绑定有差异而未给 `--conflicts`，4 降级而未给 `--allow-downgrade`，5 有资源关联要选择而未给 `--associate`，6（从 v0.11.3 起）接口不兼容，即 Runtime、UI、acsetup 与资源标准包声明的接口修订对不上；2 到 6 都停在安装改动之前。从 v0.11.3 起，`acsetup --root <abs path> --resources <标准包 zip> (--plan|--yes)` 把一个资源仓的标准包放进已有的 A/B 安装，既不换程序，也不切槽。离线版 `acsetup-full-<tag>.exe` 只用自带的版本；在线版 `acsetup.exe` 需要 `--online` 或 `--from <folder>`。`acsetup --help` 打印完整用法。acsetup 是窗口程序，交互式控制台的提示符不会等它结束；下面的一键脚本会等它结束，并原样传出它的退出码。

**一键脚本：** 从 v0.11.2 起，每个版本还附带两个脚本，它们只是发布资产，不是本仓里的文件：
- `install.ps1`：从同一个版本下载 `acsetup-full-<tag>.exe` 及其 `.sha256`，核对 SHA-256，再以命令行模式运行 acsetup，打印并原样传出它的退出码。脚本自己的退出码是 10（下载失败）和 11（SHA-256 不符）。先运行 `powershell -NoProfile -ExecutionPolicy Bypass -File install.ps1 -Root F:\AC --plan`，再把 `--plan` 换成 `--yes` 运行一次。
- `install.sh`（Git Bash）：取该版本的 `install.ps1`，按 install.sh 里记录的 SHA-256 核对，再用同样的参数运行它，例如 `bash install.sh -Root F:/AC --plan`。

**Runtime 守护：** 从 v0.11.3 起，Runtime 没经过正式关闭就结束时（崩溃、窗口被关、重启电脑），可以自动再拉起来。安装向导不会启用它：请在 Runtime 所属用户的普通（非管理员）PowerShell 里运行一次 `<root>\runtime\actingctl.exe watchdog install --root <root>`。它注册一个按用户运行的计划任务，每分钟运行 `<root>\tools\actingwatch.exe`。守护从不拉起正式关闭的 Runtime；最后一份 Runtime 日志以 FATAL 行结束时保持停止；30 分钟内最多拉起 3 次。`watchdog status --root <root>` 只报告、不改动任何东西，`watchdog uninstall --root <root>` 删除该任务（移除安装之前先运行它），记录在 `<root>\watchdog\watchdog.log`。详见 Runtime 的 INSTALL.md「Runtime watchdog」一节。

这些也可以交给智能体来做，上面的程序 skill 写了怎么做。安装向导的完整说明（包括升级、离线版、安装日志）见 UI 仓 README 的 [Setup wizard acsetup](https://github.com/HS7097/ActingCommand-UI#setup-wizard-acsetup) 一节。

## 版本发布

每个版本在本仓发布为 `vX.Y.Z`（目前都是预览版，标记为 pre-release）。各组件已解耦：每个组件仓（Runtime、UI、各资源仓）各自发布，由自己的发布工作流从确切的提交发布，并且只在有改动时才发布；只要接口保持兼容，各组件就能继续搭配使用。从 v0.11.3 起，每次 Runtime 和 UI 构建都在自己的 `BUILD-MANIFEST.json` 里声明它读写的接口修订，acsetup 在改动任何东西之前，按这些声明核对它要安装的每一种组合。本仓的每个版本带上每个成员最新的发布，资产原样不动，因此各成员的版本号可以彼此不同，也可以与本仓的版本号不同；`MEMBERS.json` 记录每个成员自己的 tag。例如 v0.11.3 带的是 Runtime v0.11.3、UI v0.11.3、Azur Lane 资源 v0.11.3、Arknights 资源 v0.11.4 和 Blue Archive 资源 v0.11.4：

| 资产 | 内容 |
|---|---|
| `actingcommand-runtime-<sha>.zip` | Runtime：`actingcommand-actingd.exe`、`actingctl.exe`、配置模板、INSTALL.md、RELEASE-NOTES.md |
| `actingcommand-tools-<sha>.zip` | 同一次构建的工具：`actinglab.exe`、`actingledger.exe`、两个检查程序、从 v0.11.3 起的守护启动器 `actingwatch.exe`，以及 adb。从 v0.11.2 起不再带 OCR 提供者：OCR 在 Runtime 进程内运行 |
| `acui-windows-<sha>.zip` | 监控台 `acui.exe` 和安装向导 |
| `<game>-resources-<sha7>.zip` | 各游戏资源仓发布的标准资源包 |
| `MEMBERS.json`、`SHA256SUMS` | 各成员的提交与发布；每个 zip 以及 `MEMBERS.json` 的 SHA-256 |
| `acsetup.exe`、`acsetup-full-<tag>.exe` | 在线版、离线版安装向导，各带一个 `.sha256` |
| `install.ps1`、`install.sh` | 从 v0.11.2 起：一键安装脚本（见“安装”一节） |

发布之前，每个成员 zip 都会对照它自带的 `BUILD-MANIFEST.json`（仓库、提交、每个文件的大小和 SHA-256）核对一遍；离线版安装向导组装好后还会回读核对。本仓没有 CI，版本由人工组装和核对。

以前每日发布留下的 `build-r<Runtime>-u<UI>` 预发布仍保留在 Releases 页面上供参考；每日发布已经停用。
