# Milestone 5 Acceptance — Operator UI

## Scope

Milestone 5 adds a server-rendered operator interface before any Replay execution capability.

## Acceptance criteria

- [x] The root route renders the Lumen Replay operator dashboard.
- [x] The dashboard performs and displays a live MongoDB connectivity check.
- [x] MongoDB credentials are masked in displayed configuration.
- [x] Runtime values loaded from `config.yml` and environment overrides are visible.
- [x] Trace recording and Replay-owned collection names are visible.
- [x] The configured Lumen endpoint is visible but no Lumen request is sent.
- [x] Recent immutable Trace recordings are listed.
- [x] Recording status and message counts are displayed.
- [x] Only completed recordings expose the Prepare action.
- [x] Prepared Replay sessions are listed newest first.
- [x] The existing Replay session service is reused by the UI.
- [x] The dashboard describes Milestones 1–5 and current capability.
- [x] UI limits and title are configurable through `config.yml`.
- [x] Templates and CSS are included as package data.
- [x] No execution, identifier rewriting, model reset, response capture, comparison, or assessment is introduced.

## Operator route

```text
GET /
POST /ui/recordings/{recording_id}/prepare
```

## Architectural boundary

The UI is an operator surface over existing Replay services. It does not access Trace collections directly, mutate Trace data, or send traffic to Lumen.
