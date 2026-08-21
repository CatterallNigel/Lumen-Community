# Lumen Pontis Milestone 6 Acceptance — ACP Tool Result Bridge

## Purpose

M6 completes the ACP tool-result bridge required by Rogare while preserving the Lumen
conversational authority boundary:

```text
client -> Pontis -> Lumen stack -> Moderari/model
                              <- tool_call
                 Pontis -> Pi over ACP
                        <- tool result over ACP
client <- Pontis <- Lumen stack <- Moderari/model
```

Pi remains a tool provider. Pontis never substitutes Pi agent prose for the model answer.

## Operational integration

For OpenAI-compatible `POST /v1/chat/completions` traffic Pontis now observes the downstream
response for structured OpenAI tool calls. When a tool call is returned:

1. Pontis preserves the tool name and argument string as emitted by the model.
2. The fork message delivered to Pi is `tool_name(<original arguments>)`.
3. Pontis lazily creates or reuses the Pi ACP session through the existing M5 session service.
4. ACP `tool_call` / `tool_call_update` events are merged by `toolCallId` using the M6 Phase 1
   extraction code.
5. Only completed ACP tool output is accepted for the model continuation. Pi agent text is
   ignored.
6. Pontis appends the current assistant tool-call message plus the matching OpenAI `role=tool`
   result to the existing client messages and sends the continuation back down the configured
   Lumen path.
7. The cycle repeats if the model requests another tool, up to the bounded safety limit.
8. The final Moderari/model answer is returned to the originating client.

Pontis does not reconstruct earlier conversation history. It only carries forward the request
messages already supplied by the client and appends the current tool exchange.

## Session correlation

Pontis prefers an existing logical session identifier from these request/response headers:

- `x-lumen-session-id`
- `x-lumen-replay-session-id`
- `x-session-id`
- `session-id`

It also accepts `session_id` / `sessionId` from a JSON request. If no identifier is present, an
HTTP-turn-local Pontis session identifier is generated.

ACP working-directory correlation prefers:

- `x-lumen-cwd`
- `x-working-directory`
- `x-cwd`
- JSON request field `cwd`

If no working directory is supplied, Pontis uses its process working directory. This preserves
M5 lazy attachment without requiring Replay or Rogare to create ACP sessions themselves.

## Failure semantics

Pontis never fabricates a tool result. If ACP produces no completed tool output, Pontis returns
the original model tool-call response unchanged and records the condition in the operational
log. This keeps evidence visible and avoids substituting Pi's agent answer.

A single HTTP turn is bounded to eight provider tool cycles to prevent an unbounded bridge
loop caused by a misbehaving model/provider combination.

## M6 diagnostics

The Phase 1 diagnostics remain available:

```text
python -m lumen_pontis m6-acp-tool-diagnostic --cwd C:\Development\Lumen-Pontis
```

Normal operational ACP traffic uses the same extended `session/update`, `tool_call`, and
`tool_call_update` logging path.

## Automated acceptance

M6 tests cover:

- JSON tool-call extraction;
- streaming OpenAI/SSE tool-call reassembly;
- exact fork-message argument preservation;
- ACP completed-tool-output selection;
- exclusion of Pi agent prose from model continuation;
- continuation payload construction with original model tool-call IDs;
- transparent preservation when ACP provides no completed tool result;
- existing M2-M5 lifecycle/session/proxy behaviour.

Run:

```text
pytest
pytest --cov=lumen_pontis --cov-report=term-missing
ruff check .
mypy src tests
```

## M6 corrective integration — provider HTTP re-entry

Operational Replay testing exposed a recursive bridge condition after a fork. While Pontis was
waiting for Pi to handle an ACP prompt, Pi legitimately re-entered Pontis over the OpenAI-compatible
HTTP path to obtain the model response. Pontis was treating that provider-internal request as a new
northbound tool-bearing conversation, creating another ACP session and feeding the response back
through the bridge again.

The corrected invariant is:

- a normal northbound chat completion may activate the ACP tool bridge;
- while an ACP provider prompt is active, HTTP chat-completion traffic is provider re-entry and is
  forwarded transparently to the downstream Lumen stack;
- the returned model tool call is returned to the already-active Pi request unchanged so Pi can
  execute it;
- the completed ACP tool result returns to the original Pontis bridge invocation;
- Pontis then continues the original model conversation with the tool result;
- the existing logical Lumen-session to ACP-session binding remains authoritative and is reused by
  subsequent tool cycles in that conversation.

This correction does not weaken the M6 tool-result validation, does not move tool execution into
Pontis, and does not change Replay, Vestigare, or Moderari responsibilities.

## M6 corrective integration — provider HTTP re-entry

Operational Replay testing exposed a recursive bridge condition after a fork. While Pontis was
waiting for Pi to handle an ACP prompt, Pi legitimately re-entered Pontis over the OpenAI-compatible
HTTP path to obtain the model response. Pontis was treating that provider-internal request as a new
northbound tool-bearing conversation, creating another ACP session and feeding the response back
through the bridge again.

The corrected invariant is:

- a normal northbound chat completion may activate the ACP tool bridge;
- while an ACP provider prompt is active, HTTP chat-completion traffic is provider re-entry and is
  forwarded transparently to the downstream Lumen stack;
- the returned model tool call is returned to the already-active Pi request unchanged so Pi can
  execute it;
- the completed ACP tool result returns to the original Pontis bridge invocation;
- Pontis then continues the original model conversation with the tool result;
- the existing logical Lumen-session to ACP-session binding remains authoritative and is reused by
  subsequent tool cycles in that conversation.

This correction does not weaken the M6 tool-result validation, does not move tool execution into
Pontis, and does not change Replay, Vestigare, or Moderari responsibilities.
