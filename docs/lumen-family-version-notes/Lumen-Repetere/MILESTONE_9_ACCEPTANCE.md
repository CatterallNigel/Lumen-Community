# Milestone 9 Acceptance — Replay Runtime

## Implemented

- Replay is a transparent HTTP proxy by default.
- Explicit `\obt replay` commands are consumed by Replay.
- A prepared replay can be started with `\obt replay start <replay-id>`.
- Recorded `/v1/chat/completions` requests are executed privately against Lumen.
- JSON and SSE model responses are canonicalised for behavioural matching.
- Generated tool-call IDs are ignored; tool names and arguments remain significant.
- Matching recorded tool results are reused through the next cumulative source request.
- The first different response is persisted as a Fork Point.
- The divergent response is returned unchanged to Trace/Pi.
- Replay becomes transparent after a Fork Point.
- A terminal post-fork answer returns Replay to transparent mode.
- Replay runs are persisted in the Replay-owned `replay_runs` collection.

## Not implemented

- Behavioural scoring or answer-quality assessment.
- Tool execution inside Replay.
- Trace route switching or automatic Replay discovery.
- Multi-process coordination of one active runtime.
- Polished operator UI; that belongs to Servire.

## Quality gate

```bat
pytest
ruff check .
mypy src tests
```
