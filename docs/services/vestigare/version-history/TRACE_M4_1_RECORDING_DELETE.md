# Lumen Trace M4.1 — Recording deletion control API

Adds the Trace-owned `DELETE /trace/recordings/{recording_id}` control operation required by Replay's operator UI.

The operation deletes both recording metadata and its captured messages, rejects deletion of the active recording with HTTP 409, returns HTTP 404 for an unknown recording, and returns HTTP 204 on success. If the most-recent completed recording is deleted, Trace resets its in-memory recording snapshot to idle.
