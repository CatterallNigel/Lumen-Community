# Lumen v3.2.10 – Release Notes

**Date:** 26 July 2026

## Purpose

v3.2.10 is a focused reliability release based on observations from the interrupted v3.2.9 evaluation run.

## Changes

### Supersession-aware checkpoint retry

When a newer checkpoint is created for a session, older checkpoints that previously failed persistence are removed from the active retry journal.

The older checkpoint remains visible for audit purposes with the state:

```text
superseded_unpersisted
```

It also records the checkpoint that superseded it. The newest checkpoint remains the authoritative persistence candidate.

A Final Cognitive Checkpoint supersedes a pending rolling checkpoint from the same generation.

### Stable checkpoint UI chronology

The checkpoint UI now explicitly orders checkpoints by generation and original creation time. Persistence retry updates no longer determine which checkpoint appears first.

The UI displays `superseded_unpersisted` as a separate state and shows the superseding checkpoint identifier.

### EOF diagnostic states

The completion report now records one of three EOF states:

```text
unknown
incomplete
verified
```

This replaces the misleading initial `eof_verified=True` condition that could occur before the first read result had been inspected.

- `unknown` — a read tool call has been issued, but its result has not yet established continuation or EOF.
- `incomplete` — a continuation read remains outstanding.
- `verified` — no continuation remains and completion is no longer a tool-call response.

## Validation

The complete automated test suite passes:

```text
135 passed
```
