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

本仓是 ActingCommand 项目族的伞仓（门面页），不承载开发。Runtime 与 UI 以子模块指针挂在本仓下：

| 目录 | 指向 |
|---|---|
| `ActingCommand-Runtime/` | Runtime 仓 `main` |
| `ActingCommand-UI/` | UI 仓 `main` |

指针由工作流 `sync-member-pointers` 每日对齐到各自 `main` 的最新提交，也可手动触发；对齐失败会开一条带 `sync-failure` 标签的 issue。资源包不随本仓分发，按游戏各自的资源仓分发。

```
git clone --recurse-submodules https://github.com/HS7097/ActingCommand.git
```
