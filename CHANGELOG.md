# Changelog

## 0.2.0 - 2026-08-14

- Preserved validation compatibility with immutable 0.1.0 records while emitting 0.2.0 records.
- Prevented audit-history conflicts from leaving partial records and added clean-retry regression coverage.
- Rebuilt the core around shared storage and validation modules; added real config handling, hardened VersionSeal provenance checks, fail-closed state handling, UUID-temporary publication, and explicit adapter capabilities.
- Hardened approval scoping, idempotent publication effects, atomic output, secret rejection, timestamps, CI, and verification coverage.

## 0.1.0 - 2026-08-14

- Initial Kujo-native release with working local records, validation, contracts, fixtures, and safety boundaries.
