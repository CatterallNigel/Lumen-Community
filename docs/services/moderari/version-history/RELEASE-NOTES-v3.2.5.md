# Lumen v3.2.5

## Checkpoint observability

- Added a minimal live checkpoint page at `GET /checkpoints`.
- The page polls every two seconds and updates when a checkpoint is received or its persistence state changes.
- Added JSON endpoints:
  - `GET /api/checkpoints`
  - `GET /api/checkpoints/latest`
- Checkpoints display generation, session, timestamp, newly covered source ranges, context before/after compaction, continuity size, full distilled continuity, and persistence status.

## Resilient checkpoint persistence

- Checkpoint generation remains authoritative in memory even when MongoDB is unavailable.
- Failed MongoDB writes are recorded in a durable local JSON journal.
- A background retry worker retries pending writes at a configurable interval.
- Checkpoint persistence is idempotent, so a partial MongoDB write can be safely retried.
- The UI distinguishes `pending`, `pending_retry`, and `persisted` checkpoints and exposes the most recent persistence error.

## Configuration

```yaml
checkpoint_retry:
  enabled: true
  interval_seconds: 15
  journal_path: data/pending-checkpoints.json
```

Environment overrides:

- `LUMEN_CHECKPOINT_RETRY_ENABLED`
- `LUMEN_CHECKPOINT_RETRY_INTERVAL_SECONDS`
- `LUMEN_CHECKPOINT_RETRY_JOURNAL`

## Validation

The complete automated test suite passes, including new tests for checkpoint observation, UI endpoints, durable journalling, and successful retry.
