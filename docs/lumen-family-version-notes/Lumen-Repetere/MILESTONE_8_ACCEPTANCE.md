# Milestone 8 Acceptance — Semantic Conversation Distillation

## Scope

Replay parses immutable OpenAI-compatible `/v1/chat/completions` request bodies and presents
semantic conversation evolution without modifying Trace data or executing a replay.

## Accepted capabilities

- Parse cumulative `messages` arrays from captured request bodies.
- Present system, user, assistant and tool events separately.
- Present assistant `tool_calls`, function names and arguments.
- Present tool results and `tool_call_id` provenance.
- Compare each cumulative request with its predecessor.
- Show only newly appended events as the reasoning-step delta.
- Preserve access to the original raw exchange and message inspectors.
- Tolerate malformed or non-chat payloads without affecting immutable recording loading.

## Explicit exclusions

- No inference or summarisation by a language model.
- No mutation of Trace recordings.
- No reconstruction beyond evidence contained in cumulative request histories.
- No replay-plan editing.
- No target execution or response capture.

## Corrected exchange correlation

The Recording Explorer now correlates requests and responses by immutable `exchange_id`, not by
adjacent sequence position. This is required because long-running streamed model requests can
complete after many unrelated polling exchanges. The UI therefore displays the captured model
response associated with the correct chat request.
