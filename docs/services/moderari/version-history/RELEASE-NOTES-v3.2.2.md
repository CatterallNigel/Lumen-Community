# Lumen v3.2.2 — Checkpoint Reliability Fixes

- Restores the Qwen checkpoint trigger to 63% of configured context.
- Makes generation-one checkpoint prompts explicitly create initial Distilled Continuity when no previous continuity exists.
- Validates checkpoint output and retries once when Qwen returns an empty, undersized, or meta-only response.
- Scrubs Lumen/OBT warnings and visible Distilled Continuity wrappers from model-facing assistant history.
- Detects control-only final completions during unfinished reads and injects the exact deterministic continuation read instead of ending the workflow.
- Preserves the v3.2.1 dynamic token budget, immutable checkpoint persistence, resume reconstruction, and rotating operational logs.

The previously planned split operational/payload file logging is deferred to v3.2.3.
