# Lumen v3.2.1 — Durable Continuity and Resume

## Highlights

- Persists every Distilled Continuity checkpoint as an immutable MongoDB document.
- Links the active session to its latest checkpoint while retaining checkpoint lineage.
- Restores the latest Distilled Continuity, objective, deterministic source coverage, and up to two retained raw source chunks instead of replaying the full historical transcript when a checkpoint exists.
- Persists checkpoint-covered ranges, retained raw ranges, last completed offset, and next unread offset with an explicit line offset unit.
- Scales the checkpoint output token budget from a 16K-input/2,000-output baseline up to a configurable ceiling (4,096 by default).
- Adds checkpoint and resume diagnostics including trigger ratios, generations, source ranges, requested/produced tokens, elapsed time, and Mongo persistence status.
- Adds rotating file logging: 25 MB per file with three backups, while retaining console output.

## MongoDB

The session collection remains configurable with `mongodb.collection`. Immutable checkpoints are stored in `mongodb.checkpoint_collection` (`checkpoints` by default).

## Logging

Default file logging is `logs/lumen.log` with:

- `max_bytes: 26214400`
- `backup_count: 3`
