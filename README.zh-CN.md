<p align="right">🌐 <a href="./README.md">English</a> · <b>简体中文</b></p>

<div align="center">

<img src="docs/assets/readme/actingcommand-icon.png" width="112" alt="ActingCommand 图标">

<h1>ActingCommand</h1>

<p><b>面向多款二次元手游的本地常驻自动化调度、视觉执行与智能体协同系统</b></p>

**首席执行官 兼 董事长** — HS7097<br/>
**首席技术官 兼 首席架构师** — Claude Opus 5.5 · GPT‑6 Astra · Claude Fable 5.2 · Claude Fable 5.1 · Claude Fable 5 · GPT‑5.5<br/>
**董事会秘书 兼 首席审计官** — Claude Opus 5.5 · Claude Fable 5.2 · Claude Fable 5.1 · Claude Fable 5 · Claude Opus 4.8<br/>
**首席技术工程师** — GPT‑6 Astra · Claude Opus 5.5 · GPT‑5.6 Sol · GPT‑5.5<br/>
**运维总监** — GPT‑6 Astra<br/>
**运维主管** — GPT‑5.6 Sol Daybreak · DeepSeek V4.1 Flash 1017<br/>
**业务运营组长** — Gemini‑4 Carbon · Gemini‑4 Argon

</div>

> [!IMPORTANT]
> **当前处于 `0.12` 收敛预览系列（正在迈向 `v1.00` 正式版）**：接口、配置与数据包格式仍在随实机验证快速收口；现阶段仅维护最新版本，不向旧系列回迁修复。

---

**ActingCommand** 是一套运行在 Windows 本机的多游戏自动化统筹系统：日常挂机 **100% 由本地 Rust 常驻内核确定性执行（零大模型 Token 消耗、不含任何联网代码）**，按净收益与资源红线自动管理多台安卓模拟器或真机；同时原生提供录制工坊与本地 MCP 服务，让 AI 智能体在你需要时协助制作任务包、查阅权威账本与诊断异常。

---

## 1. 我们现在能做什么

以下能力均已在当前最新发布（[`Runtime v0.12.54`](https://github.com/HS7097/ActingCommand-Runtime/releases/tag/v0.12.54) · [`UI v0.12.15`](https://github.com/HS7097/ActingCommand-UI/releases/tag/v0.12.15)）中落地可用：

- **多实例本地常驻与无人值守（`actingd`）**
  - 常驻后台进程独立运行，关闭控制台或命令行不影响后台调度；支持 `actingwatch` 用户级看门狗在崩溃或重启后自动拉起。
  - 原生对接 **MuMu 模拟器**（高速 Nemu IPC 截帧与触控、经 MuMuManager 自动发现实例与启停模拟器）及 **USB 安卓真机**（自带官方 `adb 37.0.1`），支持运行中热切换触控与截图后端。
  - 实例级严格故障隔离：单台卡死不影响其余实例；一段时间无待办任务时自动关闭游戏或停掉模拟器释放内存，下一轮任务前自动唤醒。
- **按净收益与双水位红线算账的智能调度**
  - 支持按时钟槽与实例实时事实（如从画面读出的体力回满时刻、冷却完成时刻）双重触发。
  - 业务任务按 **「单次净收益 ÷ 预计运行时长」** 动态排定优先级，净收益未知或非正的任务绝不盲派；资源池支持设置 **停止线** 与 **硬红线**，防止耗尽留作活动或抽卡的资源储备。
  - 支持全局/单实例持久化暂停与恢复、优先级偏移、资源目标与磁盘容量准入；**每一个未执行的任务都会给出具体原因**。
- **混合确定性视觉内核 + 分级自愈阶梯**
  - 内置 8 种识别目标：模板匹配、颜色、颜色摘要、仅点击、OCR、ONNX 神经网络、组合条件（2–8 项组合）与数值比对，另支持多尺度带遮罩的锚点多匹配检索；OCR 与神经网络在进程内按需加载运行，OCR 支持经 DirectML 调用 GPU 加速；点击落点在目标框中心候选点间自然轮换。
  - 例行任务失败后自动由配置化的 **恢复阶梯**（回主页 → 重启游戏应用 → 重启模拟器）接手自救，手动触发的调试运行不误触阶梯。
- **单写者权威账本与原生桌面监控台（`acui`）**
  - 每一步“看到了什么、点了哪里、返回什么登记结果码”均脱敏写入本地 SQLite 账本，截图按内容摘要去重存储，自动清理过期帧并将错误/录制关键帧归档至 `kept/` 目录。
  - 原生桌面监控台 `acui` 支持在线（经 Runtime）或离线（直读状态目录）查看账本六大视图、查看账号与看门狗状态、启停内核以及热切换各设备后端。
- **中立游戏经济建模工具（`acmodel`）**
  - 独立离线的 Python 3.13 建模与求解器，内置 LP/MILP 经济规划、精确抽卡保底链、样本掉率置信界与灵敏度推导图谱，可导出资源价格、红线阈值与任务评分并绑定至调度配置。
- **录包工坊与 AI 智能体接口（`actinglab` + 41 个本地 MCP 工具）**
  - `actinglab` 支持从实机实时画面录制 `linear_steps` 任务包、静态检查标准包结构、日常短命令巡检以及一键部署/回退救急。
  - `actingctl mcp-serve` 通过本地 stdio 向智能体暴露按权限分档的 **41 个工具**，让 AI 安全协助查看健康总览、单步观察、录制新包与溯源排障。

---

## 2. 我们正在实现什么

在迈向结束预览的 **`v1.00` 正式版** 之前，团队正在 `0.12.z` 系列内集中推进以下能力：

- **四款主力游戏的 `0.12` 标准包全覆盖**：**《明日方舟》《碧蓝航线》《蔚蓝档案》《少女前线2：追放》** 已在项目自有实机环境（5 个实例）持续联调，正在补齐全部日常、养成刷取、常驻活动任务包以及调度器所需的完整价值表。
- **单设备 / 单模拟器实例内的多游戏轮换调度**：突破“一实例绑死一款游戏”的限制，支持在同一部手机或同一个模拟器实例内挂载多款游戏（含安卓多用户应用分身），按跨游戏紧迫度单队列排队轮换前台、切走自动清理后台。
- **监控台（`acui`）玩家操控闭环与工具双入口收口**：在 `acui` 中补齐实例可视化增删配置、日常开关与养成目标设定；同时将命令行与 MCP 服务统一收口至 `actinglab`，形成 **「日常玩家用 `acui`、开发与智能体用 `actinglab`」** 的清晰双入口。
- **各游戏专属估值引擎联动与异常精准唤醒**：将各游戏角色/装备估值模型与刷取目标打通；完善静默卡死监测、常见结果码速查索引与智能体唤醒规则，仅在恢复阶梯无法自愈时才唤醒 AI 或提醒用户。
- **新一代图形安装向导（`acsetup`）与成套发布回归**：待 `0.12` 接口与账本格式彻底定型后，恢复开箱即用的一键安装/升级向导与本伞仓的成套版本发布。

---

## 3. 我们怎么做的

```mermaid
flowchart LR
    Pack["游戏标准包 (1280x720 素材 / 点击框 / 排程 / 价值表)"] --> RT["Rust 常驻内核 actingd (零游戏硬编码 / 净收益调度 / ONNX 视觉)"]
    RT <--> Dev["MuMu 模拟器 / USB 安卓真机"]
    RT --> Ledger[("SQLite 权威账本 (类型化事件 / 结果码 / 证据帧)")]
    Ledger --> UI["原生监控台 acui"]
    Ledger <--> Lab["actinglab 与本地 MCP (41 个智能体工具)"]
```

1. **内核绝对中立，换游戏只换包**：`Runtime`、`UI`、命令行与 MCP 的源码、契约、配置模板及测试中 **不含任何游戏逻辑或游戏信息**（无游戏名、角色名、关卡名或数值规则，由 CI 守卫自动扫描拦截）。一切游戏相关内容均封装在独立标准包内，装载前先做结构准入校验。
2. **账本是唯一事实来源**：调度器依据账本事实决定下一轮运行，监控台只读账本视图，任何异常仅凭账本与证据帧即可完整复盘。
3. **Fail Loud（绝不静默吞错）**：所有运行结果均携带统一码表登记的结果码与明确退出码；配置不合、画面尺寸不符（标准包统一按 `1280×720` 制作）、证据缺失均立即显式报错并保留现场，绝不“瞎点蒙混”或静默降级。
4. **纯本机零外网**：`actingd` 仅监听本机回环地址（`127.0.0.1` / `[::1]`），无任何外网通信代码，Runtime 自身不向外发送任何数据；接入的智能体经 MCP 读到的内容，去向由该智能体决定。
5. **成员解耦与接口契约校验**：`Runtime`、`UI` 与各游戏标准包独立按需发版；组件间能否搭配不看表面版本号是否相同，而是由各自构建清单（`BUILD-MANIFEST.json`）声明的接口修订号严格核对。

---

## 4. 用户需要做什么

### 基础环境准备
- **操作系统**：Windows（在 Windows 11 上开发与实测；普通用户权限即可，无需管理员权限）。
- **运行设备**：**MuMu 安卓模拟器**（实例分辨率设为 **`1280×720`**），或开启 USB 调试的安卓真机。
- **可选组件**：使用 OCR 或神经网络识别目标需放置 ONNX Runtime 与对应视觉模型；使用 `acmodel` 经济建模工具需安装 Python 3.13。

### 按使用场景选择上手路径

- **如果你是普通玩家（希望开箱即用托管日常）**
  - 当前 `0.12` 系列启用了全新的账本与新标准包格式，**`0.12` 系列的游戏标准包暂未公开分发，图形安装向导 `acsetup` 也暂未随 `0.12` 打包**。建议先关注本仓，等待四游标准包与安装向导在 `v1.00` 合拢发布后再一键安装。
  - *(历史存档说明：本仓 [Releases](https://github.com/HS7097/ActingCommand/releases) 页保留的 `v0.11.4` 附带旧版安装向导与前三款游戏的 `0.11` 标准包，仅供旧架构参考，与 `0.12` 互不兼容且不再维护。)*

- **如果你是开发者或自制包玩家（体验最新 `0.12` 内核与 AI 辅助制包）**
  1. **首次手动铺设 `0.12` 根目录**：从 [`ActingCommand-Runtime` Releases](https://github.com/HS7097/ActingCommand-Runtime/releases) 下载最新的 `actingcommand-runtime-<sha>.zip`、`actingcommand-tools-<sha>.zip` 与 `SHA256SUMS`，校验后分别解压至空目录 `<root>\runtime\` 与 `<root>\tools\`，并按模板编写 `<root>\actingd.config.json`（含 DirectML GPU OCR、自带 `adb`、启动与关闭说明详见 Runtime 仓的 [`INSTALL.md`](https://github.com/HS7097/ActingCommand-Runtime/blob/v0.12.54/distribution/windows/INSTALL.md)）。如需图形监控台，从 [`ActingCommand-UI` Releases](https://github.com/HS7097/ActingCommand-UI/releases) 下载 `acui-windows-<sha>.zip`。
  2. **接管后续更新与开启看门狗**：首次铺好后，运行 `<root>\runtime\actingctl.exe install adopt` 接管该根目录，后续即可用 `actingctl install`（`release stage` / `update plan` / `update apply` / `rollback`）就地升级或回退（跨单向账本修订升级时附加 `--accept-ledger-one-way`）；在普通 PowerShell 中运行一次 `<root>\runtime\actingctl.exe watchdog install --root <root>` 即可启用崩溃自动拉起看门狗。
  3. **（可选）注册本地 MCP 服务配合智能体使用**：运行以下命令打印稳定入口注册配置（不改写文件），按提示添加到你的智能体客户端（支持 `claude` / `codex` / `antigravity` / `dsh`）后重开会话即可：
     ```powershell
     <root>\runtime\actingctl.exe mcp-config --client claude --entry --root <root> --tier observer,operator,author
     <root>\runtime\actingctl.exe mcp-config --client codex  --entry --root <root> --tier observer,operator,author
     ```

<details>
<summary><b>展开查看：MCP 权限档位、41 个工具清单与智能体协作约定（Runtime v0.12.54）</b></summary>

<br/>

- **权限档位**：默认开启 `observer`（读取、运行中安全设置、性能预设、真机绑定、能力缺口记录、补救选择与健康统计）；`operator` 增加设备、应用、调度控制与证据已读确认（热切换实例触控/截图后端亦需此档）；`author` 增加 Lab 实机观察与录制。
- **协作约定**：会话先调 `ac_overview`；耗时操作立即返回跨重启有效的句柄，用 `ac_get_run`（带 `wait_s`）轮询，结果未明时先查句柄勿重发写操作；编辑配置文件、审批与重启 Runtime 均归人工执行。
- **工具参考说明**：本仓 [`skills/actingcommand/SKILL.md`](skills/actingcommand/SKILL.md) 保留了 `v0.11.3` 时期的操作范式参考；在 `0.12` Runtime 上，请以 `<root>\runtime\actingctl.exe mcp-serve --list-tools --format markdown` 打印的实时工具与参数表为准。

| 档位 | 工具清单 |
|---|---|
| `observer`（25 个） | `ac_overview`、`ac_events`、`ac_material`、`ac_observe`、`ac_get_run`、`ac_diagnose`、`ac_runs`、`ac_run_evidence`、`ac_resources_list`、`ac_targets_get`、`ac_config_set`、`ac_perf_preset`、`ac_packs`、`ac_facts`、`ac_outcome_code`、`ac_devices`、`ac_device_bind`、`ac_device_unbind`、`ac_capability_gap`、`ac_remedy_set`、`ac_remedy_get`、`ac_schedule`、`ac_routine_summary`、`ac_pack_check`、`ac_catalog_check` |
| `operator`（9 个） | `ac_run_pack`、`ac_stop_run`、`ac_pause`、`ac_resume`、`ac_emulator`、`ac_app`、`ac_ladder_drill`、`ac_evidence_confirm_read`、`ac_targets_set` |
| `author`（7 个） | `ac_lab_observe`、`ac_lab_do`、`ac_record_start`、`ac_record_mark`、`ac_record_stop`、`ac_record_status`、`ac_binding_draft` |

</details>

---

## 5. 我们现在都有哪些仓

ActingCommand 采用「中立开源内核 + 独立私有资源与估值包」的多仓解耦架构。为确保核心程序 100% 游戏无关且严守版权边界，所有包含具体游戏画面素材、坐标区域与专有经济数据的仓库均与开源底座物理隔离：

| 仓库 | 属性 | 职责定位 | 当前状态 |
|---|---|---|---|
| [**ActingCommand**](https://github.com/HS7097/ActingCommand)（本仓） | 🔓 公开 | **项目族伞仓（门面页）**：总体说明、许可策略、`skills/` 智能体手册及跨组件整包 Releases | 文档持续更新（整包 Release 停留在旧系 `v0.11.4`，待 `v1.00` 合拢更新） |
| [**ActingCommand-Runtime**](https://github.com/HS7097/ActingCommand-Runtime) | 🔓 公开 | **核心运行时仓**：常驻进程 `actingd`、调度器、视觉识别、SQLite 账本、`actingctl` / `actinglab` / `actingledger` / `actingwatch` 及本地 MCP 服务 | 活跃迭代（最新 [`v0.12.54`](https://github.com/HS7097/ActingCommand-Runtime/releases/tag/v0.12.54)） |
| [**ActingCommand-UI**](https://github.com/HS7097/ActingCommand-UI) | 🔓 公开 | **桌面端与中立建模仓**：原生监控台 `acui`、安装向导 `acsetup`、固定入口 `acforward`，以及中立经济建模工具 `acmodel` | 活跃迭代（最新 [`v0.12.15`](https://github.com/HS7097/ActingCommand-UI/releases/tag/v0.12.15)） |
| **ActingCommand-Models** | 🔒 私有 | **游戏估值与经济引擎仓**：对接 `acmodel` 宿主协议的各游戏专属角色估值、养成计算与经济适配器 | 内部研发联调中 |
| **ActingCommand-Resources-Arknights** | 🔒 私有 | **《明日方舟》标准包仓**：`1280×720` 识别锚点、点击框、任务包与排程收益数据 | `0.12` 标准包补齐与实机运行中 |
| **ActingCommand-Resources-AzurLane** | 🔒 私有 | **《碧蓝航线》标准包仓**：`1280×720` 识别锚点、点击框、任务包与排程收益数据 | `0.12` 标准包补齐与实机运行中 |
| **ActingCommand-Resources-BlueArchive** | 🔒 私有 | **《蔚蓝档案》标准包仓**：`1280×720` 识别锚点、点击框、任务包与排程收益数据 | `0.12` 标准包补齐与实机运行中 |
| **ActingCommand-Resources-GirlsFrontline2** | 🔒 私有 | **《少女前线2：追放》标准包仓**：`1280×720` 识别锚点、点击框、任务包与排程收益数据 | `0.12` 标准包补齐与实机运行中 |
| **ActingCommand-Resources-Nikke** | 🔒 私有 | **《胜利女神：NIKKE》资源数据仓**：消耗图谱、经济规则与固定参考能力视图 | 数据建模预研中 |
| **ActingCommand-Workflow** | 🔒 私有 | **工程治理与架构契约仓**：跨仓架构契约、任务单流转、结果码审计与多智能体协作规范 | 内部日常运转 |

*(注：本伞仓下的 `bundles/` 与 `docs/` 仅保留早期归档参考资料，不再随 `0.12` 更新。)*

---

## 参与反馈与开源许可

- **问题与建议**：如果你有更好的想法，或在使用与自制包过程中遇到任何问题，欢迎在本仓或对应的公开成员仓提交 [Issue](https://github.com/HS7097/ActingCommand/issues)。
- **开源许可**：本仓与 ActingCommand-Runtime 遵循 [`AGPL-3.0-only`](LICENSE) 许可证，ActingCommand-UI 为 `GPL-3.0-only`。项目各仓内没有使用其他自动化项目的源码；《明日方舟》标准包内含取自 MaaAssistantArknights 的界面模板图（`AGPL-3.0-only`）。上游许可矩阵、第三方组件与合规规则详见 [`LICENSE_POLICY.md`](LICENSE_POLICY.md) 与 [`NOTICE.md`](NOTICE.md)。
