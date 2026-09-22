# Agent instructions

Keep CLI, domain behavior, validation, storage, fixtures, release checks, and tests in Kujo. Preserve immutable records, append-only history, atomic writes, bounded I/O, path/symlink protection, offline behavior, and authority boundaries. Run `bash scripts/validate.sh` (set `KUJO_BIN` if needed). Read `docs/contracts.md` before changing approval, lock, pagination, or export behavior. Never force-push or use live credentials in tests.
