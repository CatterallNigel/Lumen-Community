# Lumen Pontis Milestone 5.2 Acceptance — Initial OBT Session Suppression

## Architectural rule

Only when the first user message of a previously unknown conversation begins with `\obt`
does Pontis treat the initiating HTTP request as control-plane traffic rather than as the
northbound conversational owner.

Existing sessions are never detached merely because a later user message begins with `\obt`.

## Behaviour

Pontis inspects only OpenAI-compatible `POST /v1/chat/completions` requests. For an initial
unknown `\obt` request it:

1. forwards the request southbound unchanged apart from the existing content-boundary cleanup;
2. does not treat the initiating HTTP caller as the conversational northbound consumer;
3. waits for the normal southbound response to unwind through Replay/Trace;
4. if that response contains a live tool call, lazily creates the configured ACP provider
   session and delivers the tool-call message to it;
5. if the response is ordinary control output or a completed/matched result, returns it to the
   initiating control caller without creating a provider session.

Replay remains unaware of Pontis, Pi and ACP. Trace remains unaware of provider selection.

## Session identity

If the OpenAI-compatible request supplies `session_id` or `metadata.session_id`, Pontis uses it.
Otherwise Pontis creates an internal `pontis-obt-*` identifier for the detached control
initiation. This identifier is owned by Pontis and is not a Replay dependency.

## Target fork flow

```text
Replay UI -> Pontis -> Trace -> Replay (\obt start)
                         -> private Replay execution -> Lumen
Lumen -> Replay (fork, transparent) -> Trace -> Pontis
Pontis -> lazy Pi ACP attachment -> Pi
Pi -> Pontis -> Trace -> Replay (transparent) -> Lumen
```
