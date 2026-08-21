# Lumen Rogare M4.5 — Session, Trace and Operator UX

## Scope

This milestone completes the remaining Rogare conversational-console operator work after the Moderari → Pontis backchannel was proven end-to-end.

### 1. Backchannel polling efficiency

Rogare polls Pontis for retained operational events every 5 seconds while a request is active. Moderari's heartbeat cadence remains unchanged.

### 2. Explicit new-session lifecycle

Rogare now provides a **New Session** action. A new session clears the Rogare conversational surface, resets provider presentation state, generates a new logical session identifier and requires an explicit Start Session action before conversation resumes.

### 3. Automatic editable session names

New sessions default to `session-rogare-YYYYMMDD-HHMMSS`. The generated name remains editable until the session is started.

### 4. Bootstrap/history presentation isolation

Rogare never presents the provider bootstrap exchange as normal conversation. A defensive presentation filter also removes the legacy Moderari startup banner or a bare bootstrap `READY` response if either leaks into a normal user turn. Explicit `\\obt` command responses are not filtered.

### 5. Rogare-native `\\obt` guidance

The composer permanently explains that `\\obt` commands are handled by Lumen and are not model prompts, and points the user to `\\obt help`.

### 6. Vestigare Trace controls

Rogare provides direct Start Trace / Stop Trace controls and current recording status. These calls go directly from Rogare to Vestigare at the configured `vestigare.base_url`; they do not traverse Pontis, Repetere or Moderari.

### 7. Working/progress presentation

The existing transient activity panel now keeps Moderari progress text separate from Rogare's local elapsed timer and heartbeat acknowledgement. Operational events remain outside the conversation history and the activity panel clears when the final response arrives.

## Configuration

```yaml
vestigare:
  base_url: http://127.0.0.1:11438
  timeout_seconds: 15
```

## Trace API contract used

- `GET /trace/recordings/status`
- `POST /trace/recordings/start` with `{ "name": "<recording name>" }`
- `POST /trace/recordings/stop`

## M4.6 Addendum — Conversation cancellation

Rogare now exposes a **Cancel** button beside **Send** while a conversational request is active.
Cancellation is explicit at both UI and server layers: the browser aborts the outstanding request and Rogare cancels the registered FastAPI request task for that session, allowing cancellation to propagate through the active `httpx` call toward Pontis and the downstream Lumen stack. The control is disabled whenever no conversation is running, and a cancelled turn is recorded in the Rogare surface as `Conversation cancelled.` rather than as an error.
