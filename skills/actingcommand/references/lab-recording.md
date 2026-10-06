# Lab recording

This is the Lab recording part of the `actingcommand` skill: §7 of the CLI manual v0, unchanged except that its references to §1–§5, R2 and R3 now point to `cli-workshop.md`, which also holds the evidence key and the citation paths. The MCP mapping comes first.

## MCP mapping (author tier)

With the server's author tier, the recording commands of §7 are tools. Each tool runs `tools\actinglab.exe --json` of the server's installation (from v0.11.2 `<root>\tools\`, outside the slots, with the server's selection pinned; in v0.11.1 that of the selected slot, `<root>\A\` or `<root>\B\`; before, `<root>\tools\`) with one argument per flag and answers actinglab's data verbatim, with the class taken from actinglab's exit code (2 usage, 3 safety, 4 device, 5 runtime, 6 usage `not_implemented`). When the server knows its state root, it sets `ACTINGCOMMAND_RUNTIME_STATE_ROOT` to it, so `cli-workshop.md` §5 item 8 does not apply.

| Tool | actinglab command |
|---|---|
| `ac_record_start` | `record start`. `--force` is never passed: overwriting an active recording stays the user's CLI step |
| `ac_record_mark` | `record mark --request-json <request>` with one `actingcommand.lab-record-mark.v1` request. The request form carries every capability of the flag forms in §7.2–§7.8: marks, click, samples, offline frame, transition, step remediation, application step, optional step (contracts/lab-recording.md "record mark") |
| `ac_record_status` | `record status` |
| `ac_record_stop` | `record stop`; its dry run is `record stop --dry-run` |
| `ac_lab_observe` | `observe`; with capture and record, `observe --capture --record` |
| `ac_lab_do` | `do`; with capture and record, `do --capture --record` |
| `ac_binding_draft` | No single command. It takes an `ac_record_stop` answer, runs `package preflight` on the recorded package and `actingd check-config --config` on the install's configuration, both read only, and changes nothing |
| `ac_pack_check` (observer) | `package digest` and `package preflight` |
| `ac_catalog_check` (observer) | `resource catalog` |
| `ac_pause`, `ac_resume` (operator) | `actingctl pause` and `resume` (§7.1) |
| none | `capture --record`, `session capture --record`, `session app … --record` and `session instance app … --record`: use the CLI (§7.2, §7.7) |

Rules that come with the tools:

- **Instance.** The tools' `instance` is the actinglab instance alias, passed through as `--instance` the way the CLI takes it.
- **State directory.** In a device recording leave the record tools' state directory unset: `--record` refuses `--state-dir` (§7.1), and a recording kept elsewhere answers `record_flag_reachable: false`.
- **Pause.** `ac_pause` the instance before `ac_record_start`, and `ac_resume` it with that pause's `owner_epoch` and `revision` after the final `ac_record_stop` or when the recording is given up (§7.1).
- **No destructive guard on the capture path.** `ac_lab_do` with capture and without a dry run does not read `destructive` or `allow_destructive`. That is actinglab's own behaviour; the server adds no guard.
- **Answers over the output budget** are written whole to `%TEMP%\actingcommand-mcp\materials\<sha256>.json` and answered as `export {path, sha256, size}` with the warning `answer_exported`. Read that file. A command that acts on the device or the recording was performed or attempted: never run it again to see its answer. An exported `ac_record_stop` answer still carries its binding parts inline when they fit; give `ac_binding_draft` that answer as it is, or `record_stop_export {path, sha256}`.
- **Handles.** A Lab call that outlasts the 25 s call budget answers a `lab_job_…` handle for `ac_get_run`, and actinglab keeps running. The handle lives in this MCP process only (`handle_unknown` afterwards); then read the recording with `ac_record_status`.
- **A second stop** after the package was generated answers `already_generated` without `binding_requires`. `ac_binding_draft` then gives `binding_requires: null` with the warning `binding_requires_unavailable`.
- **Ledger.** Only `ac_lab_observe` and `ac_lab_do` with capture open a Runtime session; the server then records one `client.action` (surface `mcp`) that carries the answer's `req_id`. The record tools change recording files only and record no `client.action`.
- **Real device.** When this was written, the capture paths of these tools had been checked on the fixture runtime only (Workflow #338 S3); their real-device check was still pending. Report what a device run does.

## 7. Lab recording (Runtime v0.10.0)

A Lab recording turns real screens into a `linear_steps` task pack. Each step is one screen with its recognition marks and one effect: a click rectangle or an application operation. The last step only recognizes. `record stop` checks the pack the way actingd will load it, writes it as `<D>.zip` (when it has template images) or `<D>.json`, where D is its content digest, and prints binding snippets (lab-recording.md; linear-steps.md). You record and write the pack. Binding it in `actingd.config.json`, the catalog task and its approval stay with the user (`cli-workshop.md` §1). A recording needs the user's request and the v0.10.0 `actinglab`; `record status` and `actingd suspended` are read only. Commands use the variables of `cli-workshop.md` §2.2 plus `$I = '<alias>'`.

### 7.1 Before and after a recording

- **Pause the instance first, resume it afterwards.** Run `& $Ctl pause --state-root $State --instance $I` before `record start`, and `& $Ctl resume --state-root $State --instance $I` after the final `record stop`, or when you give the recording up. Otherwise a scheduled task between two of your commands acts on another screen (lab-recording.md "`--record` on device commands"). A pause lives in memory only: after a daemon restart, check `status` and pause again (`cli-workshop.md` §5 item 3).
- **Device commands need the daemon and a physical instance.** `capture`, `observe`, `do` and `session app` with `--record` go through the Runtime; `record start`, `mark`, `status` and `stop` work offline. A fixture-simulated instance refuses Lab capture, clicks and application operations; for `session app` the refusal is `fixture_execution_scope_forbidden`.
- **One process per instance.** Recording state changes and `record mark` / `record stop` previews take the lock `<state>\record-<instance>.lock` without waiting. Another holder gives `record_busy` (exit 3, `details.holder`). Do not retry blindly: run `record status`, see which step is open, then decide. A `record mark` without `--step` may otherwise land on a step the other process just opened. `record status` takes no lock. `record_lock_failed` is exit 5. There is no unlock command; the lock ends with its process (lab-recording.md "Recording lock").
- **Do not mix actinglab versions during a recording.** The v0.9.0 actinglab runs `session app … --record` without recording it and does not know the lock (lab-recording.md "`session app … --record`").
- **One state directory.** `--record` commands use `ACTINGLAB_SESSION_STATE_DIR` or the default and refuse `--state-dir` (`record_state_dir_unsupported`). Give no `--state-dir` to any recording command. When `record start` prints `record_flag_reachable: false`, device frames cannot reach the recording; only `record mark --frame` can add frames.
- **Global `--instance $I` on every command.** A recording belongs to one instance (`record_instance_mismatch`).
- **Carrier package.** `observe --capture --record` and `do --capture --record` need one. Any admissible pack of the same game and resolution will do, an earlier Lab pack included. Give it as `--package <dir | D.zip | D.json> --package-ref '<reference>'` (reference from `cli-workshop.md` R3). `capture --record` and `session app … --record` need none.
- **Unknown flags.** `record mark` and `record stop` refuse unknown flags and positional arguments (`validation_failed`, exit 2), unlike most of actinglab (`cli-workshop.md` §5 item 1).

**Device recording and `--dry-run`.** All six forms below declare `dry_run_mode: "refused"` in `capabilities`. A valid recording invocation with `--dry-run` exits 2 with the listed code before recording locks, Runtime access, frame attachment or saved recording changes. The flag is recognized before or after the command. Use the `record mark` and `record stop` previews to check recorded material (§7.4; actinglab-dry-run.md "Commands").

| Recording form | Code with `--dry-run` |
|---|---|
| `capture --record` | `dry_run_unsupported` |
| `session capture --record` | `dry_run_unsupported` |
| `observe --capture --record` | `record_flag_unsupported` |
| `do --capture --record` | `record_flag_unsupported` |
| `session app <action> --record` | `record_flag_unsupported` |
| `session instance app <action> --record` | `record_flag_unsupported` |

### 7.2 Recording a path

```powershell
$I = '<alias>'
$Carrier = '<root>\packages\<game>\<digest>'
$CarrierRef = (& $Lab --json package digest --package $Carrier | ConvertFrom-Json).data.reference | ConvertTo-Json -Compress
& $Ctl pause --state-root $State --instance $I
& $Lab --json --instance $I record start --task-id <task_id> --locale <locale>        # task id ^[a-z0-9][a-z0-9_]{0,63}$
& $Lab --json --instance $I capture --record                                          # opens step 1 on the current screen
& $Lab --json --instance $I record mark --page <name> --template <id>=x,y,w,h --color <id>=x,y,w,h --click-from <id>
& $Lab --json --instance $I do --capture --record --package $Carrier --package-ref $CarrierRef   # presses inside step 1's click rectangle
& $Lab --json --instance $I capture --record                                          # opens step 2 on the screen the click led to
& $Lab --json --instance $I record mark --page <name> --template <id>=x,y,w,h --color <id>=x,y,w,h   # the last step: marks, no click
& $Lab --json --instance $I record status
& $Lab --json --instance $I record stop --dry-run                                     # §7.4
& $Lab --json --instance $I record stop --lab-dir "$Root\packages\<game>"
& $Ctl resume --state-root $State --instance $I
```

- **Marks.** `--template <id>=x,y,w,h` crops the primary frame. `--color <id>=x,y,w,h` takes the region's mean colour. The colour digest, OCR and check families exist only in the request form, `record mark --request-json '<actingcommand.lab-record-mark.v1 JSON>'` or `--request <file>`. Every mark is tested on every live frame of its step; one failure refuses the whole command without saving mark changes (`record_mark_rejected`, exit 3, `details.marks[]`). `record mark --dry-run` previews the mark changes without saving them; the session directory and lock metadata follow §7.4. `--sample <png>` adds another frame of the same screen to test on. Lab has no OCR provider, so an OCR mark is always `not_evaluated`, never passed (lab-recording.md "Marks and the mark-time self-test").
- **Clicks.** `--click x,y,w,h` or `--click-from <mark id>` declares the rectangle; `--click-guard <id>` and `--click-retry <n>` (2..5) are optional. `do --capture --record` presses its centre, or `--tap x,y` inside it. `--tap-rect x,y,w,h` declares the rectangle when the step has none.
- **Outcomes of `do --capture --record`.** Performed: the step closes. Performed with a failure: the step closes, marked `needs_review`, with `record_click_performed_with_failure` (exit 4). Indeterminate: `record_click_indeterminate` (exit 4) and the step stays open. Look at the screen, then press again after `record mark --reopen-step <n>`, or accept the step with `record mark --close-step`. Not performed: nothing is recorded and the Runtime error is returned (lab-recording.md "`--record` on device commands").
- **Step numbers.** Steps are serial numbers that are never reused. The pack numbers the remaining steps 1..n again; `record status` shows that number as `artifact_step`. Remediation acts on the last step only: `--drop-step <n>`, `--reopen-step <n>`, `--close-step`, `--to-transition <n>`. Nothing is deleted.
- **Offline frames.** `record mark --frame <png>` without `--step` adds a whole frame of the recording's size as the next screen. `record mark --step <n> …` adds marks or samples to an existing step.
- **Main interface.** Mark the main interface with `--page home`. That page counts as the main interface for restart packs (§7.7).

### 7.3 Intermediate states: three forms

What happens between a step's effect and the next step (lab-recording.md "Transitions"; linear-steps.md "Intermediate states"):

1. **None** (default): the next step's page is awaited.
2. **Page**: a loading or other recognizable screen that must be seen first. Either capture it as a step, mark it, and turn it into the previous step's transition with `record mark --to-transition <its serial>`, or declare it on the step with the effect: `record mark --step <k> --transition page --frame <png> <marks…> [--transition-timeout-ms <ms>]`. If such a screen can be shorter than one capture interval, it will sometimes be missed and the run fails; use a window instead.
3. **Window**: `record mark --step <k> --transition window --min-ms <a> --max-ms <b>`. Nothing is evaluated for `a` ms after the effect; then the next page is awaited until `b` plus the step's arrival timeout has passed. Use it for a transition that cannot be captured or recognized.

`--transition none` clears a transition, and `--replace-transition` replaces one whole. A transition belongs to a step with an effect (`record_transition_without_click` otherwise).

### 7.4 Check with `--dry-run`, add marks, then stop

1. `record stop --dry-run --lab-dir "$Root\packages\<game>"` runs generation and self-checks in memory: admission by the Runtime rules, the first decision, the cross-check of every page against the recorded frames, and the supplied `--lab-dir` checks. Use the same options as the final stop. It preserves the recording and output artifacts; both recording states stay active. Successful output has `dry_run: true` and `status: "validated"`; `not_started` means there is no session and no package was validated. Report any refusal before deciding whether to proceed. The command still creates its session directory and takes the existing recording lock with holder metadata (§7.1; actinglab-dry-run.md "Bootstrap and transient writes").
2. Read `lab.warnings` and the refusals. Add what they ask for with `record mark --step <n> …`; the stored frames suffice. Run `--dry-run` again.
3. `record stop --lab-dir "$Root\packages\<game>"` writes `<D>.zip` or `<D>.json` there. `--lab-dir` is never guessed: only its last level is created. An existing file with the same content is `present`; with other content it is `record_artifact_name_conflict`, and the file stays untouched. **After the final stop the recording is closed.** A missing mark then means recording again (lab-recording.md "Package generation").

Warnings you must act on:
- `arrival_unconfirmed`: the next step's page also passes on the frames before the click, so a run cannot tell whether the click took effect. Give the next step a mark the earlier screen does not have, or declare a window.
- `entry_overlay_insensitive`: step 1 still matches when darkened (×0.45). A notice dimming step 1 would be taken for step 1, and a prerequisite or return-home pack would not run (§7.6).
- **Dimmed pop-ups in general.** Templates match with `ccoeff_normed` and do not see an overall darkening, so a screen under a darkening pop-up still passes. Add a `--color` mark on a fixed bright area, or a `color_digest` mark through `--request-json`, or declare a window.

Other output to look at: `first_decision` (`would_click` of `step_01_click`, or `not_evaluated` when step 1 involves OCR or the pack has an application step), `cross_check.status` (`partially_evaluated` when OCR was not evaluated), `steps_needing_review`, and `timeouts`. Timeouts are set with `--arrival-timeout-ms` (default 15000), `--application-arrival-timeout-ms` (default 90000) and `--timeout-ms` (computed by default). An explicit value outside 1..1800000 is `validation_failed`. `record stop` needs game, server and locale: give `--game`, `--server` and `--locale` when the recording and the instance configuration do not state them (`record_locale_missing`).

**Self-check coverage.** Preparation uses the Runtime's `PreparedContainedTask` rules, including recognition budgets; first-decision simulation uses the same observation and input-guard owners. The recorder generates no `target_consensus`: `--sample` supplies frames for per-frame cross-checks, not a consensus declaration. These results cover the generated package, not multi-frame execution, live OCR/provider behavior or prerequisite recovery (lab-recording.md "Self-checks").

For an author-supplied declaration, `package preflight --package <directory-or-zip> --package-ref '<reference>'` checks hash-first loading and shared task preparation. It executes no recognition, provider or device; actual sample and guard behavior needs Runtime evidence. `package validate` covers the package format only. In `capabilities`, `record stop`, `record mark` and their `session record` aliases declare `preview`; `package preflight` and `resource catalog` declare `no_effect` (actinglab-dry-run.md "Commands").

### 7.5 Binding snippets

`record stop` prints these in `lab` (lab-recording.md "Output"). Show them to the user; never apply them yourself.

| Field | Use |
|---|---|
| `package_ref` | the `--package-ref` of the pack: `--package <D>.zip --package-ref '<package_ref>'` |
| `task_run_example` | one manual run (`cli-workshop.md` R2), only on the user's request |
| `binding_example` | a `policy.procedure_manifest[]` entry with the absolute `package_path`. It runs on a schedule only with a catalog task of that `procedure_ref` and its approval |
| `prerequisite_entry_example` | the `prerequisite_packages` entry for this pack, when another pack names it with `--requires` (§7.6) |
| `catalog_on_failure_example` | the catalog task's `on_failure`, `{"action":"pause","retry_limit":1,"retry_backoff_ms":0,"escalation_threshold":2}`: a qualifying failure of the pack's own steps is rerun immediately while its original activity window and budget cycle remain valid (R22); the same problem again suspends the task (§7.9; policy-suspension.md "Catalog") |
| `binding_requires` | what the binding also needs, as a checklist |

The user edits the configuration and the catalog. A changed `on_failure` is a catalog change and needs a new approval, which only the user can give. Then `actingd check-config` (read only, `cli-workshop.md` R3) and a daemon restart by the user; the configuration is read at startup.

### 7.6 Where a pack starts: return-home packs and `--requires`

A linear pack runs its path from step 1, and only step 1's page is checked at the start. **Record from the pack's first step** (linear-steps.md "Prerequisite packages", "Return-home fallback").

**Declared target consensus.** A task or recognition pack can declare `target_consensus` for an existing target. For example, this requires two passing samples out of three for `ready` (business-identity-consensus.md "Finite consensus"):

```json
{"target_consensus":{"ready":{"samples":[{"frame":0},{"frame":1},{"frame":2}],"k":2,"sample_interval_ms":50}}}
```

The total sample count is at most five; distinct frame indices are contiguous from zero, and multi-frame intervals are 1..1000 ms. Runtime uses the same target aggregation for current, optional, intermediate and terminal pages, each layer's initial entry check and post-recovery recheck, and input guards. Candidate order still picks the first passing page. Click geometry and the input reference come from the current frame. A pack without a declaration uses one frame.

Preparation checks the worst calls and sample waits against each observation phase's budget and the task limit; an insufficient declaration gives `recognition_sample_budget_insufficient`. Runtime carries the original absolute deadline through samples and repeated observations, and guards use the deadline fixed at step start. Missing samples, expiry, changed geometry, cancellation or lost permission end that observation before any following input. Sampling therefore consumes the existing budget; it does not extend an entry or recovery wait. Lab recording's coverage is described in §7.4 (linear-steps.md "Execution", "Prerequisite packages").

- **Without `--requires`**, a run that does not start on step 1 first runs the return-home pack that `return_home_packages` names for the pack's game and server, then waits for step 1. **So step 1 should be the screen that return-home pack ends on, normally the main interface.** Otherwise every run from elsewhere fails with `contained_task_return_home_entry_unmatched`. Without a configured return-home pack, the failure is `contained_task_linear_entry_unmatched`.
- **A pack recorded from another screen** names the pack that leads to its step 1: `record stop --requires <package_id>`. The user maps that id in `actingd.config.json`: `prerequisite_packages` entries are `{package_id, package_path, package_digest}`, the prerequisite pack's own `prerequisite_entry_example`. `return_home_packages` entries are `{game, server, package_id}` and name an id of `prerequisite_packages`. Both maps are read at startup, so a change needs a restart (actingd-check-config.md "Prerequisite packages").
- **Chain and depth.** A linear prerequisite pack may name its own, up to three declared layers plus the return-home layer. A chain has no cycle, at most 1000 steps in total, and one game, server and resolution. Refusals are `contained_task_prerequisite_*`, before the run starts.
- **Gate codes never start the stuck-recovery ladder.** This covers `contained_task_prerequisite_*` and `contained_task_return_home_entry_unmatched`. They point at a stale marker or a wrong declaration, which a restart does not repair.
- **Give step 1 a colour mark on a fixed bright area,** besides any templates (`entry_overlay_insensitive`, §7.4). Otherwise a notice dimming step 1 passes as step 1, and the prerequisite pack does not run.
- **The end of a chain is best a page-graph pack.** It succeeds with no step when the screen is already on its target page; a linear one fails there.
- **A flow whose first screen appears only sometimes** (for example a notice shown only at the first login of the day) stays a page-graph pack. As a linear pack it fails on every run without that screen, so do not schedule it at a fixed interval (linear-steps.md "What a linear task cannot express").
- A game's resource repository may have no pack that returns to the main interface from any screen (the Blue Archive repository had none when v0.10.0 was frozen). Tell the user; the resource maintainers add one.

### 7.7 Restart packs (application steps)

A step's effect may be `launch`, `restart` or `stop` of the application assigned to the instance; the pack never names the application. A step has one effect (lab-recording.md "Application steps"; application-lifecycle.md).

- **Recording.** On the device: `& $Lab --json --instance $I session app restart --record` (also `launch`, `stop`, `force-stop`, which is recorded as `stop`; `session instance app` works the same way). Offline: `record mark --application <action>`.
- **`--record` never goes with `--dry-run` on `session app`** (`record_flag_unsupported`).
- **Uncertain results.** `record_application_indeterminate` (exit 4) means the operation may have run. Look at the instance, then run the same command again (launch, restart and stop can be repeated) or accept the step with `record mark --close-step`. Only that code and `record_append_failed_after_input` mean it may have run or ran; every other error means it did not. Tell them apart by the code, not the exit code.
- **After `stop`, the next effect must be `launch` or `restart`** (`record_step_after_stop_invalid`).
- **Record up to the main interface.** After the last `launch` or `restart`, a later step must be the main interface, marked `--page home` (`record_application_without_home`): `any → restart → title → home`. Screens a cold start shows only sometimes go in as optional steps between title and home: `any → restart → title → [notice?] → home`. The home step itself is required (§7.8).
- **A pack whose step 1 is the application operation has no entry recognition.** Every run performs that operation first; a restart pack restarts the game every time. It cannot take `--requires`, and no return-home pack runs under it.
- **Recommended configuration:**
  - The restart pack is the instance's `startup_package`, which is also the second rung of the stuck-recovery ladder. It must be a directory: extract `<D>.zip` into `<root>\packages\<game>\<D>\`, or expand a `<D>.json` with the script in package-reference.md "Containers". Confirm with `package digest` that its reference is still D, then propose `"startup_package": {"package": "<that directory>", "expected_sha256": "<D>"}`.
  - A page-graph return-home pack goes in `return_home_packages`.
  - A conditional restart ("restart only on this screen") needs a hand-written page-graph pack.
- **A restart pack as a prerequisite or return-home pack is costly.** It restarts the game whenever the first entry observation above it does not match; that observation uses any declared consensus (§7.6). Its failures do not accumulate toward suspension and may rerun immediately within the original activity window and budget cycle (§7.9).
- **Ladder rung 1.** A return-home pack bound on the request (`--recovery-package`) keeps the default response deadline of 60 s, too short for a restart pack. The pack from `return_home_packages` gets the maximum (emulator-control.md "Stuck-recovery ladder").
- **adb failures** (`application_backend_operation_failed`, `input_backend_operation_failed`) of a scheduled linear task are only rerun (R25-2), unless the run was poisoned.
- **Application steps need a physical instance with an assigned application.** Elsewhere they are refused with `application_effect_requires_assigned_application`, and the offline simulation does not run them.

### 7.8 Optional steps (Workflow #339)

An optional step is a screen that does not always appear after the click of the step before it: a daily sign-in, a notice, a reward card, an update prompt after a cold start. When its page does not appear, the run skips it and records nothing for it (lab-recording.md "Optional steps"; linear-steps.md "Optional steps").

- **Marking.** `record mark … --optional [--settle-ms <0..60000>]`, in the same command as the step's marks and click. Clear it with `--not-optional`.
- **Steps that cannot be optional:** step 1 (`record_optional_first_step`: start one screen earlier, or use `--requires`), the last step (`record_optional_final_step`), an application step (`record_optional_application`) and the main interface after a restart (`record_optional_restart_segment_end`).
- **Settle.** After the page that follows the optional steps (N) first passes, the run keeps watching for a late pop-up for `settle_ms` before it decides that none came. Choose the longest delay seen between N appearing and the pop-up covering it, plus a margin. The default is 2000 ms; it had not been calibrated on a real device when v0.10.0 was frozen. A pop-up later than the settle makes the run fail loudly. A larger settle costs up to that time on every run in which a pop-up of the sequence does not appear.
- **Close-type steps: add `--click-retry 2`.** A swallowed close click then gets a second press once the retry decision sees the pop-up again.
- **N must not pass under the pop-up.** Otherwise `record stop` refuses with `record_optional_skip_target_insensitive`. Give N a `--color` mark on a bright area the pop-up darkens: `record mark --step <N's serial> --color <id>=x,y,w,h`. For a small pop-up that does not darken, mark the area it covers.
- **The pop-up must not pass where it is absent.** Otherwise the refusal is `record_optional_step_ambiguous`, with `against` naming the screen. Give the pop-up a mark of its own, such as its title or a button.
- **One pop-up several times.** Record it as many times as it can appear; each copy runs at most once per run. Reuse its marks with `--reuse <id>` and the same click rectangle. `optional_steps[].same_as` names the earlier copy, with one `arrival_unconfirmed` warning.
- **Inserting a pop-up that did not appear while recording.** You need a whole frame of it at the recording's size: `capture --out <png>` on a day it appears, or a failed run's frame from the evidence store. Right after the step before it was executed:
  1. `record mark --frame <popup.png> --template … --click … --optional [--settle-ms <ms>]`
  2. `record mark --close-step`
  3. `capture --record` of the next screen.

  A second copy from the same PNG also needs `--close-step` first. After the final stop nothing can be inserted: record again (the kept frames can be marked offline).
- **What optional steps cannot express:**
  - A pop-up later than its settle, shown more often than the copies recorded, or never recorded: the run fails loudly.
  - A pop-up already on the screen before the preceding click: use a prerequisite pack, or start earlier.
  - A detour that confirms and comes back to the same page (`A → start → [prompt?] → confirm → A' → start → B`): use a page-graph pack or two packs.
  - Branches and loops.
- **If an unrecorded pop-up gets a task paused:** get its frame and record the pack again with it as an optional step. The new pack lifts the pause once bound (§7.9).

### 7.9 Failures, reruns and suspension

For scheduled `linear_steps` tasks with the recommended `on_failure` (§7.5). See policy-suspension.md.

- **Rerun.** The first admitted normal trigger opens a retry round using its original activity window and budget cycle. While that round is valid, a failed run is recorded and retried at the first evaluation after `retry_backoff_ms`, without waiting for the clock trigger or cooldown. Budget, window, pause, performance and lease/permission gates still apply.
- **When immediate retries end.** Reaching any original task/activity daily or window count limit closes the round. Actual runtime is settled when a run completes; if the remaining original runtime budget cannot reserve the next run's expected duration, the round closes. Another task's use of the same instance's shared activity budget also counts. Pending reservations constrain admission. Window expiry closes the round too; an evaluated retry admitted after expiry is refused with `policy_retry_round_ended`.
- **Which window counts.** An ordinary window ends at its declared end; a full-day window ends at the next local midnight. An overnight window belongs to its opening day, so midnight inside it does not renew its budgets. A different profile, window or expected duration cannot use the original round's immediate permission. A run already admitted keeps its existing deadline and permission across a date boundary.
- **Starting again.** Budget availability alone does not trigger a run or reopen a closed round; the next legal normal trigger opens a new one. Success clears the round. Consecutive failures, a new failure identity and restarting the daemon do not extend it: restart reconstructs the original round from the ledger. Ending immediate permission preserves failure history and classification, backoff and suspension rules (policy-suspension.md "Immediate rerun (R22)").
- **Suspended (`paused_task`) when the same problem repeats.** This applies to two kinds of failure. The first is `page_confirmation_failed`, `contained_task_linear_intermediate_unobserved` or `contained_task_guard_refused` in the pack's own steps, outside the restart segment, with similar error frames twice. The second is a `contained_task_prerequisite_*` refusal while the run is prepared. A severe failure, a sensitive task, an exceeded runtime budget or an interrupted settlement pauses at the first failure.
- **Rerun only, never suspended**, and listed in `repeating` once it fails again: everything else. That includes the gate, prerequisite and return-home packs, a start not on step 1, `contained_task_linear_application_unconfirmed`, the restart segment (after a `launch` or `restart`, before the main interface), adb failures (R25-2), device errors, cancellation, deadlines, the task timeout and recognition errors. Immediate retries remain bounded by the original window and budget cycle. A stale return-home pack shows only in `repeating`; that history is not permission to retry after the round ends.
- **`& $Actingd suspended --config $Config`** is read-only and runs beside the daemon. Run it from the folder the daemon starts in, as `check-config` (`cli-workshop.md` R3). It prints one JSON object with three lists:
  - `suspended[]`: paused pairs, with `step`, `detail`, the error `frames` and their `comparison`, `lifts_when`, and `takeover`, a `task-run` command line for a manual run.
  - `lifted[]`: pairs a package update lifted. `effective` is `pending_restart` until the daemon has read the new configuration.
  - `repeating[]`: pairs failing again and again without being suspended.

  `warnings` include `config_newer_than_daemon_start`. Exit 0 whatever it lists. Exit 1 with `FATAL actingd: <code>` on any error, for example `suspended_policy_unconfigured` without a `policy` section.
- **Lifting, never by hand.** A suspension is lifted in three steps: (1) the pack is updated, for example recorded again, which gives a new D, or the prerequisite or return-home pack is fixed; (2) the user points the binding at it: the procedure binding's `package_digest` and `package_path`, the `prerequisite_packages` entry, or the `return_home_packages` entry; (3) the user restarts actingd. The pause is then lifted automatically. `actingctl task-run` is not gated by the policy and lifts nothing.
- **Known limits.** A dialog that does not darken and covers a medium part of the screen counts as "similar" and can pause the task; look at the frames. The same cause can differ on days with and without an optional pop-up, which costs one more rerun.
