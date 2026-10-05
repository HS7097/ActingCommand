# ActingCommand CLI workshop (fallback)

This is the command-line fallback of the `actingcommand` skill. Use it when the harness has no ActingCommand MCP server (`SKILL.md`, Connecting), or for a command that no MCP tool covers. It is §1–§6 of the CLI manual v0, which was the skill's `SKILL.md` before the MCP edition; its §7, Lab recording, is now `lab-recording.md`. Changed from v0: this paragraph, the first sentence below, the references to §7, and the heading and MCP note of §6.

It is a command-line manual for an AI agent that operates an installed ActingCommand. §1–§5 cover Runtime **v0.9.1** and UI **v0.9.0**; they were not checked again for v0.10.0 except the explicitly marked additions. `lab-recording.md` covers what Runtime **v0.10.0** adds: Lab recording and its self-check coverage, `linear_steps` packages with declared target consensus, and scheduled tasks' bounded immediate retries and suspension. It holds no game knowledge; that lives in the resource packs.

**Evidence key.** `[run]`: checked by running the v0.9.1 release binaries with side-effect-free arguments only. `path:line`: checked in source, in HS7097/ActingCommand-Runtime at v0.9.1 (`8e0ac191`), or in HS7097/ActingCommand-UI v0.9.0 (`3056039`) when marked `UI`. `file.md "Section"` (in `lab-recording.md`): checked in that contract and the [v0.10.0 Runtime candidate at `0d3e686e`](https://github.com/HS7097/ActingCommand-Runtime/tree/0d3e686e3ad6026c8e667597d884abb52b3e1009) (PR #608). Nothing in `lab-recording.md` is `[run]`; source checks do not establish a released build or real-device behavior. **Unverified**: not checked; check it before you rely on it.

**Citation paths.** Runtime: an unqualified `main.rs` is `apps/actingctl/src/main.rs`; `actingd main.rs`, `config.rs` and `check_config.rs` are in `apps/actingd/src/`; `actinglab main.rs` and the other actinglab files (`cli_parse.rs`, `flag_args.rs`, `lab2_cli.rs`, `run_summary.rs`, …) are in `apps/actinglab/src/`; `ledger-forensics lib.rs`/`main.rs` are in `apps/ledger-forensics/src/`; `runtime.rs`, `package.rs`, `taskflow.rs`, `event.rs`, `event/ids.rs`, `resource_targets.rs` and `contract lab.rs` are in `crates/actingcommand-contract/src/`; `client.rs` and `error.rs` are in `crates/runtime-client/src/`; `host.rs` and `host/lease.rs` are in `crates/runtime-host/src/`; `resource-tooling …` is `crates/resource-tooling/src/`; `ledger global/projection.rs` is `crates/ledger/src/global/projection.rs`. `Runtime README.md` is the repository's root README; `distribution/windows/INSTALL.md` is written out; every other `*.md` (`resource-targets.md`, `scheduling/README.md`, …) is under `contracts/`. UI: `crates/acui-setup/src/`.

## 1. Scope

**Use this skill when** an ActingCommand install is on the machine and the user asks you to report health and what each instance is doing, run one task pack once, check a pack, find out why a scheduled task failed or was suspended, set resource targets for an instance, or (v0.10.0) record a task pack with Lab.

**Ask before you change anything.** Read-only commands need no request. Every other command needs the user's request for that action.

| Kind | Commands |
|---|---|
| Read only | `actingctl status`, `status --config`, `facts --program`, `monitor-status`, `emulator status`; every `actingledger` command; `actinglab --json` `help`, `capabilities`, `schema`, `package digest`, `package validate`, `scheduling compile`, `run summary`, plus `record status`, `package preflight`, `resource catalog` in v0.10.0; `actingd check-config`, `actingd suspended` (v0.10.0) |
| Uses the device or changes scheduling: only on the user's request | `actingctl task-run`, `reset`, `observe` and `stream` (they capture frames), `selfcheck`, `emulator start`, `stop`, `restart`, `monitor-set`, `monitor-clear`, `pause`, `resume`, `task-offset`, `agent-publish-facts`, `agent-apply-resource-targets`; a Lab recording (v0.10.0, `lab-recording.md`): `actinglab record start`, `record mark`, `record stop`, and `capture`, `observe --capture`, `do --capture`, `session app` with `--record` |
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
| `actingd` = `runtime\actingcommand-actingd.exe` | Daemon: `actingd ready pid=… host=… port=…`. `check-config --config <path>`: one JSON line with `status` `ok` or `failed`. v0.10.0 `suspended --config <path>`: one JSON object (`lab-recording.md` §7.9) | stderr `FATAL actingd: <code>` | 0 ok, 1 any failure (actingd main.rs:49-57) `[run]` |
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

In v0.10.0, `--package` of `package digest` and of `task-run` may also name a content container file, `<D>.zip` or `<D>.json`, with the same content-directory reference (package-reference.md "Containers"). Recording packs with Lab: `lab-recording.md`.

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

v0.9.1 has no "suspended task" state. In v0.10.0 a scheduled `linear_steps` task can retry immediately within its original activity window and budget cycle, and qualifying repeated failures pause it; `actingd suspended` lists what is paused, lifted or failing again and again (`lab-recording.md` §7.9).

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

## 6. What changes in v0.10.0 and v0.11.0

None of these commands exist in v0.9.1. Do not call them until the installed version has them; `actinglab --json capabilities` lists the installed commands.

- **v0.10.0**: pack making with Lab, described in `lab-recording.md`. The agent marks regions of real frames as recognition points or click points by recording through `actinglab` (CLI first) and gets a draft of the configuration binding, which the user applies and approves. Linear tasks consume declared target consensus (`lab-recording.md` §7.6). Scheduled `linear_steps` tasks can retry immediately within the original activity window and budget cycle, and qualifying repeated failures suspend them (`lab-recording.md` §7.9); `actingd suspended --config` lists them. The older `record step`, `candidates`, `amend`, `build-task` and `promote` actions keep their behaviour and are not covered here.
- **v0.11.0** (Workflow #338, not released when this was written): `actingctl mcp-serve`, a local stdio MCP server with 22 task-level tools, all named `ac_*` (the Lab ones included), in three tiers: `observer` (read only, the default), `operator` (device and scheduling writes) and `author` (Lab recording). The user enables the higher tiers in the harness configuration. The skill's `SKILL.md` now works with these tools, and `tools.md` holds the tool table generated from `actingctl mcp-serve --list-tools`; this CLI manual stays as the fallback.
