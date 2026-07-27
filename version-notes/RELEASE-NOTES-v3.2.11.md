# Lumen v3.2.11 — Read Lifecycle and Finalisation Control

## Purpose

v3.2.11 addresses the execution failure observed at the end of a long, chunked
source read. The model repeatedly emitted the same high-level `read(path)` intent
while Lumen was transitioning into Final Cognitive Checkpoint generation.

The release establishes the following ownership boundary:

- Qwen expresses the intent to continue reading a resource.
- Lumen owns the authoritative path cursor, offset and EOF state.
- Replay protection operates on the resolved tool operation, not only the raw
  model-authored call.

## Changes

### Authoritative continuation offsets

When Qwen requests `read(path)` without an offset and the current tool result
contains a continuation marker, Lumen resolves the request to the authoritative
`next_offset` before replay evaluation.

### Continuation-aware replay protection

A read matching the outstanding resource and authoritative offset is treated as
a legitimate continuation, even when the raw model request resembles an earlier
`read(path)` call.

### Final Cognitive Checkpoint gate

Final Cognitive Checkpoint generation remains prohibited while:

- an outstanding read continuation exists;
- the latest source tool result is invalid or represents an error;
- a requested multi-file read still has unfinished resources; or
- the current user task has already produced its final checkpoint.

### Bounded recovery

The completion/replay recovery ceiling is reduced from twenty attempts to four.
This prevents long retry storms while preserving enough attempts for exact-read,
empty-completion and final-answer recovery.

## Required lifecycle

```text
Model requests read(path)
        ↓
Lumen resolves authoritative offset
        ↓
Pi executes the resolved read
        ↓
Result is delivered to the model
        ↓
Continuation marker present?
   Yes  → next read remains valid
   No   → EOF eligible for verification
        ↓
Generate user-facing answer
        ↓
Finalise and persist run artifacts
```

## Diagnostic priority

The operational logs must distinguish:

- raw model tool arguments;
- Lumen-resolved tool arguments;
- current authoritative cursor;
- continuation marker detection;
- replay decision;
- EOF eligibility;
- Final Cognitive Checkpoint eligibility; and
- answer delivery state.
