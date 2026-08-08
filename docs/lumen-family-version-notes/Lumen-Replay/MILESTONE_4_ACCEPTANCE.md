# Milestone 4 Acceptance — Replay Session Preparation

## Implemented

- Operator-facing `config.yml` schema for Lumen and MongoDB configuration.
- Environment-variable overrides remain supported.
- Replay-owned `replay_sessions` persistence.
- Immutable `ReplaySession` domain model.
- New `replay_id` and isolated `replay_session_id` for every preparation.
- Provenance back to the immutable Trace recording.
- `POST /recordings/{recording_id}/prepare`.
- Recording validation is reused before session persistence.

## Architectural boundaries

- Replay replaces Pi; Pi reset is neither required nor implemented.
- No request is sent to Lumen.
- No recorded payload is modified.
- No source identifier is rewritten yet.
- No response comparison or assessment is performed.
- A Replay UI must precede the execution milestone.
