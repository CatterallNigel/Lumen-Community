# Milestone 1 — Logging increment

This increment adds project-wide logging under the `lumen_trace` logger hierarchy.

Implemented:

- `configure_logging()` with console and rotating-file handlers
- `get_logger()` for project child loggers
- human-readable or JSON output
- YAML settings for level, path, rotation size, and retained backups
- idempotent reconfiguration
- focused logging tests

No FastAPI startup, MongoDB connection, proxy, or recording functionality is included.
