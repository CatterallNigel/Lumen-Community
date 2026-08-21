# Milestone 3 typing fix

This patch fixes the MyPy errors reported after the first recording-engine increment.

Changes:

- Casts `app.state.recording` to `RecordingManager` at the FastAPI state boundary.
- Defines the database lifecycle property using the recorder's structural `AsyncDatabase` protocol rather than the concrete Motor database type.
- Completes the health-test database double with the collection methods required by `AsyncCollection`.

No runtime behaviour is changed.
