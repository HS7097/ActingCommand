# LICENSE_POLICY.md

## Overall license direction

This project is planned to use:

`AGPL-3.0-only`

as the overall project license direction.

This does not automatically relicense upstream code. Upstream copyright notices and license obligations must be preserved.

## SPDX identifier

Use the following SPDX identifier for newly written project files where appropriate:

```text
SPDX-License-Identifier: AGPL-3.0-only
```

## Upstream license matrix

Before copying or adapting code, verify the license of each upstream repository from its actual `LICENSE`, `COPYING`, `NOTICE`, or equivalent files.

| Upstream project                            | URL                                                            | Expected license | Verified from repository | Direct code use status       | Notes                                           |
| ------------------------------------------- | -------------------------------------------------------------- | ---------------- | ------------------------ | ---------------------------- | ----------------------------------------------- |
| wess09/AzurPilot                            | https://github.com/wess09/AzurPilot                            | GPL-3.0 license text | Yes, local `LICENSE` file | Conditional after file-level review | Code may be incorporated or refactored only when attribution, license, source, and modification-record obligations are satisfied |
| LmeSzinc/AzurLaneAutoScript                 | https://github.com/LmeSzinc/AzurLaneAutoScript                 | GPL-3.0 license text | Yes, local `LICENSE` file | Conditional after file-level review | Code may be incorporated or refactored only when attribution, license, source, and modification-record obligations are satisfied |
| MaaAssistantArknights/MaaAssistantArknights | https://github.com/MaaAssistantArknights/MaaAssistantArknights | To verify        | No                       | No, pending license verification | AGPL/GPL boundary must be checked               |
| BlueArchiveArisHelper/BAAH                  | https://github.com/BlueArchiveArisHelper/BAAH                  | To verify        | No                       | No, pending license verification | MIT or other license must be verified from repo |
| pur1fying/blue_archive_auto_script          | https://github.com/pur1fying/blue_archive_auto_script          | To verify        | No                       | No, pending license verification | GPL/AGPL boundary must be checked               |

## Compliance rules

- This project follows `AGPL-3.0-only` as its overall project license direction.
- Compatible upstream code may be copied, adapted, referenced directly, or refactored inside this repository when the license conditions are satisfied.
- Before direct upstream code use, verify repository-level and file-level licenses.
- Preserve upstream copyright notices.
- Preserve upstream license texts.
- Record copied/adapted files in `NOTICE.md`.
- Record modification summaries for copied, adapted, or refactored upstream files.
- Provide corresponding source code when distributing builds or offering network-service access as required by `AGPL-3.0-only`.
- Keep plugin boundaries explicit.
- Prefer clean-room reimplementation when license compatibility is unclear.
- Treat non-code assets separately from source code.
- Do not assume game screenshots, templates, icons, OCR models, or other assets are covered by the same license as code.

## AGPL network-service note

If this project exposes a web UI, remote control panel, or network service, ensure that users who interact with the modified program over a network can access the corresponding source code as required by AGPL-3.0.

ActingCommand currently carries an `AGPL-3.0-only` license file in the renamed `HS7097/ActingCommand` repository and the split `HS7097/ActingCommand-UI` and `HS7097/ActingCommand-Runtime` repositories. Newly written ActingCommand runtime and UI files should follow that project license direction unless a later license review changes it.

## GPL and AGPL note

GPL-3.0 and AGPL-3.0 compatibility must be reviewed before combining code.

Do not assume that `GPL-3.0-only`, `GPL-3.0-or-later`, `AGPL-3.0-only`, and `AGPL-3.0-or-later` are interchangeable.

## MIT note

MIT-licensed code may generally be incorporated into stronger copyleft projects, but the original MIT copyright and license notice must be preserved.
