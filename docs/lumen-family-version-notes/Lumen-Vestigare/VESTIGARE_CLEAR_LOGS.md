# Lumen Vestigare — Offline Clear Logs

**Status:** Implemented

Vestigare now follows the Lumen offline log-management standard.

```cmd
python -m lumen_trace clear-logs
```

The package name remains `lumen_trace`, so the command uses that existing Python
module name even though the component's product name is Lumen Vestigare.

The operation:

- runs offline and exits immediately;
- checks the configured Vestigare listener before deletion;
- refuses to clear logs while Vestigare is active;
- clears only `$ROOT/logs/*`;
- preserves the `logs` directory itself;
- rejects a configured log directory that does not resolve to `$ROOT/logs`;
- does not touch Trace recordings, MongoDB state, configuration, or any other
  component's files;
- does not initialise the normal FastAPI/application lifecycle when `clear-logs`
  is selected.

A successful run reports the number of top-level log-directory entries removed.

## Configuration-owned service startup

Normal module startup now uses Vestigare's own `server.host` and `server.port`
configuration values:

```text
python -m lumen_trace
```

Servire therefore does not need to duplicate Vestigare's host and port as
Uvicorn command-line arguments. `python -m lumen_trace clear-logs` remains the
offline log-maintenance command and does not start Uvicorn.
