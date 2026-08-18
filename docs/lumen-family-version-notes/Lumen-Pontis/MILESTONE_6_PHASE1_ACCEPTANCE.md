# Lumen Pontis Milestone 6 — Phase 1 Acceptance

## Purpose

M6 adds the ACP return path required for Rogare-originated tool use while preserving Pontis'
single responsibility as the client/session/protocol bridge.

Phase 1 deliberately establishes observation and extraction before production routing is
changed. Pontis must first prove the exact tool event structure emitted by the installed
`pi-acp` implementation.

## Architectural invariant

For a Rogare-originated turn, the conversational chain remains Rogare → Pontis → Moderari/model.
Pi is a tool provider only.

When the model requires a tool, Pontis will ultimately send the same fork message unchanged to
Pi over ACP, receive the tool result over ACP, and return that result to the Lumen/model path.
Pontis must not reconstruct conversation history, reason about tool output, or substitute Pi's
agent answer for the model's answer.

## Phase 1 implementation

The existing ACP client already stores every `session/update` notification. Phase 1 extends
that existing behaviour by:

- merging `tool_call` and `tool_call_update` events by `toolCallId`;
- exposing merged tool states as `AcpPromptResult.tool_results`;
- retaining `title`, `kind`, `status`, `rawInput`, `rawOutput`, and `content` without semantic
  interpretation;
- preserving first-seen tool-call order;
- adding concise content-safe production logs describing tool-result presence;
- adding an explicit `m6-acp-tool-diagnostic` command that prints raw diagnostic values to the
  operator for controlled validation.

## Diagnostic command

```text
python -m lumen_pontis m6-acp-tool-diagnostic --cwd C:\Development\Lumen-Pontis
```

Optional prompt:

```text
python -m lumen_pontis m6-acp-tool-diagnostic --cwd C:\Development\Lumen-Pontis --prompt "Use the bash tool to list the files in the current workspace."
```

## Required live evidence before Phase 2

Capture the diagnostic output and confirm:

1. Pi emits `tool_call` and/or `tool_call_update` events.
2. A stable `toolCallId` correlates the lifecycle.
3. The requested tool input is visible in `rawInput` and/or equivalent ACP content.
4. The executed tool result is visible in `rawOutput` and/or equivalent ACP content.
5. The terminal tool status emitted by this installed `pi-acp` version is known.
6. Pi may continue to produce agent text, but Pontis can distinguish that text from the tool
   result and therefore ignore it for the Rogare tool-provider return path.

## Phase 2–4 gate

Do not wire the production Rogare tool-result loop against an assumed ACP payload shape.
The live diagnostic output is the input to M6 Phase 2 implementation.

Subsequent M6 work will:

- normalize the confirmed completed-tool result;
- bridge that result back into the original Lumen/model exchange;
- support repeated tool-call cycles in the same turn;
- regression-test existing M5 Replay/lazy-provider behaviour.

## Automated validation

Phase 1 unit tests cover:

- extraction from `tool_call` + `tool_call_update`;
- update merging by `toolCallId`;
- preservation of tool order;
- latest-state semantics;
- malformed/missing tool IDs;
- unrelated ACP updates;
- existing ACP agent-message extraction and M2–M5 behaviours.
