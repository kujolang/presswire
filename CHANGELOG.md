# Changelog

## Unreleased

- Bind local delivery to a verified binary snapshot, coordinate before effects, and use atomic no-overwrite publication and state writes.
- Preflight predictable receipt failures; expose partial-delivery failures and optional approval output-path scope.
- Restore documented query/config options, complete file export receipts, portable launcher discovery, and JSON version aliases.
- Enforce record/query budgets in UTF-8 bytes so Unicode cannot create unreadable records.
- Bound query pages and warnings; add calendar/helper validation, concurrency and CLI regression gates, and isolated test cleanup.


## Unreleased

- Standardized README badge ordering and repository-local artifact ignores.
- Kept Loop Engineering evidence available locally while removing it from published source.

## 0.2.0 - 2026-08-14

- Preserved validation compatibility with immutable 0.1.0 records while emitting 0.2.0 records.
- Prevented audit-history conflicts from leaving partial records and added clean-retry regression coverage.
- Rebuilt the core around shared storage and validation modules; added real config handling, hardened VersionSeal provenance checks, fail-closed state handling, UUID-temporary publication, and explicit adapter capabilities.
- Hardened approval scoping, idempotent publication effects, atomic output, secret rejection, timestamps, CI, and verification coverage.

## 0.1.0 - 2026-08-14

- Initial Kujo-native release with working local records, validation, contracts, fixtures, and safety boundaries.
