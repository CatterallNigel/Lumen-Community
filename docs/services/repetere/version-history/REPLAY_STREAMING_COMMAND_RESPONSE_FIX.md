# Replay streaming command response correction

Replay now preserves the OpenAI-compatible response mode requested by Pi when
Replay consumes a local `\\obt replay` command.

For requests with `"stream": true`, Replay emits Server-Sent Events containing:

1. a `chat.completion.chunk` content event;
2. a terminal chunk with `finish_reason: "stop"`;
3. the terminal `data: [DONE]` marker.

For non-streaming requests Replay continues to return a normal
`chat.completion` JSON response.

The implementation follows the response contract used by Lumen v3.2.14.3 while
remaining an independent Replay implementation. Transparent proxied traffic is
unchanged.
