# Lumen v3.2.13 — Dependency Validation Framework

Lumen now validates its required operating environment before accepting work.

## Added

- Structured startup validation results with `READY`, `DEGRADED`, and `FAILED` states.
- Configuration validation.
- Filesystem readability and write-access probes.
- MongoDB connectivity and read/write round-trip validation.
- OpenAI-compatible model-provider validation via `/models`.
- Configured-model availability and optional live completion probe.
- Fail-fast startup when a required dependency is unavailable.
- Structured validation report stored on `app.state.startup_validation` for the future operational UI.

## Configuration

The `startup_validation` section controls startup checks. The live model probe is enabled by default and can be disabled for lightweight development startup with `probe_model: false`.
