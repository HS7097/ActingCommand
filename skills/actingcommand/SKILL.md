---
name: actingcommand
description: Operate an installed ActingCommand through its local MCP server (actingctl mcp-serve, Runtime v0.11.0) and its ac_* tools. Use when the machine has an ActingCommand install (actingd.config.json beside runtime\, tools\ and ui\ folders) and the user wants the system's health, what each emulator instance is doing, one task pack run once or stopped, a pack checked, a failed or suspended scheduled task diagnosed, instance resource targets set, or a task pack recorded with Lab. Without the MCP server it follows references/cli-workshop.md. It never approves, never edits the daemon configuration and never starts or stops the daemon.
---

# ActingCommand program skill v1 (MCP)

`actingctl mcp-serve` is a local stdio MCP server that offers tools only. Each tool does one piece of work and passes the Runtime's answers and refusals through unchanged; every judgment stays in the Runtime. This manual names tools. Their arguments and result fields come from `tools/list` at run time, which is authoritative; the generated table is in `references/tools.md`. Nothing here is specific to a game: game knowledge lives in the resource packs.

## When to use

- An ActingCommand install is on the machine, and the user asks for its health and what each instance is doing (R1), one task pack run once (R2) or a run stopped (Stop), a pack checked (R3), why a scheduled task failed or was suspended (R4), resource targets (R5), or a pack recorded with Lab.
- Tools named `ac_*` are available. If they are not, see Connecting.
- Read-only tools need no request. Every write (the operator and author tiers) needs the user's request for that action.

## Connecting

- `<root>\runtime\actingctl.exe mcp-config --client claude` (or `--client codex`), optionally with `--tier observer,operator,author`, prints the configuration snippet and writes nothing. The user adds it to the client: for Claude Code the printed `claude mcp add --scope user actingcommand -- "<exe>" mcp-serve --tier observer`, for Codex the printed `[mcp_servers.actingcommand]` table in its `config.toml`. Never edit a client's configuration yourself.
- **Tiers.** `observer` reads and is always on. `operator` adds device and scheduling writes, `author` adds Lab recording. The default is observer only. Enabling a tier is the user's step: they change `--tier` and restart the server. A tool outside the enabled tiers answers `tier_not_enabled`. Report it; do not do the same work through the CLI instead.
- **Location.** The server takes the install root from its own path (`<root>\runtime\actingctl.exe`) and reads only `state_root` from `<root>\actingd.config.json`; `--root` and `--state-root` override them. It reads nothing at start. `ac_overview` shows what it found (`install`, `lab_tool`).
- **Time and size.** Every call answers within 25 s; longer work answers a handle at once. Every result is at most 24 KiB: lists page with cursors, and screenshots and oversized Lab answers are written to files under `%TEMP%\actingcommand-mcp\materials\`.
  - Codex: the snippet sets `startup_timeout_sec = 10` and `tool_timeout_sec = 60`. Keep the tool timeout above 25 s. The commented `enabled_tools` line is an optional second restriction.
  - Claude Code warns when a tool result passes 10,000 tokens and cuts it at 25,000 by default (`MAX_MCP_OUTPUT_TOKENS`). The 24 KiB cap keeps every result under that limit; a large result can still raise the warning.
  - The server speaks both protocol generations (`initialize` with 2025-03-26, 2025-06-18 or 2025-11-25, and per-request 2026-07-28). No client setting is needed.
- **Without the server**, use `references/cli-workshop.md`: the same work with actingctl, actinglab, actingledger and actingd on the command line.

## Handles, cursors and pauses

- A run's handle is its Runtime `request_id`. It stays valid across MCP and actingd restarts, and any process reads it with `ac_get_run`.
- Every other job handle (from `ac_pause`, `ac_resume`, `ac_emulator`, `ac_stop_run`, `ac_pack_check`, `ac_catalog_check` and the author tools) lives in its MCP process only and answers `handle_unknown` after that process ends. Then read a pause's `revision` and `owner_epoch`, or the emulator's state, from `ac_overview`, a run from `ac_get_run`, and a recording from `ac_record_status`.
- An `ac_events` cursor dies with the Runtime connection or the server (`cursor_invalid`). Read again from the first page.
- A pause made with `ac_pause` is the CLI's pause. It stays until `ac_resume` or an actingd restart, which clears every pause; it does not end with the MCP process.

## Recipes

### R1. Health, and what each instance is doing (observer)

1. `ac_overview`. If `daemon.online` is false, report `daemon.error.code` (`runtime_unavailable`, `install_state_root_unresolved`) and stop. There is then no `instances` field: that is no answer, not an empty system. Starting actingd is the user's step.
2. Read `global_pause`, and for each instance `lease_active`, `queued`, `pause` and `run` (its newest run of the last 24 h). `incomplete` means part of the snapshot is missing; say which part (`warnings`).
3. For more, `ac_events` (by instance and view, from a start time, following `next_cursor`), and `ac_get_run` for one run.

### R2. Run one task pack once (operator, on request)

1. `ac_overview`: the instance is listed, `lease_active` is false and `queued` is 0. A paused instance was paused on purpose: ask first.
2. Check the pack with `ac_pack_check` (R3) unless it was just checked.
3. **Exclusive use** of the instance means `ac_pause` for that instance first and `ac_resume` afterwards. The pause drains every in-flight run on that instance, manual runs and other sessions' runs included: when the drain times out, the Runtime asks them all to stop. Pause only with the user's consent, and keep `owner_epoch` and the pause's `revision` from the answer.
4. `ac_run_pack` answers at once with the handle (phase `submitting`). It never pauses anything: a busy instance comes back in `ac_get_run` as the Runtime's `LeaseBusy` or `ContainedTaskBusy`, class safety.
5. `ac_get_run` with the handle, waiting up to 25 s per call, until the state is `succeeded`, `failed` or `cancelled`. `interrupted_unterminated` is uncertain (Failures).
6. After step 3, `ac_resume` with the `owner_epoch` and `revision` you kept (or that `ac_overview` shows). A refusal with `details.host_failure.code` `scheduling_pause_owner_epoch_mismatch` or `scheduling_pause_revision_mismatch` means the pause is no longer the one you made: report it, never retry with other values.
7. If `ac_run_pack` was interrupted before it answered, call `ac_overview` before you submit again.

Report the state, outcome, failure code, final page and executed steps.

### Stop a run (operator, on request)

- `ac_stop_run` with the run's handle stops a manual run (MCP, CLI, UI or Lab) from any process. The package is abandoned and the screen stays where it is: no return home, no recovery package, no rerun. Only touch points that may still be held are lifted.
- `touch_release` is `done`, `not_needed` (this includes a run that had already ended) or `failed` (for example `LeaseBusy`, which can mean the still-connected submitter lifted them itself). `ac_get_run` then shows `cancelled`.
- A scheduled run cannot be stopped by a client. The Runtime refuses and `blocked_by` names `ac_pause`: the drain of an instance pause stops it, which needs the user's consent (R2 step 3).

### R3. Check a pack (observer, offline)

1. `ac_pack_check` with the package (directory, `<D>.zip` or `<D>.json`) and, if you have one, the reference you intend to use. It answers actinglab's `package digest` and `package preflight` verbatim. Keep `digest.reference`; `package_ref_check.matches_digest` says whether a given reference is the package's.
2. `preflight.coverage` says how far the check went; it recognizes and executes nothing. A refusal is class usage, code `package_invalid`, with actinglab's error in `details.lab_error`.
3. `ac_catalog_check` compiles a business identity catalog offline and writes nothing.
4. `actingd check-config` runs inside `ac_binding_draft` (author). Without that tier, use `references/cli-workshop.md` R3.

### R4. Diagnose a failed or suspended task (observer)

1. `ac_overview` for the instance: paused, held by a lease, or the daemon offline?
2. `ac_diagnose` for the instance, optionally for one task. The window is the last 24 h by default and reaches back at most 7 days. It gives the failed or open runs, the first page of errors, and this instance's rows of the actingd suspended report (`suspended`, `lift`, `repeating`) verbatim. A `suspended_report.status` other than `ok`, or `incomplete`, means part is missing: say so.
3. `ac_get_run` with a run's `run_id` for its full status: `terminal.failure_code`, `progress`, `evidence`.
4. Continue the errors with `ac_events` (view errors, same instance and window). Read evidence with `ac_material`: text in pieces of up to 8 KiB; images and archives only as exported files.
5. Report the failing step, its failure code and whether it repeats. Lifting a suspension is the user's: update the pack, point the binding at it, restart actingd (`references/lab-recording.md` 7.9). Never re-run, change packs or edit the configuration on your own.

### R5. Resource targets (setting them: operator, on request)

1. `ac_resources_list`: what the instance can target, what each target must give, and `policy_instance`. `ac_targets_get`: the policy active now.
2. `ac_targets_set` with v2 targets and, for a non-empty list, a validity in days (1 to 365). The server writes the document around them. A refusal carries its position in `details.rejection`: fix that field. An empty list without a validity withdraws the policy.
3. `ac_targets_get` again. A condition in `awaiting_observation` is not an error.

An actingd older than v0.11.0 answers `runtime_operation_unsupported`: read the scheduling catalog as in `references/cli-workshop.md` R5.

## Lab recording (author, on request)

Read `references/lab-recording.md` first. It maps each author tool to its actinglab command and keeps the full recording rules. The tool flow:

`ac_pause` (the instance) → `ac_record_start` → device frames with `ac_lab_observe` or `ac_lab_do` (capture and record), marks, clicks and offline frames with `ac_record_mark` → `ac_record_status` → `ac_record_stop` as a dry run → more `ac_record_mark` → `ac_record_stop` → `ac_binding_draft` → `ac_resume`.

- The author tools' `instance` is the actinglab instance alias.
- `ac_lab_do` with capture acts on the device and does not read the destructive guard flags.
- `ac_binding_draft` only drafts. The binding, the catalog task, its approval and the daemon restart are the user's steps (`manual_steps`).

## Failures

Every failure is `isError` with `{ok:false, error:{class, code, message, blocked_by, details}}`; `blocked_by` names what can unblock it. Answers that worked can still carry `warnings`, `incomplete` or `truncated`: report those, never as plain success.

| Class | Do |
|---|---|
| usage | Fix the arguments and call again (`arguments_invalid`, `instance_unknown`, `since_out_of_range`, `package_invalid`) |
| safety | Stop and report to the user (`tier_not_enabled`, `LeaseBusy`, `ContainedTaskBusy`, `record_session_active`) |
| device | Check the instance and its emulator with `ac_overview`. Start or stop an emulator (`ac_emulator`) only on request |
| runtime | actingd or a helper program is unreachable or failed (`runtime_unavailable`, `lab_process_failed`). Report it; never start actingd |
| uncertain | The write may have happened. Call `ac_get_run` first (by handle), then `ac_overview`. Never resend a write with new arguments |

The server's own codes:
- `tier_not_enabled` (safety): `details.required_tier`; the user restarts the server with that tier.
- `handle_unknown` (usage): the job ended with its MCP process; read the state as in "Handles, cursors and pauses".
- `cursor_invalid` (usage): read again from the first page.
- `output_budget_exceeded` (usage): the result passed 24 KiB. Narrow the request: filters, a lower limit, Lab `fields`.
- `answer_exported` (warning): a Lab answer too large to return is whole in the file of `export {path, sha256, size}`; read that file. The command was performed or attempted: never run an effectful Lab command again to see its answer. Give an exported `ac_record_stop` answer to `ac_binding_draft` as `record_stop_export`.
- `governance_client_not_allowed` (warning): actingd's allowed clients do not list `actingctl-mcp`, and the write went ahead. Tell the user.

## Never

- Approve anything or touch `catalog_approval_ids`; edit `actingd.config.json`; start, stop or restart actingd; add, remove or discover instances. Propose it; the user or the UI console does it.
- Edit a client's MCP configuration or enable a tier.
- Take over or release a lease you do not hold, or stop a scheduled run other than through a pause the user agreed to.
- Overwrite an active recording. `--force` is never passed; overwriting is the user's CLI step.
- Resend a write with new arguments, or run an effectful Lab command again to see its answer.
- Send personal data off the machine. Never print the secret salt; treat user names in paths, account names and frames as private.

## Tools

The table generated from `actingctl mcp-serve --list-tools --format markdown` at Runtime `3bf664e0` is in `references/tools.md`. At run time, `tools/list` is authoritative.

- observer (9): `ac_overview`, `ac_events`, `ac_material`, `ac_get_run`, `ac_diagnose`, `ac_resources_list`, `ac_targets_get`, `ac_pack_check`, `ac_catalog_check`
- operator (6): `ac_run_pack`, `ac_stop_run`, `ac_pause`, `ac_resume`, `ac_emulator`, `ac_targets_set`
- author (7): `ac_lab_observe`, `ac_lab_do`, `ac_record_start`, `ac_record_mark`, `ac_record_stop`, `ac_record_status`, `ac_binding_draft`

## References

- `references/tools.md`: the generated tool table.
- `references/gold-samples.md`: a real call sequence for each recipe, with abridged outputs.
- `references/lab-recording.md`: Lab recording, with the tool-to-command map.
- `references/cli-workshop.md`: the command-line manual, for work without the MCP server.
