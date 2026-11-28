# Making a package

This is the package-making part of the `actingcommand` skill, for Runtime **0.12.72**: how an agent records a business on an instance (stage one), turns the recording into a `linear_steps` package (stage two), and brings that package into its game's resource repository and onto the device (stage three). It is written for the Lab's MCP server, `actinglab-mcp`, in the order the work is done. Every tool, parameter, refusal word and answer member below is taken from the Runtime's operation table (`contracts/lab-operations.json`), the server's manual (`apps/actinglab-mcp/MANUAL.md`) and the contracts; a citation `file.md "Section"` names a contract under `contracts/` of the Runtime repository, and a `§` names a section of this file.

**How far to go.** Stop at the delivery point the task names; never go on into the next stage by yourself. Material only: stop after the archive (§3.7), and §1 to §3 are all you need to read. A package: stop after `ac_record_stop` (§4). Into the repository with its regression cases and checks: stop after §5.4. A trial: §5.5. The build: §5.6. The placement and what follows it: §5.7, for a seat that lists the placing tools (§2); any other seat stops at the build and hands it over. §6 lists what does not exist yet, and §7 is one example from the first call to the last.

A tool is `ac_` and the operation row's words with spaces and `-` as `_` (`record mark` is `ac_record_mark`, `standard-pack adopt` is `ac_standard_pack_adopt`); its arguments are the row's parameters by name. Two tools keep other names: one frame is `ac_frame`, and the authoring `observe` is `ac_lab_observe`. `tools.md` beside this file lists every tool with its module; at run time `tools/list` is authoritative. What does not exist yet is said where you would look for it, and listed again in §6.

## 1. The owner's rules

The project owner set these rules on the project board. They bind every package.

The two rules for making a package stand here in the owner's own words, unchanged. The English under each is a rendering, not the owner's words.

> 「铁律一：除极少数复杂循环战斗外，一律只准交 record 录出来的线性包，严禁再手写几千行的 page_graph 状态机。」

*Rendering:* Iron rule one: apart from the very few complex looping battles, hand in only linear packages recorded with `record`; writing page_graph state machines of thousands of lines by hand again is strictly forbidden.

> 「铁律二：只录一遍正常主路径（Happy Path），点完最后一下业务生效立即停！ 严禁在制包时提前去造“余额不足、售罄、背包满”等还没碰到的分支。正常跑起来真遇到了，0.12.61 会自动停在 unrecorded_screen 并留好现场帧，到时候用 R2 对着那张真实截图 1 分钟补一个结局就行」

*Rendering:* Iron rule two: record the normal main path (the happy path) once, and stop as soon as the last press has made the business take effect! While making a package, never build in advance branches not met yet, such as "balance not enough", "sold out" or "bag full". When a normal run really meets one, 0.12.61 stops by itself at unrecorded_screen and keeps the frame of the scene; then it is enough to add an ending with R2 from that real screenshot, in a minute. (R2 is the ending a stopped recording takes from a frame, §4.8.)

What an agent does because of them, with the owner's other rulings on making a package:

1. **Only recorded linear packages.** A package is made by recording a business and turning the recording into a `linear_steps` package. Do not write a page graph by hand. The rule's one exception is a complex looping battle, which a linear package cannot carry: it runs one declared path, with no branch and no loop of variable length (`linear-steps.md` "What a linear task cannot express"). A list that scrolls is no such exception: it is recorded (§4.5). Before you write a page graph for such a battle, tell the session that gave you the work which battle it is and why a recording cannot carry it.
2. **The main path, once.** Record the business's normal path once, and end on the screen its last press brings. Do not build branches the recording did not meet. What a run meets later (a pop-up after which the path goes on, another way the business ends) is added from that run's frame, without recording again (§5.8).
3. **The tools hand back material and propose nothing.** Every region, choice, name and number is yours. No answer carries a suggestion, a candidate box or a ready-made request.
4. **Recognition by mapping.** The region that recognises a page and the region that is pressed are different places, and neither holds account data (a name, an identifier, an amount).
5. **No back key while recording.** The back key is never pressed while a recording is open. A step that presses it is declared afterwards (§4.6).
6. **Tried first where it was recorded.** A new package is tried on the instance and the application it was recorded on (`ac_standard_pack_trial`, §5.5) before anything else is done with it on a device.
7. **Nothing enables a package by itself.** Nothing you do lifts a hold or lets a package enter scheduling by itself: that is a placement that names its authorization (`ac_place` with `authorization` and `enables`, §5.7). An adoption, a passed trial, a check or a plan is no authorization.
8. **Break-in first.** A package that enters scheduling is in its break-in period first (§5.7).
9. **A numbered level by its number.** A level, a stage or a chapter that carries a code (such as `13-4`, or a chapter's number) is recognised by that code, read as text, and never by the name written after it; an event's levels too. Where a code is not unique by itself (the same number under several places or kinds), keep what tells them apart, and say so in the package's notes. (The owner's ruling of 2026-11-17, recorded on the project board by the channel that keeps the resource repositories.) Until the offline regression can check a press that follows a located text, give such a level's press a fixed box guarded by a check that reads the code, rather than a press that follows the text.

## 2. Before you start

- **Call `ac_status` first.** The Runtime must answer (`daemon`), and the instance must be listed with nothing running on it. A Runtime that does not answer is reported offline in that answer; starting it is the person's decision, never yours. Under a global pause (`global_pause`) the Runtime starts no emulator by itself, so an emulator that is off (`emulator_state`) stays off until the resume; each start it held is the instance's record `emulator_autostart_skipped` with `skip_reason` `global_paused` (`emulator-control.md` "Pauses (PG-1)"). Where your task asks for it, start it with `ac_emu_start` (`instance`), which runs under any pause; the startup package it would run waits until the resume (`scheduling-pause.md` "The dispatch gate"), so launch the application yourself (next item).
- **The instance and its application.** Record on the instance, and the application assigned to it, that the package will be tried on (§1, rule 6). Launch the application with `ac_app_launch` (`instance`), the application operation; never by pressing a launcher icon. Bring it to the business's first screen with its own controls before `ac_record_start`: presses before the start are not recorded. The first screen should be the screen the game's return-home package ends on, normally the main interface (`page` `home`, §4.2): a run that does not start on step 1's page first runs the return-home package of its game and server (the game's standard pack declares it) and then waits for step 1, failing `contained_task_return_home_entry_unmatched` when step 1's page does not come. A package recorded from another screen names the package that leads to its step 1, a package of its game's standard pack, with `requires` of `ac_record_stop` (§4.11) (`linear-steps.md` "Return-home fallback", "Prerequisite packages").
- **The screen size.** Packages are made at 1280×720 today, the base size every standard package declares. `ac_record_start` refuses an instance whose display the Runtime last observed at another size, `record_display_not_base` (`frame_width`, `frame_height`, `base_width`, `base_height`), and the recording's first frame of another size is refused with the same code; a recording whose frames are not the base size is `record_size_not_base` (archive it and record again on a display of the base size). `display_last_observed` in the start's answer is the display a task last saw, not the display now. `ac_frame` gives the device's own size, and `ac_tap`, `ac_swipe` and `ac_long_tap` take the device's own pixels: at 1280×720 these are the recording's pixels (`lab-recording.md` "A recording's size").
- **A recording holds its instance.** `ac_record_start` pauses the instance's scheduling with a reason of its own, `lab-record-<ms>`, and waits until the task running there has ended (its wait for the pause is 335 s). An emulator start the Runtime began by itself on the instance (its configured start, the emulator watch's restart or recognition, an idle wake) is waited for until it ends, past those 335 s, and the pause is not lifted because of it (`scheduling-pause.md` "Instance pause: three stages"): a start whose wait ended first fails while its pause stands and goes on, so read `ac_record_status` and `ac_status` before anything else. A Runtime stopped during that wait keeps the pause (`scheduling_pause_kept_at_shutdown`), which the start takes for a refused pause: it fails and keeps no session header, while the next start of the Runtime restores a pause with the recording's reason that no recording holds (`distribution/windows/RELEASE-NOTES.md` of the Runtime repository, v0.12.72, the known limits): report such a pause to the session that coordinates the work. An instance already paused by another party is held by that pause, which the recording never resumes (`hold.state` `foreign`). Do not call `ac_pause` or `ac_resume` for a recording. `ac_record_archive` and `ac_record_stop` resume the recording's own pause and nothing else; a release that fails keeps the hold (`hold.kept`), and `next.release` names the call that tries it again (`lab-recording.md` "A recording holds its instance").
- **Never leave a recording open.** A forgotten recording keeps its instance's scheduling paused, with no expiry; `ac_status` shows the pause with its reason. While a recording is active (a reopened one too), every call with `record` on that instance, whoever sends it, adds a step to it. Never pass `force` to `ac_record_start`: when it answers `record_session_active`, a recording is open on that instance. **Find out whose it is before you touch it**: read `ac_record_status` (its `record_id`, its steps, its `hold`). It is yours only when its `record_id` is one this task started or was handed with the work: then go on with it or end it, as the task needs. A recording of another seat or another task, or one you cannot place, is left exactly as it is: do not archive it, do not stop it, add no step to it; report the instance as occupied to the session that coordinates the work. Ending a recording ends the other party's work on it and releases its hold, which lets scheduled runs start on the instance.
- **Text recognition.** Names in a list, OCR marks and value tests are read by the Lab's OCR model, on the CPU. Without one they are `not_evaluated` (`lab_ocr_unavailable`), never passed. A read names its model by `model_ref` and `model_sha256`: `ac_vision_check` lists the models the installed packs need (`required_models`, each `model_ref`, `model_sha256`), `ac_vision_manifest` (`provider` `cpu`) every model of the vision root. A seat that does not list these two tools (next item) takes the two values from the session that coordinates the work, which reads them there; it does not guess them.
- **The tools your seat lists.** A client lists the tools of the modules its registration names. The recording tools, `ac_standard_pack_adopt`, `ac_resource_regression`, `ac_standard_pack_trial`, `ac_standard_pack_status`, `ac_standard_pack_catalog` and `ac_standard_pack_build` are in the module `authoring`; `ac_standard_pack_check`, `ac_standard_pack_ready`, `ac_frame` and the application tools (`ac_app_launch`, `ac_app_restart`, `ac_app_stop`) are in `core`, and the presses (`ac_tap`, `ac_swipe`, `ac_long_tap`, `ac_do`) in `debugging`. **`ac_place`, `ac_op_status`, `ac_undo`, `ac_vision_check` and `ac_vision_manifest` are in the module `rescue`**, beside the tools that start, stop, deploy and roll back the Runtime: a seat registered without that module does not list them. Such a seat makes the package up to the build (§5.6) and hands the build's files and the plan's words (`next.place`) to the session that coordinates the work, which places it and reads it back (§5.7). Do not work around a tool your seat does not list.
- **How answers come back** (`apps/actinglab-mcp/MANUAL.md` sections 3 to 7). A refusal is `{outcome, message?, details?}`; warnings are in `warnings`. A call not done within 25 s answers `{handle, job_phase}` and goes on: follow it with `ac_run_show` (`id` the handle, `wait_s`). An answer over 24 KiB is written whole to the export directory with the warning `answer_exported`: read that file, and never call a writing tool again to see its answer (a rescue tool's comes as a header beside its file, §5.7). After an interrupted or uncertain write, read first (`ac_record_status`, `ac_status`) and never send it again with new arguments. Every path is absolute (`not_absolute` otherwise).

## 3. Stage one: record

Stage one plays the business normally with the recording on, and marks nothing (`lab-recording.md` "Recording while playing").

### 3.1 The calls, in order

1. `ac_record_start` with `instance`, `task_id` (`^[a-z0-9][a-z0-9_]{0,63}$`; the package's task folder) and `locale`. Give no `state_dir` and never `force`. Read `lab_recording.status` (`active`; `unavailable` with a `reason`, such as `task_id_invalid_for_lab_package`, means no Lab recording started), `lab_recording.record_id`, `hold.state` (`own` or `foreign`), `hold.warnings` (`no_runtime_listening`: nothing is held, and no recorded press can be sent), `display_last_observed`, and `record_flag_reachable`, which must be `true`.
2. Each press with `record: true`: `ac_tap` (`instance`, `x`, `y`), `ac_swipe` (`instance`, `x1`, `y1`, `x2`, `y2`, `duration_ms`), `ac_long_tap` (`instance`, `x`, `y`, `duration_ms`). An application operation in the middle of the business is `ac_app_launch`, `ac_app_restart` or `ac_app_stop` with `record` (`lab-recording.md` "In the middle of a business"). After each press read `record.step`, `record.after.path` (the stored screen after the press: look at it before the next press), `record.after.still` and `record.warnings`.
3. The end: `ac_record_archive` (`instance`), §3.7.

### 3.2 What the recording keeps of a press

- **The screen before.** One frame captured immediately before the press; the press is sent on that frame, and its point is a pixel of it (`before_frame`).
- **The screens after.** Frames captured after the press until the screen is still, every one kept; the last is the press's `after_frame` and the next step's frame. `ac_record_frames` and `ac_record_review` read them (§4.1).
- **The press** as the command made it: a tap or a long press as a box of one pixel at its point (`source: "point"`), a swipe as a drag between two such boxes (`source: "swipe"`), with its duration and the time it was executed. Nothing is marked: every step is marked in stage two.

The press's outcome: a performed press closes its step. `record_click_indeterminate` (exit 4): the press may have been made and its step stays open; look at the screen (`ac_frame`). When the press took effect, accept it with `ac_record_mark` and `close_step: true`. When it did not, `ac_record_mark` with `reopen_step` (the step's serial number), then press the step again with `ac_do` (`instance`, `capture`, `record`, `package` a carrier as below, and `next_frame_after_ms` to take the next screen): `ac_tap` refuses a step that declares a click (`record_click_exists`). `record_append_failed_after_input`: the press was sent and not recorded; read `ac_record_status` before anything else. A refusal the Runtime denied sent nothing and recorded no press. When the first capture after a performed press could not be taken (the refusal's `details.after.taken` is `false`), the press and its step stand: do not press again; take the screen with `ac_lab_observe` (`instance`, `capture`, `record`, and `package`, any admissible package of the same game and resolution as its carrier), or with `ac_frame` and then `ac_record_mark` with `frame` (an offline frame of the recording's size opens the next step).

### 3.3 Swipes and long presses

- `ac_swipe` with `record` records a drag between boxes of one pixel at its two points over `duration_ms` (1 to 60000); both points lie inside the frame. A press box cannot be drawn on it later (`record_click_rect_conflict`): a recorded swipe keeps its two ends.
- `ac_long_tap` with `record` records a box of one pixel at its point held for `duration_ms` (1 to 60000), gesture `long_press`. A press box drawn on it later keeps the long press and its duration.
- A swipe sent through `ac_do` stores no screen after it, so a scrolling choice cannot read it (§3.6): use `ac_swipe`.

### 3.4 The frames after a press

The captures after a press are set by four parameters, each given with `record`:

| Parameter | Press (`ac_tap`, `ac_swipe`, `ac_long_tap`) | Application operation (`ac_app_*`) | What it is |
|---|---|---|---|
| `after_ms` | 100 | 2000 | the wait from the press to the first capture, 0 to 60000 |
| `interval_ms` | 100 | 1000 | from the start of one capture to the start of the next, 20 to 1000 |
| `still_ms` | 300 | 2000 | the screen is still when it has not changed for this long, 0 to 5000 (0: the first capture is the screen after, untested) |
| `still_max_ms` | 5000 | 30000 | no capture starts later than this after the first, 0 to 60000 and at most 120 times `interval_ms` |

At the bound the last capture is the screen after, `still` is `false`, and `record.warnings` holds `after_not_still`; the press is recorded. A screen that settles slowly needs a larger `still_max_ms`. The captures between the press and the still screen are the frames of the transition, which stage two can declare (§4.9).

### 3.5 What is not recorded

- **The back key and anything typed.** `ac_key` and `ac_text` have no `record`. Do not send them while a recording is open (§1, rule 5): leave a screen by its own control, and declare the back key on that step afterwards (§4.6).
- **A press without `record`.** The recording does not know it. The capture before the next recorded press becomes the open step's screen, so the screen after the earlier press and the screen the next press was sent on are two different screens, and the material shows a jump that no recorded press makes. Do not send one; when it happened, archive the recording and record the business again.

### 3.6 Demonstrating a scrolling list

A choice in a list that scrolls is demonstrated as it is played: several swipes, each a step with its screens, then the press on the wanted row. Stage two folds the swipes into that press (§4.5). There are two kinds; choose before you record:

- **Read to the end** (`stop` `end`): swipe forward until the list's true last screen shows, swipe back to the wanted row, press it. Use it when what the list holds is not known, or when the wanted item may be missing on some days: such a run then ends normally (§4.5).
- **Straight to the wanted item** (`stop` `wanted`): swipe forward until the wanted row shows, press it. Use it only for an item that is always in the list.

Each swipe:

- is `ac_swipe` with `record`;
- runs clearly along one axis: its move along the axis at least twice its move across (`move_axis_unclear` otherwise);
- is shorter than the list's visible length, so that neighbouring screens share at least one row: its move along the list at most `b − e − 1` pixels, `b` the list's extent and `e` a row's extent along the axis (`scan_swipe_too_long` otherwise, which no number cures);
- comes in order: forward swipes first, then swipes back; never a forward swipe after a swipe back (`move_after_back`).

The first forward swipe is the move the package repeats: its two points and its duration are the package's advance, and its `interval_ms` is the package's settle interval. For the first kind, reach the list's true last screen once: the end mark is cut from the last screen after a forward swipe. Press the wanted row inside the region you will declare as the rows' click box. A list that fits on one screen has no swipe: its choice is the fixed-slot select (§4.4).

### 3.7 The end

`ac_record_archive` (`instance`) ends stage one: it writes the manifest of the material (`lab.manifest`; its `material` names each press with the frame it was made on and where, and `final_frame`, the screen the business ended on), keeps every file, ends the session (`archived`) and releases the hold (`hold.release` `resumed`; `not_own` for another party's pause). `lab.warnings` `final_frame_missing` means the manifest names no final screen: the last press's screen after was not taken. You may instead go on to stage two on the active recording, which keeps the instance held until `ac_record_stop`.

## 4. Stage two: turn the recording into a package

Stage two writes the recording and sends nothing to the device, **with one exception: the end of a recording that still holds its instance**. A recording archived at the end of stage one has released its hold (`ac_record_status` `hold.state` is then `none`, or `foreign` where another party's pause holds the instance): for it, all of stage two is offline. A recording you took into stage two without archiving (`own`), or one whose release failed at the archive and stays named in its header (`kept`), still holds its own pause: its `ac_record_stop` that is no dry run releases that pause once the package is written, which resumes the instance's scheduling and makes the Runtime reconnect the device and run its self-check (up to 33 s and the IO timeout, so the call may answer `{handle, job_phase}`), after which scheduled runs start there again (`lab-recording.md` "A recording holds its instance"). If the task asked for offline work only, archive first (§3.7) and reopen (§4.1).

### 4.1 Reopen and read

- `ac_record_reopen` (`instance`, `record_id`) makes the archived recording active again; it takes no hold. Refused `record_session_active` while another recording is active on the instance, `record_not_archived` for a recording that is not archived. While it is active, send no press with `record` on that instance.
- `ac_record_ready` (`instance`, and `game`, `server`, `locale` when the recording does not hold them) lists everything the recording still lacks before `ac_record_stop` can make its package. **Read it after every change.** `status`: `material` (nothing of stage two written), `draft`, `ready` or `packaged`. `missing`: each item `{order, step, artifact_step, frame_id, what, code, hint?, details?}`, `what` its word and `code` the refusal `ac_record_stop` would give. `warnings`: `press_point_only`, a step whose press is a single recorded pixel (§4.3), and `record_template_near_threshold` (§4.11). `checks` and `checks_after`: the self-checks that ran and did not run. `sources`: where the game, server and locale came from (`command_line`, `recording`, `lab_config`, `installation_config`, or `null` when unknown). It writes none of them on the recording: give `ac_record_stop` the ones you gave it.
- `ac_record_review` (`instance`, `step` for one step, `no_preview`, `game`, `server`, `locale`) answers each effective step as `before` (the screen the press was made on, with its `path` and `preview`), `press`, `after` (with `still` and `waited_ms`), `between` (the frames stored between them), `marks`, `folded` (on a press step holding a fold), `missing` (its items of `ac_record_ready`) and `warnings` (its `record_template_near_threshold`, §4.11; absent when it has none); `steps_total` counts the effective steps. Its previews (`lab/preview/s<nn>-before.png`, `s<nn>-after.png`) are copies with the step's recognize boxes drawn green, the press box orange, other marks blue and the press magenta; no stored frame carries a box. Read a long recording one `step` at a time.
- `ac_record_frames` (`instance`, `step`, `before` 0 to 100, default 3, `after` 0 to 100, default 10) lists the stored frames around one step's press, each with its `path` and `offset_ms` from the press.
- `ac_record_status` (`instance`): `lab.steps[].index` is a step's serial number, which every `step` parameter takes and which is never reused; `artifact_step` is its number in the package.

### 4.2 The marks of a page

Every effective step but an application entry needs at least one mark (`record_step_without_recognition`); the last step has marks and a page and no press (`record_final_step_has_click` otherwise). `ac_record_mark` with `step` and the marks tests every mark on every live frame of the step when it is made; one failure refuses the whole call and nothing is written (`record_mark_rejected`, `details.marks[]` with each mark's `reason`). `dry_run: true` previews. Mark ids are 1 to 128 bytes, unique, and do not begin with `page/` or `select/`. The request form (`request_json`, or `request` a file) is the document `actingcommand.lab-record-mark.v1` with its marks in `add`, each with its `id`, its `family` and the family's members, a `region` but for a check and a value test (`lab-recording.md` "record mark"; a value test's form is in its "Value tests"). A template, a colour, a colour digest, an OCR and a check mark and the page's name, on step 3:

```json
{"schema_version":"actingcommand.lab-record-mark.v1","step":3,"page":"title",
 "add":[{"id":"ui/start","family":"template","region":{"x":1180,"y":664,"width":73,"height":25}},
        {"id":"state/start","family":"color","region":{"x":1173,"y":680,"width":1,"height":1}},
        {"id":"hud/strip","family":"color_digest","region":{"x":0,"y":640,"width":1280,"height":80},
         "columns":16,"rows":2,"max_mean_milli":1500,"max_cell":12},
        {"id":"text/start","family":"ocr","region":{"x":1180,"y":664,"width":73,"height":25},
         "languages":["en"],"timeout_ms":1000,"match_mode":"contains","expected":["START"],
         "case_sensitive":false,"minimum_confidence":0.8,"model_ref":"<model_ref>","model_sha256":"<model_sha256>"},
        {"id":"check/ready","family":"check","all_of":["ui/start","state/start"]}]}
```

An OCR mark names its model by `model_ref` and `model_sha256` (§2, text recognition).

| Mark | What it proves | Where its image comes from | How it is given |
|---|---|---|---|
| recognize box | a fixed image is at its place (template, `ccoeff_normed`, threshold 0.95 by default) | cut from the step's primary frame; scored on the step's frames and on the captures stage one stored for the step whose whole frame did not change | `box` `recognize=x1,y1:x2,y2` |
| template | the same, with a search area of your own | cut from the step's primary frame, searched in its area (by default the crop grown by 48 px) | `template` `<id>=x,y,w,h[@x,y,w,h]` |
| colour | the region's mean colour is within its distance | the region's mean colour on the primary frame | `color` `<id>=x,y,w,h` |
| colour digest | a grid of cells keeps its colours | the cells of the primary frame | request form only (`request_json`) |
| OCR | a text is read there | read on each frame by the OCR model | request form only |
| check | `all_of` or `any_of` of 2 to 8 marks of the step | its members | request form only |
| value test | a number read there meets a comparison | read on each frame by the OCR model | request form only |

- **Boxes.** `box` takes two opposite corners in the step's primary frame's pixels, both inside the box (`x` the smaller `x`, `width` their distance plus 1). A recognize box becomes the template mark `box/<nn>/<x>_<y>_<w>_<h>` that the step's page requires. Its answer `boxes[].frames` says of each frame scored `own`, `added` (taken as a sample), `not_added` (with `because`: `frame_changed`, `below_threshold`, or the page's refusal there) or `elsewhere` (it also passes on another step's frame: it does not tell the two screens apart). Give all recognize boxes of a step in one call. A box on one flat grey value is refused `box_flat` with `instead: "color"`: mark it with `color`.
- **The page name.** `page` (`^[a-z0-9][a-z0-9_]{0,47}$`). The last step's page is the package's terminal page, `step_<nn>_<page>`. `page` `home` marks the main interface.
- **By mapping** (§1, rule 4): recognise a page by a region of its own (a title, a frame, a fixed label) and press another region; neither holds a player name, an identifier or an amount. The regression cases paste the regions the package reads into their scenes (§4.12).
- **Step 1** also needs a colour mark on a fixed bright area: otherwise a notice that dims step 1 is taken for step 1 (`entry_overlay_insensitive`). Templates do not see an overall darkening; use a colour or colour digest mark wherever a pop-up dims the screen.
- **Crop to what cannot change.** Cut a template or a recognize box to the fixed glyph or icon itself, with a margin of about 3 px. Leave out a translucent tile, a tab's backdrop and the art behind it: an application changes those with an event or a season, without notice. Before the package goes in, score the mark on at least two frames whose backdrops differ, the recording and a live frame of another day or another event (`ac_recognize` with `target`, `scene` and `package` answers a mark's score on a saved frame). A colour mark on a translucent element accepts every shade seen (its centre and a wide distance), or is dropped. Seen in production: a reading package's recorded template, a glyph with the tile behind it (90×40), fell from above the 0.95 threshold to 0.845 on the day the tile darkened, and every reading and fact refresh of that application failed until the crop was cut to the glyph alone (30×32: 1.0 that day, 0.987 and 0.994 on the recorded frames); a tab's colour check broke the same way when a translucent tab took a lighter shade. When a whole family of packages fails at its entry after an application's update, first compare the failing mark's score on the new frame with its score on the recorded scene.

### 4.3 The press, its guard and retries

- **The press region.** `box` `press=x1,y1:x2,y2` makes the click's rectangle. On a press recorded by its point the box must hold that point (`record_click_outside_step_rect`, `reason` `recorded_point_outside_box`), and the press stays as recorded; a second press box needs `replace_click`. Without a press box the package presses the one recorded pixel (`press_point_only`). `click` (`x,y,w,h`), `click_from` (a mark's region) and `click_follow` (a template or a located text the click follows, next item) are the other forms.
- **A click that follows a mark** (`lab-recording.md` "Following a template", "Following a located text"). `click_follow` names a template mark of the step, for a control whose position moves: the package presses the box the template matches on the frame it runs on. Give the template a search area that covers every position the control moves to, and `unique` with a margin when a second copy could appear. Or it names a located-text mark: an OCR mark of the request form that declares `locate` (`normalize` `identity_v1`; `blocks` `detect` or `region`; `line_merge`, given exactly with `detect`; optional `cells`, a grid relative to the region's corner) and may add `unique` (`{"margin_milli"}`); the package presses the block its names are found in, or with `cells` the cell that holds the block's centre, on one screen, without scrolling:

  ```json
  {"id":"text/card","family":"ocr","region":{"x":100,"y":200,"width":600,"height":240},
   "languages":["en"],"timeout_ms":1000,"match_mode":"exact","expected":["BETA"],
   "case_sensitive":false,"minimum_confidence":0.8,"model_ref":"<model_ref>","model_sha256":"<model_sha256>",
   "locate":{"normalize":"identity_v1","blocks":"detect","line_merge":{"gap_factor":0},
             "cells":{"x":0,"y":0,"width":140,"height":240,"pitch_x":150,"pitch_y":240,"columns":4,"rows":1}},
   "unique":{"margin_milli":50}}
  ```

  The followed text is the click's guard and no mark of the step's page: the step needs another mark that tells its screen from the task's other screens, such as a template of its title (`record_click_follow_invalid`, `follow_problem` `page_needs_followed_text`, otherwise). On a step whose press was recorded, the press must lie inside the box found on the primary frame (`press_outside_found_box`). Without an OCR model `ac_record_mark` on that step is refused `lab_ocr_unavailable`; a name on no recorded frame of the step cannot be declared (`text_locate_not_found`): add a frame that shows it as a `sample`. At run time a name that is not on the screen fails the run with `contained_task_guard_refused` (`cause=text_locate_not_found`), nothing pressed: where "not there" is a normal result, use a select (§4.4).
- **Where a located text is pressed, offline** (`ocr-fields.md` "Located text"). `ac_recognize` (`target` the mark's id, `scene` a saved frame, `package` the stopped recording's package) answers a located-text target that passed with `matched_rect`, the box a run's click starts from: the found cell when `cells` is declared, else the found block, in the coordinates a template's `matched_rect` has in that answer; the same function gives the run its click box. A located-text target that did not pass, and an OCR target without `locate`, answer `matched_rect` `null`. `ac_is_visible` carries the same box in its `evaluation`; `ac_tap_target` plans the target's declared static `click`, never the found box. A regression case checks only whether a target passes (`expect.recognize`, `{target, passed}`), not where its press lands.
- **The guard** is checked on the frame before the press. `click_guard` names a template, colour, colour digest, check or value-test mark of the step; without it the guard is taken from the page's required marks: the click's source mark (unless it is an OCR mark), else the first template, else the first colour or colour digest, else the first check. A page of OCR marks alone gives no guard.
- **Retries.** `click_retry` (2 to 5, interval 1000 ms) presses again when the next page does not come and the step's page is seen again: give it to a press that may be swallowed, such as a close button.

### 4.4 A choice, and the fixed-slot select

- **Mark the choice first.** A step whose press chose one item among several is marked with `request_json` `{"schema_version":"actingcommand.lab-record-mark.v1","step":<k>,"choice":true}`. The mark says only that the choice is not refined yet: `ac_record_ready` lists the step as `select_unrefined`, and `ac_record_stop` refuses `record_select_invalid` with `select_problem` `choice_not_refined`. `"choice": false` takes the mark off (`no_choice` on a step without it).
- **Refine it** by declaring the step's `select` (request form only). For a list on one screen, the fixed slots: `slots` (1 to 16, each `rect`, an optional `click` box and `name`, the region its name is read in), `read` (`languages`, `timeout_ms`, `minimum_confidence_milli` 1 to 1000, `model_ref`, `model_sha256`), `names` (every name the list may show: `id` `^[a-z][a-z0-9_]{0,63}$` and 1 to 8 `aliases`), `match` (`max_distance` 0 to 8, `minimum_margin` 1 to 9), `wanted` (`name`, the variable's name, and `default`, one of the names), `pressed` (the name the recorded press chose; absent: the default), `on_empty_key` (the run's normal end when the wanted name is on no slot; default `<task_id>_not_found`) and `confirm`. The recorded press becomes the select's click when it lies in the slot `pressed` selects on the primary frame (`select_press_mismatch` otherwise: give `pressed` the name the press chose). `clear_select` takes the select off.
- **The wanted value is a variable.** Every select's wanted value is an `ordered` variable of the package named `wanted.name`, its default `[wanted.default]`: a value is a list of names in an order of preference, of which a run takes the first the screen shows. It is set by name on an application with `ac_goals_set` (`instance`, `name`, `value`), and a trial takes it as `variable` (§5.5); `ac_package_variables` (`package`) reads what a package declares.
- **Its rules** (`record_select_invalid`, `select_problem`): no click beside it (`with_click`), no gesture, follow or retry (`with_gesture`, `with_follow`, `with_retry`), not optional (`on_optional_step`), not on the last step of a stopped recording (`on_final_step`), and a page that gives a guard, a template, colour, colour digest or check mark (`guard_missing`).
- **Its self-test** on every live frame: every slot reads a known name (`select_name_unread`), the pressed name selects a slot (`select_pressed_absent`), and no other step's page passes there (`select_other_page_passes`).
- **At run time** the choice is decided by the names read, confirmed on a fresh frame and tapped once. A wanted name on no slot ends the run normally with the `on_empty` key, a success; a name that cannot be read and could change the choice fails the run (`selection_result_blocked`); nothing is guessed.

### 4.5 The scrolling choice

**1. Fold the swipes into the press.** `ac_record_mark` with `fold_scan` `<b>` and `fold_first` `<a>`, `a` the first swipe's serial number and `b` the press's (the request form says the same with `step_action` `{"kind":"fold_scan","step":<b>,"first":<a>}`). The answer's `status` is `scan_folded` and `scan_fold` holds `{first, axis, moves, folded_at_unix_ms}`, each move `forward` or `back`. The swipes stay in the recording whole but are no steps of the package. Refused `record_scan_invalid` with `scan_problem`: `fold_range_invalid` (a step of the range is no recorded swipe, or `b` no performed tap), `inserted_in_range`, `would_lose_marks`, `would_lose_page`, `would_lose_optional`, `would_lose_choice`, `would_lose_transition`, `would_lose_inserted` (take that off first, or fold another range), `move_axis_unclear`, `move_after_back`. `unfold_scan` `<b>` undoes it, in the request form `step_action` `{"kind":"unfold_scan","step":<b>}` (`not_folded` on a step without a fold; `scan_selected` while the select stands: `clear_select` first). A step operation, a mark or a transition on a step of the fold is `step_in_fold`. Until a scrolling select reads the fold, `ac_record_ready` lists `scan_without_select` and `ac_record_stop` refuses `scan_not_selected`. The fold is taken on an active recording and on a stopped one.

**2. Mark the press step's page** (a title of the list's screen, §4.2): the select's guard comes from it.

**3. Declare the select** with `rows` and `scan` in place of `slots`, beside `read`, `names`, `match`, `wanted`, `pressed` and `on_empty_key` (request form only):

- `rows`: `row` (one row of the list as its first screen shows it), `anchor` (what repeats in every row), `name` (where a row's name is read), `click` (where the tap is drawn; absent: the row). The anchor, the name and the click lie inside the row, the row inside the list.
- `scan`: `list` (the list's area), `start` (the start mark), `end` (the end mark; with `"wanted"` it may be left out, below), `stop` (`"end"` or `"wanted"`, required) and `numbers` (numbers given in place of the Lab's, below).
- **Where each image comes from.** The screens are the fold's, in order: the first screen (before the first swipe), then the screen after each swipe. The row, its anchor, its name and the start mark are taken from the first screen; the end mark from the last screen after a forward swipe. Each mark is a template at its own region, unmoved, so it passes only where the list stands at its end. The start and end marks may lie anywhere in the frame (a header above the list, a footer below it).
- **Cut the end mark from the list's true last screen.** An end mark cut from a screen that is not the true end cannot be told offline: the package then ends its coverage early and takes the later names as not in the list.

Refused at the request, `record_select_invalid`: `scan_without_fold`, `rows_with_slots`, `confirm_with_rows`, `scan_stop_missing`, `scan_region_outside_frame`, `scan_region_outside_list`, `scan_region_outside_row`, `scan_number_not_measured`, `on_empty_without_end` (below), `kernel_refused` (`details.loader_code`). A screen the select cannot read, a swipe recorded without its screens, is `record_step_frame_missing`. Without an OCR model a scrolling select is refused `lab_ocr_unavailable`.

**The numbers.** The Lab writes them from your regions and the demonstration, each with its `origin` (`measured`, `default`, `derived`, or `given`) and what it comes from (`from`); `ac_record_review` lists them. `b` and `e` are the list's and a row's extents along the axis, `V` the first screen and the screens after a forward swipe.

| Number | Origin | The value |
|---|---|---|
| `advance_min_content_step_px` | default | 1 |
| `advance_max_content_step_px` | derived | `b − e` |
| `advance_duration_ms` | measured | the first forward swipe's duration |
| `reposition_max_content_step_px` | derived | `b − e` |
| `settle_interval_ms` | measured | the first forward swipe's `interval_ms` |
| `settle_timeout_ms` | derived | twice the longest time a swipe's screen was captured after it, plus the interval |
| `position_tolerance_px` | derived | `max(2, e / 10)` |
| `edge_margin_px` | derived | the tolerance |
| `signature_columns`, `signature_rows` | derived | from the row's shape |
| `signature_max_distance_milli` | default | 1500 |
| `min_overlap_instances` | default | 1 |
| `spacing_columns` | measured | the most rows across the axis on a screen |
| `spacing_max_axis_gap_px`, `spacing_max_trailing_gap_px`, `spacing_max_cross_lead_px`, `spacing_max_cross_gap_px` | derived | from the screens, plus the tolerance |
| `max_instances` | derived | the most anchors on a screen plus 2, at most 64 |
| `max_viewports` | derived | `2V`, at least 2, at most 128 (without an end mark `2V + 2`) |
| `max_reposition_viewports` | derived | `2V`, at most 128 |
| `max_ms` | derived | the viewports' moves and settles, plus one screen's reads |

To give a number, add it by name to `scan.numbers` (`{"max_viewports": 40}`, a whole number of at least 1); it wins, is kept with `origin` `given` beside the value the Lab would have written, and the numbers computed from it follow it. A longer list than the demonstration showed may need a larger `max_viewports` (`selection_scan_bounds_exceeded` on the device).

**The self-test at mark time** reads every screen of the demonstration with the kernel's own dry run and its offline scan. The refusals an agent meets most, and what to do:

| Refusal | What it means | What to do |
|---|---|---|
| `record_select_invalid` `scan_swipe_too_long` (`details.swipe`, `displacement_px`, `most_px`, `hint`) | a forward swipe moved the list further than `b − e − 1`: neighbouring screens share no row | no number cures it: record the business again with shorter swipes (§3.6) |
| `record_mark_rejected` `scan_end_mark_early` or `scan_end_mark_lost` (`passed_on`, `last_frame_id`) | the end mark passes on a forward screen before the last | give `end` a region whose content shows only on the list's last screen (a footer, a closing line), cut from the true last screen |
| `record_mark_rejected` `scan_end_mark_missing` | the end mark does not pass on the last forward screen | the demonstration did not reach the end, or the region is wrong; record again to the true end for the first kind |
| `record_mark_rejected` `select_name_unread` | a row the anchor finds in the list read no known name | add the name, or the alias it is read as, to `names` (every name the list may show belongs there), or move `rows.name` onto the text |

The others: `scan_rows_not_found` (the first screen shows no row), `scan_start_mark_missing`, `scan_start_mark_repeated`, `scan_step_outside_window` (`content_step_px`, `window`), `scan_coverage_incomplete`, `scan_position_unread`, `scan_decision_mismatch`, `scan_absent_not_empty`, `scan_min_step_too_large`, `self_test_error`.

**When the wanted item is not in the list.**

- The first kind, with its end mark: the run ends normally with the `on_empty` key, a success.
- The first kind without an end mark makes no package: `ac_record_ready` lists `scan_end_not_recorded`, `ac_record_stop` refuses `end_not_recorded`. Record the business again and scroll to the end once.
- The second kind may carry no end mark. Such a package cannot end normally when the item is absent: the run fails `selection_scan_early_stop_unreached`. So `on_empty_key` beside a scan without an end mark is refused `on_empty_without_end`, and the default must be a name read on a recorded screen (`scan_default_not_shown`). With an end mark it ends normally as the first kind does.
- A package whose scan declares no end needs the `package` revision 18, which `ac_standard_pack_build` counts.

**What only the device shows**, for the trial (§5.5): the list's inertia and how long it settles, a list opened in its middle, the drag's real displacement, the return to the chosen row and its confirmation, a list that loads more rows while it scrolls, and a list without the wanted item for the second kind.

### 4.6 The key step

A step may press the back key in place of a click (`lab-recording.md` "Key steps"; `linear-steps.md` "Key steps"). It is declared on the recording and never sent while recording.

- **Declaring it.** `ac_record_mark` with `step` and `key` `back` (request form `"key": {"key": "back"}`), on an active or a stopped recording. On a step without an effect the key is its effect. On a step whose press was recorded it replaces that press only with `replace_click` (`record_step_effect_exists` with `effect: "click"` otherwise): the press is kept whole inside the key, the step's frames stand, and a run presses the key on this page where the recording pressed the screen. `back` is the one word (`record_key_invalid`, `word_unknown`).
- **Refused**, nothing written, `record_key_invalid` with `key_problem`: `application_entry`, `application_step`, `holds_choice` (a select or a choice mark on the step), `step_not_effective`, `with_click`, `with_application`, `with_retry` (a key step declares no retry: send the key alone).
- **The guard** is the step's `click_guard`, else its page's first template, colour or colour digest, or check mark. A page of text or value-test marks alone gives none: `ac_record_ready` lists `key_guard_missing` and `ac_record_stop` refuses `record_key_invalid` with `guard_missing`; add a template to the page.
- **Taking it back.** `no_key: true` (request form `"key": false`) removes the key and gives back exactly the press it replaced, with the click guard the step had before.
- **A pop-up a run met, closed by the key**: `optional_after` with `key` in place of a click (§4.7).
- The package's operation is `step_<nn>_key`. A key step whose replaced press needs review still needs review (`lab.steps_needing_review`). Dropping a key step before the last is `record_drop_unproven` with `drop_problem` `key_declared`: take the key back first. A run of a package with a key step on an instance whose input sends no key ends at its start with `contained_task_linear_key_unsupported`.

### 4.7 Optional steps

A screen a run does not always show (a notice, a reward card) is an optional step: when its page does not appear, the run skips it.

- **On a recorded step**, active or stopped recording: `ac_record_mark` with `step`, `optional: true` and `settle_ms` (0 to 60000, default 2000: how long the run watches for a late pop-up after the next page first passes); `not_optional: true` clears it. Not on step 1 (`record_optional_first_step`), the last step (`record_optional_final_step`), an application step (`record_optional_application`) or a select step. Give a close-type step `click_retry` 2.
- **A pop-up a run met after the stop**: insert it from the run's frame with `optional_after` (the serial number of the step after whose effect it appeared), `frame`, its marks (`template`, `color`, `reuse`, `page`, `sample`), one effect (a click form or `key`) and `settle_ms`. Only on a stopped recording, and alone in its call. Refused `record_optional_after_invalid` with `optional_after_problem`: `recording_active`, `with_other_change`, `step_unknown`, `step_last` (a screen after the last press is an ending), `step_application`, `no_marks`, `no_effect`, `clears_optional`, `not_inserted`. `remove_inserted` gives an inserted step up.
- **Checked at stop**: the pop-up's page must not pass where it is absent (`record_optional_step_ambiguous`, `against` `previous`, `skip_target` or `other_optional`: give it a mark of its own, such as its title), and the page after it must not pass under it (`record_optional_skip_target_insensitive`: give that page a colour mark on an area the pop-up darkens or covers).

### 4.8 Endings

A business may end otherwise after its last press (too little balance, nothing to collect). A stopped recording takes such a screen as an ending, from a frame: `ac_record_mark` with `ending: true`, `frame`, `outcome_key`, `page`, and its marks (`template`, `color`, `reuse`; `sample` for more frames). `remove_ending` (its key) gives one up.

- Refused `record_ending_invalid` with `ending_problem`: `recording_active`, `with_other_change`, `key_invalid` (`^[a-z0-9][a-z0-9_]{0,127}$`), `key_reserved` (`<task_id>_done`, the usual end's key), `key_of_select`, `key_exists`, `key_unknown`, `page_exists`, `no_marks`; at stop `ocr_alone_unchecked`.
- At stop, an ending that also passes on the frames of the step before the last press, of the last step or of another ending is `record_ending_ambiguous` (`against` `previous`, `usual`, `other_ending`): give it up and take it again with a mark of its own screen, such as a template of its message. The usual page passing on an ending's frames is the warning `ending_usual_insensitive`.
- A run whose last press reaches the ending finishes Completed on it with its key, a success. Not this way: an ending that needs a press to close, a result after any press but the last, a branch that goes on, a pop-up after which the path goes on (§4.7).

### 4.9 Transitions

A loading or other screen between a press and the next step is declared on the step with the press:

- `transition` `page` with the screen's marks, from a frame the recording already stored (request form `"transition": {"kind": "page", "frame_id": "<frame id>", "sample_frame_ids": [], "add": [...], "reuse": [], "timeout_ms": null}`; `ac_record_frames` lists the frame ids) or from a file (`frame`); `transition_timeout_ms` 1 to 1800000.
- `transition` `window` with `min_ms` and `max_ms`: nothing is evaluated for `min_ms` after the press, then the next page is awaited. Use it for a screen shorter than a capture interval or one that cannot be recognised.
- `transition` `none` clears it; `replace_transition` replaces one whole. A loading screen recorded as a step of its own is turned into the previous step's transition with `to_transition`.
- Refused: `record_transition_without_click`, `record_transition_exists`, `record_transition_window_invalid`, `record_frame_unknown`.

### 4.10 Dropping a press that changed nothing

`drop_step` (a serial number) on an active recording (a stopped one takes no step operation: `record_session_not_active`, reason `recording_stopped`). The last effective step is dropped as before; a step before it only when its stored frames do not contradict that its press changed nothing, by two comparisons, the whole frame and the neighbourhood of the press. Otherwise `record_drop_unproven` with `drop_problem` `frames_not_recorded`, `screen_changed`, `next_screen_differs` or `key_declared`, and `drop_comparison` `frame` or `press_region`. A change confined to a small area away from the press is not seen: whether the press mattered is your judgement. A step holding a choice is never dropped (`record_select_invalid`, `drop_of_choice`). Request form `{"step_action": {"kind": "restore_step", "step": <n>}}` gives a dropped step back.

### 4.11 The dry run, then the stop

1. `ac_record_stop` (`instance`, `dry_run: true`, and `game`, `server`, `locale` as `ac_record_ready`'s `sources` require) runs everything and writes nothing: `status` `validated`, `lab.would_write`, `lab.warnings`, `lab.first_decision`, `lab.cross_check`, `lab.selects`, `lab.key_steps`, `lab.steps_needing_review`, `lab.cases` (`status` `to_write`). A refusal is the first pre-check the recording fails, with `details.missing_total` and `details.hint` `record ready`: read `ac_record_ready` for every item.
2. `ac_record_stop` with the same parameters, no `dry_run`. Give no `lab_dir`: the package stays in the recording (`lab/out/<package_id>.<zip|json>`), where adoption reads it, and a `lab_dir` inside an installed root is refused (`record_lab_dir_invalid`, `inside_installed_root`). Read `lab.status` (`generated`; `regenerated` after marks after the stop; `already_generated` when nothing changed), `lab.package_id` (default `<game>.<server>.<task_id>`), `lab.path`, `lab.cases`, `lab.adoption_example` and `hold`. When the recording still held its own pause, this stop released it (`hold.release` `resumed`): the instance's scheduling runs again from here, as said at the head of this section.

**The self-checks**, in order (the names of `ac_record_ready`'s `checks`): `frames`, `admission`, `first_decision`, `cross_check`, `endings`, `selects`, `final_package`, `cases`. `ac_record_stop` refuses at the first failure of the first check that meets one (`record_artifact_admission_failed`, `record_step_self_mismatch`, `record_optional_step_ambiguous`, `record_optional_skip_target_insensitive`, `record_ending_ambiguous`, `record_case_failed`, …); `ac_record_ready` lists every failure of each check.

**Warnings to act on.** `arrival_unconfirmed`: the next step's page also passes on the frames before the press, so a run cannot tell whether the press took effect: give the next page a mark the earlier screen lacks, or declare a window. `entry_overlay_insensitive` (§4.2). `first_decision_not_evaluated`: step 1's press was not simulated (OCR without a model, an application step). `ending_usual_insensitive` (§4.8). `lab.steps_needing_review`: a press performed with a failure.

**A warning that reports a fact: `record_template_near_threshold`** (`lab-recording.md` "Self-checks", item 11). A template mark's score came nearer its threshold than the bound, on the side a self-check needs it on: `side` `passes` (a mark a page requires passes on the page's own frames by less) or `misses` (a mark a page forbids misses there, or the one template mark that tells a page from a screen it must not pass on misses there, by less). It carries `check` (the code that self-check answers when it fails), the members that name its place (`step`, `record_index`, `page` and the like), `mark_id`, `frame_id` (the frame of the nearest score), `score`, `threshold`, `margin` and `bound`. The bound is `template_margin_bound` of `ac_record_stop`, `ac_record_ready` and `ac_record_review` (0 to 1, default 0.03; 0 warns of none; outside that range `validation_failed`). `ac_record_stop` and its dry run answer it in `lab.warnings` after the other warnings, `ac_record_ready` among its `warnings` once every self-check before `cases` passed, `ac_record_review` at its step (`steps[].warnings`). Colour, colour digest, OCR, value-test and check marks are not judged so. It refuses nothing and changes nothing of the package; on a `packaged` recording no self-check runs, and the bound is checked and has no effect. It proposes nothing: look at the frame it names and decide whether the mark holds on the device, which is yours to decide (`apps/actinglab-mcp/MANUAL.md` section 8).

### 4.12 The regression cases written with the package

`ac_record_stop` writes the package's regression cases beside it, `lab/out/<package_id>.cases/<case id>/` (`case.json` and `scene.png`): one per effective step with a press and a page (`step_<nn>`), on the frame its press was sent on; one for the final screen (`final`); one per ending (`ending_<outcome_key>`). A scene is a black canvas of the recording's size on which exactly the regions the package's recognition targets read are pasted where they are. Each case is run on the package first, and only cases that pass are written. `lab.cases` answers `directory`, `status` (`written`, `replaced`), `counts`, `ids`, `not_proven`, `warnings` and `limits`.

| Word | Refuses or warns | What it means |
|---|---|---|
| `record_case_failed` (`regression_problem`) | refuses, nothing written | a real miss: the case's page is not recognised on its scene, or a target it expects fails. A target depends on more of the screen than its region: give the page a mark that holds within its own region |
| `record_cases_ocr_unavailable` | warns | the Lab had no text-recognition model for a text target the page detection reads: no case's page was judged; they are judged where a model is |
| `record_case_page_ambiguous` (`other_pages`) | warns | the case's page is recognised beside another page of the package: the case is written without `expect.observe`, expecting the targets its page requires to pass and those it forbids to fail |
| `record_case_ambiguous_not_written` | warns | such a page has no target to pin: no case is written; mark a target on that page's screen |
| `record_case_bare_canvas` | warns | the page is also what the bare black canvas answers (a dark colour mark, for one): the case proves nothing of its page |
| `record_case_whole_frame` (`targets`) | warns | a target reads the whole frame, so the scene is the whole frame: the case carries the whole screen, whatever it shows. Give those targets regions of their own (§1, rule 4) |

`not_proven` names cases run in part, with the runner's reasons (`ocr_unavailable`, `frames_required`). `limits` is `["self_agreement"]`: the cases were made from the frames the package was built from and say nothing about screens it did not meet. A regeneration replaces the set whole and keeps the earlier one beside it (`lab.cases.kept`). An adoption by `record_id` takes the set into the checkout (§5.2).

### 4.13 Marking again after the stop

A stopped recording takes, on its existing steps: marks, samples, pages, press boxes, clicks, guards, retries, transitions, optional flags, a choice and a select, a fold, a key, an ending (§4.8) and an inserted optional step (§4.7). The answer says `after_stop: true`. It takes no other new step, no other step operation (a drop, `reopen_step`) and no application operation (`record_session_not_active`, reason `recording_stopped`): any other missing step means recording again. The next `ac_record_stop` generates the package and its cases again (`regenerated`), keeping the earlier files beside them (`lab.kept`). Until then the recording is not current for adoption (`recording_not_current`); once regenerated, adopt it again with `replace` (§5.6).

## 5. Stage three: into the repository and onto the device

`<checkout>` below is the game's working root in a resource repository's checkout: the directory that holds `applications.json`, `maintenance.json`, `packs/` and the root files (`schedule.json`, `pages.json`, …).

**One state directory.** The trial records, the adoption's check trees and its kept copies live in the Lab's state directory (`state_dir`; actinglab's default session directory when it is not given). Give every stage-three call that takes `state_dir` (`ac_standard_pack_adopt`, `ac_standard_pack_trial`, `ac_standard_pack_status`) the same one, outside every repository and installation, and on the checkout's volume: adopting again with `replace` moves the replaced pack directory, and the cases it replaces, into it (`replace_across_volumes` otherwise). A recording lives where the recorded presses keep it, actinglab's default session directory (or `ACTINGLAB_SESSION_STATE_DIR` where that is set), which they take in place of any `state_dir`; `ac_record_start` names it, `record_flag_state_dir`.

**Where adoption reads the recording.** By `record_id` adoption reads the recording, its package, its regression cases and `ac_record_ready`'s judgement of it in the recordings' directory, `<recordings>\record-artifacts\<record_id>\lab\`: the directory `session_state_dir` names, else the `state_dir` (`standard-pack-authoring.md` "The recordings' directory"). It only reads there: everything it writes or moves, and the trial records it reads, stay under `state_dir`. By `record_id` it takes the recording's regression cases with its package (§5.2); by `package` it takes none and judges no recording.

- **The recordings' directory is the state directory** (both actinglab's default, on the checkout's volume): adopt by `record_id`, with no `session_state_dir` and the same `state_dir` (or none) as every other stage-three call.
- **The recordings live in another directory, or on another volume than the checkout:** adopt by `record_id` with `session_state_dir` the recordings' directory (`record_flag_state_dir`) and `state_dir` a directory of yours on the checkout's volume, outside the checkout; `replace` works there. Give `ac_standard_pack_trial` and `ac_standard_pack_status` of the package that same `state_dir`: adoption reads the trial records under its own `state_dir` only, so a trial recorded under another one (actinglab's session directory, the trial's default) is not read, and the adoption answers `review.trial` `none` and warns `not_tried` for a package that was tried.
- **What cannot be.** `session_state_dir` beside `package` is refused before anything is read (`validation_failed`, `argument_problem` `conflicting`). The state directory itself cannot lie on another volume than the checkout for a `replace` (`replace_across_volumes`), nor inside a repository or an installed root for a trial (`trial_state_dir_refused`; `standard-pack-trial.md` "The record").

### 5.1 Adoption: a first adoption is held

`ac_standard_pack_adopt` (`repo`, `record_id`, with `session_state_dir` where §5 gives it, or `package`, `held_reason`, optionally `held_reference` and `state_dir`; `dry_run: true` first) makes the stopped recording's package a member of its game's standard pack in the checkout. A package that has not run through on a real device is adopted held: `held_reason` is the decision of the class `held` without a document (a non-blank text of at most 1024 bytes, such as that it has not been tried yet); `lab.adoption_example.decision` of the stop is that decision as a document for `decide`, its reason left blank for you to fill in.

- **What it writes**: `packs/<package_id>/` (its `control.json` and `resources/**`, each JSON file in the canonical form), the held entry in `schedule.json` and, by `record_id`, last, the recording's regression cases (§5.2); nothing deleted. Every check runs before the first write.
- **The answer**: `status` (`adopted`, `validated`), `class`, `review` (`recording_checked`, whether the recording was checked offline; `trial`, the trials of this content, `state` `passed`, `failed` or `none`), `held`, `written`, `cases` (§5.2), `check_tree`, `readiness` (the pack's readiness row on the check tree, `waiting` with S2 `held` for a held pack), `warnings` (each warning that names this package or no package of the root, whole and in order, adoption's own among them: `not_tried`, no passed trial of this content; `adopt_cases_left`, §5.2), `other_packages_warnings` `{total, counts}` (the warnings that name another package of the root, counted by code, none dropped; `standard-pack-authoring.md` "The warnings of other packages"; when the check tree's root could not be read again to tell them apart, every warning is in `warnings` whole with `adopt_warnings_not_split` (`path`) after them, `other_packages_warnings` is absent, and nothing is refused) and `next`: the repository's own items to write, each `{path}` (a regression case `regression/<game>/` only when the adoption took none, `docs/PACKS.md`, `ours/coverage/business-v1.csv` where the checkout holds them; `ours/model/bindings.json` while the rule has no value).
- **Refused** `adopt_refused` with `adopt_problem`: `repository_invalid`, `recording_not_current` (by `record_id`: marked since its stop, or not `packaged` as `ac_record_ready` judges it, whose answer is `details.record_ready`: do what its `missing` names and stop the recording again), `recording_not_judged` (by `record_id`: the function behind `ac_record_ready` refused the recording; the refusal is the cause), `package_invalid`, `package_mismatch`, `pack_exists` (adopted before: `replace`), `pack_dirty`, `replace_across_volumes`, `decision_invalid`, `value_origin_model`, `business_rule_in_the_way`, `recording_unreviewed`, `local_path` (a text of the package holds an absolute local path), `root_file_not_canonical` (write that root file once in the canonical form, two-space indentation, one member per line, LF line ends and a final newline, in a commit of its own), `check_tree_link`, `check_tree_unmarked`, `check_tree_busy`, `rule_refused`, and the cases' words of §5.2. A failed write is `adopt_write_failed`.
- **Adoption writes no entry of `pack-catalog.json`**: the entry is written after the pack's rule, in §5.6 (§5.3).
- **Commit** the adoption in the checkout: a later adoption with `replace` refuses changes under `packs/<package_id>` the version control does not hold (`pack_dirty`).

### 5.2 The regression cases

`standard-pack-authoring.md` "The cases". Each case of the recording's `lab/out/<package_id>.cases/` (§4.12) belongs in the checkout as `regression/<game>/<package_id>.<case id>/`, its `case.json` and `scene.png` byte for byte, `<game>` the game `applications.json` names. `resource regression` takes any directory under `regression/<game>/` as a case; this name keeps the cases of different packages apart.

- **By `record_id`** (§5) adoption takes every case as its last write. Before anything is written each case is run, by the runner `ac_resource_regression` uses, on the package as this adoption writes it; a dry run runs them too. Refused, nothing written: `case_failed` (a case fails; the details list the cases that did not pass as `ac_resource_regression` lists them, `failed_total`, `listed`, `cases`), `case_place_taken` (a case's directory is there and is no case an earlier adoption took from a recording), `case_name_invalid` (a case's name gives no directory of one file name), `pack_exists` (without `replace`, a case an earlier adoption took; or a case's `.part` an interrupted adoption left), `recording_not_current` (the cases directory cannot be listed). With `replace` the cases an earlier adoption took from a recording are replaced as a set, moved into the state directory; a case written by hand is never moved. The answer's `cases`: `state` `taken`, `from`, `written` (`would_write` in a dry run), `replaced`, `kept` (where the replaced ones went) and `not_proven` (`{case, verdict, reasons}`: a `partial` or `not_run` case, taken). `state` `none_found`: the recording's stop wrote no cases.
- **By `package`** no case is read (`cases.state` `package_path`): copy each case directory of `lab.cases.directory` into the checkout by hand, as above. An adoption that takes no case moves none and runs none: when cases an earlier adoption of the package took from a recording, or a case's `.part`, stay in `regression/<game>/`, it warns `adopt_cases_left` (`package_id`, `paths`): they may not match the content it wrote; put the new recording's cases in their place by hand.

Then run them:

`ac_resource_regression` (`repo` the checkout, `package_id`, optionally `game`, `case`) runs the cases offline on the package `packs/<package_id>/` each names and reads only. Its answer: `counts` (`cases`, `passed`, `failed`, `partial`, `not_run`) and, with a filter, `cases` (each `{path, id, package_id, verdict, checks, warnings?}`, `checks` only those that did not pass, each with its `problem` or `reason`). A failed case makes the call the refusal `resource_regression_failed`, its details the counts, `failed_total`, `listed` and the first 10 cases that did not pass; `partial` cases add the warning `resource_regression_partial`. A member the case format does not list, at a case's top level, in its `scene_construction`, its `canvas` or a rectangle (a note on how its scene was made), is read past with the warning `case_member_unknown` (`json_pointer` its place, `member` its name); `expect`, `expect.observe` and an entry of `expect.recognize` stay closed, and a member they do not know fails the case (`member_unknown`; `resource-regression-case.md` "The reader"). `ac_standard_pack_build` and `ac_resource_verify` do not run the cases yet. Commit them.

### 5.3 The pack's declaration in `pack-catalog.json`

Each game root declares its packages in `pack-catalog.json` at the checkout's root (`pack-catalog.md` "Who writes it"). It is written by a tool, never edited by hand: `ac_standard_pack_catalog` (`root` the checkout, optionally `refresh` and `displays`; `plan: true` first, then without `plan`).

- **A package without an entry gets one**, `{package_id, consumes, produces}`: its `consumes` and `produces` are the union over the package's rules in `schedule.json` at the moment the entry is written, and a display (`title`, `summary`, `group`, `risk`) comes only from a leftover `ui.json`. The call writes such an entry for every package of the registry that has none, not for yours alone: read the plan's `added` first.
- **An entry already there is kept as written** unless `refresh` names it: its differences from the rules are answered (`differences`) and warned (`standard_pack_catalog_schedule_differs`).
- **`refresh`**, a list of package ids, each once, is the one way an entry's business part is brought back to the rules: each named entry's `consumes` and `produces` are computed again from its rules by the rule a new entry is made by, its display and its place kept; a named package without an entry gets its new entry. `refreshed` answers each in the order named, `{package_id, state, before?, after}`, `state` `replaced`, `agrees` or `added`. So an entry written while its pack had no rule (an empty business part) is brought up to date by a later `refresh`. Refused before anything is written, `standard_pack_catalog_refresh_refused` with `catalog_refresh_problem` `package_not_listed` (the registry does not hold it) or `package_repeated`.
- **`displays`**, the absolute path of a display file you write, gives the entries it names their display: `{"schema_version": "actingcommand.pack-catalog-displays.v1", "groups"?: [{"id", "title"}], "packs": [{"package_id", "title", "summary", "group"?, "risk"?}]}`, `title` and `summary` each `{<locale>: text}`. Each named entry's display is replaced whole, its business part and its place untouched (`display_written`). A group the file names that the catalogue does not hold is added after its groups (`groups_added`); one it holds with the same title is there already; no group is removed or renamed. Refused before anything is written, `standard_pack_catalog_display_refused` (`standard_pack_problem` at its `json_pointer` in the display file): `malformed` (among others no `schema_version` or another, or a member the shape does not know, a business member too), `package_not_listed`, `catalog_duplicate` (a package named twice), `catalog_group_title_differs` (a group of that id with another title), `catalog_out_of_limits` (more than 64 groups in all), and every display defect the catalogue's reader names (`pack-catalog.md` "Warnings: a display defect"). Every title, summary, group and risk word is yours: no tool proposes one (§1, rule 3).

Until a pack has an entry it is neither an error nor a warning: it runs as before and shows by its id, and `ac_standard_pack_check` counts it under `unlisted` (`pack-catalog.md` "Neither: coverage"); an entry without a display shows by its `package_id`. The order is: adopt held, try, write the rule (§5.6), then the call with `refresh` naming the package (`apps/actinglab-mcp/MANUAL.md` section 8, "The package declaration"). The answer: `status` (`planned`, `written`, `unchanged`), `path`, `added`, `kept`, `removed`, `dropped`, `refreshed` (with `refresh`), `display_written` and `groups_added` (with `displays`), `document` (with `plan`), `warnings`. A file wrong in its business part makes the root wrong (`standard_pack_root_invalid`), and so does an entry the call would leave out, or a pool it would drop, whose name no warning can carry (`catalog_out_of_limits` at its place in the file). Commit the file.

### 5.4 Checking the pack: `check`, `ready`, `status`

- `ac_standard_pack_check` (`root`) checks a laid-out game root, such as the one adoption checked, `<check_tree>\installed\<game>` (`check_tree` from the adoption's answer). Its last member `catalog` is the note `standard_pack_catalog_coverage` (with `unlisted`, the packages without an entry) or `standard_pack_catalog_absent`.
- `ac_standard_pack_ready` (`root`) judges every pack by its thirteen items, S1 to S7 of the root and I1 to I6 of an installation: `readiness` `waiting` (each failing item with its `readiness_problem`), `ready_to_place`, `dispatchable` or `not_scheduled`. **It reads no installation**: the installation's items are `not_judged` (`installation_not_read`), so a pack is `ready_to_place` at best; `place --plan` does not answer readiness either.
- `ac_standard_pack_status` (`root`, `repo`, `package_id`, `state_dir`; `installation: true` on the installation's game root) says where one pack stands: `stage_state` (`not_admitted`, `not_judged`, `can_be_tried`, `supporting`, `enabled`), `formed` (`admitted`, `content_hash_prefix`, with `repo` `checkout`), `conditions` and `missing`, `held`, `trial`, `enabled` (`state` `enabled`, `held`, `not_placed`, `not_scheduled` or `not_judged`; `authorization`) and, with `installation`, `pairs`. Without `installation` the authorization is `not_recorded` (`journals_not_read`). With it `root` must be the installation's game root of its game, `<standard_packs.root>\<game>` (`package_status_root_not_installed` otherwise).

### 5.5 The trial

A trial is one real run of the pack directory on the instance and application it was recorded on, under the owner's authorization (`standard-pack-trial.md`). A passed trial is not an authorization; a trial counts nothing toward a break-in; it is one run, sent once, never retried by the tool.

1. **The plan**: `ac_standard_pack_trial` (`instance`, `package` the directory `<checkout>\packs\<package_id>`, `expect` the outcome key it should end with, `plan: true`; `variable` and `root` as needed). A container file is refused (`validation_failed`, `unusable`): what is tried is the directory adopted. It sends nothing. Read `plan.content_hash_prefix`, `plan.ends` (each `{outcome_key, end, pages}`), `plan.variables` (each `{name, value, source}`), `plan.irreversible` (`marked`, `unmarked`, or `not_judged` without `root`), `run_requires` (`authorization` when none was given) and `record_directory`.
2. **The run**: the same call without `plan`, with `authorization`, the reference of the owner's authorization (1 to 1024 bytes, for example the address of the board comment that holds it). The tool records it and judges nothing; never make one up. `variable` (`<name>=<json>`, once per name) gives a variable a value; a variable not given runs at its default. A package the irreversible mark flags also needs `irreversible: true`.
3. **Reading its end**: `request_id`, `end` (`state`, `outcome_key`, `final_page`, `failure_code`, `failure_values`), `expected`, `matched` and `record` (the trial record, `<state>/trials/<package_id>/<content12>-<ms>.json`). The trial passed when the run ended on an end the package declares and, with `expect`, on that end. Another normal end is `matched` `false` with the warning `trial_end_unexpected`. A failed or cancelled run is answered with its own code on top, the record written. Read the run in full with `ac_run_show` (`id` the `request_id`) and step by step with `ac_run_evidence` (`id`; its frames are material refs for `ac_material`).
4. **When the call outlasts 25 s** it answers `{handle, job_phase}`: follow it with `ac_run_show` (`id` the handle, `wait_s`). An end not read yet is `trial_end_unread` with `next.collect`: read it later with the same arguments and `collect` (the `request_id`).

Refused: `trial_authorization_missing`, `trial_irreversible_unacknowledged`, `trial_state_dir_refused` (a state directory inside an installed root or a repository), `trial_collect_mismatch`, `trial_run_interrupted`, `trial_terminal_unrecorded`, `trial_end_uncoded`, `trial_record_exists`. **A trial does not yet compare the content it ran with the content it planned**: the run's admission record carries the content hash its admission measured (`content_hash`), but the trial does not read it, and `collect` records the directory as it reads it then (`standard-pack-trial.md` "What `--collect` can know and what it cannot"). Change nothing in the pack directory between the plan and the end.

### 5.6 Enabling: adopt again with a decision, then build

1. **The decision document** `actingcommand.pack-decision.v1`, a file you write: `class` (`business`, a rule judged by its value; `maintenance`; or `held`), `trigger` (a predicate object as `schedule.json` writes it; a new rule needs one), `value` (`"pending"`, or `{"fixed_value_milli": <n>, "origin": "pack"}`; the origin `model` is refused, `value_origin_model`), optionally `consumes`, `produces`, `overrides` (`standard-pack-authoring.md` "The decision document"). The value is not yours to invent: with `"pending"` the rule is written and waits for its value (`waits_for_value`), and `next` names `ours/model/bindings.json`.
2. **Adopt again**: `ac_standard_pack_adopt` (`repo`, `record_id`, with `session_state_dir` where §5 gives it, or `package`, `decide` the file, `replace: true`, the same `state_dir` the trial used). A class other than `held` lifts the hold and needs `review.recording_checked`; the answer warns `enabling_needs_authorization` (adoption enables nothing: the placement does) and, without a passed trial of this content, `not_tried`. By `record_id` it replaces the cases the first adoption took with the recording's set (`cases.replaced`, `cases.kept`; §5.2).
3. **The declaration, now that the rule is written** (§5.3): `ac_standard_pack_catalog` (`root`, `refresh` a list holding the `package_id`, `plan: true`); read `refreshed` (its `after` against the rule; `state` `added` when the package had no entry, `replaced` when its entry held something else) and `added`; then the same call without `plan`. To give the entry its display, write the display file and call with `displays` (`plan: true` first): read `display_written` and `groups_added`. **Commit** the checkout (the pack, `schedule.json`, the cases, `pack-catalog.json`).
4. **Build**: `ac_standard_pack_build` (`repo` the top of the clean work tree, `source_repository` `<owner>/<name>`, `out` a new or empty directory outside the checkout, optionally `commit`, which must be HEAD). It writes `<game>-resources-<sha7>.zip`, `SHA256SUMS` and `build.json` into `out`; its answer carries `files`, `readiness` and `next.place`, the plan's words: `["place", "<zip>", "--sums", "<SHA256SUMS>", "--plan"]`.

### 5.7 The placement, the break-in and the undo

`ac_place` places a built bundle into the installation (`install-cli.md` "The authorization a placement names"). It is irreversible and journalled. It is a tool of the module `rescue`, as `ac_op_status` and `ac_undo` are: a seat that does not list them hands the build over to the session that coordinates the work (§2).

**Where their whole answer is** (`apps/actinglab-mcp/MANUAL.md` sections 5 and 6). A success of these three too large for the budget is written whole to the export directory and answered its header: `ac_place` (with `plan`, and a placement) keeps the plan's `schedule_changes` under `plan` and its `next`, `ac_undo` its `reverses` (with `plan`) and `schedule_changes`, where it carries them, `ac_op_status` its `unfinished`, `last_operations` and, with `op`, `journal`. A part that does not fit whole first loses its rows `packages` (counted `packages_total`), then has every list counted (`enters_total` and the others), then is left out; `truncated` names, as JSON pointers into the file, every member the header does not hold as the file does, `export` is `{path, sha256, size}` and `answer_exported` the first warning. Read `schedule_changes` before you place: in the header, or in the file where `truncated` names its lists. `material_export_failed` first and no `export`: the file could not be written, and the header is all there is.

1. **The plan**: `ac_place` (`bundle` the zip, `sums`, `plan: true`). Its `schedule_changes` say which packages the placement makes enter or leave scheduling: `packages` (each `{package_id, before, after, content?}`), `enters`, `leaves`, `content_only`, and `unread` when a side could not be read.
2. **The placement**: the same call without `plan`, with `authorization` (the reference of the owner's authorization; every placement whose plan changes something needs one) and `enables`, every package of the plan's `enters`, once each (absent when nothing enters). Refused before anything is held or written: `place_authorization_missing` (`install_command_refused`), `actingctl_usage_invalid` (`unusable`) for a malformed reference, `place_enables_unknown` (the plan could not name what enters), `place_enables_mismatch` (`expected`, `given`), `pack_invalid`. A failed check after the placement is `post_check_failed`, its `next` naming the undo. The tool records the authorization and grants nothing.
3. **Read it back**: `ac_op_status` (`op` the operation id) shows the journal, its inputs `authorization` and `enabled`; `ac_standard_pack_status` with `installation: true` shows `enabled.state` and `enabled.authorization`.
4. **The break-in.** A package's content is in its break-in on each pair (catalog task, instance) from its first settled run there until both hold: `scheduling.suspension.break_in_runs` such runs (20) and `scheduling.suspension.break_in_ms` since the first (604 800 000 ms, seven days). Inside it, a screen the package never recorded ends a scheduled run with `contained_task_linear_unrecorded_screen` and pauses its pair at once (`pause_cause` `unrecorded_screen`); after it, the same screen ends the run with `page_confirmation_failed` and the ordinary failure handling. A changed content starts a new break-in. `ac_standard_pack_status` with `installation` answers each pair's `break_in` (`content_hash_prefix`, `runs`, `runs_needed`, `first_settled_at_unix_ms`, `since_first_ms`, `ms_needed`, `in_break_in`) and `suspended` (`pause_cause`). There is no manual lift: a paused pair is lifted once its package's content changes (§5.8).
5. **The undo**: `ac_undo` (`op`, `plan: true` first) answers `reverses` and `schedule_changes`, with the warnings `undo_enters_scheduling` (`names`, the packages the undo lets enter scheduling again) and `undo_scheduling_unknown`. It asks for no authorization. Report what enters before you undo.

### 5.8 When a run meets a screen the package never recorded

The run names such a screen only when, after a press whose next page did not come, the screen stays still and shows nothing the package declares; then its `artifact_id` is the frame of that screen. Otherwise it ends with the failed wait's code, usually `page_confirmation_failed`. From that frame to a new package:

1. `ac_run_evidence` (`id` the run's `request_id`) lists the run's frames as material refs `<seq>:<event_id>:<artifact_id>`; the ref whose last part is the failure's `artifact_id` is the frame. `ac_material` (`ref`, `export: true`) writes it whole as a PNG and answers its file. Take it in time: a pinned frame nobody confirmed as read is removed after `frame_retention_evidence_max_age_days` (15 by default).
2. On the stopped recording: a screen that ends the business after the last press is an ending (§4.8); a pop-up after which the path goes on is an optional step inserted after the step it followed (§4.7). The run names that step by `pack_operation_id`, `step_<nn>_click`, `step_<nn>_select` or `step_<nn>_key`, `nn` its number in the package: in `ac_record_status` find the step whose `artifact_step` is `nn` and take its `index`. Give the file `ac_material` wrote as it is: a frame of a pure scale of the recording's size is converted by the Lab.
3. `ac_record_stop` (`regenerated`); then `ac_standard_pack_adopt` with `replace: true` and no `decide` (the rule and the held entry are kept byte for byte, only the pack directory is replaced; by `record_id` the new cases replace the earlier adoption's, by `package` copy them over the old ones by hand, §5.2), commit, build and place (§5.7): a content-only placement names its `authorization` and no `enables`. The changed content lifts the pair's pause at its next due evaluation.

Any other answer (a result after an earlier press, a branch that goes on, a step missing in the middle) is a new recording.

## 6. What does not exist yet

- Adoption by `package` takes no regression case and judges no recording, and adoption reads the trial records only under its own `state_dir`, which a `replace` needs on the checkout's volume (§5, §5.2).
- `standard-pack build` and `resource verify` do not run the regression cases, and a case checks whether a located text passes, not where its press lands (§5.2, §4.3).
- A trial does not yet compare the content it ran with the content it planned (§5.5).
- Readiness does not read the installation's six items (I1 to I6), and `place --plan` answers no readiness (§5.4).
- Adoption writes no entry of `pack-catalog.json`, and no tool proposes a display: the display file is yours (§5.3).
- `ac_place`, `ac_op_status`, `ac_undo`, `ac_vision_check` and `ac_vision_manifest` are listed only for a seat registered with the module `rescue` (§2).
- A list that fits on one screen is no scrolling choice: use the fixed slots (§4.4).
- Only 1280×720: no recording on another screen size, and no package for phones (§2).

## 7. A worked example

An invented application, for illustration only. Its main screen has a title at the top and a "Catalogue" button. The catalogue lists fourteen named items in a list that scrolls vertically (Amber, Birch, Cedar, Dune, Elm, Fern, Grove, Heath, Iris, Juniper, Kelp, Laurel, Moss, Nettle), each row an icon, a name and an "Open" control on the right; a header strip "All items" stands above the first row, and a footer line "No more items" below the last. Opening a row shows a detail screen with a "Collect" button; collecting shows a "Collected" banner. The business collects the wanted item; the list's content is not fixed, so the choice reads the list to its end.

The instance alias is `emu-a`; the game is `neutral` and the server `std`, both given by the installation; `<checkout>`, `<work>` (a directory of your own outside the checkout) and `<out>` stand for absolute paths of your machine, and actinglab's default state directory lies on the checkout's volume (§5). Each `→` is one MCP call with its arguments; `←` names the answer members to read before the next call.

**Before.**

```text
→ ac_status {}
← daemon answers; emu-a is listed, nothing runs on it
→ ac_app_launch {"instance": "emu-a"}
← the receipt; then bring the application to its main screen by its own controls
```

**Stage one.**

```text
→ ac_record_start {"instance": "emu-a", "task_id": "catalogue_collect", "locale": "en"}
← lab_recording.status "active", lab_recording.record_id (keep it: <record_id>), hold.state "own",
  display_last_observed {"width": 1280, "height": 720, …}, record_flag_reachable true
  (a {handle, job_phase} answer: ac_run_show {"id": "<handle>", "wait_s": 25} until it ends)

→ ac_tap {"instance": "emu-a", "x": 1140, "y": 652, "record": true}
← record.step 1, record.after.path (the list's first screen), record.warnings []

→ ac_swipe {"instance": "emu-a", "x1": 600, "y1": 520, "x2": 600, "y2": 280, "duration_ms": 600, "record": true}
← record.step 2 (moves 240 px; the list is 480 px tall, a row 80 px: at most 399)
→ ac_swipe {"instance": "emu-a", "x1": 600, "y1": 520, "x2": 600, "y2": 280, "duration_ms": 600, "record": true}
← record.step 3
→ ac_swipe {"instance": "emu-a", "x1": 600, "y1": 520, "x2": 600, "y2": 280, "duration_ms": 600, "record": true}
← record.step 4; record.after.path shows "No more items": the list's true last screen
→ ac_swipe {"instance": "emu-a", "x1": 600, "y1": 280, "x2": 600, "y2": 520, "duration_ms": 600, "record": true}
← record.step 5 (back); the Fern row shows

→ ac_tap {"instance": "emu-a", "x": 1032, "y": 380, "record": true}
← record.step 6 (the centre of the Fern row's "Open" control on step 5's screen after);
  record.after.path is the detail screen
→ ac_tap {"instance": "emu-a", "x": 1100, "y": 636, "record": true}
← record.step 7; record.after.path shows "Collected", record.after.still true

→ ac_record_archive {"instance": "emu-a"}
← status "archived", lab.manifest, lab.warnings [] (no final_frame_missing), hold.release "resumed"
```

**Stage two.**

```text
→ ac_record_reopen {"instance": "emu-a", "record_id": "<record_id>"}
← status "reopened"
→ ac_record_ready {"instance": "emu-a"}
← status "material"; missing: page_recognition for steps 1, 2, 3, 4, 5, 6, 7 and 8;
  warnings press_point_only; sources {game: "installation_config", server: "installation_config",
  locale: "recording"}
→ ac_record_review {"instance": "emu-a", "step": 1}
← steps[0].before.path and .preview, press.point, after.path, missing
```

Step 1, the main screen: its title recognises it; the Catalogue button is pressed; a bright corner proves it is not dimmed.

```text
→ ac_record_mark {"instance": "emu-a", "step": 1, "page": "home",
                  "box": ["recognize=40,24:279,63", "press=1040,620:1239,683"],
                  "color": ["home/bright=1200,24,16,16"]}
← status "marks_recorded"; boxes[0].mark "box/01/40_24_240_40", boxes[0].frames (taken own or added)
```

Fold the four swipes (steps 2 to 5) into the press of step 6:

```text
→ ac_record_mark {"instance": "emu-a", "fold_scan": 6, "fold_first": 2}
← status "scan_folded"; scan_fold.axis "vertical", scan_fold.moves (three forward, one back)
→ ac_record_mark {"instance": "emu-a", "step": 6, "page": "catalogue", "box": ["recognize=40,24:359,63"]}
← status "marks_recorded"
```

The scrolling select, written to `<work>\select-6.json` (UTF-8 without a byte order mark); `read` names the model `ac_vision_check` lists:

```json
{"schema_version": "actingcommand.lab-record-mark.v1", "step": 6,
 "select": {
   "rows": {"row": {"x": 80, "y": 172, "width": 1040, "height": 80},
            "anchor": {"x": 96, "y": 188, "width": 48, "height": 48},
            "name": {"x": 160, "y": 192, "width": 360, "height": 40},
            "click": {"x": 960, "y": 188, "width": 144, "height": 48}},
   "scan": {"list": {"x": 80, "y": 140, "width": 1040, "height": 480},
            "start": {"x": 96, "y": 144, "width": 200, "height": 24},
            "end": {"x": 96, "y": 588, "width": 240, "height": 24},
            "stop": "end"},
   "read": {"languages": ["en"], "timeout_ms": 1000, "minimum_confidence_milli": 800,
            "model_ref": "<model_ref>", "model_sha256": "<model_sha256>"},
   "names": [{"id": "amber", "aliases": ["Amber"]}, {"id": "birch", "aliases": ["Birch"]},
             {"id": "cedar", "aliases": ["Cedar"]}, {"id": "dune", "aliases": ["Dune"]},
             {"id": "elm", "aliases": ["Elm"]}, {"id": "fern", "aliases": ["Fern"]},
             {"id": "grove", "aliases": ["Grove"]}, {"id": "heath", "aliases": ["Heath"]},
             {"id": "iris", "aliases": ["Iris"]}, {"id": "juniper", "aliases": ["Juniper"]},
             {"id": "kelp", "aliases": ["Kelp"]}, {"id": "laurel", "aliases": ["Laurel"]},
             {"id": "moss", "aliases": ["Moss"]}, {"id": "nettle", "aliases": ["Nettle"]}],
   "match": {"max_distance": 1, "minimum_margin": 1},
   "wanted": {"name": "catalogue_item", "default": "fern"},
   "pressed": "fern"}}
```

```text
→ ac_record_mark {"instance": "emu-a", "request": "<work>\\select-6.json"}
← select (its self-test passed on every screen), select.scan.numbers (each name, value, origin, from);
  a refusal: §4.5's table
```

Steps 7 and 8, the detail screen and the final screen:

```text
→ ac_record_mark {"instance": "emu-a", "step": 7, "page": "detail",
                  "box": ["recognize=40,24:399,63", "press=980,600:1219,671"]}
← status "marks_recorded"
→ ac_record_mark {"instance": "emu-a", "step": 8, "page": "collected", "box": ["recognize=440,300:839,379"]}
← status "marks_recorded"
→ ac_record_ready {"instance": "emu-a"}
← status "ready", missing [], checks (all eight complete), warnings []
→ ac_record_stop {"instance": "emu-a", "dry_run": true}
← status "validated", lab.would_write, lab.warnings, lab.selects[0].scan, lab.cases.status "to_write"
→ ac_record_stop {"instance": "emu-a"}
← lab.status "generated", lab.package_id "neutral.std.catalogue_collect", lab.path,
  lab.cases.directory, lab.cases.ids ["step_01", "step_02", "step_03", "final"], lab.cases.warnings,
  lab.adoption_example
```

The package's steps are 1 (`home`), 2 (`catalogue`, the select), 3 (`detail`) and 4 (`collected`); its usual end's key is `catalogue_collect_done`, the select's empty end `catalogue_collect_not_found`, and its variable `catalogue_item`.

**Stage three.**

```text
→ ac_standard_pack_adopt {"repo": "<checkout>", "record_id": "<record_id>",
                          "held_reason": "recorded; not tried on a real application yet", "dry_run": true}
← status "validated", would_write, cases.would_write (the four cases, each run already), readiness
→ ac_standard_pack_adopt {"repo": "<checkout>", "record_id": "<record_id>",
                          "held_reason": "recorded; not tried on a real application yet"}
← status "adopted", class "held", review.recording_checked true, review.trial.state "none",
  warnings [not_tried], other_packages_warnings (the other packages' warnings, counted), written, cases.state "taken", cases.written, cases.not_proven [],
  check_tree, readiness.readiness "waiting" (S2 held), next
```

The adoption wrote the four cases as `<checkout>\regression\neutral\neutral.std.catalogue_collect.step_01\` (and `.step_02`, `.step_03`, `.final`). Run them again over the checkout, and check the root:

```text
→ ac_resource_regression {"repo": "<checkout>", "package_id": "neutral.std.catalogue_collect"}
← counts {cases: 4, passed: 4, failed: 0, partial: 0, not_run: 0}, cases
→ ac_standard_pack_check {"root": "<check_tree>\\installed\\neutral"}
← catalog (the pack is under unlisted: it has no entry yet, which is no error; §5.3), warnings
```

Commit the checkout. Then the trial, on the instance it was recorded on:

```text
→ ac_standard_pack_trial {"instance": "emu-a", "package": "<checkout>\\packs\\neutral.std.catalogue_collect",
                          "expect": "catalogue_collect_done", "plan": true}
← sent false, plan.content_hash_prefix, plan.ends, plan.variables [{name: "catalogue_item",
  value: ["fern"], source: "default"}], run_requires ["authorization"]
→ ac_standard_pack_trial {"instance": "emu-a", "package": "<checkout>\\packs\\neutral.std.catalogue_collect",
                          "expect": "catalogue_collect_done", "authorization": "<the owner's reference>"}
← (a {handle, job_phase} answer: ac_run_show {"id": "<handle>", "wait_s": 25} until it ends)
  request_id, end {state, outcome_key "catalogue_collect_done"}, matched true, record
→ ac_run_evidence {"id": "<request_id>"}
← the run's steps, pages and frames
```

Enable it. `<work>\decision.json`:

```json
{"schema_version": "actingcommand.pack-decision.v1", "class": "business",
 "trigger": {"kind": "clock", "schedule": {"kind": "interval", "clock_source": {"kind": "local"},
                                           "every_ms": 86400000, "anchor_ms": 0}},
 "value": "pending"}
```

```text
→ ac_standard_pack_adopt {"repo": "<checkout>", "record_id": "<record_id>",
                          "decide": "<work>\\decision.json", "replace": true}
← status "adopted", class "business", review.trial.state "passed",
  warnings [enabling_needs_authorization], waits_for_value, next (ours/model/bindings.json),
  cases.state "taken", cases.replaced (the four cases of the first adoption), cases.kept
→ ac_standard_pack_catalog {"root": "<checkout>", "refresh": ["neutral.std.catalogue_collect"], "plan": true}
← status "planned", refreshed [{package_id "neutral.std.catalogue_collect", state "added",
  after {consumes [], produces []}}] (from the rule just written: it names no pool), added, document,
  warnings
→ ac_standard_pack_catalog {"root": "<checkout>", "refresh": ["neutral.std.catalogue_collect"]}
← status "written"
```

The entry's display, written by you to `<work>\displays.json`:

```json
{"schema_version": "actingcommand.pack-catalog-displays.v1",
 "packs": [{"package_id": "neutral.std.catalogue_collect",
            "title": {"en": "Collect from the catalogue"},
            "summary": {"en": "Opens the catalogue, chooses the wanted item and collects it."}}]}
```

```text
→ ac_standard_pack_catalog {"root": "<checkout>", "displays": "<work>\\displays.json", "plan": true}
← status "planned", display_written ["neutral.std.catalogue_collect"], groups_added [], document
→ ac_standard_pack_catalog {"root": "<checkout>", "displays": "<work>\\displays.json"}
← status "written"
```

Commit the checkout, then build and place:

```text
→ ac_standard_pack_build {"repo": "<checkout>", "source_repository": "<owner>/<name>", "out": "<out>"}
← files, readiness, next.place ["place", "<out>\\neutral-resources-<sha7>.zip", "--sums", "<out>\\SHA256SUMS", "--plan"]
→ ac_place {"bundle": "<out>\\neutral-resources-<sha7>.zip", "sums": "<out>\\SHA256SUMS", "plan": true}
← schedule_changes.enters ["neutral.std.catalogue_collect"], leaves [], content_only []
→ ac_place {"bundle": "<out>\\neutral-resources-<sha7>.zip", "sums": "<out>\\SHA256SUMS",
            "authorization": "<the owner's reference>", "enables": ["neutral.std.catalogue_collect"]}
← the operation id and each step's result
→ ac_op_status {"op": "<operation id>"}
← the journal: inputs.authorization, inputs.enabled ["neutral.std.catalogue_collect"]
→ ac_standard_pack_status {"root": "<standard_packs.root>\\neutral", "package_id": "neutral.std.catalogue_collect",
                           "installation": true}
← packs[0].enabled.state "enabled", enabled.authorization, pairs ([] while no run has settled; then each
  pair's break_in, in_break_in true)
```

The package is placed and enabled; its rule waits for its value (`"pending"`) before it is dispatched, and its first scheduled runs are its break-in (§5.7).
