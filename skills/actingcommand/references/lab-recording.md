# Lab recording

This is the Lab recording part of the `actingcommand` skill, for Runtime **0.12.61**. It opens with the owner's two rules for making a package, then maps the recording commands to the tools of the two MCP servers, then keeps the recording rules (§7). Every command, member and code here was taken from the Runtime's contracts and the two MCP servers' tool lists as they are to be released in 0.12.61; a citation `file.md "Section"` names the contract under `contracts/` of the Runtime repository. References to §1–§5, R2 and R3 point to `cli-workshop.md`, which also holds the shell variables (`$Lab`, `$Ctl`, `$State`, `$Root`, `$Config`).

## Rules for making a package

The owner set two rules for making a package. They stand here in her own words, unchanged. The English under each is a rendering, not her words.

> 「铁律一：除极少数复杂循环战斗外，一律只准交 record 录出来的线性包，严禁再手写几千行的 page_graph 状态机。」

*Rendering:* Iron rule one: apart from the very few complex looping battles, hand in only linear packages recorded with `record`; writing page_graph state machines of thousands of lines by hand again is strictly forbidden.

> 「铁律二：只录一遍正常主路径（Happy Path），点完最后一下业务生效立即停！ 严禁在制包时提前去造“余额不足、售罄、背包满”等还没碰到的分支。正常跑起来真遇到了，0.12.61 会自动停在 unrecorded_screen 并留好现场帧，到时候用 R2 对着那张真实截图 1 分钟补一个结局就行」

*Rendering:* Iron rule two: record the normal main path (the happy path) once, and stop as soon as the last press has made the business take effect! While making a package, never build in advance branches not met yet, such as "balance not enough", "sold out" or "bag full". When a normal run really meets one, 0.12.61 stops by itself at unrecorded_screen and keeps the frame of the scene; then it is enough to add an ending with R2 from that real screenshot, in a minute. (R2 is the ending a stopped recording takes from a frame, §7.12.)

What an agent does because of them:

- **Record; do not write a page graph by hand.** The one exception is a complex looping battle: a linear package runs one declared path, with no branch and no loop of variable length (linear-steps.md "What a linear task cannot express"). A business that has to scroll a list to find its item cannot be recorded in 0.12.61: a select reads the fixed slots of one screen, and a list that scrolls is a later slice (lab-recording.md "Selecting in a list"). Such a business waits for that release; it is not written by hand.
- **Record the main path once, and stop when the last press has taken effect.** The screen that press arrives at is the last step: its marks and its `page`, no click (§7.2). Do not build a branch the recording did not meet; a run that meets one later brings its frame (next items).
- **When a run meets a screen the package never recorded** (since 0.12.60; linear-steps.md "Failure codes", "Retries"). After a step's press whose wait failed, the step's closing decision watches the screen for one step timeout and never presses. When the screen stays still and nothing the package declares shows on it, the run ends with `contained_task_linear_unrecorded_screen`; its `artifact_id` is the frame of the run's `Failure` pin, the decision's last frame. A scheduled run pauses its pair (catalog task, instance) at once, with no rerun, whatever its `on_failure`; nothing else is paused, and a direct run pauses nothing (policy-suspension.md "A failure that pauses at its first occurrence"). The decision names no such screen when any of these holds:
  - the screen is not still: the decision's last two captures, or its first and last frames, differ by the Runtime's frame-change measure;
  - a page the failed wait awaited or the step's own page passes on one of its frames, a frame is a loading (transition) frame, a shared interrupt page shows, or another page the package declares passes on its last frame;
  - the run's time left before its deadline or the end of its runtime budget is less than twice the decision's bound (or a scheduled run's budget end could not be read);
  - the run is not the task's own: a prerequisite, return-home, navigation or interrupt layer, a startup package or a ladder rung; or it is inside a restart segment;
  - the failed wait missed the intermediate page or ended on its loading allowance, or the package declares sample consensus.

  Then the run ends with the code the failed wait gave (`page_confirmation_failed` for the usual end), its pair follows its `on_failure` as before, and `run evidence` still lists the run's frames.
- **From that failure to an ending**, as the code's catalog entry gives its handling (read it with `ac_outcome_code`, or `ac_codes` on the Lab's server):
  1. `actinglab run evidence <request_id>` (MCP `ac_run_evidence`) lists the run's frames as material refs `<seq>:<event_id>:<artifact_id>`. The `artifact_id` alone is no material ref; the ref whose last part is it is the pinned frame.
  2. `actinglab material <ref> --out <dir>` writes it as `<dir>/<sha256>.png` (MCP `ac_material` with export writes the same file into the server's materials folder). Take it in time: a pinned frame nobody confirmed as read is removed once `frame_retention_evidence_max_age_days` (15 by default) have passed since its ingestion, and one confirmed as read at the ledger store's next pass.
  3. On the stopped recording, `actinglab record mark --ending --frame <png> --outcome-key <key>` with a mark of that screen only, such as `--template` of its message (MCP: `ac_record_mark` with an `ending` request, §7.12). Give the file as `material` wrote it: a frame of another resolution that is a pure scale of the recording's is converted by the Lab itself.
  4. `actinglab record stop` regenerates the package, which then ends such a run Completed on that screen with that key, its message left on the screen. A changed package content lifts the pause at the pair's next due evaluation; `actingd suspended` and `ac_diagnose` list the paused pair until then.

  An ending is only for a screen that ends the business after the path's last press (too little balance, nothing to collect). Every other answer is a new recording. A pop-up in the middle of the path, after which the path goes on, is an optional step; a stopped recording takes no new step, so the package is recorded again, and the new recording can be assembled offline from the frames the earlier one kept, with the pop-up's frame inserted as an optional step (§7.8). A result after an earlier press, or a branch that continues the business, is a step and a page. A slow screen nobody declared is declared as a page or a loading page.
- **In a list, choose by the name read, never by the position** (§7.11). The choice is confirmed on a fresh frame before its one tap. A wanted item that is not there ends the run normally, Completed with the select's `on_empty` key, a result with a name; an item that cannot be read and could change the choice fails the run (`selection_result_blocked`) and is never guessed.
- **A regenerated package reaches a running installation only through the game's resource repository and a placement** (§7.14).

## MCP mapping (author tier)

Two MCP servers serve the recording. Today's server, `actingctl mcp-serve`, has tiers: its author tier lists 12 tools. The Lab's server, `actinglab-mcp`, has no tiers and names a tool `ac_` and the operation row's words with spaces and `-` as `_` (`contracts/lab-operations.json`); it lists its authoring tools where the configuration's `lab_module_authoring` is `true`. Each tool runs the actinglab command of the server's installation and answers its data verbatim (runtime-interface.md "MCP (counter `mcp`)"; lab-two-entries.md). At run time `tools/list` is authoritative.

| actinglab command | `actingctl mcp-serve` | `actinglab-mcp` |
|---|---|---|
| `record start` | `ac_record_start` | `ac_record_start` |
| `record mark` with one `actingcommand.lab-record-mark.v1` request: marks, a click, a select or `clear_select`, samples, a transition, a step action; on a stopped recording also `ending` or `remove_ending` | `ac_record_mark` (`request`, `dry_run`) | `ac_record_mark` (`request_json` or `request`, a file) |
| `record status` | `ac_record_status` | `ac_record_status` |
| `record stop`; its dry run `record stop --dry-run` | `ac_record_stop` (`dry_run`) | `ac_record_stop` |
| `record archive`, `record reopen` | `ac_record_archive`, `ac_record_reopen` | the same names |
| `record candidates`, `record amend`, `record step`: the session's own step list, not the Lab recording's marks | `ac_record_candidates`, `ac_record_amend`, `ac_record_step` | the same names |
| `observe --capture --record` | `ac_lab_observe` (`capture`, `record`, `package`) | `ac_lab_observe` |
| `do --capture --record`, one call per step (§7.2) | `ac_lab_do` (`capture`, `record`, `package` or `zip`, `mark`, `next_frame_after_ms`, `next_frame_out`) | `ac_do` (`mark_request_json` or `mark_request` in place of `mark`) |
| `record draft`: the binding parts of a `record stop` answer, `package preflight` and `actingd check-config`, read only | `ac_binding_draft` | `ac_record_draft` |
| `capture --record`, `session capture --record`, `session app … --record`, `session instance app … --record` | none (`ac_app` records no step) | none |
| `run evidence`, `material` (observer) | `ac_run_evidence`, `ac_material` (`mode: export`) | `ac_run_evidence`, `ac_material` (`export: true`) |
| `package check`, `resource catalog` (observer) | `ac_pack_check`, `ac_catalog_check` | `ac_package_check`, `ac_resource_catalog` |
| `goals`, `goals get`, `goals set`, `goals clear`, `package variables` (§7.13) | none | `ac_goals`, `ac_goals_get`, `ac_goals_set`, `ac_goals_clear`, `ac_package_variables` |
| `standard-pack adopt`, `build`, `check`, `ready` (§7.14) | none | `ac_standard_pack_adopt`, `ac_standard_pack_build`, `ac_standard_pack_check`, `ac_standard_pack_ready` |
| `actingctl pause`, `resume` (operator, §7.1) | `ac_pause`, `ac_resume` | `ac_pause`, `ac_resume` |

Rules that come with the tools:

- **Instance.** The tools' `instance` is the actinglab instance alias, passed as `--instance`.
- **State directory.** In a device recording leave the record tools' state directory unset: `--record` refuses `--state-dir` (§7.1), and a recording kept elsewhere answers `record_flag_reachable: false`.
- **Pause.** `ac_pause` the instance before `ac_record_start`, and `ac_resume` it with that pause's `owner_epoch` and `revision` after the final `ac_record_stop` or when the recording is given up (§7.1).
- **Destructive guard flags.** `ac_lab_do` with capture and without a dry run refuses `destructive` and `allow_destructive` as `conflicting` before actinglab runs; offline and with a dry run they are read as before.
- **Answers over the output budget** are written whole to `%TEMP%\actingcommand-mcp\materials\<sha256>.json` and answered as `export {path, sha256, size}` with the warning `answer_exported`. Read that file. A command that acts on the device or the recording was performed or attempted: never run it again to see its answer. An exported `ac_record_stop` answer still carries its binding parts inline when they fit; give it to `ac_binding_draft` (`ac_record_draft`) as it is, or as `record_stop_export {path, sha256}`.
- **Handles.** A Lab call that outlasts the call budget answers a handle for `ac_get_run` (`ac_run_show` on the Lab's server), and actinglab keeps running. The handle lives in that MCP process only (`handle_unknown` afterwards); then read the recording with `ac_record_status`.
- **A second stop** answers `already_generated` without `binding_requires`; after `ac_record_mark` on the stopped recording it generates again and answers `regenerated` (§7.4).
- **Ledger.** `ac_lab_observe` and `ac_lab_do` (`ac_do`) with capture reach the Runtime, and the server then records one client action naming the Lab request. `ac_record_start`, `ac_record_mark`, `ac_record_status` and `ac_record_stop` change or read recording files only.

## 7. Lab recording (Runtime 0.12.61)

A Lab recording turns real screens into a `linear_steps` task package. Each step is one screen with its recognition marks and at most one effect: a click (a tap, a drag, a long press, a swipe), a select, or an application operation. The last step only recognizes. `record stop` checks the package the way actingd will load it, writes it as `<package_id>.zip` (when it has template marks) or `<package_id>.json`, and prints binding parts (lab-recording.md; linear-steps.md). You record and write the package; binding it, the catalog task and its approval stay with the user (`cli-workshop.md` §1). A recording needs the user's request. Commands use the variables of `cli-workshop.md` §2.2 plus `$I = '<alias>'`.

### 7.1 Before and after a recording

- **Pause the instance first, resume it afterwards.** Run `& $Ctl pause --state-root $State --instance $I` before `record start`, and `& $Ctl resume --state-root $State --instance $I` after the final `record stop`, or when you give the recording up. Otherwise a scheduled task between two of your commands acts on another screen (lab-recording.md "`--record` on device commands"). A pause survives an actingd restart with a new `owner_epoch` and revision 1: read both before you resume (`SKILL.md`, "Handles, cursors and pauses").
- **Device commands need the daemon and a physical instance.** `capture`, `observe`, `do` and `session app` with `--record` go through the Runtime; `record start`, `mark`, `status` and `stop` work offline. On a fixture instance `session app … --record` is refused (`fixture_execution_scope_forbidden`), and application steps need a physical instance (§7.7).
- **One process per instance.** Every state-writing recording command takes `<state>\record-<instance>.lock` without waiting; another holder gives `record_busy` (exit 3, `details.holder`). Do not retry blindly: run `record status`, see which step is open, then decide; a `record mark` without `--step` may otherwise land on a step the other process just opened. `record status` takes no lock. `record_lock_failed` is exit 5. There is no unlock command; the lock ends with its process (lab-recording.md "Recording lock").
- **Do not mix actinglab versions during a recording.** A build before 0.12.61 does not decode a recording that holds a select or an ending (`record_lab_unavailable`, reason `lab_recording_unreadable`), and the file is left as it is (lab-recording.md "recording.json").
- **One state directory.** `--record` commands use `ACTINGLAB_SESSION_STATE_DIR` or the default and refuse `--state-dir` (`record_state_dir_unsupported`). Give no `--state-dir` to any recording command. When `record start` prints `record_flag_reachable: false`, device frames cannot reach the recording; only `record mark --frame` can add frames.
- **Global `--instance $I` on every command.** A recording belongs to one instance (`record_instance_mismatch`).
- **Carrier package.** `observe --capture --record` and `do --capture --record` need one: any admissible package of the same game and resolution, an earlier Lab package included, given as `--package <directory | .zip | .json>` or `--zip <z>`. It carries the Runtime request only; the press is planned from the recording. No package reference is given. `capture --record` and `session app … --record` need none.
- **Unknown flags.** `record mark` and `record stop` refuse unknown flags and positional arguments (`validation_failed`, exit 2), so a mistyped `--lab-dir` never skips the package directory.

**Device recording and `--dry-run`.** All six device recording forms refuse `--dry-run` with exit 2 before recording locks, Runtime access, frame attachment or recording changes, wherever the flag stands. Use the `record mark` and `record stop` previews to check recorded material (§7.4; actinglab-dry-run.md "Commands").

| Recording form | Code with `--dry-run` |
|---|---|
| `capture --record` | `dry_run_unsupported` |
| `session capture --record` | `dry_run_unsupported` |
| `observe --capture --record` | `record_flag_unsupported` |
| `do --capture --record` | `record_flag_unsupported` |
| `session app <action> --record` | `record_flag_unsupported` |
| `session instance app <action> --record` | `record_flag_unsupported` |

### 7.2 Recording a path: one call per step

`do --capture --record` with a mark request records a whole step in one call: it marks the open step (its marks, its `page`, its click), presses, and with `--next-frame-after-ms` takes the next step's frame (lab-recording.md "One call per step"). A recording of n steps with a press takes n + 5 calls: `record start`, the first frame, one call per step, the last screen's marks, `record stop --dry-run` and `record stop`.

```powershell
$I = '<alias>'
$Carrier = '<root>\packages\<game>\<package_id>.zip'       # any admissible package of the same game and resolution
& $Ctl pause --state-root $State --instance $I
& $Lab --json --instance $I record start --task-id <task_id> --locale <locale>   # task id ^[a-z0-9][a-z0-9_]{0,63}$
& $Lab --json --instance $I capture --record                                     # opens step 1 on the current screen
& $Lab --json --instance $I do --capture --record --package $Carrier `
  --mark-request '<dir>\step-1.json' --next-frame-after-ms 800 --next-frame-out '<dir>\step-2.png'
& $Lab --json --instance $I do --capture --record --package $Carrier `
  --mark-request '<dir>\step-2.json' --next-frame-after-ms 800 --next-frame-out '<dir>\step-3.png'
& $Lab --json --instance $I record mark --request '<dir>\final.json'             # the last screen: marks and page, no click
& $Lab --json --instance $I record stop --dry-run --lab-dir "$Root\packages\<game>"   # §7.4
& $Lab --json --instance $I record stop --lab-dir "$Root\packages\<game>"
& $Ctl resume --state-root $State --instance $I
```

`step-1.json` marks the screen in front of you and declares its click; write each next file after reading the frame the call before wrote, UTF-8 without a byte order mark. One rectangle given once is both the template's crop and the click (`"click": {"from": "<mark id>"}`):

```json
{"schema_version":"actingcommand.lab-record-mark.v1","page":"title",
 "add":[{"id":"ui/start","family":"template","region":{"x":1180,"y":664,"width":73,"height":25}},
        {"id":"state/start","family":"color","region":{"x":1173,"y":680,"width":1,"height":1}}],
 "click":{"from":"ui/start"}}
```

Through MCP the same: `ac_record_start`, `ac_lab_observe` with `capture` and `record` (and `package`), then for each step `ac_lab_do` with `{"instance": "<alias>", "capture": true, "record": true, "package": "<carrier>", "mark": {…the request…}, "next_frame_after_ms": 800, "next_frame_out": "<dir>\\step-2.png"}` (on the Lab's server `ac_do`, the request as `mark_request_json` text or a `mark_request` file), then `ac_record_mark` of the last screen, `ac_record_stop` with `dry_run`, and `ac_record_stop`.

- **The open step and nothing else.** The mark request is the request `record mark` takes, applied by its code with its self-test before anything is pressed: a refusal is `record mark`'s own, and nothing is pressed or written. A request with `step_action` or `transition`, another step, or a frame file that opens a step is `validation_failed` (`details.member`): send it with `record mark` (`ac_record_mark`), then press.
- **Marks.** `--template <id>=x,y,w,h[@x,y,w,h]` crops the primary frame (the part after `@` is its search area; by default the crop grown by 48 px). `--color <id>=x,y,w,h` takes the region's mean colour. The colour digest, OCR, check and value-test families, located text (§7.10) and select (§7.11) exist only in the request form. Every mark is tested on every live frame of its step; one failure refuses the whole command and nothing is written (`record_mark_rejected`, exit 3, `details.marks[]`). `record mark --dry-run` previews; `--sample <png>` adds another frame of the same screen to test on. The Lab loads the Runtime's OCR provider (on the CPU); without a usable OCR model an OCR mark is `not_evaluated` (`lab_ocr_unavailable`), never passed (lab-recording.md "OCR in the Lab").
- **Clicks.** `--click x,y,w,h`, `--click-from <mark id>` or `--click-follow <mark id>` (§7.10) declares the rectangle; `--click-guard <id>` and `--click-retry <n>` (2..5) are optional. Drags (`--drag-to`, `--duration-ms`) and long presses (`--long-press`) are pressed as one swipe or long tap. `--swipe x1,y1,x2,y2,ms` on a step without a click records that swipe as its effect. A brake drag or path drag is not pressed by the Lab (`record_gesture_execution_unsupported`): reach the next screen otherwise and close the step with an offline frame.
- **Outcomes of the press.** Performed: the step closes. Performed with a failure: the step closes, marked `needs_review`, with `record_click_performed_with_failure` (exit 4). Indeterminate: `record_click_indeterminate` (exit 4) and the step stays open; look at the screen, then press again after `record mark --reopen-step <n>`, or accept it with `record mark --close-step`. Not performed (also refused by the Runtime, or unanswered): the marks stay written, the click has no outcome, and the operation's own error is returned. `record_append_failed_after_input` (exit 3) means the press was sent.
- **Repeating a press.** The same call sent again is refused by `record mark`'s own check (`record_click_exists`, `record_mark_id_conflict`) and presses nothing twice. Look at the screen first (`observe --capture` without `--record`, `ac_lab_observe` with capture and without record) or read `record status`. Only while the screen still shows the step's own page, send the same call without the mark request (for a step without a click, its `--tap-rect` or `--swipe` kept). If it shows the page the press leads to, the press was performed: do not send it again.
- **A next frame that was not taken.** The press and its closed step stand; take the frame with `capture --record` or `ac_lab_observe` with capture and record, and do not press again. Both a press not performed and a next frame not taken can be `device_error`: the second's details carry `record` (`step_closed` true) and `next_frame.taken` false. Today's server with `--detail codes` shows them alike; read `ac_record_status` before you press again.
- **The last screen** is a step of its own: a `record mark` request with its marks and `page` and no click. Its `page` names the package's terminal page, `step_<nn>_<page>`. `record stop` refuses a last step with a click (`record_final_step_has_click`) and any step without a mark (`record_step_without_recognition`).
- **Step numbers.** Steps are serial numbers that are never reused. The package numbers the remaining steps 1..n again; `record status` shows that number as `artifact_step`. Remediation acts on the last step only: `--drop-step <n>`, `--reopen-step <n>`, `--close-step`, `--to-transition <n>`. Nothing is deleted.
- **Offline frames.** `record mark --frame <png>` without `--step` adds a whole frame of the recording's size as the next screen. `record mark --step <n> …` adds marks or samples to an existing step.
- **Main interface.** Mark the main interface with `--page home`. That page counts as the main interface for restart packages (§7.7).

### 7.3 Intermediate states: three forms

What happens between a step's effect and the next step (lab-recording.md "Transitions"; linear-steps.md "Intermediate states"):

1. **None** (default): the next step's page is awaited.
2. **Page**: a loading or other recognizable screen that must be seen first. Either capture it as a step, mark it, and turn it into the previous step's transition with `record mark --to-transition <its serial>`, or declare it on the step with the effect: `record mark --step <k> --transition page --frame <png> <marks…> [--transition-timeout-ms <ms>]`. If such a screen can be shorter than one capture interval, it will sometimes be missed and the run fails; use a window instead.
3. **Window**: `record mark --step <k> --transition window --min-ms <a> --max-ms <b>`. Nothing is evaluated for `a` ms after the effect; then the next page is awaited. Use it for a transition that cannot be captured or recognized.

`--transition none` clears a transition, and `--replace-transition` replaces one whole. A transition belongs to a step with an effect, a click or an application operation (`record_transition_without_click` otherwise).

### 7.4 Check with `--dry-run`, add marks, then stop

1. `record stop --dry-run --lab-dir "$Root\packages\<game>"` runs the pre-checks, generation, admission, the first decision, the cross-check and the `--lab-dir` checks in memory. Use the same options as the final stop. It writes nothing and both states stay active; its output has `status: "validated"`, `lab.would_write[]` and `lab.lab_dir_status` (`to_write` or `to_replace`). Report any refusal before deciding whether to proceed.
2. Read `lab.warnings` and the refusals. Add what they ask for with `record mark --step <n> …`; the stored frames suffice. Run `--dry-run` again.
3. `record stop --lab-dir "$Root\packages\<game>"` writes `<package_id>.zip` or `<package_id>.json` there (the package id defaults to `<game>.<server>.<task_id>`; `--package-id` sets it). `--lab-dir` is never guessed: only its last level is created. A file of that name is replaced in place (`lab_dir_status: "replaced"`); no digest names the package (lab-recording.md "Container and name", "Writing").
4. **After the stop** the recording is stopped, and it still takes `record mark` on its existing steps: marks, samples, pages, clicks, guards, retries, transitions, optional steps, a select (§7.11) and an ending (§7.12). The next `record stop` generates the package again (`lab.status: "regenerated"`). It takes no new step, no step operation and no application operation (`record_session_not_active`, reason `recording_stopped`): a missing step means recording again (lab-recording.md "Marking after stop").

Warnings you must act on:
- `arrival_unconfirmed`: the next step's page also passes on the frames before the click, so a run cannot tell whether the click took effect. Give the next step a mark the earlier screen does not have, or declare a window.
- `entry_overlay_insensitive`: step 1 still matches when darkened (×0.45). A notice dimming step 1 would be taken for step 1, and a prerequisite or return-home package would not run (§7.6).
- **Dimmed pop-ups in general.** Templates match with `ccoeff_normed` and do not see an overall darkening, so a screen under a darkening pop-up still passes. Add a `--color` mark on a fixed bright area, or a `color_digest` mark through the request form, or declare a window.

Other output to look at: `first_decision` (`would_click` of `step_01_click`; `would_end_empty` for a select at step 1 that finds nothing; `not_evaluated` with `lab_ocr_unavailable` or `lab_application_effect_offline`), `cross_check.status` (`partially_evaluated` when OCR was not evaluated), `names_changed_after_press` (§7.10), `steps_needing_review`, and `timeouts`. Timeouts are set with `--arrival-timeout-ms` (default 15000), `--application-arrival-timeout-ms` (default 90000) and `--timeout-ms` (computed by default); an explicit value outside 1..1800000 is `validation_failed`. `record stop` needs game, server and locale: give `--game`, `--server` and `--locale` when the recording and the instance configuration do not state them (`record_locale_missing`).

**Self-check coverage.** Preparation uses the Runtime's `PreparedContainedTask` rules; the first decision uses the same observation and guard owners. The recorder generates no `target_consensus`. These results cover the generated package, not multi-frame execution, live provider behaviour or prerequisite recovery (lab-recording.md "Self-checks"). `package preflight --package <path>` loads a package with every file and prepares its entry task, executing nothing; `package check --package <path>` (`ac_pack_check`) is the Runtime's own structural admission.

### 7.5 Binding parts

`record stop` prints these in `lab` (lab-recording.md "Output"). Show them to the user; never apply them yourself.

| Field | Use |
|---|---|
| `package_id`, `path` | the package, which loads as `--package <package_id>.<ext>`; no member names a digest or a package reference |
| `task_run_example` | one manual run (`cli-workshop.md` R2), only on the user's request |
| `binding_example` | a `policy.procedure_manifest[]` entry with the absolute `package_path`. It runs on a schedule only with a catalog task of that `procedure_ref` and its approval |
| `prerequisite_entry_example` | `{bundle, pack, directory}`: where the package goes in its game's standard-pack registry (`<standard_packs.root>/<game>/bundle.json` and its `packs[]` entry), so that another package can name it with `--requires` (§7.6) |
| `catalog_on_failure_example` | the catalog task's `on_failure`, `{"action":"pause","retry_limit":1,"retry_backoff_ms":0,"escalation_threshold":2}` (§7.9; policy-suspension.md "Catalog") |
| `binding_requires` | what a binding also needs, as a checklist: the catalog task and its approval, the prerequisite or return-home fallback, the recommended `on_failure`, the application and optional-step limits, and with endings that the package is no prerequisite or return-home package and needs a Runtime that admits ending pages |

`ac_binding_draft` turns this answer into a draft with `package preflight` and `actingd check-config`, changes nothing, and lists the person's steps in `manual_steps`; the Lab's `record draft` (`ac_record_draft`) is that tool ported. A changed `on_failure` is a catalog change and needs a new approval, which only the user can give.

### 7.6 Where a package starts: return-home packages and `--requires`

A linear package runs its path from step 1, and only step 1's page is checked at the start. **Record from the package's first step** (linear-steps.md "Prerequisite packages", "Return-home fallback").

- **Without `--requires`**, a run that does not start on step 1 first runs the return-home package of its game and server, the `maintenance` entry of the game's `bundle.json` whose `uses` holds `return_home`, then waits for step 1. **So step 1 should be the screen that return-home package ends on, normally the main interface.** Otherwise every run from elsewhere fails with `contained_task_return_home_entry_unmatched`; without a return-home package the failure is `contained_task_linear_entry_unmatched`. When the game root's navigation data lists the package's entry, its gate navigates instead.
- **A package recorded from another screen** names the package that leads to its step 1: `record stop --requires <package_id>`. The id is resolved in the standard-pack registry of the package's own game (`packs[]` of `<standard_packs.root>/<game>/bundle.json`) when the run is prepared, with no configuration entry and no restart. The retired actingd keys `prerequisite_packages` and `return_home_packages` make a configuration fail (`config_decode_failed`).
- **Chain and depth.** A prerequisite package may name its own, up to three declared layers plus the return-home layer, at most 1000 steps in all, one game, server and resolution. Refusals are `contained_task_prerequisite_*`, before the run starts. A package with an ending or a select that may end empty is no layer (`contained_task_prerequisite_incompatible`, `reason=linear_endings` or `select_on_empty`).
- **Gate codes never start the stuck-recovery ladder.** This covers `contained_task_prerequisite_*` and `contained_task_return_home_entry_unmatched`. They point at a stale return-home package or a wrong declaration.
- **Give step 1 a colour mark on a fixed bright area,** besides any templates (`entry_overlay_insensitive`, §7.4). Otherwise a notice dimming step 1 passes as step 1, and the prerequisite package does not run.
- **Without a prerequisite package or return-home fallback**, a run that does not start on step 1's page, one already on the target page included, fails with `contained_task_linear_entry_unmatched`. The first step cannot be optional: a path whose first screen appears only sometimes starts one screen earlier or names a prerequisite package, and is not suited to a fixed-interval schedule (linear-steps.md "What a linear task cannot express").

### 7.7 Restart packages (application steps)

A step's effect may be `launch`, `restart` or `stop` of the application assigned to the instance; the package never names the application. A step has one effect (lab-recording.md "Application steps"; application-lifecycle.md).

- **Recording.** On the device: `& $Lab --json --instance $I session app restart --record` (also `launch`, `stop`, `force-stop`, which is recorded as `stop`; `session instance app` works the same way). Offline: `record mark --application <action>`, or the request member `"application": {"action": "restart"}` through `ac_record_mark`. No MCP tool records `session app`.
- **`--record` never goes with `--dry-run` on `session app`** (`record_flag_unsupported`).
- **Uncertain results.** `record_application_indeterminate` (exit 4) means the operation may have run. Look at the instance, then run the same command again (launch, restart and stop can be repeated) or accept the step with `record mark --close-step`. Only that code and `record_append_failed_after_input` mean it may have run or ran; every other error means it did not. Tell them apart by the code, not the exit code.
- **After `stop`, the next effect must be `launch` or `restart`** (`record_step_after_stop_invalid`).
- **Record up to the main interface.** After the last `launch` or `restart`, a later step must be the main interface, marked `--page home` (`record_application_without_home`): `any → restart → title → home`. Screens a cold start shows only sometimes go in as optional steps between title and home: `any → restart → title → [notice?] → home`. The home step itself is required (§7.8).
- **A package whose step 1 is the application operation has no entry recognition.** Every run performs that operation first; a restart package restarts the game every time. It cannot take `--requires` (`record_requires_invalid`, `application_entry`), and no return-home package runs under it. There is no conditional restart.
- **Where it goes.** `binding_requires` says such a package is best configured as the instance's `startup_package`, unpacked as a directory; that declaration names the package by its path only (one that still names `expected_sha256` fails `config_decode_failed`, actingd-check-config.md). A game's standard pack declares its startup and return-home packages as `maintenance` entries with `uses` `startup` and `return_home` (package-reference.md "Maintenance declarations").
- **adb failures** (`application_backend_operation_failed`, `input_backend_operation_failed`) of a scheduled linear task are only rerun (R25-2), unless the run was poisoned.
- **Application steps need a physical instance with an assigned application.** Elsewhere they are refused with `application_effect_requires_assigned_application`, and the offline simulation does not run them.

### 7.8 Optional steps

An optional step is a screen that does not always appear after the click of the step before it: a daily sign-in, a notice, a reward card, an update prompt after a cold start. When its page does not appear, the run skips it and records nothing for it (lab-recording.md "Optional steps"; linear-steps.md "Optional steps").

- **Marking.** `record mark … --optional [--settle-ms <0..60000>]`, in the same command as the step's marks and click (request form `"optional": true`, `"optional_settle_ms"`). Clear it with `--not-optional`. A stopped recording takes it on its existing steps too.
- **Steps that cannot be optional:** step 1 (`record_optional_first_step`: start one screen earlier, or use `--requires`), the last step (`record_optional_final_step`), an application step (`record_optional_application`), the main interface after a restart (`record_optional_restart_segment_end`), and a select step.
- **Settle.** After the page that follows the optional steps (N) first passes, the run keeps watching for a late pop-up for `settle_ms` before it decides that none came; the default is 2000 ms. Choose the longest delay seen between N appearing and the pop-up covering it, plus a margin. A larger settle costs up to that time on every run in which a pop-up of the sequence does not appear.
- **Close-type steps: add `--click-retry 2`.** A swallowed close click then gets a second press once the retry decision sees the pop-up again.
- **N must not pass under the pop-up.** Otherwise `record stop` refuses with `record_optional_skip_target_insensitive`. Give N a `--color` mark on a bright area the pop-up darkens: `record mark --step <N's serial> --color <id>=x,y,w,h`. For a small pop-up that does not darken, mark the area it covers.
- **The pop-up must not pass where it is absent.** Otherwise the refusal is `record_optional_step_ambiguous`, with `against` naming the screen. Give the pop-up a mark of its own, such as its title or a button.
- **One pop-up several times.** Record it as many times as it can appear; each copy runs at most once per run. Reuse its marks with `--reuse <id>` and the same click rectangle. `optional_steps[].same_as` names the earlier copy, with one `arrival_unconfirmed` warning.
- **Inserting a pop-up that did not appear while recording.** You need a whole frame of it at the recording's size: `capture --out <png>` on a day it appears, or a failed run's frame from the evidence (§ "Rules for making a package"). Right after the step before it was executed:
  1. `record mark --frame <popup.png> --template … --click … --optional [--settle-ms <ms>]`
  2. `record mark --close-step`
  3. `capture --record` of the next screen.

  A second copy from the same PNG also needs `--close-step` first. A stopped recording takes no new step: a pop-up found missing after the stop means recording again, which can be assembled offline from frames kept with the earlier recording.
- **What optional steps cannot express:**
  - A pop-up later than its settle, shown more often than the copies recorded, or never recorded: the run fails loudly, or ends with `contained_task_linear_unrecorded_screen` (§ "Rules for making a package").
  - A pop-up already on the screen before the preceding click: use a prerequisite package, or start earlier.
  - A detour that confirms and comes back to the same page (`A → start → [prompt?] → confirm → A' → start → B`): the contract's answers are two packages or a page-graph package; under the first rule, record two packages.
  - Branches and loops.

### 7.9 Failures, reruns and suspension

For scheduled `linear_steps` tasks with the recommended `on_failure` (§7.5). See policy-suspension.md.

- **Rerun (R22).** A failure that carries a failure identity is rerun at the first evaluation after `retry_backoff_ms`, without waiting for its trigger or cooldown, while the retry round of its original activity window and budget cycle lasts. Budgets, windows, pause, performance and lease gates still apply. Reaching an original count limit, a runtime budget that cannot reserve the next run, or the window's end closes the round (`policy_retry_round_ended`). Budget availability alone opens no round; success clears it; a restart rebuilds it from the ledger.
- **Suspended (`paused_task`) when the same problem repeats.** `page_confirmation_failed`, `contained_task_linear_intermediate_unobserved` or `contained_task_guard_refused` in the package's own steps, outside the restart segment, twice with the same failure identity; a `contained_task_prerequisite_*` refusal while the run is prepared; the navigation and interrupt codes of a standard pack. A severe failure, a sensitive task and an exceeded runtime budget pause at the first failure.
- **Paused at its first occurrence:** `contained_task_linear_unrecorded_screen`, with no rerun (§ "Rules for making a package").
- **Rerun only, never suspended**, and listed in `repeating` once it fails again: everything else. That includes the gate, prerequisite and return-home packages, a start not on step 1, `contained_task_linear_application_unconfirmed`, the restart segment (after a `launch` or `restart`, before the main interface), adb failures (R25-2), device errors, cancellation, deadlines, the task timeout and recognition errors.
- **A success, never a failure:** a run that ends on a select's `on_empty` key (§7.11) or on an ending page (§7.12).
- **`& $Actingd suspended --config $Config`** is read-only and runs beside the daemon; `ac_diagnose` shows the instance's rows of it. It lists `suspended` (paused pairs, with the failure, its frames and `lifts_when`), `lifted` (pairs a package change lifted) and `repeating` (pairs failing again and again without being suspended).
- **Lifting, never by hand.** A paused pair is admitted again once a lift check finds that the content hash of its package, at the place its binding names, or of a layer it names, differs from what the failure recorded: a package changed in place lifts the pause at the pair's first due evaluation after it, without a restart and without a configuration change (policy-suspension.md "Lifting (R19)"). There is no manual lift; `actingctl task-run` is not gated by the policy and lifts nothing.
- **Known limit.** One cause can fail at the step before a run of optional steps on a day without a pop-up and at the optional step on a day with one, so the two count apart; this favours rerunning and hides no failure.

### 7.10 Pressing what is found by its name

A click may follow a mark instead of a fixed rectangle (lab-recording.md "Following a template", "Following a located text"):

- **A template** (`--click-follow <id>`, `"click": {"follow": "<id>"}`): the package presses the box the template matches on the frame it runs on, for a button whose position moves; its search area should cover every position the button moves to. Declare `unique` with a margin when a second copy could appear.
- **A located text**: an OCR mark of the request form that declares `locate` (`normalize` `identity_v1`, `blocks` `detect` or `region`, `line_merge`, optional `cells`, and optional `unique`); the click follows it, and the package presses the box (or the cell) its names are found in. There is no shortcut flag.
- **The guard, not the page.** The followed located text is the click's guard and no mark of the step's page, so "the wanted name is not on the screen today" never looks like "the package does not know this screen". The step needs another mark that tells its screen from the other screens of the task, such as a template of its title; without one it is `record_click_follow_invalid`, `page_needs_followed_text`.
- **OCR is required.** The self-test reads the names on every live frame with the Lab's OCR provider; without an OCR model `record mark` on that step is refused `lab_ocr_unavailable` (exit 3). A name that is on no recorded frame cannot be declared (`text_locate_not_found`): record a sample that shows it.
- **At run time** the guard is evaluated again on one fresh frame of the step's page, and the press is drawn from that frame's box (`input_recheck: fresh_frame`). A name not on the screen fails the run with `contained_task_guard_refused` and `cause=text_locate_not_found`, with no input. A located-text click has no normal end for "not found": where "not there" is a normal result, use a select (§7.11).
- **Changing the names**, also after the stop: one `record mark` that removes the mark and adds it again with the same id, every other member equal and only its new `expected` (read the members in `record status`, `steps[].marks[].located_text`). The click's `follow` then records `names_changed_after_press: true`, and `record stop` lists the step in `names_changed_after_press`. A change of any other member is held to the recorded press again.

```json
{"schema_version":"actingcommand.lab-record-mark.v1","frame":"<dir>\\cards.png","samples":["<dir>\\cards_shuffled.png"],
 "add":[{"id":"ui/title","family":"template","region":{"x":40,"y":40,"width":200,"height":48}},
        {"id":"text/card","family":"ocr","region":{"x":100,"y":200,"width":600,"height":240},
         "languages":["en"],"timeout_ms":1000,"match_mode":"exact","expected":["BETA"],
         "case_sensitive":false,"minimum_confidence":0.8,"model_ref":"model","model_sha256":"<64 hex>",
         "locate":{"normalize":"identity_v1","blocks":"detect","line_merge":{"gap_factor":0},
                   "cells":{"x":0,"y":0,"width":140,"height":240,"pitch_x":150,"pitch_y":240,"columns":4,"rows":1}},
         "unique":{"margin_milli":50}}],
 "click":{"follow":"text/card"}}
```

### 7.11 Choosing in a list

A step may choose in a list of fixed slots on one screen by the name read in each slot: the request member `select` (request form only, also inside the one-call mark request). `record stop` generates it as the select step of the `linear_steps` task (lab-recording.md "Selecting in a list"; linear-steps.md "Select steps").

```json
{"schema_version":"actingcommand.lab-record-mark.v1","step":3,
 "add":[{"id":"ui/list_title","family":"template","region":{"x":40,"y":20,"width":160,"height":32}}],
 "select":{
   "slots":[{"rect":{"x":100,"y":200,"width":180,"height":240},"name":{"x":110,"y":400,"width":160,"height":30}},
            {"rect":{"x":300,"y":200,"width":180,"height":240},"name":{"x":310,"y":400,"width":160,"height":30}}],
   "read":{"languages":["en"],"timeout_ms":1000,"minimum_confidence_milli":850,"model_ref":"model","model_sha256":"<64 hex>"},
   "names":[{"id":"item_a","aliases":["Item A"]},{"id":"item_b","aliases":["Item B"]}],
   "match":{"max_distance":1,"minimum_margin":1},
   "wanted":{"name":"list_item","default":"item_b"},
   "pressed":"item_a"}}
```

- **The members.** `slots` (1 to 16: each its `rect`, an optional `click` box and its `name` region), `read` (the OCR members of every name read), `names` (every name the list may show: an `id` of `^[a-z][a-z0-9_]{0,63}$` and 1 to 8 `aliases`), `match` (`max_distance` 0 to 8, `minimum_margin` 1 to 9), `wanted` (its `name` and its `default`), `pressed` (the name you press now as a demonstration; absent: the default), `on_empty_key` (the key of the run's end when the wanted name is on no slot; default `<task_id>_not_found`) and `confirm`.
- **The click.** The select is the step's effect: its click is the slot the pressed name selects on the primary frame, pressed by the one call as any recorded click. A tap recorded before, also after the stop, becomes the select's click when it lies in that slot (`select_press_mismatch` otherwise); `clear_select` gives the tap back.
- **Refusals** (`record_select_invalid`, `details.select_problem`): a click that follows a mark, a gesture, a click declared beside it, a retry, an optional step, an application operation, the last step of a stopped recording (`on_final_step`), and a page that gives no guard (`guard_missing`: the page needs a template, colour, colour digest or check mark).
- **The self-test** is the kernel's own dry run of the select on every live frame of the step: every slot must read a known name (`select_name_unread`), the pressed name must select a slot on the primary frame (`select_pressed_absent`), and no other step's page may pass there (`select_other_page_passes`). Without an OCR model it is `not_evaluated`, and a select whose step has no recorded press is refused `lab_ocr_unavailable`.
- **At run time** the choice is decided on the step's frame by its policy document, confirmed on a fresh frame, and one tap is drawn in the chosen slot. An `empty` decision taps nothing and ends the run Completed on the step's page with the `on_empty` key, a success. An item the package does not know or cannot read, which could change the decision, fails the run (`selection_result_blocked`); an ambiguous or insufficient decision fails it too (`selection_not_selected`). Nothing is skipped or guessed.
- **The wanted value is a constant of the package** in 0.12.61: the select's `wanted.default`, written as the one gate of its policy document. Change it by declaring the select again with another default (also on a stopped recording), then `record stop` (`regenerated`). A list that scrolls, and the wanted value as a variable, are later slices.
- **Which to use.** A control found by its fixed label whose position moves: a click that follows a located text (§7.10). An item chosen by a value that may change, where "not there" is a normal result: a select.
- **Revision.** A `linear_steps` task with a select step needs `package` 13; an earlier Runtime refuses it (`contained_task_linear_invalid`, `reason=operation_effect`). `record stop` writes no revision; the bundle's packaging computes it (§7.14).

### 7.12 Endings after the stop

A business may end otherwise after its last press (too little balance, nothing to collect). A stopped recording takes such a screen as an **ending**, from a frame and without recording again; `record stop` generates it as an ending page of the task (lab-recording.md "Endings"; linear-steps.md "Endings").

```json
{"schema_version":"actingcommand.lab-record-mark.v1",
 "ending":{"frame":"<dir>\\<sha256>.png","samples":[],"page":"short","outcome_key":"daily_open_short",
           "add":[{"id":"ui/short_dialog","family":"template","region":{"x":480,"y":260,"width":320,"height":48}}],
           "reuse":[]}}
```

The command form is `record mark --ending --frame <png> --outcome-key <key> [--sample <png>]… [--page <name>] [--template …]… [--color …]… [--reuse <id>]…`; `record mark --remove-ending <key>` (request `"remove_ending"`) gives one up.

- **Only on a stopped recording, and alone.** An active recording refuses it (`record_ending_invalid`, `recording_active`); a request that names a step's member beside it is `with_other_change`. An archived recording takes none.
- **The key** is `^[a-z0-9][a-z0-9_]{0,127}$` (`key_invalid`), not `<task_id>_done`, the usual end's (`key_reserved`), not a select's `on_empty` key (`key_of_select`), and one per ending (`key_exists`); each ending has its own page name (`page_exists`) and at least one mark (`no_marks`).
- **Its marks** are self-tested on its frames as a step's are, and share the recording's ids. Its frame must have the recording's size or a pure scale of it, which the Lab converts (`record_frame_size_mismatch` otherwise).
- **`record stop`** generates its page `ending_kk` or `ending_kk_<page>` and its outcome mapping, and checks it (self-check item 9): an ending that also passes on the frames of the step before the last press (`previous`), of the last step (`usual`) or of another ending (`other_ending`) is `record_ending_ambiguous`; give it up and take it again with a mark of its own screen, such as a template of its message. An ending that rests on OCR marks alone cannot be checked without an OCR model and is refused (`ocr_alone_unchecked`). The usual page passing on an ending's frames is the warning `ending_usual_insensitive`.
- **At run time** the endings are awaited after the last press, before the target page; the first that passes wins, and the run finishes Completed on it, keyed by its key, a success for the policy. The run still waits for one of its ends, so a swallowed press is no success.
- **What cannot be added this way:** an ending that needs a press to close (its message is left on the screen; the next task's start clears it); a result after any press but the last; a branch that continues the business; a pop-up after which the path goes on (an optional step, §7.8); an ending that looks like the screen before the last press, the last step or another ending.
- **Revision.** A `linear_steps` task with an ending page needs `package` 15; an earlier Runtime refuses it (`contained_task_linear_invalid`, `reason=terminal_page`). A package with an ending is no prerequisite and no return-home package (`reason=linear_endings`).

### 7.13 Variables and values by name

A **variable** is a named slot a package's policies read at run time and do not choose themselves: "which one", "how many", "in which order" (package-variables.md).

- **The declaration** is the top-level member `variables` of the package's `control.json`: 1 to 16 entries, each `name` (`^[a-z][a-z0-9_]{0,63}$`), `type` (`choice`, `integer`, `switch` or `ordered`), `title`, the type's members and its `default`. A package that provides an irreversible capability declares none (`variables_on_irreversible`). A bundle whose packs declare variables declares `package` 14; an earlier Runtime refuses such a pack.
- **No recording writes a variable in 0.12.61**: a select's wanted value is a constant of the package (§7.11); generating a variable from a recording is a later slice.
- **Read what a package declares:** `actinglab package variables --package <dir>` (`ac_package_variables`), offline, without a Runtime.
- **Read and set an application's value by name** (the Lab's rows `goals`; MCP on the Lab's server only):
  - `goals <inst> [--package <id>]` (`ac_goals`): the application's goal page, each entry with its value, default and whether it is in effect, and the record's `version`;
  - `goals get <inst> <name>` (`ac_goals_get`): one entry;
  - `goals set <inst> <name> <json> [--expected-version <n>]` (`ac_goals_set`): one value, checked against its declaration by the dispatch's own function (refusals name their `goal_problem`, or `variable_not_applying`, `not_offered`, `not_checked_yet`, `root_failing`, `no_schedule`); `--expected-version` is the `version` `goals` answered;
  - `goals clear <inst> <name> [--expected-version <n>]` (`ac_goals_clear`): the next run reads the default.

  `goals set` and `goals clear` write: only on the user's request.
- **Which runs read the value.** A scheduled dispatch hands its run the value set on the application, judged against the declaration when the run is to read it; a value that no longer fits, or a variable that does not apply by the game root's check, reads its default (the warning `goal_value_not_applied`, once). A direct run and a Lab run read every variable at its default.
- **On the goal page** a `goals.json` (version 2) entry of type `variable` places the variable of that name; a variable no entry places is shown after the entries (standard-pack-goals.md).

### 7.14 Adopting, building and checking a game's packages

A recorded package becomes a member of its game's standard pack in a resource repository's checkout, is built into the bundle an installation places, and is judged for readiness. These are the Lab's rows; over MCP they are tools of the Lab's server only.

- **`standard-pack adopt --repo <checkout> (--record-id <id> | --package <path>) [--decide <decision.json>] [--replace] [--state-dir <dir>]`** (`ac_standard_pack_adopt`): writes the pack directory and its rule, held entry or maintenance entry into the checkout, every check before the first write (`adopt_refused`, `adopt_problem` naming the check), nothing deleted. The decision document `actingcommand.pack-decision.v1` holds only what the Lab cannot know (its `class`, `trigger`, `value` `"pending"` or `{fixed_value_milli, origin: "pack"}`); a value with origin `model` is refused (`value_origin_model`). `--replace` adopts a regenerated package again. The global `--dry-run` writes nothing (standard-pack-authoring.md).
- **`standard-pack build --repo <checkout> --source-repository <owner>/<name> --out <dir> [--commit <40-hex>]`** (`ac_standard_pack_build`): builds the game's whole standard pack from one clean commit into `<out>`: `<game>-resources-<sha7>.zip`, `SHA256SUMS` and `build.json`, its `package` revision computed from the content, never written by hand (13 for a select step, 15 for an ending page) (resource-verify.md "`standard-pack build`").
- **`standard-pack ready --root <game root>`** (`ac_standard_pack_ready`): judges every pack by its thirteen readiness items: `dispatchable`, `ready_to_place`, `waiting` with every reason, or `not_scheduled`. It reads no installation, so a pack answers `ready_to_place` at best (standard-pack-schedule.md "Readiness").
- **`standard-pack check --root <game root>`** (`ac_standard_pack_check`): checks an installed root, the root check of its variables included.

After a package is regenerated (§7.4, §7.12), it reaches a running installation only through the game's resource repository and a placement: adopted into the checkout again, built, and placed with `place <zip> --sums <SHA256SUMS>` (package-reference.md "The built standard pack").
