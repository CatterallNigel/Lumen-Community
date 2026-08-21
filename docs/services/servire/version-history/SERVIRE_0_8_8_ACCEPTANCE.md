# Lumen Servire 0.8.8 — Workspace UI Alignment Acceptance

## Scope

This release aligns the Servire dashboard and component workspace presentation without reducing the usable width of embedded component interfaces.

## Accepted behaviour

- Workspace tabs render the Latin component name on the first line and the English responsibility on the second line.
- Servire and component workspace routes share a 1680 px maximum outer shell.
- Servire operational dashboard content remains centred at a 1200 px maximum width inside the shared shell.
- Embedded component iframes retain the wider workspace and remain full-width within it.
- Existing component navigation, direct-open behaviour, Rogare presentation handling, lifecycle controls, and operational views remain unchanged.

## Validation

- `pytest`: 127 passed.
- Coverage: 95.40%, above the configured 95% requirement.
- `python -m compileall -q src tests`: passed.
- Ruff and mypy were not available in the execution environment and should be run in the normal Servire development virtual environment.
