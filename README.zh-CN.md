<p align="right">🌐 <a href="./README.md">English</a> · <b>简体中文</b></p>

<div align="center">

<img src="docs/assets/readme/actingcommand-icon.png" width="112" alt="ActingCommand 图标">

**首席执行官 兼 董事长** — HS7097<br/>
**首席技术官 兼 首席架构师** — Claude Opus 5.5 · GPT‑6 Astra · Claude Fable 5.1 · Claude Fable 5 · GPT‑5.5<br/>
**董事会秘书 兼 首席审计官** — Claude Opus 5.5 · Claude Fable 5.1 · Claude Fable 5 · Claude Opus 4.8<br/>
**首席技术工程师** — Claude Opus 5.5 · GPT‑6 Astra · GPT‑5.6 Sol · GPT‑5.5<br/>
**运维总监** — GPT‑6 Astra<br/>
**运维主管** — GPT‑5.6 Sol Daybreak · DeepSeek V4.1 Flash 1017

</div>

**⚠️ 预发布、调试阶段：所有发布都是预发布，接口、配置与文件格式还会变化；只支持最新版。**

# ActingCommand

- [ActingCommand-Runtime](https://github.com/HS7097/ActingCommand-Runtime) — Rust 常驻运行时，核心程序，含命令行、工具与本地 MCP 服务
- [ActingCommand-UI](https://github.com/HS7097/ActingCommand-UI) — 安装向导与只读控制台
- **明日方舟**、**碧蓝航线**、**蔚蓝档案**三个标准包 — 在不公开的单独仓库里制作；已公开的标准包作为本仓 [Releases](https://github.com/HS7097/ActingCommand/releases) 的资产发布

## 本仓

本仓是 ActingCommand 项目族的伞仓（门面页），不承载代码；代码在上面的各成员仓。本仓放 README、`skills/` 下的程序 skill，以及 Releases 里按版本号发布的版本：每个版本包含安装向导，以及各成员为该版本发布的原样资产。`bundles/` 保留以前每日发布用过的蔚蓝档案资源包，仅作参考、不再更新；`docs/` 放 README 用的图片和较早的笔记。

**我们能做什么：** 让智能体部署我们的程序，或者自己动手安装；在 Harness 里加载程序 skill、注册 MCP 服务之后（见[配合智能体使用](#配合智能体使用mcp)），你就可以让智能体为运行在安卓模拟器上的程序制作所需的素材，然后定期重复运行。

**智能体需要做什么：** 制作一些图片和点击区域。

**我们怎么做的：** Runtime 是一个常驻的 Rust 程序，本身不含任何游戏逻辑。它通过 ADB（以及模拟器厂商提供的接口）连上安卓模拟器，按节拍截帧，在帧上识别素材（模板、颜色、OCR、神经网络），在事先声明的区域内点击，并把每一步——看到了什么、做了什么、结果如何——作为类型化事件写进一本只追加的账本。账本是权威记录：调度器按它决定下一次运行，控制台只读它，出了问题也只从它溯源。游戏相关的一切——图片、点击区域、任务顺序、数据表——都放在按游戏分开的标准包里，其中每个任务包在装载前按它声明的结构准入；Runtime 只装载、核对、执行。换游戏换包，程序不动。

**欢迎参与：** 如果您有更好的想法，或者在使用中遇到了什么问题，可以在本仓或对应的仓创建 issue。

## 开发状态

- **调试阶段。** 主循环是：发现问题 → 修复 → 对照预期检查 → 再改。部署与易用性是次要的。
- **只有预发布。** 每个成员的每次发布都是预发布。只支持最新版，旧系列不出修复。
- **各成员现状。** Runtime 已进入 0.12 系列，最新是 v0.12.10。UI 最新是 v0.11.4；0.12 系列没有对应的 UI 版本，v0.11.4 的安装向导和控制台都不能用于 0.12 的 Runtime（见[安装](#安装)）。本仓最新的版本仍是 v0.11.4，带的是 Runtime v0.11.4（见[版本发布](#版本发布)）。
- **Runtime 0.12 带来的变化。** 新账本：0.12 的 Runtime 从空的状态根开始，打不开 0.11 的账本，0.11 的程序也读不了 0.12 的账本。新的包格式：0.11 的标准包在 0.12 里装载不了。命令行和看门狗都以登记过的结果码作答，按结果的类别以 0、1 或 2 退出；MCP 工具报错也带同样的码。Runtime v0.12.10 是第一个账本修订 4 的版本，从它退回更早的 Runtime 不受支持。
- **实机现状。** 项目自己的机器上，0.12 系列的 Runtime 在五台 MuMu 实例上全天运行。三个标准包每天跑例行批次；另有两台实例接入了少女前线2，它的标准包正在制作，未公开。标准包还没有覆盖全部任务。
- **版本号规则。** X 为重大或不兼容的变化，Y 为新特性或新覆盖面，Z 为修复（含为修复服务的特性）。各成员各自发版；能否搭配看各自声明的接口版本，不看版本号是否一致。

## 现在能用的

Runtime 各行按 Runtime v0.12.10 写；控制台与安装向导两行按 UI v0.11.4 写。

| 部分 | 做什么 |
|---|---|
| Runtime（`actingcommand-actingd.exe`） | 常驻进程，只在本机回环地址（127.0.0.1 / [::1]）上收请求，没有网络代码；客户端关掉不影响它运行。调度器：按已批准的目录运行任务包，可按时钟槽，也可按条件（例如从实例事实里读到的时刻）；每个实例一个请求队列、一个工作线程，租约与围栏在每次触碰设备时复核；暂停与恢复（全局或单个实例，熬过重启）、优先级偏移、资源目标、磁盘容量准入。每个没跑的任务都写明原因。例行运行失败后，由恢复阶梯接手这台实例，阶梯的触发条件和各级都写在配置里（回主页、重启应用、重启模拟器）；手动发起的运行不走阶梯。一台实例出故障（截图、应用控制、一次观察）只影响这一台。 |
| 识别与设备 | 八种识别目标：模板、颜色、仅点击、OCR、神经网络、颜色摘要、组合（由 2 到 8 个其他目标组成）、数值检验（比较读到的值）；OCR 与神经网络在进程内用 ONNX Runtime 推理，模型第一次用到才加载。MuMu：Nemu IPC 截图与输入，经 MuMuManager 发现实例、启停模拟器，可选由 Runtime 自动开启实例的模拟器。USB 连接的安卓真机也可以绑定为实例。自带 adb 37.0.1。在框里点击时，落点集中在框的中部，并在几个候选点之间轮换。 |
| 账本 | 单写者的 SQLite 账本，记的是脱敏后的类型化事件，每个结果都带登记过的结果码；截图按内容摘要存放。截图清理器删去不再需要的截图，并把为错误与 Lab 保留的截图移进 `<state root>\kept\<date>\`，由人删除。 |
| 命令行与工具 | `actingctl`（状态与总览、事件、运行列表与运行证据、「为什么没跑」、暂停与恢复、模拟器控制、设置、设备、请求关闭、看门狗、MCP）；`actinglab` 从实机画面录制 `linear_steps` 任务包，检查任务包、标准包与资源仓；`actingledger` 离线读账本；看门狗启动器 `actingwatch.exe`。 |
| MCP 服务 | `actingctl mcp-serve`：给智能体用的 38 个工具，见[配合智能体使用](#配合智能体使用mcp)。 |
| 控制台（`acui.exe`） | 只读账本的原生控制台：实例卡、时间线、详情；六个页签对应账本的六个视图；可离线直接读状态根，也可经 Runtime 在线读。顶栏启动器可拉起 Runtime 或请求它关闭，「实例配置」窗口可编辑实例。只能用于 0.11 的 Runtime。 |
| 安装向导（`acsetup`） | 安装本仓的版本：A/B 安装与升级，改动之前逐个文件核对；也有命令行（见[安装](#安装)）。它是整个项目族里唯一有网络代码的程序。它拒绝安装 0.12 的 Runtime。 |
| 标准包 | 明日方舟、碧蓝航线、蔚蓝档案：各游戏的全部任务包，加上程序与智能体所需的数据。本仓（v0.11.4）里公开的是 0.11 的标准包；0.12 系列的标准包未公开。标准包都按 1280×720 的画面制作；截图尺寸不符时任务明确失败。 |

## 安装

**Runtime 0.12 系列。** 目前发布的安装向导都装不了它：UI v0.11.4 的安装向导与它声明的接口对不上，会在改动任何东西之前以退出码 6 拒绝。0.12 的 Runtime 要手动铺在一个空的状态根上：从 [Runtime 的 Releases](https://github.com/HS7097/ActingCommand-Runtime/releases) 取 Runtime zip、Tools zip 和 `SHA256SUMS`，核对后把 Runtime zip 解到 `<root>\runtime\`，把 Tools zip 解到 `<root>\tools\`，再按模板写好私有配置 `<root>\actingd.config.json`。配置、自带 adb、启动 / 查看 / 关闭以及看门狗，见 Runtime 的 [INSTALL.md](https://github.com/HS7097/ActingCommand-Runtime/blob/main/distribution/windows/INSTALL.md)。0.12 系列内升级时替换 `runtime\` 和 `tools\`，配置和账本不动。0.12 系列的标准包未公开。

**本仓 v0.11.4（0.11 系列），用安装向导安装。** 需要 Windows；游戏实例运行在 MuMu 安卓模拟器上，分辨率 1280×720。按当前用户安装，不需要管理员权限。

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

**命令行：** acsetup 也可以不开窗口运行：

```
acsetup --root <abs path> (--plan|--yes) [--conflicts new|old] [--associate <alias>=<bundle>/<server>] [--allow-downgrade] [--online|--from <folder>]
```

`--plan` 列出全部改动和差异，安装根目录下不改任何文件；`--yes` 执行安装或升级。`--associate` 可以重复，每个实例别名一次。退出码：0 完成，1 失败，2 用法错误，3 维护绑定有差异而未给 `--conflicts`，4 降级而未给 `--allow-downgrade`，5 有资源关联要选择而未给 `--associate`，6 接口不兼容，即 Runtime、UI、acsetup 与标准包声明的接口修订对不上；2 到 6 都停在安装改动之前。`acsetup --root <abs path> --resources <标准包 zip> (--plan|--yes)` 把一个标准包放进已有的 A/B 安装，既不换程序，也不切槽。离线版 `acsetup-full-<tag>.exe` 只用自带的版本；在线版 `acsetup.exe` 需要 `--online` 或 `--from <folder>`。`acsetup --help` 打印完整用法。acsetup 是窗口程序，交互式控制台的提示符不会等它结束；下面的一键脚本会等它结束，并原样传出它的退出码。

**一键脚本：** 每个版本还附带两个脚本，它们只是发布资产，不是本仓里的文件：
- `install.ps1`：从同一个版本下载 `acsetup-full-<tag>.exe` 及其 `.sha256`，核对 SHA-256，再以命令行模式运行 acsetup，打印并原样传出它的退出码。脚本自己的退出码是 10（下载失败）和 11（SHA-256 不符）。先运行 `powershell -NoProfile -ExecutionPolicy Bypass -File install.ps1 -Root <install root> --plan`，再把 `--plan` 换成 `--yes` 运行一次。
- `install.sh`（Git Bash）：取该版本的 `install.ps1`，按 install.sh 里记录的 SHA-256 核对，再用同样的参数运行它，例如 `bash install.sh -Root <install root> --plan`。

**Runtime 看门狗：** 从 Runtime v0.11.3 起，Runtime 没经过正式关闭就结束时（崩溃、窗口被关、重启电脑），可以自动再拉起来；0.12 系列里，上一次启动以错误结束时也会拉起。它不会自动启用：请在 Runtime 所属用户的普通（非管理员）PowerShell 里运行一次 `<root>\runtime\actingctl.exe watchdog install --root <root>`。它注册一个按用户运行的计划任务，每分钟运行 `<root>\tools\actingwatch.exe`。看门狗从不拉起正式关闭的 Runtime；上一次启动以致命结局收场时保持停止；30 分钟内最多拉起 3 次。它看管 A/B 安装，从 Runtime v0.12.1 起也看管上面那样手动铺开的根。`watchdog status --root <root>` 只报告、不改动任何东西，`watchdog uninstall --root <root>` 删除该任务（移除安装之前先运行它），记录在 `<root>\watchdog\watchdog.log`。详见 Runtime 的 INSTALL.md「Runtime watchdog」一节。

这些也可以交给智能体来做，程序 skill 写了怎么做。安装向导的完整说明（包括升级、离线版、安装日志）见 UI 仓 README 的 [Setup wizard acsetup](https://github.com/HS7097/ActingCommand-UI#setup-wizard-acsetup) 一节。

## 配合智能体使用（MCP）

`actingctl mcp-serve` 是一个走 stdio 的本地 MCP 服务（从 Runtime v0.11.0 起）。它只提供工具（不提供 resources、prompts），只和同一台机器上的 Runtime 通信，Runtime 的回答与拒绝原样传出。

**注册。** `mcp-config` 打印给客户端用的注册内容，自己不写任何文件：

```
<root>\runtime\actingctl.exe mcp-config --client claude [--tier observer,operator,author]
<root>\runtime\actingctl.exe mcp-config --client codex  [--tier observer,operator,author]
```

- Claude Code：运行打印出来的命令，`claude mcp add --scope user actingcommand -- "<root>\runtime\actingctl.exe" mcp-serve --tier …`。
- Codex：把打印出来的 `[mcp_servers.actingcommand]` 一段加进 Codex 的 `config.toml`。

在 A/B 安装上打印出的程序是固定入口 `<root>\runtime\actingctl.exe`，切槽之后注册照样有效。注册后新开一个客户端会话才会加载这个服务。

**档位。** `observer` 总是开着，也是默认；除了读取，它还含 Runtime 运行中可改的设置、手机设备绑定、本地能力缺口记录和实例的补救选择。`operator` 加上设备、应用与调度，`author` 加上 Lab 录制。不在已开档位里的工具回答 `tier_not_enabled`；要换档位，用别的 `--tier` 重新注册。Runtime v0.12.10 有下面 38 个工具：

| 档位 | 工具 |
|---|---|
| observer（23 个） | `ac_overview`、`ac_events`、`ac_material`、`ac_observe`、`ac_get_run`、`ac_diagnose`、`ac_runs`、`ac_run_evidence`、`ac_resources_list`、`ac_targets_get`、`ac_config_set`、`ac_packs`、`ac_facts`、`ac_outcome_code`、`ac_devices`、`ac_device_bind`、`ac_device_unbind`、`ac_capability_gap`、`ac_remedy_set`、`ac_remedy_get`、`ac_schedule`、`ac_pack_check`、`ac_catalog_check` |
| operator（8 个） | `ac_run_pack`、`ac_stop_run`、`ac_pause`、`ac_resume`、`ac_emulator`、`ac_app`、`ac_ladder_drill`、`ac_targets_set` |
| author（7 个） | `ac_lab_observe`、`ac_lab_do`、`ac_record_start`、`ac_record_mark`、`ac_record_stop`、`ac_record_status`、`ac_binding_draft` |

`actingctl mcp-serve --list-tools --format markdown` 打印完整的工具表，含每个工具的参数。工具报错是 `{class, outcome}`，`mcp-serve --detail codes|full` 决定每个 outcome 带多少细节。

**行为约定。** 先调 `ac_overview`。长操作立即返回句柄，用 `ac_get_run`（带 `wait_s`）轮询。运行的句柄就是 Runtime 的请求 id，跨重启有效。结果不确定时先问 `ac_get_run`，不要重发写操作。批准、改 Runtime 配置文件、重启 Runtime 都归人来做；Runtime 运行中可改的设置用 `ac_config_set` 改，每次改动都有记录。

**程序 skill。** [`skills/actingcommand/SKILL.md`](skills/actingcommand/SKILL.md) 是智能体使用这些工具的手册，含查看健康状况、单次运行、检包、诊断、资源目标与 Lab 录制的做法；没有 MCP 服务时退回命令行手册 `skills/actingcommand/references/cli-workshop.md`。安装时把整个 `skills/actingcommand` 目录复制或链接到 `~/.claude/skills/actingcommand`（Claude Code）和 `~/.agents/skills/actingcommand`（Codex）。这份 skill 按 Runtime v0.11.3 及其 22 个工具写，没有包含 Runtime v0.11.5 及以后的变化，0.12 系列也不在内。用 0.12 的 Runtime 时，以 `mcp-serve --list-tools` 打印的工具说明为准。

**0.11 系列的已知问题**（都已在 0.12 系列修复）：
1. 实例别名含大写字母时，`ac_pause` 与 `ac_resume` 回答 `client_action_invalid`，Lab 工具的操作也不留痕迹。请改用 `actingctl pause` 与 `actingctl resume`。v0.12.0 修复。
2. `ac_lab_observe`（以及 `actinglab observe`）的最小输出超过 2048 字节时直接报错，而不是截断。v0.12.0 修复。
3. 大量 Lab 操作之后，`ac_overview` 报 `incomplete`（`run_status_event_limit_exceeded`）；运行本身不受影响。v0.12.1 修复。

## 版本发布

每个版本在本仓发布为 `vX.Y.Z`，标记为预发布（pre-release）。各成员已解耦：每个成员（Runtime、UI、各标准包）各自从确切的提交发布，并且只在有改动时才发布；只要接口保持兼容，各成员就能继续搭配使用。每次 Runtime 和 UI 构建都在自己的 `BUILD-MANIFEST.json` 里声明它读写的接口修订，acsetup 在改动任何东西之前，按这些声明核对它要安装的每一种组合。本仓的每个版本带上每个成员最新的发布，资产原样不动，因此各成员的版本号可以彼此不同，也可以与本仓的版本号不同；`MEMBERS.json` 记录每个成员自己的 tag。

本仓最新的版本 **v0.11.4** 带的是 Runtime v0.11.4、UI v0.11.3、明日方舟标准包 v0.11.4、碧蓝航线标准包 v0.11.3 和蔚蓝档案标准包 v0.11.4。此后发布、还没有进本仓版本的有：[Runtime 仓 Releases 页面](https://github.com/HS7097/ActingCommand-Runtime/releases)上的 Runtime v0.11.5、v0.11.6 以及 0.12 系列（到 v0.12.10），[UI 仓 Releases 页面](https://github.com/HS7097/ActingCommand-UI/releases)上的 UI v0.11.4。所以安装向导装的是 Runtime v0.11.4 和 UI v0.11.3。0.12 的 Runtime 与 0.11 的 UI、0.11 的标准包搭配不上。

| 资产 | 内容 |
|---|---|
| `actingcommand-runtime-<sha>.zip` | Runtime：`actingcommand-actingd.exe`、`actingctl.exe`、配置模板、INSTALL.md、RELEASE-NOTES.md |
| `actingcommand-tools-<sha>.zip` | 同一次构建的工具：`actinglab.exe`、`actingledger.exe`、`actingcommand-vision-provider-check.exe`、看门狗启动器 `actingwatch.exe`，以及 `platform-tools\` 下 Google 官方 Android platform-tools 37.0.1（adb）。到 Runtime v0.11.5 为止还带设备测试探针 `actingcommand-device-test.exe`，v0.11.6 起退役。OCR 在 Runtime 进程内运行，所以不带 OCR 提供者 |
| `acui-windows-<sha>.zip` | 控制台 `acui.exe`、安装向导 `acsetup.exe` 和固定入口 `acforward.exe` |
| `<game>-resources-<sha7>.zip` | 一个游戏的标准包：该游戏的全部任务包（各自在装载前核对），加上程序与智能体所需的数据 |
| `MEMBERS.json`、`SHA256SUMS` | 各成员的提交与发布；每个 zip 以及 `MEMBERS.json` 的 SHA-256 |
| `acsetup.exe`、`acsetup-full-<tag>.exe` | 在线版、离线版安装向导，各带一个 `.sha256` |
| `install.ps1`、`install.sh` | 一键安装脚本（见「安装」一节） |

发布之前，每个成员 zip 都会对照它自带的 `BUILD-MANIFEST.json`（仓库、提交、每个文件的大小和 SHA-256）核对一遍；离线版安装向导组装好后还会回读核对。本仓没有 CI，版本由人工组装和核对。

以前每日发布留下的 `build-r<Runtime>-u<UI>` 预发布仍保留在 Releases 页面上供参考；每日发布已经停用。

## 原则

- **程序本体保持中立。** Runtime、它的工具和 MCP 服务的代码、契约、默认配置、码表和测试里，不含任何游戏逻辑或游戏信息（游戏名、页面、角色、关卡、资源名、数值、规则）。Runtime 的 CI 里有守卫测试扫描保证。
- **游戏经资源包接入。** 一切游戏内容——图片、识别区域、点击框、任务顺序、数据表——只在资源包里：任务包按游戏汇成一个标准包，装载前核对。
- **Fail Loud。** 错误必须看得见：明确的错误信息、非零退出码、账本里的 Error 或 Warning。拒绝也是一份回执，从不用沉默代替；配置无效、非回环地址、证据缺口都明确失败，不悄悄降级；「未知」从不当作「否」。
- **账本是权威记录。** 账本里写的就是 Runtime 里发生过的事；出了问题只凭账本就能追。
- **结果码。** Runtime、它的命令行、看门狗和 MCP 服务的每个结果都带一个登记在同一份码目录里的码，由 Runtime CI 里的守卫把关；每个 Runtime 版本都带一份码目录快照。Lab 自己的工具还在这道守卫之外。
- **只在本机。** Runtime 只监听本机回环，没有网络代码。
- **成员解耦。** Runtime、UI 与各标准包各自发版；能否搭配看各自声明的接口版本。
