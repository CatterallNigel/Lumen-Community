# Milestone 2 acceptance criteria

## Scope: read-only Trace recording catalogue

- [x] Retain Milestone 1 service, configuration, logging, MongoDB lifecycle and health behaviour.
- [x] Remove the duplicated unreachable application-factory lines found in the supplied Milestone 1 package.
- [x] Add immutable typed Trace recording metadata.
- [x] Add a read-only recording repository abstraction.
- [x] Add a MongoDB repository for `trace_recordings`.
- [x] Add bounded newest-first recording listing.
- [x] Add lookup by exact `recording_id`.
- [x] Add `GET /recordings` with optional status filtering and bounded pagination.
- [x] Add `GET /recordings/{recording_id}`.
- [x] Return 404 for an unknown recording and 503 for repository failures.
- [x] Keep MongoDB collection names configurable.
- [x] Test domain mapping, repository behaviour, API behaviour and application wiring.
- [x] Preserve Trace recordings unchanged; no update, delete or copy operation exists.

## Explicitly excluded

- Trace message loading or BSON binary decoding.
- Replay session creation.
- Session identifier remapping.
- Model reset or model-provider calls.
- Request dispatch or response capture.
- Timing reproduction.
- Comparison, behavioural assessment or scoring.
- Replay UI.

## Commit

```bash
git add .
git commit -m "feat(replay): add read-only trace recording catalogue"
```
