# Lumen Rogare 0.4.3

## Stale session validation

- Validates persisted Rogare session identity against Pontis `/_pontis/sessions/{session_id}` before UI reattachment.
- Clears stale browser session state after a full stack/Pontis restart.
- Prevents stale sessions from sending Rogare's placeholder `model: lumen` downstream without Pontis provider enrichment.
- Keeps the existing detach/reattach continuity behaviour when the Pontis binding is still live.
