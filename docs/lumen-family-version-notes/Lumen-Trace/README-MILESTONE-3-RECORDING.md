# Milestone 3 — Recording Engine

This increment adds the recording engine without adding the live UI or recording browser.

## Control API

```text
GET  /trace/recordings/status
POST /trace/recordings/start  {"name": "operations baseline"}
POST /trace/recordings/stop
```

Stopping waits for any response streams that began while the recording was active. New exchanges are not accepted once stopping begins.

## Replay fidelity

Each exchange stores two ordered MongoDB documents:

- `pi_to_lumen`: exact raw path, query bytes, duplicate headers, and request body bytes.
- `lumen_to_pi`: status, duplicate headers, complete response bytes, response-stream chunk sizes, duration, and completion state.

The original recording is never rewritten. Session-ID remapping remains a future Lumen Replay responsibility.
