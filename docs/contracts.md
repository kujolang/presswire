# Contracts

Contract 1.0.0. PressWire owns: Destination Profile; Destination Capability; Preflight Result; Publication Request; Publication Receipt; Correction Receipt; Rollback Result; Idempotency Record. Records carry schema/tool versions, stable IDs, actor, timestamp, provenance, command, and payload. Consumers accept compatible 1.x, preserve safe unknown payload metadata, and reject incompatible majors. JSON uses `ok/data/error/tool_version/contract_version`. Offline upstream fixtures identify repository, tag, schema, and checksum.
