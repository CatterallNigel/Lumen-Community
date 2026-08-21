# Lumen v3.1.2

## Corrected session-command boundary

- Intercepts `\obt` commands before MongoDB request persistence.
- Removes prior Lumen command requests and synthetic command responses from model-bound and persisted Pi history.
- Excludes the active Pi/Lumen session from `\obt session list` and from numeric resume selection.

## Corrected restored tool calls

- Converts legacy object/list `tool_calls[].function.arguments` values to JSON strings during save, restore, and final upstream request preparation.
- Preserves existing MongoDB sessions without requiring manual document migration.
- Prevents Ollama OpenAI-compatible API failures such as `cannot unmarshal object ... function.arguments of type string`.

## Validation

- Added command-history filtering, active-session exclusion, and legacy tool-call restore tests.
- Full suite: 104 tests passing.
