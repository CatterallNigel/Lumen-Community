# Lumen Moderari v3.2.14.24
## Architecture and Technical Description

## 1. Purpose

Moderari is Lumen's **model-facing orchestration and reasoning-flow service**.

Its primary responsibility is not simply to proxy requests to a model provider. It takes an OpenAI-compatible conversational request arriving from the Lumen execution path, establishes the effective model context, applies Moderari policy, manages continuity and context pressure, submits a model-ready request to the configured provider, interprets the response, and returns an OpenAI-compatible response to the caller.

Conceptually:

```text
Client / Agent
      │
      ▼
    Pontis
      │
      ▼
  [Lumen path]
      │
      ▼
   Moderari
      │
      ├── Prompt policy
      ├── Context observation
      ├── Continuity / checkpointing
      ├── Session restoration
      ├── Model profile
      ├── Message policy
      ├── Tool-call translation
      ├── Read-workflow recovery
      ├── Operational state
      ├── Evidence / protocol logging
      │
      ▼
OpenAI-compatible provider endpoint
      │
      ▼
     Model
```

Moderari therefore represents the boundary between:

**what the client believes the conversation is**

and

**what the model actually receives.**

That distinction is fundamental to the current architecture.

---

# 2. External architecture

Moderari currently interacts with four principal external components.

```text
                        ┌─────────────────────┐
                        │       Nuntius       │
                        │   Control routing   │
                        └──────────┬──────────┘
                                   │
                    configuration │ control
                                   │
                                   ▼
┌──────────────┐           ┌─────────────────────┐
│    Pontis    │──────────▶│      Moderari       │
│              │           │                     │
│ client/model │◀──────────│ reasoning-flow      │
│ session path │           │ adaptation          │
└──────┬───────┘           └──────┬───────┬──────┘
       ▲                           │       │
       │                           │       │
       │ backchannel              │       │ persistence
       │                           ▼       ▼
       │                    ┌──────────┐ ┌──────────┐
       └────────────────────│ Provider │ │ MongoDB  │
                            │ / Model  │ │          │
                            └──────────┘ └──────────┘
```

There are two distinct paths between Moderari and the rest of Lumen.

### Conversational/execution path

The ordinary OpenAI-compatible request/response path carries the actual model interaction.

### Control/operational path

Moderari separately communicates operational state and control information through Nuntius and Pontis.

This deliberately keeps operational telemetry and Lumen control information out of the model conversation.

---

# 3. Startup architecture

The application entry point is:

```text
lumen_moderari.app:main
```

The executable registered by `pyproject.toml` is:

```text
lumen-moderari
```

Startup proceeds broadly as follows:

```text
main()
  │
  ├── parse command-line arguments
  │
  ├── optionally clear logs
  │
  ├── request runtime provider configuration
  │      │
  │      └── Moderari → Nuntius
  │                    │
  │                    └── \obt servire ollama
  │
  ├── populate settings.ollama_base_url
  │
  ├── run startup validation
  │
  └── start Uvicorn / FastAPI
```

A significant architectural decision is visible here.

Moderari **does not own the Ollama endpoint configuration**.

At startup it sends Nuntius:

```text
\obt servire ollama
```

Nuntius returns the Servire-owned host and port configuration.

Moderari converts that to:

```text
http://<host>:<port>/v1
```

and stores it as its runtime `ollama_base_url`.

Therefore:

```text
Servire
   │
   │ owns provider endpoint configuration
   ▼
Nuntius
   │
   │ communicates configuration
   ▼
Moderari
```

Moderari's own `config.yaml` deliberately does not establish the provider endpoint.

---

# 4. Startup validation

Moderari validates:

```text
Configuration
Filesystem
MongoDB
```

Provider availability and model residency checks exist in the source but are disabled by default.

That reflects the current Lumen authority boundary:

```text
Praebere
    │
    ├── provider availability
    ├── model discovery
    └── model residency

Moderari
    │
    └── uses the provider selected for execution
```

Moderari therefore does not attempt to load or independently validate the configured model merely because Moderari starts.

---

# 5. FastAPI application

The FastAPI application combines several routers:

```text
FastAPI
 │
 ├── control_router
 ├── proxy router
 ├── checkpoint UI/API
 ├── operational UI/API
 ├── system-prompt UI/API
 └── sessions UI/API
```

The primary runtime API is:

```text
/v1/*
```

with special handling for:

```text
POST /v1/chat/completions
```

Other `/v1/*` requests are essentially forwarded to the configured upstream provider.

There is also:

```text
GET /health
```

which reports Moderari health, translator and current upstream endpoint.

---

# 6. The primary chat request path

`POST /v1/chat/completions` is the heart of Moderari.

At a high level:

```text
Incoming OpenAI request
        │
        ▼
Detect Lumen command
        │
        ▼
Remove Lumen command exchanges
        │
        ▼
Prepare chat request
        │
        ├── identify session
        ├── reconnect sparse session
        ├── observe context
        ├── apply existing checkpoint
        ├── apply system-prompt policy
        ├── apply pending session restore
        ├── select model profile
        ├── apply message policy
        ├── normalize tool calls
        └── apply runtime options
        │
        ▼
Persist request snapshot
        │
        ▼
Check context boundary
        │
        ▼
Potentially create model checkpoint
        │
        ▼
Provider / model request
        │
        ▼
Translate model response
        │
        ├── tool calls?
        ├── final answer?
        ├── read continuation?
        └── recovery required?
        │
        ▼
Persist result / checkpoint
        │
        ▼
Return OpenAI-compatible response
```

---

# 7. `prepare_chat_request()`

The principal preprocessing pipeline is in:

```text
chat_pipeline.py
```

and specifically:

```text
prepare_chat_request()
```

The ordering is important.

## 7.1 Client session identification

Moderari first attempts to determine whether the request contains an explicit client session identifier.

This matters particularly for Rogare.

Pi-style clients generally resend conversation history with every request.

Rogare can instead send:

```text
stable session ID
+
current user message
```

Moderari can reconstruct the earlier context from MongoDB.

---

# 8. Sparse-session reconnection

`reconnect_client_session_context()` handles this.

If the incoming request contains only one non-system user message and a known session ID, Moderari retrieves the previous persisted session.

It reconstructs:

```text
previous conversation
+
latest persisted assistant result
+
new user message
```

before continuing.

Thus Moderari supports two client models:

```text
Full-history client

Client ──▶ complete conversation ──▶ Moderari
```

and:

```text
Session-oriented client

Client ──▶ session ID + new turn
                         │
                         ▼
                      MongoDB
                         │
                         ▼
               reconstructed history
```

This is an important abstraction because the downstream model receives an effective conversation regardless of how the client maintains history.

---

# 9. Context observation

The next stage is `context_manager`.

Despite historical naming around "context compaction", ordinary deterministic context compaction is disabled in the supplied configuration:

```yaml
context:
  enabled: false
```

The client history therefore remains intact at this stage.

The context manager still observes the conversation to derive workflow state, including such things as:

```text
session identity
message count
character count
discovered files
attempted reads
successful reads
failed reads
remaining files
successful tool calls
read offsets
```

This information is used later for continuation, recovery and checkpoint decisions.

---

# 10. Model-facing checkpoint application

This is one of the most important pieces of Moderari.

`checkpoint_manager.apply_checkpoint()` can transform the history that is sent to the model.

It does **not** rewrite the client/Pi conversation.

Instead Moderari maintains a distinction:

```text
CLIENT HISTORY
authoritative interaction record

        │
        │ Moderari transformation
        ▼

MODEL-FACING HISTORY
effective reasoning context
```

When a checkpoint exists, old covered model-facing history can be replaced by a continuity checkpoint.

Conceptually:

```text
Original conversation

System
User objective
Assistant
Tool
Assistant
Tool
Assistant
Tool
...
Recent messages
```

becomes:

```text
System
Lumen Continuity Checkpoint
Original objective
Recent messages
```

for subsequent model requests.

The original conversation remains available to Lumen.

This makes checkpointing a **model-context transformation**, rather than destructive conversation summarisation.

---

# 11. System prompt policy

The next stage is `prompt_builder.py`.

Moderari currently supports three meaningful modes.

## Pass-through

The client system prompt is passed to the model unchanged.

```text
Client system prompt
        │
        ▼
      Model
```

## Default/generated

Moderari constructs a deterministic system prompt based upon:

```text
model profile
available tools
behaviour rules
batch workflow rules
```

A Pi system prompt can then be replaced with this generated prompt.

```text
Pi system prompt
       │
       X
       │
Moderari generated prompt
       │
       ▼
     Model
```

## Custom

A researcher-defined prompt previously explicitly applied through Moderari is inserted.

Client system messages are removed and the applied custom prompt becomes authoritative.

This provides controlled experimental prompt conditions.

---

# 12. Effective system-prompt provenance

Moderari does not merely change the system prompt.

When a Vestigare trace correlation is present, Moderari records exactly what happened to it.

The request header is:

```text
x-lumen-trace-exchange-id
```

Moderari records:

```text
incoming system messages
effective system messages
policy mode
policy action
policy strategy
prompt source
hashes
character counts
Moderari request ID
session ID
trace exchange ID
```

It then sends this evidence:

```text
Moderari
   │
   ▼
Nuntius
   │
   ▼
Vestigare
```

The evidence distinguishes the incoming prompt from the prompt actually supplied to the model.

That is particularly important to Replay because the **effective prompt**, rather than merely the original client prompt, is authoritative for reproducing model execution.

---

# 13. Model profiles

Model-specific behaviour is moved out of general deployment configuration into profile YAML.

The supplied profile is:

```text
profiles/qwen2.5-coder.yaml
```

It defines:

```text
context window
checkpoint thresholds
checkpoint token budget
sampling
request timeout
system prompt strategy
read-result policy
behaviour rules
batch workflow behaviour
keep-alive behaviour
```

For Qwen 2.5 Coder, for example:

```text
context window       = 32768 tokens
checkpoint trigger   = 63%
warning               = 75%
soft boundary         = 80%
critical boundary     = 95%
temperature           = 0.1
top_p                 = 0.95
```

This separates:

```text
deployment policy       → config.yaml
model behaviour policy  → profiles/*.yaml
```

---

# 14. Message policy

After prompt processing, `message_policy.py` prepares the messages for the actual model.

Its responsibilities include:

```text
context estimation
read-result limiting
model-profile limits
context warnings
```

Moderari estimates token usage from the effective model-facing request.

That estimate drives:

```text
checkpoint decisions
warnings
hard continuation boundary
operational reporting
```

---

# 15. Runtime options

Profile runtime settings are applied immediately before execution.

For example:

```text
temperature
top_p
keep_alive
```

are added where appropriate.

Moderari then explicitly sets:

```json
"stream": false
```

for the provider request.

This is deliberate.

---

# 16. Why Moderari disables upstream streaming

A client such as Pi can request:

```json
"stream": true
```

but Moderari does **not** pass that streaming mode directly to Qwen.

Instead:

```text
Pi asks for stream=true
        │
        ▼
Moderari
        │
        ├── remembers client requested streaming
        │
        └── sends stream=false to provider
                         │
                         ▼
                       Qwen
```

Moderari needs the complete Qwen response so that it can inspect and translate it.

Once processed, Moderari can construct an OpenAI-compatible SSE stream back to the client.

Thus:

```text
Client-facing streaming ≠ provider-facing streaming
```

---

# 17. Qwen tool-call translation

The supplied translator is:

```text
translators/qwen.py
```

Qwen 2.5 Coder may return tool calls as JSON embedded in ordinary assistant content rather than OpenAI-native `tool_calls`.

Moderari detects formats including:

```text
plain JSON
JSON fences
embedded fenced JSON
JSONL
multiple adjacent JSON objects
```

For example, Qwen might return conceptually:

```json
{
  "name": "read",
  "arguments": {
    "path": "src/app.py"
  }
}
```

Moderari converts this into:

```text
assistant
  content = null
  tool_calls = [...]
  finish_reason = tool_calls
```

using the OpenAI tool-call structure expected by Pi.

Therefore:

```text
Qwen textual tool instruction
          │
          ▼
      Moderari
          │
          ▼
OpenAI native tool_call
          │
          ▼
          Pi
```

This is one of Moderari's original bridge responsibilities and remains central.

---

# 18. Tool execution ownership

Moderari does **not** execute Pi's ordinary tools itself.

Its job is to make the model's requested tool operation understandable to the upstream client/agent environment.

The flow is approximately:

```text
Model
 │
 │ requests tool
 ▼
Moderari
 │
 │ converts to OpenAI tool_call
 ▼
Pi
 │
 │ executes tool
 ▼
Tool
 │
 │ result
 ▼
Pi
 │
 │ next chat request containing result
 ▼
Moderari
 │
 ▼
Model
```

Moderari observes this cycle and uses it to maintain reasoning/workflow state.

---

# 19. Read workflow supervision

Moderari contains substantial special handling for long source-reading operations.

It tracks:

```text
files discovered
files attempted
files successfully read
files failed
remaining files
read offsets
completed read ranges
successful tool signatures
```

It can detect situations where the model prematurely stops during an expected multi-file or continuation workflow.

Recovery mechanisms include:

```text
authoritative read-offset enforcement
successful-tool replay suppression
read-continuation nudges
empty-result continuation
stalled-action nudges
batch completion nudges
final-answer recovery
```

This means Moderari is not simply translating requests.

It is enforcing parts of the **reasoning/workflow contract** expected of the model.

---

# 20. Checkpoint generation

Model checkpointing activates as the effective context approaches the configured threshold.

For the supplied Qwen profile:

```text
trigger = 0.63
```

At this point Moderari can ask the model to produce a distilled representation of its working knowledge.

This checkpoint includes concepts such as:

```text
objective
distilled continuity
source coverage
read progress
retained recent raw messages
generation
covered message count
```

The checkpoint becomes an authoritative model-facing continuity object.

Later requests can therefore use:

```text
previous checkpoint
+
checkpoint delta
+
recent raw context
```

instead of repeatedly presenting the complete historical context.

---

# 21. Critical context boundary

There is a second protection mechanism.

If estimated context reaches the profile's critical ratio:

```text
0.95
```

Moderari does not simply continue sending requests until the model's context fails.

It creates a continuation record and returns a Lumen continuation response.

Conceptually:

```text
Context usage
     │
     ├── <63% ───── normal
     │
     ├── ≥63% ───── checkpointing
     │
     ├── ≥75% ───── warning region
     │
     ├── ≥80% ───── soft boundary
     │
     └── ≥95% ───── continuation required
```

This provides deterministic handling of context exhaustion.

---

# 22. Session persistence

Moderari uses MongoDB extensively.

The principal configured database is:

```text
database: lumen
collection: sessions
checkpoint_collection: checkpoints
```

There is also persistence for final results and saved system prompts.

The session snapshot records the latest complete client-visible history.

The implementation takes advantage of Pi's behaviour of resending conversation history: the latest snapshot can atomically represent the conversation up to that point.

Persisted information includes, depending upon lifecycle stage:

```text
session metadata
model
profile
conversation context
continuation state
checkpoint metadata
result metadata
source/read progress
recovery material
```

---

# 23. Session restoration

Moderari can resume a saved session.

The command:

```text
\obt session resume <number|session_id>
```

does not immediately call the model.

Instead it stages the session for restoration.

The next ordinary user request causes the restored context to be inserted.

Where a modern checkpoint exists, restoration is based primarily upon:

```text
Lumen Continuity Checkpoint
+
original objective
+
retained raw messages
+
completed-but-undistilled read material
+
resume instruction
```

rather than blindly inserting the entire historical conversation.

---

# 24. Lumen commands

Moderari has a local command plane.

Examples include:

```text
\obt help
\obt status
\obt context

\obt prompt default
\obt prompt pass-through
\obt prompt status

\obt session info
\obt session list
\obt session resume <number|session_id>
\obt session continuation
```

A critical property is:

```text
Lumen commands are NOT sent to the model.
```

Moderari detects them before model execution and returns a direct OpenAI-compatible assistant response.

This creates a service-control channel that can coexist with a conversational client without contaminating model context.

---

# 25. Nuntius control interface

Moderari also exposes:

```text
POST /control/obt
```

for Nuntius.

Nuntius sends a structured envelope containing:

```text
request_id
timestamp
session_id
origin
command
```

Commands explicitly addressed to Moderari use:

```text
\obt moderari ...
```

The control handler translates these into the existing internal Moderari command representation.

This establishes the architecture:

```text
User / service
      │
      ▼
   Nuntius
      │
      ▼
 /control/obt
      │
      ▼
   Moderari
```

rather than requiring control operations to pass through the conversational model path.

---

# 26. Pontis session termination

Moderari's control API also accepts a special session-termination event from Pontis.

The origin must be:

```text
pontis
```

This lets Moderari mark an active execution as terminated/incomplete.

Once a session is marked incomplete, late model responses can be discarded rather than accidentally converting a terminated execution into a completed one.

That behaviour is particularly relevant to the package name:

```text
v3.2.14.24-Incomplete-Terminal-State
```

and represents an important lifecycle safeguard.

---

# 27. Pontis backchannel

Long model operations create a separate problem: the ordinary request may be blocked for minutes.

Moderari therefore has a direct operational backchannel to Pontis:

```text
POST /_pontis/backchannel/events
```

Events contain:

```text
channel
source
type
request_id
session_id
elapsed_ms
sequence
message
phase
context_ratio
```

Typical event types are:

```text
heartbeat
progress
```

This path is deliberately separate:

```text
Moderari ─────────────▶ Pontis
        operational
        backchannel
```

It does not travel through Vestigare/Repetere and does not become part of the conversation.

Backchannel failure is best-effort: it is logged but does not fail the active model request.

---

# 28. Operational state

`operational_state.py` maintains per-session execution state.

The lifecycle contains states such as:

```text
READY
PREPARING_MODEL_REQUEST
WAITING_FOR_MODEL
EXECUTING_TOOL
PERSISTING
COMPLETED
FAILED
CANCELLED
INCOMPLETE
```

The state records information including:

```text
session ID
request ID
model
profile
provider
root ask
final answer
current phase
context ratio
tool history
timings
timeline
solution path
errors
```

The architecture has moved toward session-scoped state rather than one global execution state, although a legacy/default process snapshot remains.

---

# 29. Results

A final model answer is treated as an artefact rather than merely transient HTTP content.

Moderari can persist the result and associate it with the session.

Operational state distinguishes:

```text
answer received
```

from:

```text
persistence complete
```

and ultimately:

```text
execution complete
```

This distinction is important because a model producing text does not necessarily mean that the complete Lumen execution lifecycle has successfully finished.

---

# 30. Protocol evidence

Moderari has several logging streams:

```text
moderari.log
interaction.log
ui.log
audit.log
protocol.jsonl
```

`protocol.jsonl` is deliberately isolated from the human-readable logs.

Protocol events include the actual boundaries of execution, for example:

```text
client request received
model-ready upstream request
upstream response
```

with secret redaction available.

This gives Lumen evidence of the difference between what entered Moderari and what Moderari actually sent to the provider.

---

# 31. User/operational interfaces

Moderari exposes several local UI/API surfaces.

### Checkpoints

```text
GET /api/checkpoints
GET /api/checkpoints/latest
GET /checkpoints
```

### Results

```text
GET /api/results
GET /api/results/latest
```

### Operations

```text
GET /api/operations
GET /operations
```

### Sessions

```text
GET /api/sessions
GET /sessions
```

### System prompt

```text
GET    /api/system-prompt
POST   /api/system-prompt/policy
POST   /api/system-prompt/custom/apply

GET    /api/system-prompt/saved
GET    /api/system-prompt/saved/{id}
POST   /api/system-prompt/saved
PUT    /api/system-prompt/saved/{id}
DELETE /api/system-prompt/saved/{id}

GET /system-prompt
```

These are primarily observability, research and operational surfaces around the central model execution pipeline.

---

# 32. Complete input map

Moderari therefore has several distinct categories of input.

## A. OpenAI-compatible HTTP input

Primary:

```text
POST /v1/chat/completions
```

Input contains:

```text
model
messages
tools
stream
sampling parameters
session metadata
client metadata
```

Other `/v1/*` requests are forwarded substantially unchanged.

---

## B. Lumen conversational commands

Embedded in the incoming conversation:

```text
\obt ...
```

These are intercepted and handled locally.

They are not forwarded to the model.

---

## C. Nuntius control commands

```text
POST /control/obt
```

Structured control envelope:

```text
request_id
timestamp
session_id
origin
command
```

---

## D. Servire runtime configuration

Indirectly received through Nuntius during startup:

```text
Ollama host
Ollama port
```

---

## E. MongoDB state

Moderari reads:

```text
sessions
checkpoints
results
saved prompts
restore state
```

---

## F. Configuration

From:

```text
config.yaml
environment variables
profiles/*.yaml
```

---

## G. Trace correlation

From request header:

```text
x-lumen-trace-exchange-id
```

used to associate effective prompt provenance with Vestigare evidence.

---

## H. Tool results

Tool results arrive indirectly as ordinary OpenAI conversation messages from Pi.

Moderari observes them but does not itself execute the Pi tools.

---

# 33. Complete output map

Moderari produces several categories of output.

## A. OpenAI-compatible completion response

Either:

```text
JSON completion
```

or:

```text
SSE streaming completion
```

depending upon the client's original request.

---

## B. OpenAI tool calls

Qwen textual tool requests are transformed into:

```text
assistant.tool_calls[]
```

for Pi.

---

## C. Provider requests

Moderari sends:

```text
POST <provider>/chat/completions
```

with its **model-ready context**, which may differ materially from the incoming client context.

---

## D. MongoDB persistence

Moderari writes:

```text
session snapshots
continuation state
checkpoints
final result artefacts
system-prompt data
```

---

## E. Pontis backchannel events

```text
heartbeat
progress
```

sent directly to Pontis.

---

## F. Nuntius control/evidence messages

Most notably:

```text
effective system-prompt provenance
```

for onward delivery to Vestigare.

---

## G. Logs/evidence

```text
operational logs
interaction logs
UI logs
audit logs
protocol JSONL
```

---

# 34. End-to-end normal interaction

A normal conversational turn can therefore be represented as:

```text
Pi
 │
 │ OpenAI chat request
 ▼
Pontis / Lumen execution path
 │
 ▼
Moderari
 │
 ├─ identify session
 ├─ observe workflow
 ├─ reconnect history if required
 ├─ apply checkpoint
 ├─ establish effective system prompt
 ├─ apply restore state
 ├─ select profile
 ├─ apply model message policy
 ├─ estimate context
 ├─ persist client snapshot
 ├─ record protocol evidence
 │
 ▼
Provider
 │
 ▼
Model
 │
 │ completion
 ▼
Moderari
 │
 ├─ translate Qwen tool syntax
 ├─ inspect completion
 ├─ enforce workflow/recovery rules
 ├─ update operational state
 ├─ checkpoint if required
 ├─ persist result if terminal
 └─ create client-facing response
 │
 ▼
Pontis
 │
 ▼
Pi
```

---

# 35. End-to-end tool interaction

```text
User
 │
 ▼
Pi
 │
 ▼
Moderari
 │
 ▼
Model
 │
 │ textual Qwen tool request
 ▼
Moderari Qwen translator
 │
 │ OpenAI tool_call
 ▼
Pi
 │
 ▼
Tool execution
 │
 │ tool result
 ▼
Pi
 │
 │ next conversation request
 ▼
Moderari
 │
 ├─ observes completed tool
 ├─ updates workflow/read state
 └─ sends effective context
 │
 ▼
Model
```

The loop continues until a terminal assistant response is produced.

---

# 36. End-to-end checkpoint interaction

```text
Conversation grows
       │
       ▼
Moderari context estimator
       │
       ▼
threshold reached
       │
       ▼
Checkpoint request to model
       │
       ▼
Distilled continuity
       │
       ├── stored in memory
       ├── persisted to MongoDB
       └── recorded as evidence
       │
       ▼
Subsequent model request
       │
       ├── system prompt
       ├── continuity checkpoint
       ├── original objective
       └── recent/raw uncovered messages
       │
       ▼
Model continues
```

The client history itself remains intact.

---

# 37. Architectural responsibility summary

The current code divides Moderari into approximately these responsibility areas:

```text
app.py
    Service lifecycle and startup

config.py
    Deployment configuration

runtime_configuration.py
    Servire configuration retrieval through Nuntius

startup_validation.py
    Local dependency validation

proxy.py
    Main execution pipeline

chat_pipeline.py
    Model-request preparation pipeline

context_manager.py
    Deterministic conversation/workflow observation

prompt_builder.py
    Effective system-prompt construction

system_prompt_policy.py
    Prompt policy state

system_prompt_provenance.py
    Effective-prompt evidence

model_profiles.py
    Model-specific behaviour

message_policy.py
    Model-facing message/context policy

translators/qwen.py
    Qwen → OpenAI response translation

checkpoint_manager.py
    Continuity/checkpoint construction and application

checkpoint_persistence.py
    Durable checkpoint retry

checkpoint_observer.py
    Checkpoint observability

continuity.py
    Hard-boundary continuation

session_store.py
    MongoDB session/checkpoint/result persistence and restore

result_observer.py
    Result observability

operational_state.py
    Per-session execution lifecycle

backchannel.py
    Direct operational events to Pontis

command_plane.py
    Moderari-owned \obt commands

control.py
    Nuntius-facing control endpoint

protocol_logging.py
    Machine-readable execution evidence

*_ui.py
    Research/operational UI surfaces
```

---

# 38. The architectural core

The most useful condensed description of Moderari is:

```text
                     CLIENT TRUTH
                         │
                         ▼
                  Incoming history
                         │
                         ▼
                  ┌──────────────┐
                  │   MODERARI   │
                  │              │
                  │ observe      │
                  │ adapt        │
                  │ checkpoint   │
                  │ restore      │
                  │ supervise    │
                  │ translate    │
                  │ evidence     │
                  └──────┬───────┘
                         │
                         ▼
                     MODEL TRUTH
                 Effective request
                         │
                         ▼
                       Model
```

Moderari is therefore best understood as the component that establishes and records **the effective reasoning environment presented to the model** while preserving the distinction between that environment and the original client interaction.

That includes:

- what history the model actually sees;
- which system prompt is actually effective;
- what tools the model believes are available;
- what previous reasoning state has been distilled;
- which source material has actually been read;
- where a long-running task should continue;
- how model-specific output becomes client-compatible output;
- and whether an apparent answer actually represents a completed Lumen execution.

## 39. One implementation issue worth correcting

The supplied `config.yaml` contains a MongoDB URI with credentials directly embedded in the file.

That is operationally significant because this ZIP is a distributable source package.

For the research distribution, the credential should be supplied through deployment/runtime configuration or a secret rather than committed in a packaged configuration file. Even where the credentials are intended only for a controlled environment, keeping them out of the source/package prevents accidental reuse or disclosure.