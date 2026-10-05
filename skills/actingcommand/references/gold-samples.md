# Gold samples

One real call sequence for each recipe of `SKILL.md`, with abridged real answers. Every call and answer below was printed by a scripted MCP client in a CI one-off check of Workflow #338 (HS7097/ActingCommand-Runtime PRs #614 S1, #618 S2 and #619 S3). The client ran the exact-SHA Windows build of the product commit named, against the fixture runtime: a test actingd with fixture instances and packs, no emulator. Nothing was run for this file.

- `→` is a call: the tool and its arguments as sent. `←` is the answer, the `structuredContent` that equals its text content, with the time the client measured when it printed one. Lines starting with `#` are notes added here.
- `…` marks text left out here. `...(+N bytes)` marks a cut that is already in the CI log.
- Paths are temporary paths of the CI runner. `neutral.instance`, `fixture-instance-a` and `node.a` are fixture instances, and the packs are fixture packs.
- The tool table (`tools.md`) is generated at `3bf664e0`, whose tree equals product commit `24b752b7`. For the samples taken on earlier product commits of the same stack, the same scripted S1 and S2 checks passed again on the `24b752b7` build in run 37264244025 (S1 4/4, S2 3/3).

| Sample | One-off run | Job, step | Product commit (exact-SHA build run) |
|---|---|---|---|
| R1 | 37214752888 | S1 review fixes, "Review fixes F1 F2 F3 on the fixture runtime" | `6a58f199` (37213496844) |
| R2 | 37222145246 | S2 operator tools, "Fixture runtime - runs, stop, restart" and "Host fixture catalog - R5 tools, targets, card warning, pause and resume" | `d362dfe5` (37221085637) |
| Stop | 37222145246 | S2 operator tools, "Fixture runtime - runs, stop, restart" | `d362dfe5` (37221085637) |
| R3 | 37264244025 | S3 fix round evidence, "S3 evidence and fix round" | `24b752b7` (37260586003) |
| R4 | 37214752888, 37208525376 | S1 review fixes, "Review fixes F1 F2 F3 on the fixture runtime" and "Regression - the earlier scripted dual-era client"; S1 scripted MCP client, "Scripted MCP client on the fixture runtime" | `6a58f199` (37213496844), `16e7da0b` (37207267829) |
| R5 | 37222145246 | S2 operator tools, "Host fixture catalog - R5 tools, targets, card warning, pause and resume" | `d362dfe5` (37221085637) |
| Lab | 37264244025 | S3 fix round evidence, "S3 evidence and fix round" | `24b752b7` (37260586003) |

## R1. Health

Run 37214752888. The fixture instance had a failed manual run. Between the calls the test restarted actingd and then stopped it.

```text
→ ac_overview {}
← {"ok":true,"result":{"daemon":{"online":true,"owner_epoch":"epoch_18db550de4cc40340000000000000001"},"instances":[{"alias":"neutral.instance","instance_id":"instance_18db55eae43df5a40000000000000001","lease_active":false,"queued":0,"run":{"schema_version":"actingcommand.run-status.v1",…,"dispatch":"manual","origin":"cli",…,"state":"failed","terminal":{"outcome":"failure","failure_code":"page_confirmation_failed","executed_steps":1,"sequence":324,"event_id":"evt_18db550de4cc40340000000000000175"},"lease":{"lease_ ...(+334 bytes)

→ ac_events {"limit":2}
← {"ok":true,"result":{"events":[{"seq":1,"ts":1791129440321,"type":"perf.summary","severity":"info","request_id":"request_18db550de4cc40340000000000000002"},{"seq":2,"ts":1791129440325,"type":"command.validated","severity":"info","request_id":"request_18db550de4cc40340000000000000009"}],"next_cursor":"v1.57a5168cf3e34cdd.1.328.2.a3d4bc3bf157d5fa696eec60c00cd5105f410ebeeafb93cb3c72d77bab1906e2","snapshot_ledger_position":328}}

# the test restarts actingd; one ac_overview that reads the new owner_epoch is left out
→ ac_events {"limit":2,"cursor":"v1.57a5168cf3e34cdd.1.328.2.a3d4bc3bf157d5fa696eec60c00cd5105f410ebeeafb93cb3c72d77bab1906e2"}
← {"ok":false,"error":{"class":"usage","code":"cursor_invalid","message":"this cursor belongs to another query, Runtime connection or server process; read again from the first page without a cursor","blocked_by":null,"details":{}}}

# three ac_diagnose calls are left out (one is in R4); then the test stops actingd
→ ac_overview {}
← {"ok":true,"result":{"daemon":{"online":false,"error":{"code":"runtime_unavailable","message":"actingd does not answer for state root D:\\a\\_temp\\.tmpacnuuU: runtime client error runtime_info_unavailable during discover_runtime"}},"lab_tool":{"present":false},"install":{"root":"D:\\a\\_temp\\.tmpuEtscz","state_root":"D:\\a\\_temp\\.tmpacnuuU"}}}
```

The offline answer has no `instances` field: no answer, not an empty system. The cursor died with the restart.

## R2. Run one task pack once

Run 37222145246. (a) A pack run on the fixture instance, first from a server without the operator tier:

```text
→ ac_run_pack {"instance":"neutral.instance","package":"D:\\a\\_temp\\.tmpdMs6Y4\\neutral-task.zip","package_ref":"cd5be7eb0bc148e4b5662ba2f15d1504ca33423f75eaddeec356c1a43f6850a0"}
← (0 ms) {"ok":false,"error":{"class":"safety","code":"tier_not_enabled","message":"ac_run_pack belongs to tier operator, which this server was not started with","blocked_by":"the person restarts mcp-serve with --tier including this tier","details":{"required_tier":"operator"}}}

# a server with the operator tier
→ ac_run_pack {"instance":"neutral.instance","package":"D:\\a\\_temp\\.tmpdMs6Y4\\neutral-task.zip","package_ref":"cd5be7eb0bc148e4b5662ba2f15d1504ca33423f75eaddeec356c1a43f6850a0","deadline_s":120}
← (42 ms) {"ok":true,"result":{"handle":"request_18db7478e453c1e00000000000000007","correlation_id":"correlation_18db7478e453c1e00000000000000003","phase":"submitting"}}

→ ac_get_run {"handle":"request_18db7478e453c1e00000000000000007","wait_s":20}
← (1044 ms) {"ok":true,"result":{"schema_version":"actingcommand.run-status.v1","request_id":"request_18db7478e453c1e00000000000000007",…,"dispatch":"manual","origin":"cli",…,"state":"succeeded","terminal":{"outcome":"success","final_page":"neutral/terminal","executed_steps":1,"sequence":72,"event_id":"evt_18db6cdcdcf8e498000000000000007b"},"lease":{"lease_id":"lease_18db74dcdd0f0a200000000000000001","terminal":"released"},…,"job":{"kind":"run_pack","phase":"done","warnings":[],"outcome":{"receipt_state":"completed"}}}}

# the test's ledger check of that correlation:
S2|PROVENANCE|correlation correlation_18db7478e453c1e00000000000000003: ["governance.identity_declared", "client.action(surface_mcp=true,control=Some(\"ac_run_pack\"))", "cli.command", "command.received", …
```

`origin` is `cli` because the server uses the CLI's connection identity; the `client.action` with surface `mcp` in the same correlation marks the call as coming from MCP.

(b) Exclusive use: pause, then resume exactly that pause. The fixture paused globally (no instance); with an instance the pair works the same way for that instance.

```text
→ ac_pause {}
← {"ok":true,"result":{"paused":{"kind":"scheduling_paused","scope":{"kind":"global"},"revision":1},"owner_epoch":"epoch_18db7dbf5b69b9240000000000000001"},"warnings":[{"code":"governance_client_not_allowed","message":"actingd's allowed_clients does not list actingctl-mcp; the Runtime recorded a warning and the write went ahead",…}]}

# another client resumes it (revision 2); the test pauses again
→ ac_pause {}
← {"ok":true,"result":{"paused":{"kind":"scheduling_paused","scope":{"kind":"global"},"revision":3},"owner_epoch":"epoch_18db7dbf5b69b9240000000000000001"},"warnings":[…]}

→ ac_resume {"expected_owner_epoch":"epoch_18db7dbf5b69b9240000000000000001","expected_revision":1}
← {"ok":false,"error":{"class":"usage","code":"runtime_request_rejected","message":"runtime client error runtime_request_rejected during resume_scheduling with runtime code InvalidRequest state=Denied fatal=false host code scheduling_pause_revision_mismatch during resume_scheduling","blocked_by":null,"details":{…,"host_failure":{"code":"scheduling_pause_revision_mismatch","operation":"resume_scheduling"},"receipt_state":"denied",…}}}

→ ac_resume {"expected_owner_epoch":"epoch_18db7dbf5b69b9240000000000000001","expected_revision":3}
← {"ok":true,"result":{"resumed":{"kind":"scheduling_resumed","scope":{"kind":"global"},"revision":4}},"warnings":[…]}
```

(c) An uncertain run: actingd was killed during the run and started again.

```text
→ ac_run_pack {"instance":"neutral.instance",…,"deadline_s":120}
← (32 ms) {"ok":true,"result":{"handle":"request_18db77da6081d1f40000000000000007","correlation_id":"correlation_18db77da6081d1f40000000000000003","phase":"submitting"}}

# the test kills actingd and starts it again
→ ac_get_run {"handle":"request_18db77da6081d1f40000000000000007","wait_s":5}
← (27 ms) {"ok":true,"result":{"schema_version":"actingcommand.run-status.v1",…,"state":"interrupted_unterminated","lease":{"lease_id":"lease_18db74fa4e15f2480000000000000001","terminal":null},…,"evidence":{"snapshot_ledger_position":251,"admitted_sequence":212,"restarted_after_admission":true},"job":{"kind":"run_pack","phase":"failed","warnings":[],"outcome":{"error":{"class":"uncertain","code":"runtime_receipt_header_failed",…
```

That build did not contain Runtime PR #617, which closes such a run at the next start as cancelled with `contained_task_recovered_after_restart`.

## Stop a run

Run 37222145246. MCP process A submits and is killed; a new process B stops the run by its handle.

```text
# process A
→ ac_run_pack {"instance":"neutral.instance","package":"D:\\a\\_temp\\.tmpdMs6Y4\\neutral-task.zip","package_ref":"cd5be7eb0bc148e4b5662ba2f15d1504ca33423f75eaddeec356c1a43f6850a0","deadline_s":120}
← (32 ms) {"ok":true,"result":{"handle":"request_18db76946710ada00000000000000007","correlation_id":"correlation_18db76946710ada00000000000000003","phase":"submitting"}}

# process A is killed; process B
→ ac_stop_run {"handle":"request_18db76946710ada00000000000000007","wait_s":25}
← (2637 ms) {"ok":true,"result":{"cancellation":{"state":"terminal","deadline_monotonic_ms":120250,"outcome":"cancelled","reason":"client_requested","task_terminal":{"sequence":156,"event_id":"evt_18db6e6d597c7f48000000000000005b"},"lease_terminal":{"sequence":159,"event_id":"evt_18db6e6d597c7f48000000000000005f"},"lease_disposition":"released"},"touch_release":"done","job_phase":"done"}}

→ ac_get_run {"handle":"request_18db76946710ada00000000000000007","wait_s":20}
← (31 ms) {"ok":true,"result":{…,"state":"cancelled","terminal":{"outcome":"cancelled","failure_code":"contained_task_cancelled","executed_steps":1,"cancellation_reason":"client_requested","sequence":156,"event_id":"evt_18db6e6d597c7f48000000000000005b"},"lease":{"lease_id":"lease_18db766d5991fda00000000000000001","terminal":"released"},"progress":{"step_index":0,"operation_label":"open_terminal","page":"neutral/home"},…}}
```

The regression run 37264244025 on the `24b752b7` build printed the same answer shape from a fresh process (`touch_release` `done`, `lease_disposition` `released`).

## R3. Check a pack

Run 37264244025. The package is the one the Lab sample below generated.

```text
→ ac_pack_check {"package":"D:\\a\\_temp\\.tmpcYmY4k\\lab-out\\60b38b749b7e59e689a0d99b5c1c153c48663fb9d73c3e767049e5bee3ff60dd.json"}
← (63 ms) {"ok":true,"result":{"digest":{"status":"valid","package":"…","reference":{"schema_version":"actingcommand.package.content-directory.v1","sha256":"60b38b749b7e59e689a0d99b5c1c153c48663fb9d73c3e767049e5bee3ff60dd"},"package_id":"neutral.test.neutral_task","server":"test","entry_task_id":"neutral_task","file_count":3,"byte_count":92058},"preflight":{"schema_version":"actingcommand.package-preflight.v1","status":"prepared","executed":false,"task_id":"neutral_task",…,"coverage":{"declaration_parse":"passed","package_load":"passed","task_prepare":"passed","recognition_execution":"not_run","execution":"not_run"},…,"production_global_ledger_written":false}}}

→ ac_pack_check {"package":"…","package_ref":"{\"schema_version\":\"actingcommand.package.content-directory.v1\",\"sha256\":\"60b38b749b7e59e689a0d99b5c1c153c48663fb9d73c3e767049e5bee3ff60dd\"}"}
# the test's summary of the answer:
S3|PACK|given the digest's reference -> ok true package_ref_check {"given":"{\"schema_version\":\"actingcommand.package.content-directory.v1\",\"sha256\":\"60b38b749b7e59e689a0d99b5c1c153c48663fb9d73c3e767049e5bee3ff60dd\"}","matches_digest":true}

→ ac_pack_check {"package":"…","package_ref":"0000000000000000000000000000000000000000000000000000000000000000"}
← (56 ms) {"ok":false,"error":{"class":"usage","code":"package_invalid","message":"fatal containment error: hash mismatch for instance semantic_package_preflight: expected 0000000000000000000000000000000000000000000000000000000000000000, actual d6ccccb04cd4caf9a00a0798972480aa380c917ab2445c3b09e8d845654fc1df","blocked_by":null,"details":{"lab_exit_code":2,"lab_error":{…"details":{"coverage":{"declaration_parse":"not_completed","package_load":"failed","task_prepare":"not_run","execution":"not_run"}}},"digest":{…},"package_ref_check":{"given":"0000000000000000000000000000000000000000000000000000000000000000","matches_digest":false}}}}
```

## R4. Diagnose a failed task

(a) Run 37214752888, step "Review fixes F1 F2 F3". Three manual runs of a fixture pack had failed on the instance.

```text
→ ac_diagnose {"instance":"neutral.instance"}
← {"ok":true,"result":{"instance":{"alias":"neutral.instance","instance_id":"instance_18db55eae43df5a40000000000000001"},"window":{"since_unix_ms":1791043042794},"runs":[{"schema_version":"actingcommand.run-status.v1","request_id":"request_18db56019638381c0000000000000005",…,"run_id":"run_18db550de4cc4034000000000000012f",…,"state":"failed","terminal":{"outcome":"failure","failure_code":"page_confirmation_failed","executed_steps":1,"sequence":324,"event_id":"evt_18db550de4cc40340000000000000175"},"lease":{"lease_id":"lease_18db4d0defbf9eb40000000000000004","terminal":"released"},"e ...(+3434 bytes)
# the test's summary:
S1F|F2|runs in the window: [String("failed"), String("failed"), String("failed")]
```

The suspended-report rows of that step came from a stand-in actingd the test built, so they are not quoted here.

(b) Same run, step "Regression - the earlier scripted dual-era client". A server started with `--root` of a test install whose configuration actingd could not decode. The step prints only the answer; the same script printed the call in run 37208525376 as `ac_diagnose {"instance":"neutral.instance"}`.

```text
← {"ok":true,"result":{"instance":{"alias":"neutral.instance","instance_id":"instance_18db4e4a211282ec0000000000000001"},"window":{"since_unix_ms":1791043136065},"runs":[],"errors_page":{"events":[],"more":false,"snapshot_ledger_position":86},"suspended":[],"lift":[],"repeating":[],"report_warnings":[],"suspended_report":{"status":"failed","code":"suspended_report_unreadable","message":"actingd suspended printed no JSON report: EOF while parsing a value at line 1 column 0","exit_code":1,"stderr_tail":"FATAL actingd: config_decode_failed\n"},"incomplete":true}}
```

The failed report is in the answer and `incomplete` is true; report both.

(c) Run 37208525376: the window limit, and evidence from `ac_events` rows' materials.

```text
→ ac_diagnose {"instance":"neutral.instance","since_unix_ms":1}
← {"ok":false,"error":{"class":"usage","code":"since_out_of_range","message":"since_unix_ms reaches back at most 7 days, to 1790518771502","blocked_by":null,"details":{"oldest_unix_ms":1790518771502}}}

→ ac_material {"ref":"26:evt_18db7af1a186b3700000000000000003:artifact_18db42f1a1868df00000000000000002","mode":"export"}
← {"ok":true,"result":{"path":"D:\\a\\_temp\\actingcommand-mcp\\materials\\7c7ad8bf09a8146fafd61b4c67202cc924e802fa938878b9ad3a73df3b9eea65.png","sha256":"sha256:7c7ad8bf09a8146fafd61b4c67202cc924e802fa938878b9ad3a73df3b9eea65","size":141}}

→ ac_material {"ref":"26:evt_18db7af1a186b3700000000000000003:artifact_18db42f1a1868df00000000000000002","mode":"text"}
← {"ok":false,"error":{"class":"usage","code":"material_not_text","message":"this material is image/png; read it with mode export","blocked_by":"ac_material mode export","details":{}}}
```

## R5. Resource targets

Run 37222145246, on the host fixture catalog.

```text
# a server with the observer tier only
→ ac_resources_list {"instance":"fixture-instance-a"}
← {"ok":true,"result":{"instance_alias":"fixture-instance-a","policy_instance":"fixture-instance-a","evaluated_at_unix_ms":1791136582540,"as_of_ledger_position":15,"catalog_hash":"sha256:b006a9d28fdee221a553bb621c1f44deed4f076cc99d201ae87a19f4f506c472","targetable":[{"resource":"fixture-pool-a","fact_key":"resource.primary","producing_tasks":["fixture.observe"],"scale_required":true,"importance_required":true,"observation":{"pending":"missing"}}],"not_targetable":[]}}

→ ac_targets_set {"instance":"fixture-instance-a","targets":[]}
← {"ok":false,"error":{"class":"safety","code":"tier_not_enabled","message":"ac_targets_set belongs to tier operator, which this server was not started with","blocked_by":"the person restarts mcp-serve with --tier including this tier","details":{"required_tier":"operator"}}}

# a server with the operator tier
→ ac_targets_get {"instance":"fixture-instance-a"}
← {"ok":true,"result":{"instance_alias":"fixture-instance-a","policy_instance":"fixture-instance-a","evaluated_at_unix_ms":1791136582600,"as_of_ledger_position":16,"catalog_hash":"sha256:b006a9d28fdee221a553bb621c1f44deed4f076cc99d201ae87a19f4f506c472","active":null}}

→ ac_targets_set {"instance":"fixture-instance-a","valid_days":1,"targets":[{"id":"target-a","resource":"fixture-pool-a","condition":{"kind":"at_least","amount":50},"scale":10,"importance_milli":500,"apply":{"mode":"adjust","weight":"score_stage"}}]}
← {"ok":true,"result":{"applied":{"instance_alias":"fixture-instance-a","policy_sha256":"sha256:1a562c49a15724b3f29b911e329fdf3c456f9c9121f50714d3ebe0c0500daebe","version":19,"event_id":"evt_18db7dbf5b69b924000000000000003b","previous_version":null,"replayed":false,"checked_catalog_hash":"sha256:b006a9d28fdee221a553bb621c1f44deed4f076cc99d201ae87a19f4f506c472","valid_until_unix_ms":1791222982608,"conditions_at_position":18,"conditions":[{"target_id":"target-a","resource":"fixture-pool-a","fact_key":"resource.primary","state":{"kind":"awaiting_observation","reason":"missing"}}]}}}

→ ac_targets_get {"instance":"fixture-instance-a"}
← {"ok":true,"result":{…,"active":{"policy_sha256":"sha256:1a562c49a15724b3f29b911e329fdf3c456f9c9121f50714d3ebe0c0500daebe","schema_version":"actingcommand.resource-targets.v2","version":19,"event_id":"evt_18db7dbf5b69b924000000000000003b","valid_until_unix_ms":1791222982608,"expired":false,"targets":[{"id":"target-a",…}],"conditions":[{"target_id":"target-a","resource":"fixture-pool-a","fact_key":"resource.primary","state":{"kind":"awaiting_observation","reason":"missing"}}]}}}

→ ac_targets_set {"instance":"fixture-instance-a","targets":[]}
← {"ok":true,"result":{"applied":{"instance_alias":"fixture-instance-a","policy_sha256":"sha256:3fa3012310b22733c12809622dfddf268a3b61b2071bfb4715f027b6b64293d2","version":23,"event_id":"evt_18db7dbf5b69b924000000000000003f","previous_version":19,"replayed":false,"checked_catalog_hash":"sha256:b006a9d28fdee221a553bb621c1f44deed4f076cc99d201ae87a19f4f506c472","valid_until_unix_ms":null,"conditions_at_position":22,"conditions":[]}}}

→ ac_targets_get {"instance":"fixture-instance-a"}
← {"ok":true,"result":{"instance_alias":"fixture-instance-a","policy_instance":"fixture-instance-a","evaluated_at_unix_ms":1791136582691,"as_of_ledger_position":24,"catalog_hash":"sha256:b006a9d28fdee221a553bb621c1f44deed4f076cc99d201ae87a19f4f506c472","active":null}}
```

`awaiting_observation` is not an error. The empty list without `valid_days` withdrew the policy.

## Lab recording

Run 37264244025: a recording made offline from PNG frames, with the record tools' state directory set by the test (so `record_flag_reachable` is false). The first mark carried many colour marks so that its answer would pass the output budget.

```text
# a server with the observer tier only
→ ac_record_status {"instance":"node.a"}
← (0 ms) {"ok":false,"error":{"class":"safety","code":"tier_not_enabled","message":"ac_record_status belongs to tier author, which this server was not started with","blocked_by":"the person restarts mcp-serve with --tier including this tier","details":{"required_tier":"author"}}}

# a server with the author tier
→ ac_record_start {"instance":"node.a","task_id":"neutral_task","locale":"en","record_id":"oneoff338","state_dir":"D:\\a\\_temp\\.tmpcYmY4k\\record-state"}
← (26 ms) {"ok":true,"result":{"status":"started","record":{"schema_version":"session-record-v0","record_id":"oneoff338","task_id":"neutral_task","instance":"node.a","status":"active",…},…,"lab_recording":{"status":"active",…,"defaults":{"template_threshold":0.95,"color_max_distance":20,"match_metric":"ccoeff_normed"}},…,"record_flag_reachable":false}}

→ ac_record_start {"instance":"node.a","task_id":"neutral_task","state_dir":"D:\\a\\_temp\\.tmpcYmY4k\\record-state"}
← (22 ms) {"ok":false,"error":{"class":"safety","code":"record_session_active","message":"recording session already active for node.a with task neutral_task","blocked_by":null,"details":{"lab_exit_code":3,"lab_error":{"code":"record_session_active","message":"recording session already active for node.a with task neutral_task","blocked_by":["session_record"]}}}}

→ ac_record_mark {"instance":"node.a","request":{"schema_version":"actingcommand.lab-record-mark.v1","frame":"D:\\a\\_temp\\.tmpcYmY4k\\record-frame.png","page":"home","add":[{"id":"m001","family":"color","region":{"x":0,"y":0,"width":1,"height":1}},{"id":"m002",…
← (50 ms) {"ok":true,"result":{"req_id":null,"export":{"path":"D:\\a\\_temp\\actingcommand-mcp\\materials\\8dbdacbdba1dc251ee181f9ddbe10590463d69e2f2e4c2c45ea443afb82e19cf.json","sha256":"sha256:8dbdacbdba1dc251ee181f9ddbe10590463d69e2f2e4c2c45ea443afb82e19cf","size":30737},"overflowed":true},"warnings":[{"code":"answer_exported","message":"actinglab record mark was performed or attempted as its answer reports; the answer is larger than the output budget and is whole in the exported file D:\\a\\_temp\\actingcommand-mcp\\materials\\8dbdacbdba1dc251ee181f9ddbe10590463d69e2f2e4c2c45ea443afb82e19cf.json. Do not run it again to see the answer.",…}]}

→ ac_record_mark {"instance":"node.a","request":{"schema_version":"actingcommand.lab-record-mark.v1","frame":"D:\\a\\_temp\\.tmpcYmY4k\\arrival-frame.png","page":"done","add":[{"id":"state/done","family":"color","region":{"x":0,"y":0,"width":1,"height":1}}]},"state_dir":"D:\\a\\_temp\\.tmpcYmY4k\\record-state"}
← (27 ms) {"ok":true,"result":{"status":"marks_recorded","record_id":"oneoff338","step":2,"step_opened":true,…,"marks":[{"id":"state/done","family":"color","step":2,"region":{"x":0,"y":0,"width":1,"height":1},"expected":[0,0,255],"self_test":{"status":"passed","frames":1,"single_sample":true,"margin":20.0,"max_observed_distance":0.0},…}],…,"closed_step":1,"dry_run":false}}

→ ac_record_stop {"instance":"node.a","game":"neutral","server":"test","lab_dir":"D:\\a\\_temp\\.tmpcYmY4k\\lab-out","state_dir":"D:\\a\\_temp\\.tmpcYmY4k\\record-state","dry_run":true}
← (53 ms) {"ok":true,"result":{"req_id":null,"export":{"path":"D:\\a\\_temp\\actingcommand-mcp\\materials\\4feb8941f78e85d9b9f83286a458d5e9325b2f3be2c097d5fa4de5a6e092c14c.json","sha256":"sha256:4feb8941f78e85d9b9f83286a458d5e9325b2f3be2c097d5fa4de5a6e092c14c","size":13395},"overflowed":true,"lab":{"binding_example":{…
# the test's check: the export equals the CLI's record stop --dry-run answer field by field
S3|PARITY|record stop --dry-run|identical, field by field (13395 bytes)

→ ac_record_stop {"instance":"node.a","game":"neutral","server":"test","lab_dir":"D:\\a\\_temp\\.tmpcYmY4k\\lab-out","state_dir":"D:\\a\\_temp\\.tmpcYmY4k\\record-state"}
← (57 ms) {"ok":true,"result":{"req_id":null,"export":{"path":"D:\\a\\_temp\\actingcommand-mcp\\materials\\4511758e8f11bb54f5edc26b891b75130675f1b347491e8865621a9051c2a59f.json","sha256":"sha256:4511758e8f11bb54f5edc26b891b75130675f1b347491e8865621a9051c2a59f","size":13371},"overflowed":true,"lab":{"binding_example":{"procedure_ref":"neutral.test.neutral_task","package_digest":{"schema_version":"actingcommand.package.content-directory.v1","sha256":"60b38b749b7e59e689a0d99b5c1c153c48663fb9d73c3e767049e5bee3ff60dd"},"operation_id":"operation.contained_task","yield_points":[],"scheduled_execution":{"mode":"device_registry","package_path":"D:\\a\\_temp\\.tmpcYmY4k\\lab-out\\60b38b749b7e59e689a0d99b5c1c153c48663fb9d73c3e767049e5bee3ff60dd.json"}},"binding_requires":["policy.catalog has a task whose procedure_ref is the binding's procedure_ref (neutral.test.neutral_task)","catalog_approval_ids match the approval of the catalog",…
S3|OVERFLOW|record stop: the five binding parts inline equal the exported answer's: true; lab.status "generated"

→ ac_binding_draft {"record_stop_export":{"path":"D:\\a\\_temp\\actingcommand-mcp\\materials\\4511758e8f11bb54f5edc26b891b75130675f1b347491e8865621a9051c2a59f.json","sha256":"sha256:4511758e8f11bb54f5edc26b891b75130675f1b347491e8865621a9051c2a59f"}}
← (297 ms) {"ok":true,"result":{"binding_example":{"procedure_ref":"neutral.test.neutral_task",…},"binding_requires":["policy.catalog has a task whose procedure_ref is the binding's procedure_ref (neutral.test.neutral_task)",… ...(+3741 bytes)
# the test's summaries of this answer:
S3|DRAFT|check_config inline: {"config":"D:\\a\\_temp\\.tmpcYmY4k\\install\\actingd.config.json","exit_code":1,"status":"failed","error":{"code":"config_invalid","stage":"assemble"},"report_export":{"path":"D:\\a\\_temp\\actingcommand-mcp\\materials\\a3592d93b30f1eeb22d5eb8a995472f6618ff325f7243df08247b94aba070f02.json","sha256":"sha256:a3592d93b30f1eeb22d5eb8a995472f6618ff325f7243df08247b94aba070f02","size":130}}
S3|DRAFT|manual_steps: ["policy.catalog has a task whose procedure_ref is the binding's procedure_ref (neutral.test.neutral_task)","catalog_approval_ids match the approval of the catalog",…,"edit the configuration, approve in the UI, restart the daemon"]

→ ac_binding_draft {"record_stop_export":{"path":"D:\\a\\_temp\\actingcommand-mcp\\materials\\4511758e8f11bb54f5edc26b891b75130675f1b347491e8865621a9051c2a59f.json","sha256":"sha256:0000000000000000000000000000000000000000000000000000000000000000"}}
← (0 ms) {"ok":false,"error":{"class":"usage","code":"arguments_invalid","message":"argument record_stop_export has a sha256 that does not match the file's content","blocked_by":null,"details":{"field":"record_stop_export"}}}

# a second ac_record_stop answers lab.status "already_generated" without binding_requires; the draft from it:
S3|F4|draft: ok true binding_requires null manual_steps ["edit the configuration, approve in the UI, restart the daemon"] warnings [{"code":"binding_requires_unavailable","message":"actinglab gives binding_requires only in the answer of the record stop that generated the package; this answer has none, so manual_steps holds only the closing step","blocked_by":null,"details":{}}]

# a Lab call that outlasts the call budget
→ ac_lab_observe {"instance":"node.a","scene":"D:\\a\\_temp\\.tmpcYmY4k\\frame.png","zip":"D:\\a\\_temp\\.tmpcYmY4k\\semantic.zip","expected_sha256":"d84b89494c08ef9df737147646687822d5dce03219befa75585380869afa524d","verbose":true,"test_capture_delay_ms":30000}
← (23001 ms) {"ok":true,"result":{"handle":"lab_job_046ae3caf0aeb1fc_23","job_phase":"running"}}
# in the same process, ac_get_run with wait_s 20 later answered the job done (left out); the process is killed; a new process:
→ ac_get_run {"handle":"lab_job_046ae3caf0aeb1fc_23"}
← (0 ms) {"ok":false,"error":{"class":"usage","code":"handle_unknown","message":"this job handle is not held by this MCP process: the process that started it ended, or it is another one's","blocked_by":null,"details":{}}}
```

The fixture install's configuration does not pass `check-config` (`config_invalid`), and the draft reports that instead of hiding it. The manual steps end with the user's three steps.
