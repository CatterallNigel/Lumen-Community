# Lumen v3.1.3

## Restore identity and diagnostics fix

- Rebinds a single staged MongoDB restore to the next ordinary Pi request when Pi's fallback conversation hash changes after local `\\obt` command exchanges are removed.
- Retains exact-key matching whenever Pi supplies a stable key or the fallback key remains unchanged.
- Keeps restore one-shot: the staged document is consumed only by the next non-command request.
- Logs `pending_restore_rebound` when the fallback key changes.
- Logs `restore_applied` with saved session id, staged key, active key, message counts before/after injection, restored message count, and restored character count.
- Logs `restore_not_applied` for ordinary requests with no staged restore.
- Updates persisted bridge source version to 3.1.3.

The running v3.1.2 process is unaffected until it is stopped and v3.1.3 is started.
