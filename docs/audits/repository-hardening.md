# PressWire repository hardening audit

Date: 2026-09-22. Repository: `kujolang/presswire`. Branch: `main`.
Starting SHA: `566469012efbf10e86d48f61fdd64cdd050dddd6` (clean worktree).
Ending implementation SHA: `321153c7bba6f5c78f35e49af073441429a046b3`. The documentation-only audit receipt commit follows this implementation revision; use `git log -1 --format=%H -- docs/audits/repository-hardening.md` to identify it.

## Repository and scope

PressWire is an offline Kujo CLI for approval-gated local publication and immutable request/receipt records. Production paths are `presswire.kujo` → `src/core.kujo`, with argument parsing, common validation, storage, and separately exported hardening helpers. The shell launcher only discovers and executes Kujo. No production shell/network calls, model/provider SDKs, prompts, database, or package dependencies exist. VersionSeal-style local approval records are an integration contract; external CMS/Git/newsletter adapters are conformance fixtures, not connected live effects. Kujo supplies JSON, crypto, filesystem and subprocess primitives.

Reviewed all five source modules, entrypoint/launcher, every original test and fixture, schemas, documentation, package metadata, release/version files, ignore rules and CI. Independent security and architecture reviews supplemented implementation inspection. Read relevant Kujo filesystem primitives without changing sibling source: recursive `create_dir` is not exclusive; atomic writes with overwrite=false use exclusive temporary creation and hard-link publication; rename overwrites. Local state and ancestors are operator-controlled under SECURITY.md, not tenant isolation boundaries.

## Baseline

`/usr/bin/time -p bash scripts/validate.sh` passed: entrypoint check, 44 assertions in five suites, seven JSON documents, CLI smoke checks, foreign-runtime boundary, repository hygiene and diff checks. Baseline elapsed/user/system: 1.52/0.74/0.47 seconds on the available Kujo 1.4.0 runtime. There were no baseline suite failures. Additional probes exposed missing coverage rather than existing failing tests.

A repeatable five-publication × 4 MiB fixture and a 100-record × 4 KiB payload export fixture were run against the starting source and updated implementation. The source snapshot is preserved locally in ignored `.audit-artifacts/baseline-repo`; concise evidence is committed here. Full logs, large exports and the pinned-runtime build remain in `.audit-artifacts/`.

## Findings

| ID | Priority | Area | Finding and evidence | Action | Status |
|---|---|---|---|---|---|
| PW-01 | P0 | Integrity | `publish_local` hashed the source twice, then reopened it for copy; the checked bytes could differ from delivery. | One checked binary snapshot passed to atomic write. | Fixed |
| PW-02 | P0 | Concurrency | Directory creation was recursive/idempotent; the record lock was also acquired after the effect. | Atomic lock files; a separate operation lock covers identity check through receipt persistence. | Fixed |
| PW-03 | P0 | Overwrite | Existence check followed by rename could overwrite a competing publication without force. | Atomic no-overwrite write; record and event writes also use overwrite=false. | Fixed |
| PW-04 | P1 | Failures | Record expansion and existing history conflicts could reject a request after delivery. | Pre-effect record-size/history checks; `publication_unrecorded` exposes later persistence failures and the delivered record. | Fixed within documented crash boundary |
| PW-05 | P1 | Approval | Destination label comparison did not bind actual `--output`. | Optional approval `payload.output_path` exact-string enforcement; preserve legacy immutable evidence. | Mitigated; migration remains |
| PW-06 | P1 | CLI | Approval parsing reused `parsed`, losing explicit publication ID/timestamp in observed execution. | Use a distinct approval result binding; regress both fields. | Fixed |
| PW-07 | P1 | Queries | README flags and config limit were rejected; warning accumulation was unbounded; prefix IDs could be skipped by filename ordering. | Expose options; 1000-candidate and 8 MiB compact-data page ceilings; bare-ID sorting and resumable opaque cursors. | Fixed |
| PW-08 | P1 | Export | `export --output` ignored the destination and emitted full data. | Atomic complete-selection export plus concise receipt; reject partial/corrupt file export. | Fixed |
| PW-09 | P2 | Validation | Huge integer conversion, impossible calendar dates and malformed helper types lacked guards; artifact verification omitted its size cap. | Validate before conversion/operations; apply existing artifact cap to validation. | Fixed |
| PW-10 | P2 | Developer UX | Launcher default was a developer absolute path; version alias discarded JSON flags; JSON gate started seven runtimes; tests leaked temporary state. | Portable runtime lookup, preserve alias flags, one recursive JSON batch, isolated fixture root and cleanup trap. | Fixed |
| PW-11 | P2 | Contracts | Record schema excluded supported 0.1.0 records; quickstart validate lacked ID; helper/CLI capability distinctions were unclear. | Widen schema to supported versions; correct examples and authority/recovery documentation. | Fixed |
| PW-12 | P2 | Supply chain | CI did not explicitly require the committed lockfile. | Use `cargo build --locked --no-default-features`; retain pinned runtime/actions and read-only permissions. The entire gate verifies the minimal runtime. | Fixed |
| PW-13 | P2 | Scale | `list_dir` still materializes and sorts all directory names. | Bound per-page loading/output; document total-name memory cost. | Needs workload evidence/runtime iterator |
| PW-15 | P1 | Byte accounting | String length counts Unicode characters; oversized UTF-8 records could be saved and then rejected by the byte-bounded reader. | Exact UTF-8 byte accounting for record and query limits; duplicated-Unicode-provenance regression proves no unreadable record is saved. | Fixed |
| PW-14 | P1 | Recovery | Publication, record and event cannot commit atomically across process crashes/filesystems. | Clear partial-effect errors, fail-closed stale locks and recovery instructions; retain existing reconciliation follow-up. | Architectural remaining work |

## Changes and proof

- **Publication/storage:** `src/core.kujo`, `src/storage.kujo` use snapshot bytes, explicit no-overwrite writes, early operation coordination and pre-delivery rejection. `tests/audit_test.kujo` proves held locks prevent effects, deterministic history rejection precedes delivery, valid-size input whose envelope exceeds 1 MiB is rejected, explicit IDs/timestamps survive approval parsing, path-bound evidence rejects another target, dry-run creates no destination parents, and duplicate delivery is rejected. `tests/concurrency_test.kujo`/`lock_worker.kujo` launch 16 competitors, require exactly one lock owner and verify reacquisition. Worker failures propagate through the launcher. Locks use a hidden operation filename, avoiding collisions with valid record IDs.
- **Queries/export:** same core/storage modules add bounded filters/cursors, limit configuration, complete-file export and small receipts. Tests cover prefix IDs, cursor continuation across corrupt records, limits, malformed configuration, unsafe/huge numeric input, no overwrite, partial export rejection and record preservation. Existing stdout export remains available. The aggregate ceiling is for compact serialized record data, not total process RSS or directory-name allocation.
- **Validation:** `src/common.kujo`, `src/args.kujo`, `src/hardening.kujo` reject calendar impossibilities, missing flag values, oversized numeric strings and wrong helper types. Existing valid hardening fixtures remain green. Runtime errors now retain their underlying diagnostic inside a structured operational error. Exact UTF-8 sizes are recovered from ASCII Base64 length because the compatible runtimes do not expose a direct string-byte-length primitive; tests cover multibyte text and oversized Unicode provenance. This deliberately avoids treating character counts as byte budgets. Removed unused storage parent-path helper and import; no generic rewrite or style-only reformatting.
- **Gate/ergonomics:** launcher, validation script, `tests/support.kujo`, CLI tests, CI, schemas, README, AGENTS and contracts were updated together. Temporary state created by the gate is removed on success or failure. Test-only shell usage coordinates subprocesses; production behavior remains Kujo. JSON validation still discovers nested fixture/schema files but starts one interpreter for the entire list.

## Performance and efficiency

| Measurement | Before | After | Interpretation |
|---|---:|---:|---|
| Export result bytes, 100 records with 4 KiB payload each | 434,800 | 142 | Full export is retained in a file; record arrays compared equal. Compact result bytes, not token counts. |
| JSON-validation runtime launches, seven documents | 7 | 1 | Six launches removed; discovery remains recursive. |
| Per-page corrupt-record warnings | Unbounded | ≤1000 | Cursor continuation returns remaining warnings without loss. |
| Retained compact record data per query page | Up to 1000 × 1 MiB files, no aggregate byte cap | ≤8 MiB | Total memory also includes parsed data, warnings and all directory names. |
| Publication source reads | Two hashes plus copy | One snapshot read | Code-supported operation count; not a syscall trace. |
| Publication benchmark RSS, five × 4 MiB | 27,803,648 bytes | 40,321,024 bytes | Snapshot safety has an explicit memory cost. |
| Publication benchmark elapsed/user/system | 1.20/0.71/0.13 s | 1.78/0.86/0.24 s | Single samples; after run overlapped runtime compilation/system load. No speedup claim or latency conclusion. |
| Pinned runtime dependency graph packages on this host, including root/build/dev | 408 | 274 | Unused database/image/archive/JIT feature trees removed from CI; not a production package count. |
| Production package dependencies | 0 beyond Kujo | 0 beyond Kujo | No new package/runtime dependency. |

No model prompts, tool registries, MCP schema payloads or model calls exist here; token-specific budgets would not be meaningful. Pagination and file receipts reduce agent-visible context without removing evidence. There is no PressWire binary/build step separate from Kujo checking; whole-gate elapsed time is not comparable because the expanded suite does substantially more work. Benchmark wall time is not used as a flaky CI gate.

## Security and compatibility

Reviewed CLI/config/imported JSON, approval/actor scope, artifact bytes, outputs, state/event/lock paths, symlinks, failure cleanup, concurrency, helper schemas, subprocess boundaries and CI provenance. The initial source-grounded security scan is locally finalized at `artifacts/security/repository-hardening/report.md`; it retains one low-severity legacy approval-path finding. The final parent review additionally found and fixed Unicode byte accounting, with regression proof on both supported runtime samples. No remote privilege boundary is claimed. Forced writes, explicit record IDs and operator-controlled ancestors retain their documented authority model. Partial effects after crashes are not claimed solved.

Public command names, record payloads and contract/tool versions remain unchanged. Existing 0.1.0 records and old opaque-label approvals remain readable/usable. Additions: query flags, config limit, pagination fields, file export receipt, optional approval output_path, structured failure codes and doctor scan-completeness fields. The record schema now accepts both supported tool versions. Invalid dates/helper inputs/missing flag values now fail intentionally. `--version --json` now produces JSON as requested. Export with --output now performs the documented write; stdout-only behavior is preserved without that flag.

The internal lock format changes from directories to exclusive files. Stop old writers before upgrading; existing stale lock directories still fail closed. New test-only PRESSWIRE_TEST_ROOT does not affect product configuration. KUJO_BIN is retained; PATH and sibling fallback replace the hardcoded launcher default. Filesystem ancestors are trusted; no adversarial symlink-race sandbox is promised. New signed approvals should carry output_path, but mandatory adoption requires a coordinated producer migration.

## Cross-repository follow-ups and remaining work

| Class | Repository/contract | Evidence and impact | Next action / compatibility |
|---|---|---|---|
| P1 | PressWire + VersionSeal/workflow approval producers | Old destination labels authorize no concrete path; new output_path is enforced when present. | Stage producer migration before requiring path binding. Current fixes do not require sibling changes; preserve old records. |
| P1 | PressWire recovery | Existing SignalBox reconciliation item covers partial publication/receipt commits. | Design staged receipts and crash reconciliation; never expire locks or retry blindly. |
| P2 / needs evidence | Kujo directory iteration and bounded streaming publication | Legacy compatible builtins require full name listing and snapshot buffering for exact-byte atomic publication. | Investigate an iterator and verified staged-file publication contract for a future runtime upgrade; not required by this patch. |
| Existing duplicate | Kujo nested dictionary assignment | Audit fixture reproduced silently unchanged nested assignment on 1.4.0; existing Signal `sig_ff7d3cd8-d044-42cf-b9d8-fbfd8f103f7d` already tracks it. | Current tests use explicit child replacement; no sibling changes or duplicate Capture. |
| P3 / not worth changing | Dense helper formatting, schema/version constants | No measured runtime problem or contract bug from layout alone. | Preserve rather than reformat or redesign. |

No P0 remains from this audit. Hosted adapters/signature policy are explicitly separate capabilities, not undisclosed blockers to local operation. Linux CI execution and adversarial filesystem mutation are not inferred from local macOS tests.

## Verification receipt

Kujo 1.4.0 and minimum supported Kujo 1.0.1 both pass all **99 assertions** across eight suites, including 16 competing lock processes. Both pass the complete gate. The 1.4.0 expanded gate took 17.59 s elapsed (9.79 s user / 4.79 s system) under concurrent host build load. Shell syntax and diff checks pass. The CI-pinned Kujo 1.0.2 minimal-feature release build passed (24m 18s under host contention and with warmed dependencies). Upstream unused-code warnings remain in the build log. Its complete gate also passed all 99 assertions. The initial default-feature build was deliberately stopped when the unnecessary feature tree was identified; no default-versus-minimal elapsed-time improvement is claimed.

Commands include:

```sh
/usr/bin/time -p bash scripts/validate.sh
KUJO_BIN=/Users/robertdevore/2026/prompts.robertdevore.com/tmp/kujo-1.0.1/kujo bash scripts/validate.sh
cargo build --release --locked --no-default-features --manifest-path .audit-artifacts/pinned-runtime/Cargo.toml
KUJO_BIN="$PWD/.audit-artifacts/pinned-runtime/target/release/kujo" bash scripts/validate.sh
/usr/bin/time -l ../kujo/target/release/kujo run tests/benchmarks/publication.kujo
../kujo/target/release/kujo run tests/benchmarks/export.kujo
bash -n scripts/validate.sh
sh -n bin/presswire
git diff --check
```

The gate expands to `kujo check presswire.kujo`, all eight `tests/*test.kujo` suites, one `scripts/validate_json.kujo` invocation over discovered JSON files, launcher smoke checks and repository hygiene. The baseline source was archived from its exact SHA for comparative benchmark runs. The final oversized-record regression checks that the destination remains absent. The complete final suites include this test and the Unicode byte-accounting regression on all three runtimes. Dependency graph commands: `cargo tree --locked --manifest-path .audit-artifacts/pinned-runtime/Cargo.toml --prefix none --format '{p}'` and the same command with `--no-default-features`; distinct package lines were counted after removing repeat markers. No failing test was disabled or assertion weakened. Intermediate fixture mistakes and implementation regressions were corrected before final verification.

## Durable records

SignalBox: new Capture `cap_460fadca-c287-453e-aa05-f9144d970613` and Signal `sig_72904c72-5b75-48af-a716-421174972899` track concrete approval-path migration. Both passed exact-ID and conceptual retrieval. Skipped duplicates: existing reconciliation Capture/Signal and Kujo nested-assignment Signal; rejected completed-work recaps as Capture candidates. Strata consolidation uses Agent Notes and the final pushed SHA; its confirmed result is reported in the final assistant receipt.
