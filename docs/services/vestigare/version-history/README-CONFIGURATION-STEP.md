# Lumen Trace — Milestone 1 Configuration Step

Copy the contents of this package into the root of the existing `Lumen-Trace` repository, allowing the files to overwrite their placeholders.

Changed files:

- `pyproject.toml`
- `src/lumen_trace/config.py`
- `tests/test_config.py`

Then run from the activated Python 3.12 virtual environment:

```powershell
python -m pip install -e ".[dev]"
pytest tests/test_config.py -v
ruff check .
mypy
```

This increment implements configuration loading only. It does not yet configure logging, connect to MongoDB, initialise FastAPI, or expose the health endpoint.
