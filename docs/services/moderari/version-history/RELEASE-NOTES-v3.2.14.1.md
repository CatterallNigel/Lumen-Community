# Lumen v3.2.14.1 – Protocol Evidence Logging

This maintenance release adds a dedicated JSON Lines protocol trace for model-behaviour and interoperability investigations.

## Added

- `logs/protocol.jsonl`, containing one structured JSON object per event.
- Full client request, prepared upstream request, every upstream request attempt, raw upstream response, translated response, replay-guard recovery request, cancellation, and correlation metadata.
- Exact replay-guard injected message and resulting outbound payload.
- Correlation through session ID, request ID, gate attempt and mode.
- Recursive secret redaction for authorization headers, API keys, tokens, cookies, passwords and credentials embedded in connection URIs.
- Independent rotation using the existing log size and backup settings.

## Configuration

```yaml
logging:
  protocol_enabled: true
  protocol_file: logs/protocol.jsonl
  protocol_redact_secrets: true
```

The JSONL protocol stream is intentionally separate from the human-readable operational logs and does not propagate into `lumen.log`.
