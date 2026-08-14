# PressWire

Bounded publication gateway for approved packages with adapter capabilities, preflight, idempotency, and receipts.

PressWire 0.1.0 is an independently installable, local-first Kujo tool. It requires no hosted service, Chain of Command, WebOps, or sibling Publishing House tool. The canonical entrypoint is `presswire.kujo`; `bin/presswire` contains no product logic.

## CLI

Commands: preflight; adapters; capabilities; schedule; publish; status; correct; unpublish; receipt; history; doctor; version; init; show; export; validate; report. Run `./bin/presswire help` for flags. Mutations require `--actor`; JSON input uses `--input`. Common flags include `--json`, `--dry-run`, `--state`, `--output`, `--config`, and `--force`. Exit codes: 0 success, 1 validation/operation failure, 2 usage error.

State defaults to `.presswire/`. Immutable JSON records and append-only history use atomic writes. IDs reject traversal; symlinks and oversized inputs are rejected. See [contracts](docs/contracts.md), [security](docs/security.md), and [quickstart](examples/quickstart.md).

Test with `/Users/robertdevore/2026/Kujolang/kujo-repos/kujo/target/release/kujo run tests/test.kujo`, then run `./bin/presswire doctor --json`.

0.1.0 covers the documented local records, fixtures, validation, checksums, deterministic fixed-time IDs, and structured export. It does not manufacture human judgment, consent, rights, approval, or causation. 0.1.0 ships offline fixture and bounded local-filesystem adapters; hosted providers and Git/static-site effects are unavailable unless explicitly configured through a future adapter.

Local publication requires an approval-scoped JSON input, matching source SHA-256, \`--path\`, \`--output\`, and \`--act --yes\`. It writes atomically, returns a Publication Receipt, refuses existing targets unless \`--force\` is explicitly used for the safe replacement, and derives an idempotency key that prevents duplicate effects across timestamps.
