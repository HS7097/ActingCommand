# NOTICE.md

ActingCommand is licensed under `AGPL-3.0-only` (this repository and ActingCommand-Runtime) and `GPL-3.0-only` (ActingCommand-UI). State as of 2026-11-12; the details and the rules are in [`LICENSE_POLICY.md`](LICENSE_POLICY.md).

Former project name:

- GachaPilot

## This repository

This repository holds documentation, the agent manual under `skills/`, and archived material under `bundles/` and `docs/`. It contains no source code copied or adapted from another project.

## The member repositories

- **ActingCommand-Runtime** keeps its own [`NOTICE.md`](https://github.com/HS7097/ActingCommand-Runtime/blob/main/NOTICE.md): the source files adapted from HS7097/codex-goal-lifecycle (MIT), and the third-party components its releases carry (Android SDK Platform-Tools, the scrcpy server, OpenH264, the MaaTouch binary), each with its source, licence and notice.
- **ActingCommand-UI** is built with Slint under its GPLv3 option.

## Upstream material in use

- **MaaAssistantArknights/MaaAssistantArknights** — https://github.com/MaaAssistantArknights/MaaAssistantArknights — AGPL-3.0-only, with the project's own user agreement. The Arknights standard pack contains interface template images from its `resource/template` (the home-screen themes, the return button, quick switch, the store and the confirmation prompts), unchanged. This applies to the Arknights packs published in this repository's releases v0.10.3 to v0.11.4 as well. No source code of MaaAssistantArknights is in the project's public repositories or in any pack. The private Arknights resource repository keeps a mirror of MAA's `resource/` tree for reference; that tree includes one script of MAA's, `resource/tasks/MiniGame/PV-Reproduction.py`, kept as imported.

Apart from this and from what the member repositories' own notices list, no other project's source code or assets are in use. The other standard packs are made from the project's own captures.

Game names, art and other game material remain the property of their rights holders.

## Before any upstream code or asset is copied or adapted

Record in the receiving repository's `NOTICE.md`:

- upstream project name and repository URL
- copied or adapted file path, and the upstream commit
- original licence and copyright notice
- local destination path
- modification summary
