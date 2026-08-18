# Lumen v3.1.0

## MongoDB session persistence and restore

- Adds MongoDB-backed `lumen.sessions` persistence using `mongodb://192.168.0.60:27017`.
- Adds `\obt session list`.
- Adds `\obt session resume <number|session_id>` (`continue` is accepted as an alias).
- Restores persisted context in sequence order after the current normal system prompt.
- Stages restore without calling Qwen; the model is contacted only after the user's next non-command instruction.
- Persists the latest Pi-visible conversation snapshot atomically on each request, excluding transport heartbeat/progress events.
- Persists continuation metadata when the hard context boundary is reached.
