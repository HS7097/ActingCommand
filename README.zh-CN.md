<p align="right">🌐 <a href="./README.md">English</a> · <b>简体中文</b></p>

<div align="center">

<img src="docs/assets/readme/actingcommand-icon.png" width="112" alt="ActingCommand 图标">

**首席执行官 兼 董事长** — HS7097<br/>
**首席技术官 兼 首席架构师** — Claude Opus 5.5 · GPT‑6 Astra · Claude Fable 5.1 · Claude Fable 5 · GPT‑5.5<br/>
**董事会秘书 兼 首席审计官** — Claude Opus 5.5 · Claude Fable 5.1 · Claude Fable 5 · Claude Opus 4.8<br/>
**首席技术工程师** — Claude Opus 5.5 · GPT‑6 Astra · GPT‑5.6 Sol · GPT‑5.5<br/>
**正在面试** — DeepSeek

</div>

**⚠️ 预发布、调试阶段：所有发布都是预发布，接口、配置与文件格式还会变化；只支持最新版。0.12 系列会带来破坏性变化（见[路线图](#路线图计划中)）。**

# ActingCommand

- [ActingCommand-Runtime](https://github.com/HS7097/ActingCommand-Runtime) — Rust 常驻运行时，核心程序，含命令行、工具与本地 MCP 服务
- [ActingCommand-UI](https://github.com/HS7097/ActingCommand-UI) — 安装向导与只读控制台
- **明日方舟**、**碧蓝航线**、**蔚蓝档案**三个标准包 — 作为本仓 [Releases](https://github.com/HS7097/ActingCommand/releases) 的资产发布；它们在不公开的单独仓库里制作

## 本仓

本仓是 ActingCommand 项目族的伞仓（门面页），不承载代码；代码在上面的各成员仓。本仓放 README、`skills/` 下的程序 skill，以及 Releases 里按版本号发布的版本：每个版本包含安装向导，以及各成员为该版本发布的原样资产。`bundles/` 保留以前每日发布用过的蔚蓝档案资源包，仅作参考、不再更新；`docs/` 放 README 用的图片和较早的笔记。

**我们能做什么：** 让智能体部署我们的程序，或由人使用随每个版本一并发布的安装向导（在线 `acsetup.exe`、离线 `acsetup-full-<tag>.exe`）自行安装；在 Harness 里加载程序 skill、注册 MCP 服务之后（见[配合智能体使用](#配合智能体使用mcp)），你就可以让智能体为运行在安卓模拟器上的程序制作所需的素材，然后定期重复运行。

**智能体需要做什么：** 制作一些图片和点击区域。

**我们怎么做的：** Runtime 是一个常驻的 Rust 程序，本身不含任何游戏逻辑。它通过 ADB（以及模拟器厂商提供的接口）连上安卓模拟器，按节拍截帧，在帧上识别素材（模板、颜色、OCR、神经网络），在事先声明的区域内点击，并把每一步——看到了什么、做了什么、结果如何——作为类型化事件写进一本只追加的账本。账本是权威记录：调度器按它决定下一次运行，控制台只读它，出了问题也只从它溯源。游戏相关的一切——图片、点击区域、任务顺序、数据表——都放在按游戏分开的标准包里，其中每个任务包按内容摘要封印；Runtime 只装载、校验、执行。换游戏换包，程序不动。

**欢迎参与：** 如果您有更好的想法，或者在使用中遇到了什么问题，可以在本仓或对应的仓创建 issue。

## 开发状态

- **调试阶段。** 主循环是：发现问题 → 修复 → 对照预期检查 → 再改。部署与易用性是次要的。
- **只有预发布。** 每个成员的每次发布都是预发布。只支持最新版，旧系列不出修复。
- **会有破坏性变化。** 从 0.12.0 起 Runtime 用新的账本格式，0.11 的状态根不带过去；命令行的输出与退出码会变；MCP 档位取消；Lab 默认不再安装（见[路线图](#路线图计划中)）。
- **实机现状。** 自 10 月上旬起，Runtime 每天在真实的 MuMu 实例上按已批准的目录跑例行批次，三个游戏的标准包都在用。标准包还没有覆盖全部任务，多日无人值守长跑仍在验证。
- **版本号规则。** X 为重大或不兼容的变化，Y 为新特性或新覆盖面，Z 为修复（含为修复服务的特性）。各成员各自发版；能否搭配看各自声明的接口版本，不看版本号是否一致。

## 现在能用的

| 部分 | 做什么 |
|---|---|
| Runtime（`actingcommand-actingd.exe`） | 常驻进程，只在本机回环地址（127.0.0.1 / [::1]）上收请求，没有网络代码；客户端关掉不影响它运行。调度器：按已批准的目录与时钟槽运行任务包，租约与围栏在每次触碰设备时复核；暂停与恢复（全局或单个实例，从 v0.11.3 起熬过重启）、优先级偏移、资源目标、磁盘容量准入；从 v0.11.5 起每个实例一个请求队列，从 v0.11.6 起每个实例一个工作线程。例行运行失败后交给三级恢复阶梯：回主页、重启应用、重启模拟器（手动发起的运行不走阶梯）。从 v0.11.6 起，重启后每个没结束的运行只结算一次（有终态按终态，没有就按中断）；某个运行在启动时结算不了，只暂停它那个实例并报 Error。 |
| 识别与设备 | 七种识别目标：模板、颜色、仅点击、OCR、神经网络、颜色摘要、组合（由 2 到 8 个其他目标组成）；OCR 与神经网络在进程内用 ONNX Runtime 推理，模型第一次用到才加载。MuMu：Nemu IPC 截图与输入，经 MuMuManager 发现实例、启停模拟器；自带 adb 37.0.1。 |
| 账本 | 单写者的 SQLite 账本，记的是脱敏后的类型化事件；截图按内容摘要存放。从 v0.11.6 起，截图清理器删去不再需要的截图，并把为错误与 Lab 保留的截图移进 `<state root>\kept\<date>\`，由人删除。 |
| 命令行与工具 | `actingctl`（状态、暂停与恢复、模拟器控制、请求关闭、看门狗、MCP）；`actinglab` 从实机画面录制 `linear_steps` 任务包、检包与检目录；`actingledger` 离线读账本；看门狗启动器 `actingwatch.exe`。 |
| MCP 服务 | `actingctl mcp-serve`（从 v0.11.0 起）：给智能体用的 22 个工具，见[配合智能体使用](#配合智能体使用mcp)。 |
| 控制台（`acui.exe`） | 只读账本的原生控制台：实例卡、时间线、详情；六个页签对应账本的六个视图；可离线直接读状态根，也可经 Runtime 在线读。顶栏启动器可拉起 Runtime 或请求它关闭，「实例配置」窗口可编辑实例。 |
| 安装向导（`acsetup`） | A/B 安装与升级，改动之前逐个文件核对；也有命令行（见[安装](#安装)）。它是整个项目族里唯一有网络代码的程序。 |
| 标准包 | 明日方舟、碧蓝航线、蔚蓝档案：各游戏的全部任务包，加上程序与智能体所需的数据。现有的包都按 1280×720 的画面制作；截图尺寸不符时任务明确失败。 |

## 安装

需要 Windows；游戏实例运行在 MuMu 安卓模拟器上，分辨率 1280×720。按当前用户安装，不需要管理员权限。

1. 在 [Releases](https://github.com/HS7097/ActingCommand/releases) 页面找到最新的版本 `vX.Y.Z`，**只下载一个**安装向导：
   - `acsetup.exe`（在线版）：自己去下载最新版本的其余文件；
   - `acsetup-full-<tag>.exe`（离线版）：已带上整个版本。

   两者旁边各有一个 `.sha256` 文件，可以用来校验下载。
2. 运行安装向导，共五步：
   1. **位置**：选择安装根目录，默认 `%LOCALAPPDATA%\Programs\ActingCommand`。该目录里已有安装时，这次运行就是升级。
   2. **安装**：下载（或解出）该版本，逐个文件对照 `SHA256SUMS` 和每个 zip 里的 `BUILD-MANIFEST.json` 核对，有任何不符就停下；全部核对通过后，才把程序核心（`runtime\` 和 `ui\`）铺进程序槽 `A\`，并在安装根目录的 `runtime\`、`ui\` 里放好固定入口，固定入口总是启动 `install\active.json` 选中的槽。槽里只有程序核心：工具在安装根目录的 `tools\` 里，视觉模型和 ONNX Runtime 在 `vision\` 里（模型在 `vision\models\<model_ref>\`，ONNX Runtime 在 `vision\ort\`），资源包（`packages\`）和状态（`state\`）也都留在安装根目录；切换槽位时它们都保持原样。发布里不带视觉模型和 ONNX Runtime：它们放进 `vision\` 之前，用到 OCR 或神经网络的目标会明确失败。
   3. **选项**：写好 Runtime 配置 `actingd.config.json` 和控制台设置。开机启动、开始菜单快捷方式、桌面快捷方式都是可选项。
   4. **实例**（可跳过）：找到 MuMu 及其实例，并列出该版本附带的标准包。跳过的话，以后可以用控制台顶栏的「实例配置」按钮补上。
   5. **完成**：汇总装了什么，以及安装日志在哪里。
3. 升级时，在同一个安装根目录上运行更新版本的安装向导即可。它先在另一个槽（`B\` 或 `A\`）里准备好新版本，再切换过去。原有的配置和状态都会保留，被替换的版本留在它的槽里，可以用 `ui\acsetup.exe --rollback` 切回。已装的 Runtime 是 v0.11.1、v0.11.2 或 v0.11.3 时，升级前先正式关掉它（`<root>\runtime\actingctl.exe request-shutdown --state-root <state root> --wait 60`；因忙被拒时等它空闲再试），再运行安装向导，之后 Runtime 没在运行就把它启动。v0.11.1 之前的安装在第一次升级时，原有程序和配置会移到 `install\initial-backup-<generation>\`。

**回滚限制。** 不要手工把 `install\active.json` 改回去，请用 `--rollback`。以下回滚做不到：
- 从 v0.11.3 起 Runtime 按接口修订 2 写账本，v0.11.2 及更早的版本读不了；`--rollback` 到这样的槽会以退出码 1 停下，不做任何改动。
- 状态根很大时，从 v0.11.1–v0.11.3 升到 v0.11.4 或更新的版本实际上是单向的：回滚要跑旧版本的账本校验，在这样的状态根上会失败，安装保持不变。
- Runtime v0.11.6 的截图清理器运行过一次之后，v0.11.5 及更早的版本会拒绝打开这份状态根。配置里不写清理器时它是开着的；升级到 v0.11.6 后第一次启动，可以考虑先关掉它（`"frame_retention_enabled": false`），确认一切正常后再打开。设置方法以及配置改动如何进入 A/B 安装，见 Runtime 的 RELEASE-NOTES.md。

**命令行：** acsetup 也可以不开窗口运行：

```
acsetup --root <abs path> (--plan|--yes) [--conflicts new|old] [--associate <alias>=<bundle>/<server>] [--allow-downgrade] [--online|--from <folder>]
```

`--plan` 列出全部改动和差异，安装根目录下不改任何文件；`--yes` 执行安装或升级。`--associate` 可以重复，每个实例别名一次。退出码：0 完成，1 失败，2 用法错误，3 维护绑定有差异而未给 `--conflicts`，4 降级而未给 `--allow-downgrade`，5 有资源关联要选择而未给 `--associate`，6（从 v0.11.3 起）接口不兼容，即 Runtime、UI、acsetup 与标准包声明的接口修订对不上；2 到 6 都停在安装改动之前。从 v0.11.3 起，`acsetup --root <abs path> --resources <标准包 zip> (--plan|--yes)` 把一个标准包放进已有的 A/B 安装，既不换程序，也不切槽。离线版 `acsetup-full-<tag>.exe` 只用自带的版本；在线版 `acsetup.exe` 需要 `--online` 或 `--from <folder>`。`acsetup --help` 打印完整用法。acsetup 是窗口程序，交互式控制台的提示符不会等它结束；下面的一键脚本会等它结束，并原样传出它的退出码。

**一键脚本：** 从 v0.11.2 起，每个版本还附带两个脚本，它们只是发布资产，不是本仓里的文件：
- `install.ps1`：从同一个版本下载 `acsetup-full-<tag>.exe` 及其 `.sha256`，核对 SHA-256，再以命令行模式运行 acsetup，打印并原样传出它的退出码。脚本自己的退出码是 10（下载失败）和 11（SHA-256 不符）。先运行 `powershell -NoProfile -ExecutionPolicy Bypass -File install.ps1 -Root C:\AC --plan`，再把 `--plan` 换成 `--yes` 运行一次。
- `install.sh`（Git Bash）：取该版本的 `install.ps1`，按 install.sh 里记录的 SHA-256 核对，再用同样的参数运行它，例如 `bash install.sh -Root C:/AC --plan`。

**Runtime 看门狗：** 从 v0.11.3 起，Runtime 没经过正式关闭就结束时（崩溃、窗口被关、重启电脑），可以自动再拉起来。安装向导不会启用它：请在 Runtime 所属用户的普通（非管理员）PowerShell 里运行一次 `<root>\runtime\actingctl.exe watchdog install --root <root>`。它注册一个按用户运行的计划任务，每分钟运行 `<root>\tools\actingwatch.exe`。看门狗从不拉起正式关闭的 Runtime；最后一份 Runtime 日志以 FATAL 行结束时保持停止；30 分钟内最多拉起 3 次。从 Runtime v0.11.6 起，热升级中 held 启动失败或超时之后，它也会把 Runtime 拉起来；held 启动期间已接受的关闭则保持停止。`watchdog status --root <root>` 只报告、不改动任何东西，`watchdog uninstall --root <root>` 删除该任务（移除安装之前先运行它），记录在 `<root>\watchdog\watchdog.log`。详见 Runtime 的 INSTALL.md「Runtime watchdog」一节。

这些也可以交给智能体来做，程序 skill 写了怎么做。安装向导的完整说明（包括升级、离线版、安装日志）见 UI 仓 README 的 [Setup wizard acsetup](https://github.com/HS7097/ActingCommand-UI#setup-wizard-acsetup) 一节。

## 配合智能体使用（MCP）

从 Runtime v0.11.0 起，`actingctl mcp-serve` 是一个走 stdio 的本地 MCP 服务。它只提供工具（不提供 resources、prompts），只和同一台机器上的 Runtime 通信，Runtime 的回答与拒绝原样传出。

**注册。** `mcp-config` 打印给客户端用的注册内容，自己不写任何文件：

```
<root>\runtime\actingctl.exe mcp-config --client claude [--tier observer,operator,author]
<root>\runtime\actingctl.exe mcp-config --client codex  [--tier observer,operator,author]
```

- Claude Code：运行打印出来的命令，`claude mcp add --scope user actingcommand -- "<root>\runtime\actingctl.exe" mcp-serve --tier …`。
- Codex：把打印出来的 `[mcp_servers.actingcommand]` 一段加进 Codex 的 `config.toml`。

从 v0.11.2 起，在 A/B 安装上打印出的程序是固定入口 `<root>\runtime\actingctl.exe`，切槽之后注册照样有效。注册后新开一个客户端会话才会加载这个服务。

**档位（0.11 系列）。** `observer` 只读、总是开着，也是默认；`operator` 加上设备与调度，`author` 加上 Lab 录制。不在已开档位里的工具回答 `tier_not_enabled`；要换档位，用别的 `--tier` 重新注册。0.12 系列计划取消档位。

| 档位 | 工具 |
|---|---|
| observer | `ac_overview`、`ac_events`、`ac_material`、`ac_get_run`、`ac_diagnose`、`ac_resources_list`、`ac_targets_get`、`ac_pack_check`、`ac_catalog_check` |
| operator | `ac_run_pack`、`ac_stop_run`、`ac_pause`、`ac_resume`、`ac_emulator`、`ac_targets_set` |
| author | `ac_lab_observe`、`ac_lab_do`、`ac_record_start`、`ac_record_mark`、`ac_record_stop`、`ac_record_status`、`ac_binding_draft` |

`actingctl mcp-serve --list-tools --format markdown` 打印完整的工具表，含每个工具的参数。

**行为约定。** 长操作立即返回句柄，用 `ac_get_run`（带 `wait_s`）轮询。运行的句柄就是 Runtime 的请求 id，跨重启有效。结果不确定时先问 `ac_get_run`，不要重发写操作。批准、改 Runtime 配置、重启 Runtime 都归人来做。

**程序 skill。** [`skills/actingcommand/SKILL.md`](skills/actingcommand/SKILL.md) 是智能体使用这些工具的手册，含查看健康状况、单次运行、检包、诊断、资源目标与 Lab 录制的做法；没有 MCP 服务时退回命令行手册 `skills/actingcommand/references/cli-workshop.md`。安装时把整个 `skills/actingcommand` 目录复制或链接到 `~/.claude/skills/actingcommand`（Claude Code）和 `~/.agents/skills/actingcommand`（Codex）。这份 skill 按 Runtime v0.11.3 写，还没有包含 v0.11.5、v0.11.6 的变化；计划在 0.12 系列整体改写。

**已知问题**（在 Runtime v0.11.6 上核实，修复计划在 0.12 系列）：
1. 实例别名含大写字母时，`ac_pause` 与 `ac_resume` 回答 `client_action_invalid`，Lab 工具的操作也不留痕迹。请改用 `actingctl pause` 与 `actingctl resume`。
2. `ac_lab_observe`（以及 `actinglab observe`）的最小输出超过 2048 字节时直接报错，而不是截断。
3. 大量 Lab 操作之后，`ac_overview` 报 `incomplete`（`run_status_event_limit_exceeded`）；运行本身不受影响。

## 版本发布

每个版本在本仓发布为 `vX.Y.Z`，标记为预发布（pre-release）。各成员已解耦：每个成员（Runtime、UI、各标准包）各自从确切的提交发布，并且只在有改动时才发布；只要接口保持兼容，各成员就能继续搭配使用。从 v0.11.3 起，每次 Runtime 和 UI 构建都在自己的 `BUILD-MANIFEST.json` 里声明它读写的接口修订，acsetup 在改动任何东西之前，按这些声明核对它要安装的每一种组合。本仓的每个版本带上每个成员最新的发布，资产原样不动，因此各成员的版本号可以彼此不同，也可以与本仓的版本号不同；`MEMBERS.json` 记录每个成员自己的 tag。

本仓最新的版本 **v0.11.4** 带的是 Runtime v0.11.4、UI v0.11.3、明日方舟标准包 v0.11.4、碧蓝航线标准包 v0.11.3 和蔚蓝档案标准包 v0.11.4。此后 Runtime 又发布了 v0.11.5 和 v0.11.6，在 [Runtime 仓的 Releases 页面](https://github.com/HS7097/ActingCommand-Runtime/releases)上，还没有进本仓的版本，所以安装向导目前装的是 Runtime v0.11.4。

| 资产 | 内容 |
|---|---|
| `actingcommand-runtime-<sha>.zip` | Runtime：`actingcommand-actingd.exe`、`actingctl.exe`、配置模板、INSTALL.md、RELEASE-NOTES.md |
| `actingcommand-tools-<sha>.zip` | 同一次构建的工具：`actinglab.exe`、`actingledger.exe`、`actingcommand-vision-provider-check.exe`、从 v0.11.3 起的看门狗启动器 `actingwatch.exe`，以及 `platform-tools\` 下 Google 官方 Android platform-tools 37.0.1（adb）。到 Runtime v0.11.5 为止还带设备测试探针 `actingcommand-device-test.exe`，v0.11.6 起退役。从 v0.11.2 起不再带 OCR 提供者：OCR 在 Runtime 进程内运行 |
| `acui-windows-<sha>.zip` | 控制台 `acui.exe`、安装向导 `acsetup.exe` 和固定入口 `acforward.exe` |
| `<game>-resources-<sha7>.zip` | 一个游戏的标准包：该游戏的全部任务包（各以内容摘要为身份，装载前核对），加上程序与智能体所需的数据 |
| `MEMBERS.json`、`SHA256SUMS` | 各成员的提交与发布；每个 zip 以及 `MEMBERS.json` 的 SHA-256 |
| `acsetup.exe`、`acsetup-full-<tag>.exe` | 在线版、离线版安装向导，各带一个 `.sha256` |
| `install.ps1`、`install.sh` | 从 v0.11.2 起：一键安装脚本（见「安装」一节） |

发布之前，每个成员 zip 都会对照它自带的 `BUILD-MANIFEST.json`（仓库、提交、每个文件的大小和 SHA-256）核对一遍；离线版安装向导组装好后还会回读核对。本仓没有 CI，版本由人工组装和核对。

以前每日发布留下的 `build-r<Runtime>-u<UI>` 预发布仍保留在 Releases 页面上供参考；每日发布已经停用。

## 原则

- **程序本体保持中立。** Runtime、它的工具和 MCP 服务的代码、契约、默认配置、码表和测试里，不含任何游戏逻辑或游戏信息（游戏名、页面、角色、关卡、资源名、数值、规则）。Runtime 的 CI 里有守卫测试扫描保证。
- **游戏经资源包接入。** 一切游戏内容——图片、识别区域、点击框、任务顺序、数据表——只在资源包里：任务包按游戏汇成一个标准包，按摘要封印，装载前核对。
- **Fail Loud。** 错误必须看得见：明确的错误信息、非零退出码、账本里的 Error 或 Warning。拒绝也是一份回执，从不用沉默代替；配置无效、非回环地址、证据缺口都明确失败，不悄悄降级；「未知」从不当作「否」。
- **账本是权威记录。** 账本里写的就是 Runtime 里发生过的事；出了问题只凭账本就能追。
- **结果码。** 码目录与 CI 守卫从 v0.11.5 起就有，目前覆盖 Runtime 的一部分；把所有程序的结果都登记进去计划在 0.12 系列完成。
- **只在本机。** Runtime 只监听本机回环，没有网络代码。
- **成员解耦。** Runtime、UI 与各标准包各自发版；能否搭配看各自声明的接口版本。

## 路线图（计划中）

本节内容都还没有发布。同一系列内的先后次序还可能调整。

**0.12 系列**（以 Runtime 为主，部分随下一版 UI）：

| 主题 | 计划 |
|---|---|
| 结果码统一 | 所有程序共用一张登记过的码目录，每个码有类别；每条命令输出一行结果，退出码收敛为 0/1/2；错误按码解释（中英）。 |
| 账本做观测者 | 实时状态放进两块内存库（调度器工作台、实例基础信息库）；账本经「探针」——数据必经之路上的静默闸门——记录一切，仍是唯一的权威记录。写入失败先重试，用尽则转储并明确停机，下次启动导入；状态查询不再写账本，启动不再重放整本账本。 |
| 新账本 | 0.12.0 起用新的账本格式（修订 3），从空账本开始；0.11 的状态根不带过去（旧安装整体留作归档，不删）。 |
| 统一接口 | 一道门、三个前端（命令行、MCP、UI）一一对应。一般接口总在（查询、暂停与恢复、监视、优先级偏移、可排包状态、请求关闭、供安装器用的升级态、实例目标与资源目标）。每次请求只记来自哪个前端，不再按调用者设权限，MCP 档位取消。 |
| Lab 改为可拆的调试模块 | 默认不装。安装器新增一页，有两个相互独立的选项，勾任一个就装 Lab：**创作**（给用智能体制作资源的人：录制与制包工具）、**调试**（给高级用户：直接操控 Runtime 的内部动作）。不装时 Lab 通道仍在，但不可调用。 |
| 恢复阶梯配置化 | 触发条件（按结果码类别）、次数与时间窗、级序、每级时限、冷却与抑制规则全部进配置，随发布带默认值；改配置文件下次启动生效，或经命令行、MCP、UI 的一条指令立即生效并记账。新增两类触发：短时间内多次稳定性警告，以及不符合包逻辑的行为。 |
| 性能节奏 | 宿主卡顿时不停派单，而是拉长步与步之间、步内动作之间的间隔；持续测 CPU、GPU 与响应率，只有响应率持续下降才暂停调度，恢复后自动续跑。先只做测量，看过实测数据再开控制。 |
| 调度规则 | 实例装了哪个标准包，就按规则调度其中全部任务，**每个没跑的任务都写明原因**，在状态与 MCP 里看得到。时段用黑名单，进黑名单前渐进静默；任务包可挂起；功能未解锁作为数据声明。 |
| 数据刷新 | 实例数据未知或过期时不再整轮失败；由普通的读数包去相应页面读取、刷新。 |
| 选择与共享数据表 | 标准包里的共享数据表（按摘要引用、按键取值）、一步多选、候选布局与跨帧追踪；Runtime 只提供通用机制，表的内容在包里。 |
| 每实例目标 | 每个实例一份目标、一个入口，两种写法可单用可并用：**任务权重**（让某个任务多跑以拿特定收益）与**特殊目标**（例如某种资源「攒到 X」）；经一般接口写入、记进账本；目标优先于标准包默认。 |
| MCP 增补 | 日常读（带筛选与长轮询的事件、运行列表、实例数据、截图导出、单次运行证据、全实例诊断、分节的总览）、查码、包列表、「为什么没跑 / 下次何时到点」、请调度器排一次、读写实例目标；智能体收件箱与简报；修好上面的已知问题。命令行是正本，MCP 与之一一对应。 |
| 自动起模拟器 | 一个配置键，默认关。 |
| 分辨率支持（0.12 系列末段） | 任意 16:9 横屏与 9:16 竖屏的画面统一换算到包声明的基准坐标系（今天是 1280×720），短边不低于 720；其他比例明确拒绝并写明原因。 |
| 下一版 UI | 安装器新增 Lab 两个选项的页面、按码解释报错（中英）、监控台写明任务为什么没跑、读新账本、安装器机读输出（`--json`）；计划为 UI 的大版本号升级。 |

**0.13 系列：**

| 主题 | 计划 |
|---|---|
| 战斗层 | 通用的战斗流程件：读取结算、执行走格子，以及公开的数据格式；各游戏的战斗内容都在各自的包里。0.13.0 先放共用底座，首个使用者是明日方舟标准包的自动战斗。 |
| 通用空间组件（净室重写） | 多指按住的手势输入与灵敏度、延迟标定，小地图定位，路线图找路，闭环行走与转视角。按公开原理净室重写，不用任何第三方代码、底图或模型，不拆包；做成与颜色匹配、颜色摘要、OCR 同列的进程内后端。地图、路线图、操控布局都是包数据；定位与行走在 0.13.x 期间到来。 |
| MaaFramework 流水线格式导入导出 | 「包视图」：我们的任务包与 MaaFramework 形状的流水线 JSON 互转；转回的包要过检包才保存，过不了就报错、不写。跟随最新格式，格式有变化就报出。编辑用我们自己的 UI（节点图编辑器进下一代制作台），发行版不带第三方编辑器。 |

**0.12 系列期间的破坏性变化：** 新账本，0.11 的状态根不带过去；命令行的输出与退出码变化；MCP 档位取消；Lab 默认不装。当前的安装向导与监控台（UI v0.11.3）预计用不了 0.12.0 的 Runtime，要随下一版 UI 才支持。
