# Lumen v3.2.14.2 – Operational UI Refinement

This release extends the Operational Intelligence dashboard with execution-level timing, tool and source evidence, terminal state correction, and a unified execution timeline.

## Added

- Live Execution Summary with start time, elapsed time, model time, tool time, persistence time and orchestration overhead.
- Current phase duration and latest meaningful activity.
- Ordered tool history with target, status and duration.
- Source coverage with character count, offset and EOF status.
- Runtime execution events combined with checkpoint and result persistence in one timeline.
- Terminal checkpoint fallback for Distilled Cognition, including objective, outcome and observed solution path.
- Explicit operational phases and correct `COMPLETED`, `FAILED`, and `CANCELLED` terminal states.

## Behaviour

Execution timing is retained across Pi tool-call round trips within the same Lumen session. Tool calls are opened when Qwen selects them and closed when Pi returns the matching tool result. Final persistence now transitions the dashboard to `COMPLETED / Result persisted` rather than leaving the last transient checkpoint state visible.
