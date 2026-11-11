# LICENSE_POLICY.md

State as of 2026-11-12. This file records facts and the project's own rules; it is not legal advice.

## Licences of the project's own work

| Repository | Visibility | Licence of the project's own files |
| --- | --- | --- |
| HS7097/ActingCommand (this repository) | public | `AGPL-3.0-only` |
| HS7097/ActingCommand-Runtime | public | `AGPL-3.0-only` |
| HS7097/ActingCommand-UI | public | `GPL-3.0-only` (since 2026-09-17; its UI toolkit, Slint, is used under its GPLv3 option) |
| Per-game resource repositories, the Models repository, the Workflow repository | private | not published; game material in them stays the property of its rights holders |

Use the matching SPDX identifier in newly written files of each repository, for example:

```text
SPDX-License-Identifier: AGPL-3.0-only
```

This does not relicense anything that came from elsewhere. Upstream copyright notices and licence obligations are preserved where upstream material is present (see below).

## Third-party material: what is in the project today

Checked on 2026-11-12 against the `main` of each repository. Two questions are kept apart: **source code**, and **assets and data** (images, recognition templates, task data, tables).

### Upstream license matrix

Only upstream projects whose material is in use are listed.

| Upstream project | Its licence today | Source code in our repositories | Assets and data in our repositories |
| --- | --- | --- | --- |
| [MaaAssistantArknights/MaaAssistantArknights](https://github.com/MaaAssistantArknights/MaaAssistantArknights) | AGPL-3.0-only, with the project's own user agreement | None copied. The Runtime has its own implementation that reads MAA's published task format as data. | **In use.** The Arknights standard pack contains interface template images from MAA's `resource/template` (the home-screen themes, the return button, quick switch, the store and the confirmation prompts) unchanged: 711 of the 2284 image files of its 126 task packs today. The Arknights packs published in this repository's releases v0.10.3 to v0.11.4 contain such files too (386 of 1014 image files in v0.11.4). The private Arknights resource repository also keeps a mirror of MAA's `resource/` tree for reference. |

How this was checked: the public repositories were searched for licence headers, copyright lines and statements of origin, and every file of the packs, today's and the published ones, was compared by content hash with the upstream files the resource repositories recorded at import. The comparison finds unchanged files only.

### Source code adapted from other projects

| Where | From | Licence | Record |
| --- | --- | --- | --- |
| ActingCommand-Runtime, `crates/agent-forward` (ten files) | HS7097/codex-goal-lifecycle, the owner's own project | MIT | the Runtime's `NOTICE.md`; the licence text ships in the crate and each file says so at its top |

No other source file in the public repositories is copied or adapted from another project. Libraries are used as unmodified dependencies under their own licences (the Rust crates of each `Cargo.lock`).

### Third-party components that releases carry

The Runtime's own `NOTICE.md` is the authoritative list, with versions, sources, checksums and the notices that ship. In summary: Android SDK Platform-Tools (adb) in the Tools archive; the scrcpy server and OpenH264 inside the Runtime executable; the MaaTouch binary in the Runtime repository. ONNX Runtime, DirectML and the recognition models are not shipped; an operator obtains them.

### Game material

Game names, art, screenshots and anything derived from them (templates, cropped patches, coordinates, colours) remain the property of the games' rights holders. The project claims no copyright in them, and they are not covered by the licences above.

### Data used for modelling

Tables and pages taken from wikis and community sources are kept in private repositories with their source, author and terms recorded, and are not redistributed in the public repositories. Some of them carry non-commercial terms.

## Rules

- Before any upstream code or asset is copied or adapted, verify its licence from the upstream repository's own `LICENSE`, `COPYING`, `NOTICE` or equivalent files, at repository level and at file level.
- Record every copied or adapted file in the `NOTICE.md` of the repository that receives it: the upstream project and URL, the upstream path and commit, the licence and copyright notice, the local path and a summary of the changes. Keep the upstream licence text beside it.
- Whatever is distributed carries its notices: a release or a pack that contains third-party material ships the attribution and the licence text with it.
- Provide the corresponding source when distributing builds, as the licences above require.
- Prefer an own implementation from public specifications when licence compatibility is unclear, and say in the code what was used as a reference.
- Treat assets and data separately from source code. Do not assume that game screenshots, templates, icons, recognition models or data tables are covered by the same licence as code.
- `GPL-3.0-only`, `GPL-3.0-or-later`, `AGPL-3.0-only` and `AGPL-3.0-or-later` are not interchangeable; review compatibility before combining code under different ones.
- MIT-licensed code may be incorporated, with its copyright and licence notice preserved.

## Network use

`actingd` listens on the loopback address only and offers no network service. If a later component offers one, users who interact with it over a network must be able to obtain the corresponding source, as AGPL-3.0 section 13 requires.
