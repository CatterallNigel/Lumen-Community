# Lumen Replay Runtime Architecture

Version: 1.0  
Status: Architectural baseline  
Applies to: Lumen Replay (Repetere)

## Product boundary

> Trace captures what happened. Replay reproduces what mattered. Assess determines what it means.

Trace remains an immutable recorder of all traffic between Pi and Lumen. Replay derives the
meaningful model conversation, repeats the original opportunity to solve the task and records the
first Fork Point. Assess later combines Trace, Replay and Lumen evidence to evaluate quality,
stability, tool use and model capability.

Replay does not score answers, qualify reasoning or decide whether a divergent path is better.

## Runtime topology

Normal deployment requires one routing change in Trace:

```text
Pi -> Trace -> Replay -> Lumen -> model provider
```

Replay is a transparent proxy by default. Requests and responses are forwarded unchanged until an
explicit `\obt replay` command is received.

## Runtime states

```text
TRANSPARENT
    |
    | \obt replay start <replay-id>
    v
COMPARING
    |                      |
    | complete match       | first behavioural difference
    v                      v
TRANSPARENT            PASSTHROUGH
                           |
                           | final answer / stop / cancel
                           v
                       TRANSPARENT
```

### Transparent

Replay forwards unmatched traffic to Lumen without recording, classifying or comparing it.

### Comparing

Replay loads a prepared source recording and privately submits each recorded model request to
Lumen. Model responses are canonicalised into objective evidence containing assistant content,
tool names, tool arguments and finish reasons.

While the response matches the source response, Replay may use the next recorded cumulative request.
That request already contains the original matching tool result, so Pi is not required while the
model remains on the nominal path.

Generated tool-call identifiers are ignored during matching. Tool names and canonical JSON
arguments are compared because they describe the behavioural action.

### Fork Point

The first different model response is the Fork Point. Replay records:

- run identifier;
- source recording and prepared replay identifiers;
- number of matching model steps;
- first divergent step;
- expected response summary;
- observed response summary;
- detection timestamp.

The observed divergent response is returned unchanged through Trace to Pi. Replay then becomes a
transparent pass-through. Pi supplies tools and Lumen continues the same live conversation.

### Match completion

When all recorded model responses match, Replay records a matched run and returns a concise command
result to Pi. The already-recorded nominal conversation is not emitted again through Trace.

## Explicit commands

Replay consumes these commands rather than forwarding them to Lumen:

```text
\obt replay
\obt replay start <replay-id>
\obt replay status
\obt replay stop
```

Prepared sessions continue to be created through the Replay engineering UI or REST API.

## Persistence

Replay-owned runtime evidence is stored in `replay_runs`. Trace-owned collections remain read-only.

## Known boundary

Replay currently buffers upstream responses so it can compare complete OpenAI-compatible JSON or
SSE evidence before deciding whether to suppress a nominal response or expose a Fork Point. A later
runtime-hardening milestone may introduce bounded disk-backed buffering for very large streams.
