# Contracts

Contract 1.0.0. PressWire owns local publication requests, preflights, receipts, correction/unpublish requests and idempotency records. Records retain schema/tool versions, stable IDs, actor, UTC timestamp, provenance, command and payload. Emitted records use tool version 0.2.0; readers retain 0.1.0 compatibility and safe unknown payload metadata. The JSON envelope is `ok/data/error/error_code/tool_version/contract_version`. Exit codes remain 0 success, 1 operational failure and 2 usage error.

## Publication and approval

`publish --act --yes --path SOURCE --output TARGET` is the connected local-filesystem effect. VersionSeal approval records from tool versions `0.1.0`, `0.2.0`, and `0.3.0` are accepted only with schema and contract `1.0.0`. `version.supported_approval_record_versions` advertises this producer allowlist separately from `supported_record_versions`, which describes PressWire records accepted by `validate`. Approval evidence must bind identity, actor, action, destination label and artifact checksum. New approval records may add `payload.output_path`; when present it must exactly equal `--output` (no path alias normalization). Old opaque destination labels remain accepted and are **not concrete filesystem path authorization**. Migrate approval producers to `output_path` before relying on path-specific approval. `--force` does not bypass approval or size checks.

Publication reads one binary snapshot, checks its size and SHA-256, and writes those same bytes atomically. A destination created concurrently is never replaced unless `--force` was requested. Accepted artifacts remain limited to 64 MiB; snapshot buffering increases memory compared with the earlier streaming copy. Parent directories are operator-trusted. Input files must remain stable during reads; this is not a sandbox for adversarial directory owners.

`--id` and `--timestamp` are honored after approval parsing. Default effect IDs derive from approval ID, action, destination label and checksum. Explicit IDs deliberately select another record identity; they do not provide approval-global exactly-once execution. Dry-run creates no destination file or destination parents, but may initialize local state and briefly acquire its operation lock.

`schedule`, `correct` and `unpublish` validate approval and record requests; they do not call remote providers. CMS/Git-static/newsletter conformance, prepared/sent/confirmed/failed/compensated receipts, reconciliation, compensation rules, HMAC evidence verification and deterministic fault injection are exported library helpers. They are not a connected hosted-provider workflow or a CLI signature-enforcement policy.

## Storage and recovery

All CLI record mutations acquire `<state>/locks/.publication-operations.lock` before checking identity or delivering output. Record persistence separately uses `<state>/locks/<id>.lock`. Locks are atomically created files, not recursive directory creation. Existing lock directories still fail closed. Stop older PressWire processes before upgrading; mixed-version writers are unsupported.

No-overwrite record/history writes preserve immutable records and append-only events. Record size and known history conflicts are checked before publication. A later persistence failure returns `publication_unrecorded` with the delivered record and storage error code in `data`; inspect the destination and state before retrying. Filesystem publication and record/history persistence are not one crash-atomic transaction.

A crashed writer may leave a lock. Do not automatically expire or delete it: first confirm no writer is running, inspect output, records and history, then remove only that stale lock. A lock-cleanup failure is reported explicitly. There is no implicit retry. Cross-filesystem reconciliation remains an architectural follow-up.

## Queries and exports

`history`, `report` and `export` accept `--type`, `--after` and `--limit` (1–1000, default 1000). Config accepts `state`, `actor` and integer `limit`; explicit flags take precedence. `--after` is an opaque bare-filename cursor, used only for comparison, never joined into a path. Consume returned `next_after` while `truncated` is true. Ordering compares bare IDs, including prefix IDs. Concurrent changes are not a snapshot.

Each page inspects at most 1000 candidate records and retains at most 8 MiB of compact serialized record data. Warnings are bounded by the scan count. A filtered page can contain no matches yet be truncated; advance its cursor. Directory-name enumeration and sorting still scale with total directory entries. `doctor` reports `truncated` and `next_after`, so a healthy sampled page is not a full-state audit.

`history` continues to list records, not the separate event directory. `export` without `--output` preserves stdout data. With `--output`, it writes a JSON export atomically and emits a small path/count receipt. Existing files require `--force`. File export refuses truncated or unreadable selections rather than silently claiming a complete backup. Paginate stdout for larger selections. `--dry-run` skips file writing.

## Developer verification

Run `bash scripts/validate.sh`. Set `KUJO_BIN` to choose the runtime; the launcher otherwise tries PATH and then the sibling checkout. The validation gate owns an isolated temporary fixture directory through test-only `PRESSWIRE_TEST_ROOT` and removes it on exit. Direct suite runs default to ignored `tests/tmp/`.
