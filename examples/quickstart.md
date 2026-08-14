# Quickstart

`./bin/presswire init --state /tmp/presswire-demo --json`

`./bin/presswire preflight --state /tmp/presswire-demo --input fixtures/core.json --actor operator --timestamp 2026-08-14T00:00:00Z --json`

The fixed timestamp makes fixture IDs deterministic; repeating the command is rejected.
