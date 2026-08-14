# Contracts

Contract 1.0.0. PressWire owns: Destination Profile; Destination Capability; Preflight Result; Publication Request; Publication Receipt; Correction Receipt; Rollback Result; Idempotency Record. Records carry schema/tool versions, stable IDs, actor, timestamp, provenance, command, and payload. Consumers accept compatible 1.x, preserve safe unknown payload metadata, and reject incompatible majors. JSON uses `ok/data/error/tool_version/contract_version`. Offline upstream fixtures identify repository, tag, schema, and checksum.

Hardening contracts add offline CMS/Git-static/newsletter adapter conformance, prepared/sent/confirmed/failed/compensated effect phases, reconciliation without implicit retries, adapter-aware correction/unpublish compensation, optional explicit-key VersionSeal evidence verification, and deterministic partial-provider fault injection.
