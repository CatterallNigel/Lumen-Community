# Lumen — Repetere Increment 01 integration progress

**Status:** Partial implementation, not a deployable integration release.

## Completed
- Inspected Repetere v0.20.20 and the tested Moderari draft's control command.
- Added `infrastructure/recorded_sep_control.py` as a dedicated Nuntius handover client.
- The client copies an opaque recorded SEP and rebinds only `session_id`; it does not interpret Adapter, Filter or execution semantics.
- Uses `\\obt moderari session execution-plan register <JSON>` with the established control envelope.
- Requires an authoritative acknowledgement of `accepted=true`, matching session, source `recorded_sep`, and both `materialised=false` and `session_ready=false`.
- Distinguishes definite HTTP rejection from uncertain transport failure.
- Added contract tests for acceptance, invalid acknowledgement and timeout.
- Python syntax compilation passed.

## Not yet completed
- Wire authoritative SEP retrieval from Vestigare `trace_recordings` into the Repetere recording loader. Existing `TraceRecording` metadata omits `session_execution_plan` and its SHA-256.
- Replace the runtime's universal `prompt pass-through` prerequisite with Original Setup registration, without breaking New Setup.
- Verify exact model reservation and the point where the Nuntius handover occurs.
- Preserve existing Run, child trace, cleanup, and divergence semantics.
- Execute pytest, Ruff, mypy and end-to-end regression tests. The current container lacks `pymongo`, which is imported by Repetere's pytest `conftest.py`; pytest could not start.

## Deployment
Do **not** deploy this partial package as the completed Original Setup integration. It contains an isolated client and contract tests only; the runtime still follows the old pass-through path.
