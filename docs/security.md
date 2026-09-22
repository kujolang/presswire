# Security and authority

PressWire is a local operator tool, not a multi-tenant service. The operator owns configuration, approval provenance, state and destination ancestors. It exposes no production network or shell-execution interface. State IDs reject traversal; managed directories and file leaves reject symlinks. Ancestor directories, including platform aliases such as `/tmp`, remain trusted; there is no hostile-filesystem sandbox or TOCTOU-proof descriptor-relative traversal guarantee.

Inputs are limited to 1 MiB, config to 64 KiB, artifacts to 64 MiB, and serialized records to 1 MiB. Sensitive JSON keys are rejected recursively; this is not a secret scanner for arbitrary string values or artifact content. Calendar dates, actor strings, configuration types and exported helper types are validated. See query aggregate limits in [contracts](contracts.md).

Effects require exact approval identity, actor, action, label and checksum plus `--act --yes`. An optional approval `payload.output_path` binds a concrete CLI output string; legacy labels alone do not. Signatures are available as a library helper, not automatically enforced by the CLI. Do not treat local unsigned approval files as hosted identity proof.

Verified snapshot bytes are the bytes written. Operation locks precede delivery, record/history writes never overwrite existing files, and concurrent output writers cannot replace a destination without explicit `--force`. Predictable storage rejection is checked before delivery. Crash recovery and cross-filesystem commit remain separate concerns; `publication_unrecorded` and stale locks require inspection, not blind retries.
