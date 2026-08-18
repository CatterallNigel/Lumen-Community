# Lumen Rogare — Servire Integration

## Purpose

Servire manages Rogare as the Lumen-native conversational client while Rogare remains an independent service.

## Runtime

- Rogare: `127.0.0.1:11432`
- Working directory: `C:\Development\Lumen-Rogare`
- Python: `.venv\Scripts\python.exe`
- Launch: `python -m lumen_rogare`
- Health: `http://127.0.0.1:11432/health`
- Servire role: `conversational_client`
- Dependency: Pontis / `communication_bridge`

Servire resolves the configured relative Python executable against the Rogare working directory, so the effective launch is equivalent to:

```text
C:\Development\Lumen-Rogare\.venv\Scripts\python.exe -m lumen_rogare
```

## Workspace

Rogare is exposed as the Servire **Rogare — Console** workspace. The default presentation embeds the native Rogare UI.

Selecting **Open directly** opens Rogare in a named browser window and moves the Servire workspace to `?presentation=direct`. In that presentation Servire no longer renders a Rogare iframe. Instead it shows a Servire-owned status panel stating that Rogare is open in a separate window, with actions to **Go to Rogare** or **Use embedded console**.

Servire retains the named pop-out handle while the page is alive and can reacquire the same named browsing context after Servire navigation. **Go to Rogare** focuses/reuses that window without deliberately navigating it again, preserving the detached Rogare browser/session state. Direct-presentation state is also held in the Servire browser session so moving between Servire workspaces and returning to Rogare continues to show the detached-status surface rather than silently embedding a second Rogare UI.

**Use embedded console** is deliberately guarded. It reveals an in-page Servire warning explaining that returning Rogare to Servire closes the detached window. Rogare now persists its active logical session identity and provider-control metadata in Rogare-origin browser storage. Because both the detached Rogare window and the embedded iframe use the same Rogare origin, the new embedded surface reconnects to that existing session without issuing another session-start command. The Lumen/Pontis session therefore continues, while the previous visible browser transcript is not yet redisplayed. The user may either **Keep Rogare separate** or explicitly **Return to Servire**.

Opening Rogare directly does not create a second Rogare backend process, Pontis provider session, or ACP session. It changes presentation only; the known-good Rogare → Pontis → Vestigare → Repetere → Moderari protocol path remains unchanged.

## Future

Rogare Trace Start/Stop should reuse the existing Vestigare/Replay operational control APIs. Rogare must not implement Trace lifecycle logic itself. Moderari heartbeat/progress propagation into the Rogare UI is a separate follow-on milestone.
