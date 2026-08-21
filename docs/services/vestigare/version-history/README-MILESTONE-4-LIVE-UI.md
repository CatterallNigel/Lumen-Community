# Milestone 4 — Live HTML UI

This increment adds the first Lumen Trace browser interface at:

    http://127.0.0.1:11438/trace

It provides:

- test-name input
- start and stop recording controls
- current recording state
- live one-second polling
- conversation timeline
- polling-traffic filter
- message detail inspection
- JSON, text, and base64 body presentation
- duplicate header display
- stream chunk metadata

The interface remains a Trace control and observation surface only. It does not add replay, assessment, recording browsing, or Servire functionality.

## Replay-private exchange ingestion

Trace exposes `POST /trace/recordings/current/exchanges` for Replay-private model
calls that intentionally bypass Trace's transparent proxy. The endpoint accepts the
complete request/response exchange from Replay and persists it into the currently
active Trace recording through the same recording manager and sequence allocation
used by proxied traffic.

The endpoint returns HTTP 204 after successful capture and HTTP 409 when no Trace
recording is active. It is an evidence-ingestion boundary only; Trace does not
interpret or assess the exchange.

## Offline Clear Logs command

Vestigare now implements the standard Lumen component-owned offline log cleanup command:

```cmd
python -m lumen_trace clear-logs
```

The command clears only the contents of the component's `$ROOT/logs/` directory,
preserves the directory itself, and refuses to run while the configured Vestigare
listener is active. It does not start FastAPI, connect to MongoDB, connect downstream,
or initialise Trace recording/proxy behaviour.
