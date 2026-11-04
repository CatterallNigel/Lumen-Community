# Lumen — Repetere Increment 01: Implementation status

**Date:** 31 October 2026  
**Status:** Draft Moderari implementation; NOT integration-tested or release-ready.

## Source packages reviewed

Repetere 0.20.20, Moderari 30 October, Vestigare 18 October, Praebere ExPro Handover Corrected, Nuntius 0.4.5, and the approved Increment 01 audit.

## Implemented in draft Moderari package

- `src/lumen_moderari/control.py`: `\obt moderari session execution-plan register <JSON>` handling; Repetere origin restriction, envelope/payload session check, structured accepted/not-ready response, conflict handling.
- `src/lumen_moderari/emd_handover.py`: recorded-SEP storage per session, immutable registration semantics, parsing of source SEP into typed references, provider/model EMD check, First Ask branch without Execution Profile lookup, comparison of resolved installed execution definition against recorded definition, cleanup of pending registrations.
- `tests/test_recorded_sep_handover_i01.py`: draft HTTP control tests for acceptance, idempotency, conflict, origin, mismatch and malformed reference.

## Current limitations / release blockers

1. **Tests could not execute**: the container lacks `pymongo` and network package installation is unavailable. Python syntax compilation passed, but this is not a substitute for pytest, mypy and integration testing.
2. **No Repetere client integration yet**: Repetere still needs the Nuntius command invocation, session preparation state, and failure handling; existing replay execution path is unchanged.
3. **No source-SEP retrieval integration yet**: verify the exact Vestigare read API, source integrity and provenance; Repetere must treat the SEP as opaque.
4. **Uncertain-outcome reconciliation**: Nuntius 504 outcomes require idempotent retry and/or status read. Idempotent registration is implemented; a dedicated status command is not.
5. **Prompt-bearing payload logging**: inline SEP may contain system prompt content; Nuntius currently logs control envelopes. Redaction or approved reference transport is needed before production deployment.
6. **First Ask enabled-path tests**: test exact Adapter and Filter versions, protocol/CIM, response mode, no substitution, and replay session/recording SEP linkage. The draft compares a newly resolved plan against the recorded plan and rejects differences; no live test has verified all cases.
7. **Schema hardening**: expand malformed/injected payload tests, field types, size limits, and robust structured error taxonomy before release.
8. **Failure evidence policy**: persistence of failed preparation attempts remains a later decision.

## Verification attempted

- `python -m compileall` on Moderari source and draft test: passed.
- `pytest tests/test_n7_control_obt.py`: could not collect because `pymongo` is unavailable.
- `pip install pymongo==4.10.1`: failed because external package index DNS/network access is unavailable.

## Unchanged services

Nuntius, Praebere, Vestigare and Repetere are not modified by this draft package. The architecture continues to require Pontis-owned session IDs, Praebere EMD handover, First Ask authoritative validation, Vestigare SEP acknowledgement, and replay child trace association by session ID.

## Next steps

1. Run complete Moderari pytest, ruff, mypy on the user's development environment; address failures.
2. Add strict payload validation and full enabled/disabled First Ask tests.
3. Settle inline SEP logging redaction or alternate transport and timeout reconciliation.
4. Integrate Repetere as an automated client and verify Vestigare linkage.
5. Deliver synchronized complete affected-service packages only after cross-service tests pass.
