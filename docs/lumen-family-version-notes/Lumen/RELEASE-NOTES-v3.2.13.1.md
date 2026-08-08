# Lumen v3.2.13.1 — Controlled Startup Validation Failure

This maintenance release refines the dependency-validation behaviour introduced in v3.2.13.

## Changes

- Runs startup validation before Uvicorn starts when launched with `python app.py`.
- Exits cleanly with status code `1` after an expected validation failure.
- Avoids the Starlette/FastAPI lifespan traceback for normal fail-fast dependency failures.
- Marks configured-model validation as `SKIPPED` when the model provider is unavailable, rather than reporting a second misleading failure.
- Prints a concise failure summary identifying root failures and dependent skipped checks.

Unexpected programming errors still retain their normal Python tracebacks.
