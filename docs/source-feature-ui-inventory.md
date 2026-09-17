# Source Feature And UI Inventory

Last checked: 2026-05-29

Sources:

- Azur Lane baseline: `LmeSzinc/AzurLaneAutoScript`, commit `5efbd5e`
- Azur Lane UI/additions: `wess09/AzurPilot`, commit `e9a160f43`
- Blue Archive baseline: `pur1fying/blue_archive_auto_script`, commit `3c858d9`

This document records the user-visible functions and UI structure that GachaPilot should treat as upstream behavior. Azur Lane behavior should follow Alas first, then add AzurPilot conveniences where useful.

## Shared UI Shape

### Alas / AzurPilot

Primary navigation:

- Home page: language/theme, project notice, updater/remote/developer entries depending on fork.
- Instance area: one button per config file, such as `alas`, `AlasR`, or user-created profiles.
- Manage: create/copy/import/export config profiles.

After opening an instance:

- Title/status: current profile name and process state.
- Overview: scheduler control, running queue, pending queue, waiting tasks, and large log panel.
- Secondary navigation: generated from `module/config/argument/menu.json`.
- Detail page: clicking a secondary group expands all task cards in that group; each task card is generated from `module/config/argument/task.yaml`.

AzurPilot keeps the same pattern but adds richer status icons, dashboard panels, resource charts, import/OOBE helpers, announcements, extra themes, and more task groups.

### BAAS

Primary navigation in the Qt client:

- Home: start/stop task runner, current task text, asset display switch, asset widget, log box, hotkey, auto-start support.
- Scheduler: current running task, waiting task list, default state for newly added scheduled tasks, feature switch panel.
- Config: cards generated from `switch.json`; cards open detailed setting panels under `gui/components/expand`.
- Settings: server/device, script behavior, emulator launch, formation, stage pushing, plot pushing, activity map, miscellaneous, push notification, theme/language/DPI/card mode.
- Update Settings: global update configuration.

Profiles are tabbed by config directory. The UI supports multiple profile tabs, profile rename, profile creation, and profile deletion.

## Azur Lane Function Inventory

### Alas Core Menus

| Secondary group | User label | Tasks |
| --- | --- | --- |
| `Alas` | Alas / global | `Alas`, `General`, `Restart` |
| `Farm` | 出击 | `Main`, `Main2`, `Main3`, `GemsFarming` |
| `Event` | 活动 | `EventGeneral`, `Event`, `Event2`, `Raid`, `Hospital`, `Coalition`, `MaritimeEscort`, `WarArchives` |
| `EventDaily` | 活动每日 | `EventA`, `EventB`, `EventC`, `EventD`, `EventSp`, `RaidDaily`, `CoalitionSp` |
| `Reward` | 收获 | `Commission`, `Tactical`, `Research`, `Dorm`, `Meowfficer`, `Guild`, `Reward`, `Awaken` |
| `DailyMission` | 每日任务 | `Daily`, `Hard`, `Exercise`, `ShopFrequent`, `ShopOnce`, `Shipyard`, `Gacha`, `Freebies`, `Minigame`, `PrivateQuarters` |
| `Opsi` | 大世界 | `OpsiGeneral`, `OpsiAshBeacon`, `OpsiAshAssist`, `OpsiExplore`, `OpsiShop`, `OpsiVoucher`, `OpsiDaily`, `OpsiObscure`, `OpsiAbyssal`, `OpsiArchive`, `OpsiStronghold`, `OpsiMonthBoss`, `OpsiMeowfficerFarming`, `OpsiHazard1Leveling`, `OpsiCrossMonth` |
| `Tool` | 工具 | `Daemon`, `OpsiDaemon`, `EventStory`, `Benchmark`, `AzurLaneUncensored`, `GameManager` |

### Alas Task Detail Structure

| Task | User label | Detail setting groups |
| --- | --- | --- |
| `Alas` | Alas设置 | `Emulator`, `EmulatorInfo`, `Error`, `Optimization`, `DropRecord` |
| `General` | 通用设置 | `Retirement`, `OneClickRetire`, `Enhance`, `OldRetire` |
| `Restart` | 重启设置 | `Scheduler` |
| `Main` | 主线图 | `Scheduler`, `Campaign`, `StopCondition`, `Fleet`, `Submarine`, `Emotion`, `HpControl`, `EnemyPriority` |
| `Main2` | 主线图-2 | `Scheduler`, `Campaign`, `StopCondition`, `Fleet`, `Submarine`, `Emotion`, `HpControl`, `EnemyPriority` |
| `Main3` | 主线图-3 | `Scheduler`, `Campaign`, `StopCondition`, `Fleet`, `Submarine`, `Emotion`, `HpControl`, `EnemyPriority` |
| `GemsFarming` | 刷紧急委托 | `Scheduler`, `GemsFarming`, `Campaign`, `StopCondition`, `Fleet` |
| `EventGeneral` | 活动通用设置 | `EventGeneral`, `TaskBalancer` |
| `Event` | 活动图 | `Scheduler`, `Campaign`, `StopCondition`, `Fleet`, `Submarine`, `Emotion`, `HpControl`, `EnemyPriority` |
| `Event2` | 活动图-2 | `Scheduler`, `Campaign`, `StopCondition`, `Fleet`, `Submarine`, `Emotion`, `HpControl`, `EnemyPriority` |
| `Raid` | 共斗活动 | `Scheduler`, `Raid`, `Campaign`, `StopCondition`, `Emotion` |
| `Hospital` | 深谷来信 | `Scheduler`, `Hospital`, `StopCondition`, `Emotion` |
| `Coalition` | 共斗/联动活动 | `Scheduler`, `Campaign`, `Coalition`, `StopCondition`, `Emotion` |
| `MaritimeEscort` | 商船护航 | `Scheduler`, `MaritimeEscort` |
| `WarArchives` | 作战档案 | `Scheduler`, `Campaign`, `StopCondition`, `Fleet`, `Submarine`, `Emotion`, `HpControl`, `EnemyPriority` |
| `EventA` | 活动每日A图 | `Scheduler`, `EventDaily`, `Campaign`, `StopCondition`, `Fleet`, `Submarine`, `Emotion`, `HpControl`, `EnemyPriority` |
| `EventB` | 活动每日B图 | `Scheduler`, `EventDaily`, `Campaign`, `StopCondition`, `Fleet`, `Submarine`, `Emotion`, `HpControl`, `EnemyPriority` |
| `EventC` | 活动每日C图 | `Scheduler`, `EventDaily`, `Campaign`, `StopCondition`, `Fleet`, `Submarine`, `Emotion`, `HpControl`, `EnemyPriority` |
| `EventD` | 活动每日D图 | `Scheduler`, `EventDaily`, `Campaign`, `StopCondition`, `Fleet`, `Submarine`, `Emotion`, `HpControl`, `EnemyPriority` |
| `EventSp` | 活动每日SP图 | `Scheduler`, `Campaign`, `StopCondition`, `Fleet`, `Submarine`, `Emotion`, `HpControl`, `EnemyPriority` |
| `RaidDaily` | 共斗活动每日 | `Scheduler`, `RaidDaily`, `Campaign`, `StopCondition`, `Emotion` |
| `CoalitionSp` | 共斗/联动SP | `Scheduler`, `Campaign`, `Coalition`, `StopCondition`, `Emotion` |
| `Commission` | 委托 | `Scheduler`, `Commission` |
| `Tactical` | 战术学院 | `Scheduler`, `Tactical`, `ControlExpOverflow`, `AddNewStudent` |
| `Research` | 科研 | `Scheduler`, `Research` |
| `Dorm` | 后宅 | `Scheduler`, `Dorm`, `BuyFurniture` |
| `Meowfficer` | 指挥喵 | `Scheduler`, `Meowfficer`, `MeowfficerTrain` |
| `Guild` | 大舰队 | `Scheduler`, `GuildLogistics`, `GuildOperation` |
| `Reward` | 收获 | `Scheduler`, `Reward` |
| `Awaken` | 认知觉醒 | `Scheduler`, `Awaken` |
| `Daily` | 每日任务 | `Scheduler`, `Daily` |
| `Hard` | 困难图 | `Scheduler`, `Hard` |
| `Exercise` | 演习 | `Scheduler`, `Exercise` |
| `ShopFrequent` | 通用商店 | `Scheduler`, `GeneralShop` |
| `ShopOnce` | 其他商店 | `Scheduler`, `GuildShop`, `MedalShop2`, `MeritShop`, `CoreShop` |
| `Shipyard` | 开发船坞 | `Scheduler`, `ShipyardDr`, `Shipyard` |
| `Gacha` | 每日抽卡 | `Scheduler`, `Gacha` |
| `Freebies` | 白嫖奖励 | `Scheduler`, `BattlePass`, `DataKey`, `Mail`, `SupplyPack` |
| `Minigame` | 小游戏 | `Scheduler` |
| `PrivateQuarters` | 宿舍计划 | `Scheduler`, `PrivateQuarters` |
| `OpsiGeneral` | 大世界通用 | `OpsiGeneral` |
| `OpsiAshBeacon` | META作战 | `Scheduler`, `OpsiAshBeacon` |
| `OpsiAshAssist` | META支援 | `Scheduler`, `OpsiAshAssist` |
| `OpsiExplore` | 每月开荒 | `Scheduler`, `OpsiExplore`, `OpsiFleet` |
| `OpsiShop` | 大世界商店 | `Scheduler`, `OpsiShop` |
| `OpsiVoucher` | 兑换商店 | `Scheduler`, `OpsiVoucher` |
| `OpsiDaily` | 大世界每日 | `Scheduler`, `OpsiDaily`, `OpsiFleet` |
| `OpsiObscure` | 隐秘海域 | `Scheduler`, `OpsiObscure`, `OpsiFleet` |
| `OpsiAbyssal` | 深渊海域 | `Scheduler`, `OpsiAbyssal`, `OpsiFleetFilter` |
| `OpsiArchive` | 档案坐标 | `Scheduler`, `OpsiFleet` |
| `OpsiStronghold` | 塞壬要塞 | `Scheduler`, `OpsiStronghold`, `OpsiFleetFilter` |
| `OpsiMonthBoss` | 月度Boss | `Scheduler`, `OpsiMonthBoss`, `OpsiFleetFilter` |
| `OpsiMeowfficerFarming` | 短猫相接 | `Scheduler`, `OpsiMeowfficerFarming`, `OpsiFleet` |
| `OpsiHazard1Leveling` | 侵蚀1练级 | `Scheduler`, `OpsiHazard1Leveling`, `OpsiFleet` |
| `OpsiCrossMonth` | 跨月每日 | `Scheduler` |
| `Daemon` | 半自动点击 | `Daemon` |
| `OpsiDaemon` | 大世界半自动 | `OpsiDaemon` |
| `EventStory` | 活动剧情 | `EventStory` |
| `Benchmark` | 性能测试 | `Benchmark` |
| `AzurLaneUncensored` | 反和谐 | `AzurLaneUncensored` |
| `GameManager` | 游戏管理器 | `GameManager` |

### AzurPilot Additions On Top Of Alas

AzurPilot keeps the same nested UI model and adds these visible areas:

- Profile sidebar status icons: idle/running/error/update.
- Overview dashboard inside logs: resource display can be toggled on/off.
- Dashboard task group: `Oil`, `Coin`, `Gem`, `Pt`, `Cube`, `ActionPoint`, `YellowCoin`, `PurpleCoin`, `Core`, `Medal`, `Merit`, `GuildCoin`.
- Statistics panels: AP chart, all-resource trend chart, Operation Siren monthly stats, ship EXP stats, commission income stats.
- Event calculator with save-back helpers for event PT planning.
- OOBE / old data import flow for importing older AzurPilot or Alas configs.
- Announcement page, auto-update switch, extra themes, and modified home notices.
- Extra task entries: `ThreeOilLowCost`, `Ambush11`, `Event3`, `RaidScuttle`, `EventShop`, `Island`, `OpsiScheduling`, `OpsiSimulator`, `BoxDisassemble`, `OcrBenchmark`, `EmulatorManager`.

Extra AzurPilot task detail structure:

| Task | User label | Detail setting groups |
| --- | --- | --- |
| `ThreeOilLowCost` | 三油低耗+ | `Scheduler`, `GemsFarming`, `Campaign`, `StopCondition`, `Fleet`, `Submarine`, `Emotion` |
| `Ambush11` | 1-1刷伏击 | `Scheduler`, `GemsFarming`, `Campaign`, `StopCondition`, `Fleet`, `Submarine`, `Emotion` |
| `Event3` | 活动图-3+ | `Scheduler`, `Campaign`, `StopCondition`, `Fleet`, `Submarine`, `Emotion`, `HpControl`, `EnemyPriority` |
| `RaidScuttle` | 共斗沉船+ | `Scheduler`, `RaidScuttle`, `Raid`, `Campaign`, `StopCondition`, `Fleet`, `Submarine`, `Emotion` |
| `EventShop` | 活动商店 | `Scheduler`, `EventShop` |
| `Island` | 岛屿计划 | `Scheduler`, `IslandTransport`, `Island1` ... `Island16` |
| `OpsiMeowfficerFarming` | 耄耋相接 | `Scheduler`, `OpsiMeowfficerFarming`, `OpsiSirenBug`, `OpsiTarget`, `OpsiFleet` |
| `OpsiHazard1Leveling` | 侵蚀1练级 | `Scheduler`, `OpsiHazard1Leveling`, `OpsiSirenBug`, `OpsiFleet`, `OpsiCheckLeveling`, `OpsiFleetAutoChange` |
| `OpsiScheduling` | 智能调度+ | `Scheduler`, `OpsiScheduling` |
| `OpsiSimulator` | 大世界模拟器 Alpha | `OpsiSimulatorParameters` |
| `IslandInfo` | 岛屿信息 | `IslandSeasonTask`, `IslandTechnology` |
| `IslandSeasonTask` | 赛季任务 | `Scheduler` |
| `BoxDisassemble` | 拆装备箱 | `BoxDisassemble` |
| `OcrBenchmark` | OCR性能测试 | `OcrBenchmark` |
| `EmulatorManager` | 模拟器管理器 | `EmulatorManager` |

## Blue Archive Function Inventory

### BAAS Top-Level Gameplay Features

| Feature | User behavior |
| --- | --- |
| 角色好感度 | 咖啡厅摸头、指定学生邀请、日程优先找指定学生或高好感度区域 |
| 主线 | 普通/困难主线自动推图与扫荡 |
| 咖啡厅 | 1号/2号咖啡厅奖励、摸头、邀请券、指定/收藏学生 |
| 商店 | 普通商店、竞技场商店，支持商品白名单和刷新次数 |
| 收获 | 小组体力、免费体力、邮箱、每日体力、竞技场奖励、总力战累计积分、每日任务、Pass奖励 |
| 体力清理 | 普通关、困难关、特别委托、活动关卡，支持扫荡次数 |
| 日程 | 指定区域次数、指定学生、好感度优先 |
| 竞技场 | 自动挑战到无票，自动领取每日奖励 |
| 制造 | 三阶段制造、优先级、次数、加速券 |
| MomoTalk | 自动完成未结束对话、剧情、青辉石领取 |
| 总力战 | 清挑战券、领奖，自动凹分由 `BAAS_Cpp` 方向接入 |
| 战术综合测试 | 活动开启时自动清票 |
| 剧情 | 主线剧情、小组剧情、迷你剧情 |
| 活动 | 活动剧情、任务、挑战、走格子、扫荡；多服活动资源独立更新 |

### BAAS Scheduled Task List

| Event name | Function name | Default state | Timing model |
| --- | --- | --- | --- |
| 凌晨四点重启 | `restart` | on | daily reset |
| 竞技场 | `arena` | on | daily reset |
| 咖啡厅 | `cafe_reward` | on | interval + daily reset |
| 1号咖啡厅邀请 | `no1_cafe_invite` | off | daily reset |
| 2号咖啡厅邀请 | `no2_cafe_invite` | off | daily reset |
| 日程 | `lesson` | on | daily reset |
| 收集小组体力 | `group` | on | daily reset |
| 收集免费购买体力 | `collect_daily_free_power` | on | daily reset |
| 查收邮箱 | `mail` | on | daily reset |
| 收集每日体力 | `collect_daily_power` | on | daily reset |
| 商店购买 | `common_shop` | on | daily reset |
| 竞技场商店购买 | `tactical_challenge_shop` | on | daily reset |
| 悬赏通缉 | `rewarded_task` | on | daily reset |
| 普通关清体力 | `normal_task` | on | daily reset |
| 困难关清体力 | `hard_task` | on | daily reset |
| 学院交流会 | `scrimmage` | on | daily reset |
| 每日特别委托 | `clear_special_task_power` | on | daily reset |
| 自动制造 | `create` | on | daily reset |
| 总力战 | `total_assault` | off | daily reset |
| 活动扫荡 | `activity_sweep` | on | daily reset |
| 收集奖励 | `collect_reward` | on | daily reset |
| 自动MomoTalk | `momo_talk` | on | interval |
| 日常小游戏 | `dailyGameActivity` | on | daily reset |
| 清理好友 | `friend` | off | interval + daily reset |
| 综合战术测试 | `joint_firing_drill` | on | daily reset |
| 领取pass奖励 | `pass` | on | daily reset |

### BAAS Config Cards And Detail Panels

| Card / panel | Purpose |
| --- | --- |
| `cafeInvite` | 咖啡厅奖励、摸头轮次、邀请策略、二号咖啡厅、重复邀请等 |
| `schedulePriority` | 日程区域次数、指定学生、好感度优先 |
| `shopPriority` | 普通商店商品选择和刷新次数 |
| `arenaShopPriority` | 竞技场商店商品选择 |
| `mainlinePriority` | 普通/困难主线刷体力目标 |
| `arenaPriority` | 竞技场挑战策略和买票次数 |
| `createPriority` | 制造阶段、节点优先级、制造次数、持有量 |
| `totalForceFightPriority` | 总力战最高难度 |
| `sweepCountConfig` | 各类扫荡次数和买票次数 |
| `friendWhiteList` | 自动清好友白名单 |
| `drillConfig` | 战术综合测试清票/扫荡配置 |
| `serverConfig` | 游戏服务器、ADB IP/端口、设备识别 |
| `scriptConfig` | 截图间隔、启动后自动运行、任务完成后行为、截图方法、控制方法 |
| `emulatorConfig` | 模拟器路径和启动方式 |
| `formationConfig` | 编队参数 |
| `exploreConfig` | 推图关卡参数 |
| `proceedPlot` | 主线/小组/迷你剧情推进 |
| `eventMapConfig` | 活动剧情、任务、挑战、扫荡 |
| `otherConfig` | 杂项功能 |
| `pushConfig` | 任务完成/异常等推送配置 |
| `featureSwitch` | 调度任务开关、默认启用状态、队列显示 |
| `baasUpdateConfig` | 更新设置 |

## GachaPilot UI Model Proposal

Use a common model for all games:

```text
App
  Overview
    Resource cards from all profiles
    Resource package sync status
    Recent events / global warnings
  Profile sidebar
    Profile name
    Game.server label, for example Azur.JP or BA.JP
    Runtime state
  Profile detail
    Scheduler state
    Queue / waiting / running tasks
    Large log panel
    Secondary menu generated from adapter schema
    Detail cards generated from adapter setting groups
  Settings
    Global resource repositories
    Adapter installation/update
    Device/emulator pools
    Theme/language
```

Adapter schema:

```text
GameAdapter
  id
  displayName
  supportedServers
  profiles[]
  resources[]
  menus[]
    menu id / label
    tasks[]
      task id / label
      scheduler fields
      setting groups[]
        fields[]
```

Initial mapping:

- `azur-lane` adapter: source menus/tasks from Alas; dashboard/statistics/import helpers from AzurPilot.
- `blue-archive` adapter: source tasks from BAAS `EVENT_DEFAULT_CONFIG`; setting cards from BAAS `SWITCH_DEFAULT_CONFIG` and `gui/components/expand`.
- Later `arknights` adapter: keep MAA core logic and expose it through the same profile/detail/settings contract.
