# Lumen Replay — Milestone 3 Acceptance

## Objective

Load one completed immutable Trace recording into a validated in-memory Replay model without
modifying, adapting, dispatching, comparing or scoring any captured interaction.

## Implemented

- read-only `trace_messages` repository;
- exact preservation of BSON binary paths, query strings, headers and bodies as immutable bytes;
- immutable `TraceMessage`, `TraceHeader` and `ReplayRecording` domain models;
- completed-recording eligibility validation;
- metadata/message-count consistency validation;
- contiguous sequence validation beginning at sequence 1;
- recording ownership validation for every message;
- request/response direction and exchange-pair validation;
- safe load summary endpoint: `GET /recordings/{recording_id}/load`;
- explicit API handling for missing, incomplete, malformed and unavailable source recordings;
- unit tests for repository parsing, loading, integrity checks, API mapping and application wiring.

## Explicitly excluded

- creation of replay sessions;
- replay identifier generation or remapping;
- body or header interpretation and rewriting;
- model reset or provider control;
- request dispatch;
- replay response capture;
- behavioural comparison, scoring or assessment;
- Replay UI.

## Required quality gate

```text
pytest
ruff check .
mypy src tests
```

Coverage must remain at or above 95%.
