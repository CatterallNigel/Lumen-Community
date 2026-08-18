# Lumen Rogare v0.4.4 — Presentation History Reattachment

## Summary

Rogare now restores the visible user/assistant transcript when an active session is moved between the direct Rogare window and the embedded Servire console.

## Behaviour

- Rogare persists only user-visible `user` and `assistant` presentation events in browser storage, keyed to the active logical session.
- System notices, errors, provider bootstrap traffic, tool calls/results, and Moderari internal context are not persisted as chat presentation history.
- A fresh session starts with an empty presentation transcript.
- A verified active Pontis session reattachment restores the persisted transcript before displaying the reconnection notice.
- Presentation history is capped at the most recent 200 visible messages.
- If an older session has no stored presentation history, Rogare reconnects normally and reports that no earlier presentation history is available.
- Stale-session validation remains authoritative: if Pontis no longer has the binding, the browser state is cleared rather than restored.

## Architecture

This keeps the responsibility boundary explicit:

- Moderari owns model context and conversational continuity.
- Pontis owns provider/session arbitration.
- Rogare owns user-facing conversation presentation.

Rogare does not derive UI history from raw Moderari context, avoiding leakage of system prompts, tool traffic, distilled checkpoints, or other model-internal state.
