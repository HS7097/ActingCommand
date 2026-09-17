# AGENTS.md

## Project license direction

This project is planned to use `AGPL-3.0-only` as the overall project license direction.

However, do not copy, merge, or rewrite upstream source code before the license matrix and license boundary review are completed.

When upstream code is used later, preserve upstream copyright notices, license files, and attribution.

## Required reading before work

Before making project changes, read these files if they exist:

- `PLANS.md`
- `CHECKPOINT.md`
- `LICENSE_POLICY.md`
- `NOTICE.md`

Do not rely only on chat history or compacted context.

## Error handling rules

- Severe errors must never silently fail.
- Silent failure is not acceptable for serious faults.
- Critical failures must not return empty objects, empty arrays, null, fake default values, or silently skipped results.
- Less severe transient issues such as API timeouts or emulator connection failures may use fallback behavior, but the fallback path must be fully logged.
- Fallback logs must include the trigger reason, fallback path, affected module, and user-visible impact if any.

## Coding style

- Prefer explicit errors over silent fallback.
- Prefer guard clauses over deeply nested if-else.
- Prefer handler maps, strategy objects, or explicit error types when they make branch behavior clearer.
- Do not add production dependencies without asking first.
- Do not perform broad rewrites unless the current plan explicitly requires it.
- Use comments sparingly. Prefer clear names, small functions, type hints, and explicit state models over comments that restate code.
- Add comments or docstrings only for public APIs, non-obvious design decisions, process/runtime boundaries, state-machine invariants, recovery behavior, concurrency/lifecycle rules, security/license boundaries, or intentional limitations.
- For `AliceRuntimeOrchestrator`, document architectural boundaries clearly: the UI does not own runtime lifecycle, the runtime survives UI crash/reload/close, communication uses local API/IPC, upstream code remains behind adapter/license boundaries, runtime commands are idempotent and state-aware, and ADB/device/game automation failures must be logged and classified.

## License compliance rules

- Do not copy upstream source code until the license matrix is complete.
- Do not assume MIT, GPL, and AGPL code can be freely mixed without attribution.
- Keep AGPL/GPL/MIT source boundaries visible.
- Prefer plugin boundaries and clean interfaces over direct code merging.
- Game assets, screenshots, templates, icons, OCR data, and model files require separate review and must not be assumed to follow the code license.

## Progress tracking

After each meaningful step, update `CHECKPOINT.md` with:

- current status
- files changed
- commands run
- test results
- current blocker
- next step

When project goals, current phase, scope boundaries, or next steps change, update `PLANS.md` as well.

Planning files must stay aligned with the actual task state and must not rely only on chat history.

## ActingCommand task completion

These rules apply only to ActingCommand work. They are not global Codex rules and do not apply to unrelated projects or conversations.

When an ActingCommand task is completed and verified, push the completed repository changes to GitHub unless the user explicitly says not to push.

Cooperation workspace syncing is temporarily paused. Do not copy planning files to `C:\合作工作区\ActingCommand` unless the user explicitly asks for it again.

After each completed ActingCommand task, commit and push the relevant `PLANS.md` and `CHECKPOINT.md` in the same repository as the changed local files.

If the task changes `HS7097/ActingCommand-Runtime`, keep the Runtime repository's own planning and checkpoint files updated and commit them with the Runtime source changes.

If the task changes `HS7097/ActingCommand-UI`, keep the UI repository's own planning and checkpoint files updated and commit them with the UI source changes.

Use the umbrella `HS7097/ActingCommand` repository only for umbrella-level planning, cross-repository policy, or meta-documentation changes. Do not mirror planning files into the umbrella repository after every Runtime/UI task by default.

Do not merge, copy, or synchronize routine Runtime updates into the umbrella/main `HS7097/ActingCommand` repository by default. Runtime updates stay in `HS7097/ActingCommand-Runtime` until the user explicitly confirms they are ready to merge into the main repository.

For rollback and provenance, record important completed milestones with Git commit hashes in `CHECKPOINT.md`. Use checkpoint tags for stable milestones when they create a meaningful rollback point. Do not rewrite `main` history for ActingCommand repositories unless the user explicitly requests a history rewrite.
