# Milestone 6 Acceptance — Recording Explorer

## Scope

Milestone 6 provides complete read-only operator visibility into validated Trace recordings.

## Accepted capabilities

- Dashboard wording is **Replay Capabilities**.
- Dashboard displays recording, completion, session, and message totals.
- Recording table displays messages, exchanges, duration, timestamps, status, View and Prepare actions.
- Recording Explorer displays metadata, statistics, timeline, grouped request/response exchanges, and prepared sessions.
- Search covers body text, headers, paths, sequence numbers, status values, and exchange identifiers.
- Exchange Inspector displays both sides of a captured exchange.
- Message Inspector displays all fields available in the immutable Replay model.
- Captured bytes are decoded for display with replacement characters only; source data remains unchanged.
- No Replay execution or target traffic is introduced.

## Quality gate

```text
pytest
ruff check .
mypy src tests
```
