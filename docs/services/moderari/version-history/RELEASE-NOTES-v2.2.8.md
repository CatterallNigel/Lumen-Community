# OBT v2.2.8

## Rolling summary archive

Each successful model-authored summary is now preserved as its own model-facing system message. Later summary operations process only newly retired raw chunks; they no longer re-summarize previous summaries. The active context therefore contains the original objective, an ordered archive of distinct summaries, and the newest raw read chunks.

## Per-model summary timeout

`model_checkpointing.timeout_seconds` controls the dedicated rolling-summary request timeout. The Qwen 32K profile defaults to 3600 seconds. `model_checkpointing.max_output_tokens` is also profile-configurable and defaults to 2000.

## Empty completion recovery

When Qwen returns an empty completion during an unfinished large-file read, OBT extracts the exact next read offset from the latest model-facing large-read continuation notice and performs one retry with a temporary exact read instruction. The retry instruction is not retained in Pi history.

## Prompt cleanup and observability

The obsolete normal-system-prompt explanation of working-memory checkpointing has been removed. Summary logging now includes source offsets, archived summary count, configured timeout, output-token limit, and summary elapsed time.

## Validation

88 tests passing.
