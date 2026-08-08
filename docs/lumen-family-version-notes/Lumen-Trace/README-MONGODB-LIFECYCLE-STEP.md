# Milestone 1 — MongoDB and FastAPI Lifecycle

This increment adds:

- `DatabaseManager` with explicit connect/disconnect ownership
- MongoDB startup verification using `ping`
- FastAPI lifespan as the application composition root
- shared configuration and database services on `app.state`
- `GET /health`
- isolated database and lifecycle tests

## Apply

Extract this archive into the repository root and allow the included files to overwrite placeholders.

## Verify

```powershell
pytest -v
ruff check .
mypy
```

## Run against the configured MongoDB

```powershell
python -m uvicorn lumen_trace.main:app --host 127.0.0.1 --port 11435
```

Then request:

```text
http://127.0.0.1:11435/health
```

Expected response:

```json
{"status":"ok","mongodb":"connected"}
```
