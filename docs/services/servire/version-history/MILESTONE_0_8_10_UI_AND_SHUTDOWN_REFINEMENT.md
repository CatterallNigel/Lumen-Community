# Lumen Servire 0.8.10 — UI and Shutdown Refinement

## Scope

This release refines the Servire workspace navigation and the graceful managed-stack shutdown path introduced in 0.8.9.

## Workspace navigation

- Component tabs retain the two-line Latin-name / English-role presentation.
- Tab text is centred horizontally within each pill.
- On the Servire operational dashboard, the tab rail is now aligned with the centred 1200px dashboard content rather than the wider 1680px workspace shell.
- Embedded component workspaces retain the full 1680px workspace width and are not compressed.

## Graceful shutdown

A cold model/provider shutdown was observed to reach the exact 180-second Praebere managed-lifecycle timeout. The Praebere lifecycle allowance is therefore increased to 300 seconds in both `config.yml` and `config.example.yml`.

The process termination timeout remains independent; this change only affects how long Servire allows Praebere's managed `/lifecycle/start` and `/lifecycle/stop` operation to complete.

Lifecycle timeout diagnostics now include the configured timeout duration explicitly.

## Static analysis

The quoted local class type annotation in `tests/test_app.py` was corrected for Ruff UP037.
