# Replay command capture corrective fix

This corrective change keeps Replay transparent for normal traffic while ensuring
supported `\obt replay` commands are intercepted before they reach Lumen.

## Behaviour

Replay now recognises supported commands in:

- OpenAI-compatible `messages` payloads with string content
- structured message content parts
- OpenAI Responses-style `input` payloads
- single `prompt` payloads
- plain-text request bodies

Supported commands are consumed by Replay:

- `\obt replay`
- `\obt replay help`
- `\obt replay start <replay-id>`
- `\obt replay status`
- `\obt replay stop`

Unknown Replay subcommands are not consumed. They continue through the transparent
proxy to Lumen, preserving command ownership boundaries.

## Regression coverage added

- structured content command capture
- Responses-style input command capture
- prompt command capture
- unknown Replay subcommand pass-through
- proxy-level proof that recognised commands are not forwarded
