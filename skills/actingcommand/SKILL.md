---
name: actingcommand
description: Operate an installed ActingCommand (Runtime v0.9.1 and v0.10.0, UI v0.9.0) through its command-line programs actingctl, actinglab, actingledger and actingd. Use when the machine has an ActingCommand install (actingd.config.json beside runtime\, tools\ and ui\ folders) and the user wants the system's health, what each emulator instance is doing, one task pack run once, a pack checked, a failed or suspended scheduled task diagnosed, instance resource targets applied, or (v0.10.0) a task pack recorded with Lab. CLI workshop manual v0. It never approves, never edits the daemon configuration and never starts or stops the daemon on its own.
---

# ActingCommand workshop manual (CLI v0)

This is version 0 of the ActingCommand program skill: a command-line manual for an AI agent that operates an installed ActingCommand. §1–§5 cover Runtime **v0.9.1** and UI **v0.9.0**; they were not checked again for v0.10.0 except the explicitly marked additions. §7 covers what Runtime **v0.10.0** adds: Lab recording and its self-check coverage, `linear_steps` packages with declared target consensus, and scheduled tasks' bounded immediate retries and suspension. It holds no game knowledge; that lives in the resource packs.

**Evidence key.** `[run]`: checked by running the v0.9.1 release binaries with side-effect-free arguments only. `path:line`: checked in source, in HS7097/ActingCommand-Runtime at v0.9.1 (`8e0ac191`), or in HS7097/ActingCommand-UI v0.9.0 (`3056039`) when marked `UI`. `file.md "Section"` (§7): checked in that contract and the [v0.10.0 Runtime candidate at `0abfe843`](https://github.com/HS7097/ActingCommand-Runtime/tree/0abfe84338c86df8b37730934490db8807e8db34) (PR #608). Nothing in §7 is `[run]`; source checks do not establish a released build or real-device behavior. **Unverified**: not checked; check it before you rely on it.

**Citation paths.** Runtime: an unqualified `main.rs` is `apps/actingctl/src/main.rs`; `actingd main.rs`, `config.rs` and `check_config.rs` are in `apps/actingd/src/`; `actinglab main.rs` and the other actinglab files (`cli_parse.rs`, `flag_args.rs`, `lab2_cli.rs`, `run_summary.rs`, …) are in `apps/actinglab/src/`; `ledger-forensics lib.rs`/`main.rs` are in `apps/ledger-forensics/src/`; `runtime.rs`, `package.rs`, `taskflow.rs`, `event.rs`, `event/ids.rs`, `resource_targets.rs` and `contract lab.rs` are in `crates/actingcommand-contract/src/`; `client.rs` and `error.rs` are in `crates/runtime-client/src/`; `host.rs` and `host/lease.rs` are in `crates/runtime-host/src/`; `resource-tooling …` is `crates/resource-tooling/src/`; `ledger global/projection.rs` is `crates/ledger/src/global/projection.rs`. `Runtime README.md` is the repository's root README; `distribution/windows/INSTALL.md` is written out; every other `*.md` (`resource-targets.md`, `scheduling/README.md`, …) is under `contracts/`. UI: `crates/acui-setup/src/`.

## 1. Scope

**Use this skill when** an ActingCommand install is on the machine and the user asks you to report health and what each instance is doing, run one task pack once, check a pack, find out why a scheduled task failed or was suspended, set resource targets for an instance, or (v0.10.0) record a task pack with Lab.

**Ask before you change anything.** Read-only commands need no request. Every other command needs the user's request for that action.

| Kind | Commands |
|---|---|
| Read only | `actingctl status`, `status --config`, `facts --program`, `monitor-status`, `emulator status`; every `actingledger` command; `actinglab --json` `help`, `capabilities`, `schema`, `package digest`, `package validate`, `scheduling compile`, `run summary`, plus `record status`, `package preflight`, `resource catalog` in v0.10.0; `actingd check-config`, `actingd suspended` (v0.10.0) |
| Uses the device or changes scheduling: only on the user's request | `actingctl task-run`, `reset`, `observe` and `stream` (they capture frames), `selfcheck`, `emulator start`, `stop`, `restart`, `monitor-set`, `monitor-clear`, `pause`, `resume`, `task-offset`, `agent-publish-facts`, `agent-apply-resource-targets`; a Lab recording (v0.10.0, §7): `actinglab record start`, `record mark`, `record stop`, and `capture`, `observe --capture`, `do --capture`, `session app` with `--record` |
| Never on your own | `actingctl request-shutdown`, `emulator discover`; starting `actingd`, `actingd unlock-owner`, `actingd ledger-maintenance`; `actinglab config set` |

**Never:**
- Approve anything. Approval decisions are accepted only from User/Ui (Runtime README.md:58). Do not touch `policy.catalog_approval_ids`.
- Edit `actingd.config.json`, start, stop or restart the daemon, or add or remove instances. Propose the change; the user or the UI console makes it.
- Use the device of an instance whose `lease_active` is `true`, or try to take over or release a lease you do not hold.
- Send personal data off the machine. Do not print or copy `secret_fingerprint_salt`. Treat user names in paths, account names, frames and screenshots as private.

## 2. Finding things

### 2.1 Install root

The setup wizard installs per user. The default root is `%LOCALAPPDATA%\Programs\ActingCommand` (UI platform.rs:31-34). The console's settings file `%APPDATA%\ActingCommand\acui.toml` holds the real paths in the keys `state_root`, `actingd_config` and `actingd_exe` (UI install.rs:221-260, platform.rs:45-48). Read it when the default folder does not exist.

| Path | Content | Source |
|---|---|---|
| `<root>\actingd.config.json` | Daemon configuration: `state_root`, `instances[]`, optional `policy` (scheduling catalog, approvals), the secret salt | UI install.rs:190; config.rs:60-118 |
| `<root>\runtime\` | `actingcommand-actingd.exe` (the daemon, "actingd"), `actingctl.exe`, `actingd.config.example.json`, `INSTALL.md`, `RELEASE-NOTES.md`, `BUILD-MANIFEST.json` | UI install.rs:30-43; `[run]` release zip |
| `<root>\tools\` | `actinglab.exe`, `actingledger.exe`, `ac_fastdeploy_ppocr.dll`, nothing else | UI verify.rs:32 |
| `<root>\ui\` | `acui.exe` (the console) and the other UI files | UI install.rs:30-56 |
| `<root>\packages\<game>\` | Resource packs from bundles: `<digest>\` content directories (bundle v2) or pack ZIPs (bundle v1) | UI bundle.rs:7-17 |
| `<root>\state\` | The state root the wizard creates; the config's `state_root` is the authority | UI main.rs:931 |

### 2.2 State root

Every client needs the **state root**, not its `ledger` subfolder (Runtime README.md:158). Read it from the configuration once and keep it in a variable:

```powershell
$Root    = Join-Path $env:LOCALAPPDATA 'Programs\ActingCommand'
$Config  = Join-Path $Root 'actingd.config.json'
$State   = (Get-Content $Config -Raw | ConvertFrom-Json).state_root   # read this key only; never print the file
$Ctl     = Join-Path $Root 'runtime\actingctl.exe'
$Actingd = Join-Path $Root 'runtime\actingcommand-actingd.exe'
$Lab     = Join-Path $Root 'tools\actinglab.exe'
$Ledger  = Join-Path $Root 'tools\actingledger.exe'
$env:ACTINGCOMMAND_RUNTIME_STATE_ROOT = $State   # actinglab reads this, not the config
```

```bash
ROOT="$(cygpath -u "$LOCALAPPDATA")/Programs/ActingCommand"
STATE='C:\...\state'   # state_root from the config; JSON "\\" is one "\"; single quotes keep it
CTL="$ROOT/runtime/actingctl.exe"; LAB="$ROOT/tools/actinglab.exe"; LEDGER="$ROOT/tools/actingledger.exe"
export ACTINGCOMMAND_RUNTIME_STATE_ROOT="$STATE"
```

How each program finds the state root:
- `actingctl` and `actingledger`: `--state-root` is required and has no default (actingctl main.rs:569; ledger-forensics lib.rs:177-186).
- `actinglab`: commands that talk to the Runtime read `ACTINGCOMMAND_RUNTIME_STATE_ROOT`, else `%LOCALAPPDATA%\ActingCommand\runtime` (state_roots.rs:11-24), which is not the installed state root. `run summary` and `runtime observe`/`reset` take `--state-root` instead (run_summary.rs:48-59; runtime_slice_cli.rs:17).

### 2.3 Instance identifiers

| Identifier | Form | Defined in | Use it for |
|---|---|---|---|
| alias | text the user chose | config `instances[].alias` | every `actingctl --instance`, `selfcheck <alias>`, a pause scope, the `instance` of a resource-target document; status `instance_alias`. The Runtime resolves requests by alias only (runtime-host host/lease.rs:2308-2331) |
| instance_id | `instance_` + 32 lowercase hex | config `instances[].instance_id`; the wizard makes a random one (UI instance_step.rs:242-250) | status `instance_id`; ledger event `links.instance_id`; `actingledger views --query '{"instance_id":"…"}'`; `actinglab lab watch --instance-id` (contract event/ids.rs:12-45, 122) |
| ADB port | integer | config `instances[].port`, or found by MuMu discovery when the daemon starts | status `adb_port`, absent while unbound or serial-configured (runtime.rs:1286-1289); `actingledger views --instance-port <port>`, which also covers every older `instance_id` once bound to that port (global-ledger-query.md:118-125, 180-184) |

### 2.4 The four programs

| Program | Output | Errors | Exit codes |
|---|---|---|---|
| `actingd` = `runtime\actingcommand-actingd.exe` | Daemon: `actingd ready pid=… host=… port=…`. `check-config --config <path>`: one JSON line with `status` `ok` or `failed`. v0.10.0 `suspended --config <path>`: one JSON object (§7.9) | stderr `FATAL actingd: <code>` | 0 ok, 1 any failure (actingd main.rs:49-57) `[run]` |
| `actingctl` | One line of the bare result JSON, no envelope | stderr `FATAL actingctl: …`, nothing on stdout | 0 ok; 1 on failure, and also when the printed JSON has a non-null `receipt.error` (main.rs:33-57) `[run]` |
| `actinglab` | Envelope `{"schema_version":"0.2","cli_version","runtime_version","ok","command","data"}`; on failure `"ok":false` and `error {code, message, blocked_by, details}`. JSON when stdout is not a terminal, or with `--json` (actinglab main.rs:214-224; cli_parse.rs:25) | the same envelope | 0 ok, 2 usage or validation, 3 safety blocked, 4 device or instance, 5 Runtime not running, 6 not implemented (cli_result.rs:86-96) `[run]` |
| `actingledger` | One line of JSON report; bare `export` prints text | stderr `actingledger: <code> during <operation>: <detail>` | 0 ok, 1 failure. An incomplete report is printed first, then exit 1 with a `*_incomplete` code or `runtime_facts_not_available` (ledger-forensics lib.rs:92-157, main.rs:3-8) `[run]` |

None of them has `--help` (§5).

## 3. Recipes

The recipes use the variables of §2.2. In PowerShell use 7.3 or later (`pwsh`); for Windows PowerShell 5.1 see §5 item 7.

### R1. Health, and what each instance is doing

```powershell
& $Ctl status --state-root $State            # daemon up? instances, leases, queues, pauses, ports
& $Ctl status --config --state-root $State   # what the daemon runs: subsystems and effective parameters
& $Ledger --state-root $State open           # is the ledger readable? latest_sequence
```

`status` (runtime.rs:1272-1301, 1446-1457) has `owner_epoch`, `instances[]` and, only while scheduling is paused globally, `scheduling_pause`. Each instance has `instance_alias`, `instance_id`, `lease_active`, `queued_request_count`, `takeover_cooldown_active`, `destructive_step_active`, `preempt_requested` and, when present, `adb_port`, `resource_package`, `game_id` and `pause {revision, reason_code, since_unix_ms, stage}`. In `status --config`, the subsystem `policy_driver` tells whether the scheduler runs (actingd-check-config.md:132-147).

`status` has no current task, run or last result. Read them from the ledger, here for one instance over the last hour:

```powershell
$Since = [DateTimeOffset]::UtcNow.AddHours(-1).ToUnixTimeMilliseconds()
$Q = @{ from_timestamp_unix_ms = $Since } | ConvertTo-Json -Compress
& $Ledger --state-root $State views --query $Q --instance-port <adb_port> --limit 256
```

```bash
SINCE=$(( ($(date +%s) - 3600) * 1000 ))
"$LEDGER" --state-root "$STATE" views --query '{"from_timestamp_unix_ms":'"$SINCE"'}' --instance-port <adb_port> --limit 256
```

A page is in ledger order and holds at most 256 events. When `has_more` is `true`, run it again with `--cursor '<next_cursor as compact JSON>'` and the same query (global-ledger-query.md:134-141, 165-197). The last `task.started`, `task.completed`, `task.failed` and `policy.dispatch_*` events (contract event.rs:261-331) show the current or last run; `links.run_id` names it.

When the daemon is down, `actingctl` exits 1 with `runtime client error runtime_info_unavailable during discover_runtime` (no `runtime-info.json`, normal after a clean stop) or `runtime_connect_failed` (a stale file) (runtime-client client.rs:5190-5224). Report it. `actingledger` still reads the ledger. Starting the daemon is the user's action, from the console or the Startup launcher.

### R2. Run one task pack once (only on the user's request)

1. Run `status`. The instance must be listed, with `adb_port` present, `lease_active` `false` and `queued_request_count` `0`. If a lease is active, wait or ask; what `task-run` does against a held lease (refuse or queue) is unverified. A pause does not block `task-run` (scheduling-pause.md:47-48), but a paused instance means the user stopped it on purpose: ask first.
2. Get the pack identity from R3: a ZIP and its SHA-256, or a content directory and its reference object.
3. Run it. **The command blocks until the run ends, up to 30 minutes, with no progress output** (main.rs:336-357; contract taskflow.rs:12). Give the shell call a timeout of at least 31 minutes, or start it in the background and watch the ledger as in R1.

```powershell
# content directory: pass the reference object as compact JSON
$Pack = '<root>\packages\<game>\<digest>'
$Ref  = (& $Lab --json package digest --package $Pack | ConvertFrom-Json).data.reference | ConvertTo-Json -Compress
& $Ctl task-run --state-root $State --instance <alias> --package $Pack --package-ref $Ref
# the same with a literal reference
& $Ctl task-run --state-root $State --instance <alias> --package $Pack --package-ref '{"schema_version":"actingcommand.package.content-directory.v1","sha256":"<64 hex>"}'
# ZIP pack
& $Ctl task-run --state-root $State --instance <alias> --package <pack.zip> --expected-sha256 <64 hex>
```

```bash
"$CTL" task-run --state-root "$STATE" --instance '<alias>' --package "$PACK" --package-ref '{"schema_version":"actingcommand.package.content-directory.v1","sha256":"<64 hex>"}'
```

`--package-ref` and `--expected-sha256` fill one slot: give exactly one (main.rs:547-552). A value that starts with `{` is read as a reference object, anything else as a ZIP SHA-256 (contract package.rs:168-178). An optional return-home pack is `--recovery-package <locator>` with `--recovery-package-ref <json>` or `--recovery-expected-sha256 <hex>`: both or neither (main.rs:305-315, 674-676).

Read the outcome:
- **Exit 0.** One JSON line `{"receipt":{…,"state":"completed","terminal":{"sequence","event_id"},"result":{"kind":"contained_task_completed","run_id","task_id","outcome":"success","final_page","executed_steps"}},"events":[…]}` (client.rs:217-224; runtime.rs:4243-4253, 4345-4364). Keep it: `run summary` does not cover manual runs (§5 item 2).
- **Exit 1.** Nothing on stdout. stderr is `FATAL actingctl: runtime client error <code> during run_contained_task with runtime code <Code> state=<state> fatal=<bool> host code <host_code> during <operation>` (runtime-client error.rs:228-268). `runtime_contained_task_response_timeout`: the deadline passed and the client ran its timeout recovery (client.rs:1398-1402); the result is uncertain, so look in the ledger before you do anything else. `runtime_contained_task_paused`: someone paused the instance. For any failure, find the run with `views --query '{"event_type":"task.failed","from_timestamp_unix_ms":<start>}' --instance-port <port>` and continue with R4 step 3.
- **`FATAL actingctl: usage: …` although the arguments look right.** The `--package-ref` JSON was damaged by shell quoting (§5 item 7). `failed to resolve contained task package` means the `--package` path does not exist (main.rs:304).

### R3. Check a pack (offline, no daemon needed)

```powershell
& $Lab --json package digest --package <pack directory>
& $Lab --json package validate --zip <pack.zip> --expected-sha256 <64 hex>
& $Actingd check-config --config $Config
```

- `package digest` reads the directory once, computes its content-directory reference and admits the directory in full against it (package-reference.md:126-137). `data` holds `reference` (the `--package-ref` object), `package_id`, `server`, `entry_task_id`, `file_count` and `byte_count` (resource-tooling api.rs:125-134). A directory named by a digest that differs from its content fails `content_directory_name_mismatch`. A refusal is `package_invalid`, exit 2, with the loader's code in the message.
- `package validate` runs the pack-containment metadata validation on a ZIP, against the hash when you give one, and answers `status: "valid"`, `input_sha256`, `hash_source`, `task_count` and more (package_cli.rs:25-39; resource-tooling package_validate.rs:14-60). Whether it accepts exactly the ZIPs that `task-run` accepts is unverified. The loader `task-run` uses also runs in `check-config` for each instance's `resource_package` (actingd-check-config.md:247-252).
- `check-config` has no side effects. It loads, assembles and validates the configuration exactly as startup does, admits each instance's `resource_package`, and never reads, creates or locks anything under the state root (actingd-check-config.md:3-13). With `"status":"ok"` it lists `instances[].resource_package`; a refused pack gives `error.code` `resource_package_invalid` or `resource_package_missing` with `error.detail.loader_code` (actingd-check-config.md:200-233). The salt is never printed. Run it from the folder the daemon starts in, because relative policy package paths resolve against the current directory (actingd-check-config.md:27-31). `[run]` on scratch copies of the shipped template: unfilled gives `{"status":"failed","error":{"code":"config_invalid","stage":"assemble"}}`, exit 1; filled gives `"status":"ok"` with `"not_checked":["vision_provider_manifest","state_root"]`, exit 0.
- An independent digest in Git Bash (package-reference.md:85-87):

```bash
cd <pack directory> && { printf 'actingcommand.package.content-directory.v1\n'; find . -type f -printf '%P\0' | LC_ALL=C sort -z | xargs -0 sha256sum -b | sed 's/ \*/  /'; } | sha256sum -b | cut -c1-64
```

In v0.10.0, `--package` of `package digest` and of `task-run` may also name a content container file, `<D>.zip` or `<D>.json`, with the same content-directory reference (package-reference.md "Containers"). Recording packs with Lab: §7.

### R4. Diagnose a failed scheduled task

1. `& $Ctl status --state-root $State`: is the instance or all scheduling paused, the instance held by a lease, or unbound (no `adb_port`)?
2. Recent problems on the instance. The `errors` view selects warning, error and fatal events (global-ledger-query.md:96-97).

```powershell
$Since = [DateTimeOffset]::UtcNow.AddHours(-24).ToUnixTimeMilliseconds()
$Q = @{ view = 'errors'; from_timestamp_unix_ms = $Since } | ConvertTo-Json -Compress
& $Ledger --state-root $State views --query $Q --instance-port <adb_port> --limit 256
```

```bash
SINCE=$(( ($(date +%s) - 86400) * 1000 ))
"$LEDGER" --state-root "$STATE" views --query '{"view":"errors","from_timestamp_unix_ms":'"$SINCE"'}' --instance-port <adb_port> --limit 256
```

3. Pick the failed run: a `task.failed`, `policy.dispatch_rejected` or `policy.dispatch_completed` event. Take `links.run_id` (`run_` + 32 hex). Then read every event of that run with full payloads:

```powershell
& $Ledger --state-root $State views --query '{"run_id":"run_<32 hex>"}' --profile forensic --limit 256
```

Profiles `cli` and `concise` omit payloads, `ui` (the default) and `normal` show the public part, `lab`, `verbose` and `forensic` the full payload (ledger global/projection.rs:316-337).

4. While the daemon runs, get the scheduler's summary of a dispatched run:

```powershell
& $Lab --json run summary run_<32 hex> --state-root $State
```

It covers only runs the scheduler dispatched (client.rs:4720-4732). `run_summary_not_found` is exit 2 (runtime_slice_cli.rs:56-63).

5. Step evidence (frames, recognition) for the run's sequence range. `--after` is exclusive, `--through` inclusive:

```powershell
& $Ledger --state-root $State export --task-evidence --after <first sequence - 1> --through <last sequence> --limit 1024
```

Exit 1 with `task_evidence_export_incomplete` after the report means the evidence has gaps; say so (ledger-forensics lib.rs:150-156).

6. Report the failing step, its failure code from the payload, and whether earlier runs failed the same way. Do not re-run, change packs or edit the configuration on your own; propose it to the user.

v0.9.1 has no "suspended task" state. In v0.10.0 a scheduled `linear_steps` task can retry immediately within its original activity window and budget cycle, and qualifying repeated failures pause it; `actingd suspended` lists what is paused, lifted or failing again and again (§7.9).

### R5. Resource targets, document v2 (Workflow #335; only on the user's request)

1. **Find what can take a target.** v0.9.1 has no command that lists it. Read the active scheduling catalog named by `policy.catalog.pools` and `policy.catalog.tasks` in `actingd.config.json`; relative paths resolve against the config file's folder (config.rs:588-592, 1065-1076). A pool qualifies when its `observation` is `{"kind":"fact","fact_key":…}` with a `fact_key` that starts with `resource.` or `inventory.`, and its `scope` covers the instance. A task produces it when one of its `produces[]` entries names that `pool_id` (resource-targets.md:54, 60-63). Without a `policy` section the Runtime refuses every document with `catalog_unavailable` (resource-targets.md:65-68). To check the catalog offline: `& $Lab --json scheduling compile --tasks <file> --pools <file> --activity <file> --timeline <file>` (scheduling/README.md:10-18).
2. **Write the document** (resource-targets.md:87-118). `instance` is the alias: the policy's instance id is the alias (runtime-host host.rs:322-331). `valid_until_unix_ms` must lie in `(now, now + 31_536_000_000]`, that is at most 365 days (resource-targets.md:51). Give 0 to 16 targets, at most one per resource. Leave out `scale` and `importance_milli` only when the pool declares a `valuation` with a `gap`. `tasks` is optional in `adjust` mode and required in `override` mode. The file is at most 65,536 bytes of UTF-8; write it without a byte-order mark.

```powershell
$Until = [DateTimeOffset]::UtcNow.AddDays(7).ToUnixTimeMilliseconds()
$Doc = [ordered]@{
  schema_version      = 'actingcommand.resource-targets.v2'
  instance            = '<alias>'
  valid_until_unix_ms = $Until
  targets = @([ordered]@{
    id = 'floor-1'; resource = '<pool id>'
    condition = [ordered]@{ kind = 'at_least'; amount = 1000 }
    scale = 1000; importance_milli = 1000
    apply = [ordered]@{ mode = 'adjust'; weight = 'score_stage' }
  })
} | ConvertTo-Json -Depth 6 -Compress
$File = Join-Path $env:TEMP 'actingcommand-resource-targets.json'
[IO.File]::WriteAllText($File, $Doc, [Text.UTF8Encoding]::new($false))
& $Ctl agent-apply-resource-targets --state-root $State --policy-file $File
```

```bash
UNTIL=$(( ($(date +%s) + 7*86400) * 1000 ))
F=/tmp/actingcommand-resource-targets.json
printf '%s' '{"schema_version":"actingcommand.resource-targets.v2","instance":"<alias>","valid_until_unix_ms":'"$UNTIL"',"targets":[{"id":"floor-1","resource":"<pool id>","condition":{"kind":"at_least","amount":1000},"scale":1000,"importance_milli":1000,"apply":{"mode":"adjust","weight":"score_stage"}}]}' > "$F"
"$CTL" agent-apply-resource-targets --state-root "$STATE" --policy-file "$(cygpath -w "$F")"
```

To withdraw every target, send `{"schema_version":"actingcommand.resource-targets.v2","instance":"<alias>","targets":[]}` with no `valid_until_unix_ms` (resource-targets.md:51-52).

3. **Read the result.**
- Exit 0: `{"applied":{…}}` with `instance_alias`, `policy_sha256`, `version` (the ledger sequence of the stored policy), `event_id`, `previous_version`, `replayed`, `checked_catalog_hash`, `valid_until_unix_ms`, `conditions_at_position` and, per target, `conditions[] {target_id, resource, fact_key, state}`. `state` is `{"kind":"computed","current","observed_at_unix_ms","gap"}` (gap 0 means satisfied) or `{"kind":"awaiting_observation","reason"}`; waiting for an observation is not an error (contract resource_targets.rs:20-39; resource-targets.md:167-189).
- Exit 1 with `{"receipt":{…,"state":"denied","resource_targets_rejection":{"field_path","line","column","reason"}}}`: nothing changed; fix that field (resource-targets.md:153-160). `FATAL actingctl: resource_targets_file_invalid: …`: the file is unreadable, empty, over 65,536 bytes or not UTF-8 (main.rs:87-102). `fact_observation_not_newer`: two submissions in one millisecond; send it once more (resource-targets.md:164-165).
- Later: v0.9.1 has no read-back command. The stored policy is the `fact.published` event at sequence `version`: `views --query '{"from_sequence":<version>,"to_sequence":<version>}' --profile forensic`. Its effect shows as the reasons `resource_targets` and `resource_weights` on the scheduler's dispatch decisions (resource-targets.md:457-475). Sending the identical, unexpired document again appends nothing and answers `replayed: true` with fresh `conditions` (resource-targets.md:257-262); after expiry the same document is refused with `validity_out_of_range`.

## 4. Failure handling

| What you see | Class | Do |
|---|---|---|
| `FATAL actingctl: usage: …`; actinglab exit 2; `actingledger: invalid_arguments …` | usage | Fix the arguments. Verb first, flags after; always `--state-root`; `--instance` only where the verb needs it (main.rs:688-694, 716-729) |
| actinglab exit 3 | safety | Stop and report to the user |
| `runtime code InstanceUnknown`, `instance_unknown` | usage | Wrong alias; read `status` again |
| `host code instance_not_running during require_bound_adb_endpoint` | device | The emulator instance is stopped (distribution/windows/INSTALL.md:101-108); ask the user before `emulator start` |
| `runtime code LeaseBusy`, or `lease_active: true` | device | Someone else holds the instance; wait or ask; never take it over |
| actinglab exit 4 `device_error` | device or Runtime | Read `error.message` first; it often holds a Runtime code (§5 item 2) |
| `runtime_info_unavailable`, `runtime_connect_failed`; actinglab exit 5 | runtime | The daemon is not reachable; report it; do not start it yourself |
| `runtime_contained_task_response_timeout`; any changing command without a clear answer | uncertain | Check the ledger first; never resend a changing command blindly |
| actingledger `*_incomplete` or `runtime_facts_not_available` after a printed report | evidence gap | Report the gap; the report is partial |

## 5. Known pitfalls

1. **actinglab ignores unknown flags and keeps the last of repeated flags** (flag_args.rs:22-66). `[run]` `actinglab --json capabilities --no-such-flag 1` exits 0 with output identical to plain `capabilities`. A misspelt flag is dropped without a word. `lab watch`, `run summary` and `runtime observe`/`reset` do reject unknown flags (runtime_debug.rs:310-337; run_summary.rs:49-58; runtime_slice_cli.rs:81-93).
2. **actinglab reports many Runtime failures as `device_error`, exit 4**, with the real code only in `error.message` (runtime_slice_cli.rs:64-78; runtime_debug.rs:101-106, 278-286). Client-side failures without a Runtime code become `runtime_not_running`, exit 5, even while the daemon runs; for example `run summary` on a manual run fails with `run_summary_event_count_invalid` (client.rs:5138-5152). The envelope has no class field (contract lab.rs:250-259): the exit code is the class.
3. **A pause lives in memory only.** A daemon restart clears every pause, global and per instance (scheduling-pause.md:191-197; runtime.rs:1297-1299, 1453-1456). Check `status` again after a restart. A pause gates scheduler dispatch only, not `task-run` (scheduling-pause.md:47-48).
4. **`task-run` blocks up to 30 minutes** without progress (main.rs:354; contract taskflow.rs:12).
5. **`actinglab schema observe` omits `--package`.** `[run]` its `parameters` list only `--scene <png> or --capture` plus options (lab2_cli.rs:1340-1353), but `observe` reads `--package <dir> --package-ref <json>` or `--zip` (lab2_cli.rs:83, 129; contained_resources.rs:188-199; package-reference.md:186-188).
6. **No `--help`.** `[run]` `actingctl --help` prints the usage line to stderr, exit 1 (main.rs:791-792); `actingledger --help` prints `actingledger: invalid_arguments during parse_arguments: expected replay or --state-root`, exit 1; `actingd --help` prints `FATAL actingd: usage_invalid`, exit 1. Use the usage line, `actinglab --json help`, `capabilities` and `schema`, and this manual. The actingctl usage line omits `agent-publish-facts` and `agent-apply-resource-targets` (main.rs:572-603), and `facts` needs `--program` (main.rs:627-633).
7. **Shell quoting of JSON arguments** (`--package-ref`, `--query`, `--cursor`, `--request`). `[run]` with an argument echo: PowerShell 7.3+ passes `'{"a":"b"}'` intact; Windows PowerShell 5.1 strips the inner quotes and needs `'{\"a\":\"b\"}'`; that 5.1 form sent from PowerShell 7 arrives with literal backslashes. The two forms are not interchangeable. In PowerShell 7, build the value with `ConvertTo-Json -Compress`. In bash, use single quotes.
8. **actinglab's default state root is not the installed one** (§2.2). Set `ACTINGCOMMAND_RUNTIME_STATE_ROOT`.
9. **actingledger argument order.** `--state-root <root>` must be the first two arguments (ledger-forensics lib.rs:164-186).

## 6. What changes in v0.10.0 and v0.10.1

None of these commands exist in v0.9.1. Do not call them until the installed version has them; `actinglab --json capabilities` lists the installed commands.

- **v0.10.0**: pack making with Lab, described in §7. The agent marks regions of real frames as recognition points or click points by recording through `actinglab` (CLI first) and gets a draft of the configuration binding, which the user applies and approves. Linear tasks consume declared target consensus (§7.6). Scheduled `linear_steps` tasks can retry immediately within the original activity window and budget cycle, and qualifying repeated failures suspend them (§7.9); `actingd suspended --config` lists them. The older `record step`, `candidates`, `amend`, `build-task` and `promote` actions keep their behaviour and are not covered here.
- **v0.10.1** (Workflow #338, not released when this was written): `actingctl mcp-serve`, a local stdio MCP server with task-level tools (`ac_*`, `lab_*`) in three tiers: `observer` (read only, the default), `operator` (device and scheduling writes) and `author` (Lab recording). The user enables the higher tiers in the harness configuration. This skill then moves to the MCP tools, with a tool table generated from `actingctl mcp-serve --list-tools`; this CLI manual stays as the fallback.

## 7. Lab recording (Runtime v0.10.0)

A Lab recording turns real screens into a `linear_steps` task pack. Each step is one screen with its recognition marks and one effect: a click rectangle or an application operation. The last step only recognizes. `record stop` checks the pack the way actingd will load it, writes it as `<D>.zip` (when it has template images) or `<D>.json`, where D is its content digest, and prints binding snippets (lab-recording.md; linear-steps.md). You record and write the pack. Binding it in `actingd.config.json`, the catalog task and its approval stay with the user (§1). A recording needs the user's request and the v0.10.0 `actinglab`; `record status` and `actingd suspended` are read only. Commands use the variables of §2.2 plus `$I = '<alias>'`.

### 7.1 Before and after a recording

- **Pause the instance first, resume it afterwards.** Run `& $Ctl pause --state-root $State --instance $I` before `record start`, and `& $Ctl resume --state-root $State --instance $I` after the final `record stop`, or when you give the recording up. Otherwise a scheduled task between two of your commands acts on another screen (lab-recording.md "`--record` on device commands"). A pause lives in memory only: after a daemon restart, check `status` and pause again (§5 item 3).
- **Device commands need the daemon and a physical instance.** `capture`, `observe`, `do` and `session app` with `--record` go through the Runtime; `record start`, `mark`, `status` and `stop` work offline. A fixture-simulated instance refuses Lab capture, clicks and application operations; for `session app` the refusal is `fixture_execution_scope_forbidden`.
- **One process per instance.** Every recording command that writes state, `--dry-run` included, takes the lock `<state>\record-<instance>.lock` without waiting. Another holder gives `record_busy` (exit 3, `details.holder`). Do not retry blindly: run `record status`, see which step is open, then decide. A `record mark` without `--step` may otherwise land on a step the other process just opened. `record status` takes no lock. `record_lock_failed` is exit 5. There is no unlock command; the lock ends with its process (lab-recording.md "Recording lock").
- **Do not mix actinglab versions during a recording.** The v0.9.0 actinglab runs `session app … --record` without recording it and does not know the lock (lab-recording.md "`session app … --record`").
- **One state directory.** `--record` commands use `ACTINGLAB_SESSION_STATE_DIR` or the default and refuse `--state-dir` (`record_state_dir_unsupported`). Give no `--state-dir` to any recording command. When `record start` prints `record_flag_reachable: false`, device frames cannot reach the recording; only `record mark --frame` can add frames.
- **Global `--instance $I` on every command.** A recording belongs to one instance (`record_instance_mismatch`).
- **Carrier package.** `observe --capture --record` and `do --capture --record` need one. Any admissible pack of the same game and resolution will do, an earlier Lab pack included. Give it as `--package <dir | D.zip | D.json> --package-ref '<reference>'` (reference from R3). `capture --record` and `session app … --record` need none.
- **Unknown flags.** `record mark` and `record stop` refuse unknown flags and positional arguments (`validation_failed`, exit 2), unlike most of actinglab (§5 item 1).

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

- **Marks.** `--template <id>=x,y,w,h` crops the primary frame. `--color <id>=x,y,w,h` takes the region's mean colour. The colour digest, OCR and check families exist only in the request form, `record mark --request-json '<actingcommand.lab-record-mark.v1 JSON>'` or `--request <file>`. Every mark is tested on every live frame of its step; one failure refuses the whole command, which writes nothing (`record_mark_rejected`, exit 3, `details.marks[]`). `record mark --dry-run` tests without writing. `--sample <png>` adds another frame of the same screen to test on. Lab has no OCR provider, so an OCR mark is always `not_evaluated`, never passed (lab-recording.md "Marks and the mark-time self-test").
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
| `task_run_example` | one manual run (R2), only on the user's request |
| `binding_example` | a `policy.procedure_manifest[]` entry with the absolute `package_path`. It runs on a schedule only with a catalog task of that `procedure_ref` and its approval |
| `prerequisite_entry_example` | the `prerequisite_packages` entry for this pack, when another pack names it with `--requires` (§7.6) |
| `catalog_on_failure_example` | the catalog task's `on_failure`, `{"action":"pause","retry_limit":1,"retry_backoff_ms":0,"escalation_threshold":2}`: a qualifying failure of the pack's own steps is rerun immediately while its original activity window and budget cycle remain valid (R22); the same problem again suspends the task (§7.9; policy-suspension.md "Catalog") |
| `binding_requires` | what the binding also needs, as a checklist |

The user edits the configuration and the catalog. A changed `on_failure` is a catalog change and needs a new approval, which only the user can give. Then `actingd check-config` (read only, R3) and a daemon restart by the user; the configuration is read at startup.

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
- **`& $Actingd suspended --config $Config`** is read-only and runs beside the daemon. Run it from the folder the daemon starts in, as `check-config` (R3). It prints one JSON object with three lists:
  - `suspended[]`: paused pairs, with `step`, `detail`, the error `frames` and their `comparison`, `lifts_when`, and `takeover`, a `task-run` command line for a manual run.
  - `lifted[]`: pairs a package update lifted. `effective` is `pending_restart` until the daemon has read the new configuration.
  - `repeating[]`: pairs failing again and again without being suspended.

  `warnings` include `config_newer_than_daemon_start`. Exit 0 whatever it lists. Exit 1 with `FATAL actingd: <code>` on any error, for example `suspended_policy_unconfigured` without a `policy` section.
- **Lifting, never by hand.** A suspension is lifted in three steps: (1) the pack is updated, for example recorded again, which gives a new D, or the prerequisite or return-home pack is fixed; (2) the user points the binding at it: the procedure binding's `package_digest` and `package_path`, the `prerequisite_packages` entry, or the `return_home_packages` entry; (3) the user restarts actingd. The pause is then lifted automatically. `actingctl task-run` is not gated by the policy and lifts nothing.
- **Known limits.** A dialog that does not darken and covers a medium part of the screen counts as "similar" and can pause the task; look at the frames. The same cause can differ on days with and without an optional pop-up, which costs one more rerun.
