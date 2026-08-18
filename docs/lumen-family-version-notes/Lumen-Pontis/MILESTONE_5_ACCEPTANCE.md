# Lumen Pontis Milestone 5 Acceptance — Lazy Provider Session Attachment

## Architectural invariant

Pontis owns the relationship between downstream Lumen sessions and northbound providers.
When traffic requires a provider and no provider session is attached, Pontis creates the
provider session lazily, binds it to the existing Pontis/Lumen session, and thereafter reuses
that binding.

The current provider is Pi and the current provider-session protocol is ACP. Neither Replay
nor Rogare should contain Pi/ACP session-creation logic.

## Provider delivery API

```text
POST /_pontis/sessions/{session_id}/deliver
```

For the first delivery to an unknown session:

```json
{
  "message": "northbound message",
  "cwd": "C:/Development/project"
}
```

Pontis creates the Pi ACP session, binds it, and delivers the message. The response identifies
whether a provider session was created. Subsequent deliveries may omit `cwd` because the
binding is reused.

## Provider content boundary

For `POST /v1/chat/completions`, Pontis removes only recognised Lumen operational assistant
text that Pi has retained in conversation history:

- the Lumen `\obt` informational banner;
- `[OBT progress ...]` operational progress chunks.

If such text is attached to an assistant tool-call message, Pontis preserves the tool call and
sets assistant content to null. Genuine conversational assistant content is unchanged.

## Normal stack routing

The supplied operational configuration uses:

```text
Pontis :11435 -> Trace :11438 -> Replay :11437 -> Lumen :11436
```

Standalone development can still point the configurable downstream elsewhere.

## Replay integration boundary

A browser-started Replay is private and therefore does not naturally have an originating Pi
HTTP request waiting north of Pontis. On first divergence Replay must stop comparing, become
transparent, and hand the fork-causing northbound message to Pontis' provider-delivery
operation. Pontis — not Replay — creates or reuses the provider session.

Forking ends comparison; it never ends traffic.
