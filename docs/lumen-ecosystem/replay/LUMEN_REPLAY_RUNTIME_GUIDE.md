# Lumen Replay Runtime Guide

Version: 0.9.0  
Applies to: Lumen Trace, Lumen Replay and Lumen runtime development environment

---

## Purpose

This document describes how to run the Lumen stack with Replay in the request path, the Replay-specific `\obt` commands currently available, and how to perform both ordinary and replay runs.

Replay is a transparent proxy by default. Unless it receives an explicit Replay command, it forwards requests and responses unchanged between Trace and Lumen.

---

## Runtime topology

With Replay enabled, the development chain is:

```text
Zed
  ↓
Pi
  ↓
Lumen Trace        http://127.0.0.1:11435
  ↓
Lumen Replay       http://127.0.0.1:11437
  ↓
Lumen              http://127.0.0.1:11436
  ↓
Ollama             http://127.0.0.1:11434
  ↓
Configured model
```

The current port allocation is:

| Service |    Port | Responsibility                                  |
| ------- | ------: | ----------------------------------------------- |
| Ollama  | `11434` | Model provider                                  |
| Trace   | `11435` | Immutable capture of Pi–Lumen traffic           |
| Lumen   | `11436` | Continuity, model orchestration and persistence |
| Replay  | `11437` | Transparent proxy and controlled replay runtime |

Replay's configured target is Lumen:

```yaml
target:
  provider: lumen
  url: http://127.0.0.1:11436
  health_path: /health
  request_timeout_ms: 5000
```

Trace must point to Replay instead of directly to Lumen:

```text
Previous Trace target: http://127.0.0.1:11436
Replay-enabled target: http://127.0.0.1:11437
```

Only the Trace downstream target changes. Pi and Lumen require no routing change.

---

## Startup order

Use this order so each downstream dependency is available before the next service starts.

### 1. Start Ollama

Confirm Ollama is listening on:

```text
http://127.0.0.1:11434
```

Confirm the configured model is installed and available.

### 2. Start Lumen

From the Lumen project directory:

```bat
.venv\Scripts\Activate.bat
python app.py --clear-logs
```

Confirm startup validation reaches `READY` and Uvicorn is listening on:

```text
http://127.0.0.1:11436
```

### 3. Start Replay

From `C:\Development\Lumen-Replay`:

```bat
.venv\Scripts\Activate.bat
lumen-replay
```

Equivalent command:

```bat
python -m lumen_replay
```

Replay loads the committed `config.yml`, verifies MongoDB and Lumen connectivity, and listens on:

```text
http://127.0.0.1:11437
```

Useful Replay URLs:

| URL | Purpose |
|---|---|
| `http://127.0.0.1:11437/` | Replay engineering dashboard |
| `http://127.0.0.1:11437/health` | Replay, MongoDB and target health |
| `http://127.0.0.1:11437/docs` | FastAPI API documentation |
| `http://127.0.0.1:11437/openapi.json` | OpenAPI definition |

### 4. Start Trace

Start Trace after changing its target from Lumen port `11436` to Replay port `11437`.

Confirm Trace is listening on:

```text
http://127.0.0.1:11435
```

### 5. Start Pi

Configure Pi to use Trace as before:

```text
http://127.0.0.1:11435
```

Pi remains unaware that Replay is present behind Trace.

---

## Replay operating states

Replay has three runtime states.

### Transparent

Default state.

```text
Pi → Trace → Replay → Lumen
```

Replay forwards traffic unchanged and performs no replay comparison.

### Comparing

Entered only after a prepared Replay session is started explicitly.

Replay privately executes the recorded path against Lumen and compares meaningful model events with the original recording.

Nominal matching traffic is not returned to Pi or recorded again by Trace.

### Passthrough

Entered when Replay reaches the first Fork Point.

The divergent response is returned through Trace to Pi. Replay then forwards subsequent traffic transparently so Pi can provide tools and the changed conversation can continue normally.

---

## Available Replay `\obt` commands

The current Replay command parser recognises the following commands in the latest user message.

### Show Replay help

```text
\obt replay
```

Returns usage guidance, including the Replay UI address.

### Start a prepared replay

```text
\obt replay start <replay-id>
```

Example:

```text
\obt replay start replay-2af76d8c...
```

The `<replay-id>` is created when a Trace recording is prepared in the Replay UI or through the preparation API.

Possible outcomes include:

- replay starts successfully;
- replay ID does not exist;
- prepared session is not ready;
- another replay is already active;
- Lumen or Replay storage is unavailable.

### Show runtime status

```text
\obt replay status
```

When idle, Replay returns its current state and reports that no run is active.

During a run, it reports:

- runtime state;
- run ID;
- run status;
- number of matched steps.

### Stop the active replay

```text
\obt replay stop
```

Cancels the active Replay run and returns Replay to transparent mode.

If no replay is active, Replay reports that it is already transparent.

---

## Commands not consumed by Replay

Replay consumes only exact commands beginning with:

```text
\obt replay
```

All other requests and commands are forwarded unchanged to Lumen.

This means existing Lumen `\obt` commands continue to operate normally unless they are specifically part of the Replay command namespace.

---

## Preparing a Replay session

A replay must be prepared before it can be started.

### Using the UI

1. Open:

   ```text
   http://127.0.0.1:11437/
   ```

2. Locate a completed Trace recording.
3. Select **View** to inspect the recording and Replay Plan.
4. Enter an optional experiment name.
5. Select **Prepare Replay session**.
6. Copy the generated `replay_id` from the **Prepared Replay sessions** table.

Preparation creates Replay-owned identifiers and persists the session in MongoDB. It does not send anything to Lumen.

### Using the API

```http
POST /recordings/{recording_id}/prepare
Content-Type: application/json
```

Example body:

```json
{
  "experiment_name": "Simply Test Math baseline"
}
```

The response includes the `replay_id` required by:

```text
\obt replay start <replay-id>
```

---

## Performing a normal non-replay run

A normal run requires no Replay command.

### Procedure

1. Start Ollama, Lumen, Replay, Trace and Pi.
2. Confirm Replay health:

   ```text
   http://127.0.0.1:11437/health
   ```

3. Confirm Replay is idle:

   ```text
   \obt replay status
   ```

   Expected state:

   ```text
   transparent
   ```

4. Submit an ordinary ask through Pi, for example:

   ```text
   Calculate 12 * 9 / 3
   ```

5. Replay forwards the request unchanged:

   ```text
   Pi → Trace → Replay → Lumen → Ollama → Model
   ```

6. Responses return through the same chain.
7. Trace records the interaction normally.
8. Pi continues to provide any requested tools normally.

### Expected behaviour

Replay does not:

- compare the conversation;
- suppress responses;
- create a Replay run;
- alter model requests;
- alter tool calls;
- persist Fork Point evidence.

It behaves only as a transparent proxy.

---

## Performing a replay run

### Before starting

1. Ensure the source Trace recording is complete.
2. Inspect the recording in the Replay UI.
3. Confirm the Replay Plan contains the intended conversational path.
4. Prepare a Replay session.
5. Copy its `replay_id`.
6. Confirm no other replay is active:

   ```text
   \obt replay status
   ```

### Start the run

From Pi, submit:

```text
\obt replay start <replay-id>
```

Replay consumes this command; it is not forwarded to Lumen.

### While the recorded path matches

Replay:

1. loads the immutable source recording;
2. creates an isolated run and session identity;
3. privately submits the recorded chat request to Lumen;
4. receives and parses the model response;
5. compares the response with the expected recorded event;
6. ignores generated tool-call identifiers when comparing;
7. compares tool names and canonical tool arguments;
8. supplies the next recorded cumulative request when the event matches;
9. increments the matched-step count.

During this nominal phase:

```text
Replay ↔ Lumen
```

Pi does nothing, and Trace receives no duplicate nominal conversation.

### Fully matching result

If the entire meaningful path and final answer match the source recording:

```text
status: matched
fork_point: none
```

Replay saves the result and returns to transparent mode.

No duplicate conversation is sent to Pi or Trace.

### Divergent result

A Fork Point is reached when the live model produces the first meaningful event that differs from the recorded path.

Examples include:

- different assistant content;
- different tool name;
- different tool arguments;
- extra tool call;
- omitted tool call;
- different finish reason;
- different final answer.

Replay records objective evidence:

```text
last matching step
first divergent step
expected event
observed event
run ID
Replay session ID
source recording ID
timestamps
```

Replay then returns the divergent response through Trace to Pi and enters passthrough mode:

```text
Lumen → Replay → Trace → Pi
```

From that point:

- Pi provides any required tools;
- Trace records the changed path;
- Replay forwards traffic unchanged;
- Lumen continues the same conversation;
- the run completes or fails normally.

Replay records the Fork Point but does not judge whether the changed path is better or worse. That responsibility belongs to Lumen Assess.

---

## Monitoring a replay

Use:

```text
\obt replay status
```

The Replay dashboard is available at:

```text
http://127.0.0.1:11437/
```

Replay run evidence is stored in MongoDB collection:

```text
replay_runs
```

Prepared sessions are stored in:

```text
replay_sessions
```

Trace source data remains immutable in:

```text
trace_recordings
trace_messages
```

---

## Stopping or recovering

To cancel an active replay:

```text
\obt replay stop
```

After cancellation, verify:

```text
\obt replay status
```

Expected state:

```text
transparent
```

If Replay itself is unavailable, Trace must currently be manually pointed back to Lumen on port `11436`. Automatic routing and a Trace UI routing selector are deferred until the later product-polish stage.

---

## Current architectural boundaries

### Trace

Captures everything that transpires between Pi and its downstream target.

### Replay

- transparent proxy by default;
- consumes explicit Replay commands;
- privately reproduces the recorded path;
- records matched steps;
- identifies the first Fork Point;
- becomes transparent after divergence;
- persists run evidence;
- performs no behavioural assessment.

### Lumen

- manages model interaction;
- maintains continuity;
- persists checkpoints and answers;
- communicates with Ollama and the model.

### Pi

- remains the tool-capable agent;
- is inactive during a nominal private replay;
- resumes when Replay passes through a divergent event.

### Assess

Will later combine Trace, Replay and Lumen evidence to evaluate answer quality, path stability, tool efficiency and model capability.

---

## Quality gate

Before running an experiment after code changes:

```bat
pytest
ruff check .
mypy src tests
```

The Milestone 9 baseline passed:

```text
117 tests
95.16% coverage
Ruff clean
mypy clean
```
