# Milestone 8 Acceptance — Rogare / Servire Integration

## Scope

Milestone 8 completes the Servire-side Rogare integration while preserving the established conversational and tool-provider protocol path.

## Accepted behaviour

- Servire manages Rogare as the `conversational_client` service on `127.0.0.1:11432`.
- Rogare remains dependent on Pontis / `communication_bridge`.
- The default **Rogare — Console** workspace embeds the native Rogare UI.
- **Open directly** opens Rogare in its own named browser window.
- After pop-out, Servire changes to a lightweight Rogare status surface instead of continuing to display a second conversational console.
- The status surface states **Rogare is open in a separate window.**
- The status surface provides **Go to Rogare** and **Use embedded console** actions.
- **Go to Rogare** focuses/reuses the named Rogare pop-out without deliberately reloading it.
- Servire retains/reacquires the named pop-out handle so it can close the detached Rogare window when the user explicitly returns to the embedded console.
- **Use embedded console** first displays an in-page Servire warning; it does not immediately close Rogare or use a browser-native alert.
- The warning explicitly states that the active Rogare/Lumen session will reconnect in the embedded console, while the previous visible browser transcript will not yet be redisplayed.
- Confirming **Return to Servire** closes the detached Rogare window, clears direct-presentation state and restores the embedded Rogare UI.
- Cancelling with **Keep Rogare separate** leaves the pop-out and its current browser state untouched.
- Direct presentation is represented by `?presentation=direct` and retained in the Servire browser session, so Servire navigation and refreshes do not silently recreate a second Rogare console.
- No live cross-window transcript synchronisation is introduced; Rogare persists only the active logical session/control metadata needed for single-surface reattachment.
- No additional Rogare process, Pontis provider session, or ACP session is created by the presentation change.
- Non-Rogare component workspace behaviour remains unchanged.

## Protocol invariant

This milestone does not alter:

```text
Rogare
  ↓
Pontis :11435
  ↓
Vestigare :11438
  ↓
Repetere :11437
  ↓
Moderari :11436
```

Tool execution remains Pontis ↔ Pi via ACP.

## Verification

The package test suite includes explicit coverage for both the embedded Rogare presentation and the direct/status presentation. Static checks remain part of release verification.
