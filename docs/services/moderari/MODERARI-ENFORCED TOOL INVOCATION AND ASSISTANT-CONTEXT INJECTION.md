# Moderari-Enforced Tool Invocation and Assistant-Context Injection

**Status:** Proposed architectural direction  
**Scope:** Lumen Moderari / Lumen Pontis / external tool providers / Trace / Replay / Assess / Fiducia  
**Motivating example:** MoniGarr Engineering Glyphs (MEG)  
**Principle:** Deterministic orchestration requirements must not depend on probabilistic model tool selection.

---

## 1. Background

Lumen distinguishes the model from the environment around model inference. Models may be provided with tools and may choose to invoke those tools when they determine that they are useful.

This is appropriate where tool use forms part of the model's reasoning and execution process.

However, some external capabilities represent requirements that must be satisfied before inference rather than capabilities that the model may optionally choose to use.

Examples may include:

- semantic interpretation;
- policy or governance lookup;
- configuration resolution;
- trust constraints;
- environment facts;
- session metadata;
- capability declarations;
- security or authority information;
- externally maintained specifications;
- other deterministic contextual requirements.

For these cases, allowing the model to decide whether to invoke the required tool introduces unnecessary probabilistic behaviour.

Lumen therefore distinguishes between:

1. **Model-selected tool invocation**
2. **Moderari-enforced tool invocation**

This distinction is independent of any particular external tool or provider.

---

## 2. Core Architectural Principle

### 2.1 Model-Selected Tools

A model may be informed that tools are available.

The model then determines whether a tool is required and may choose to invoke it.

Tool invocation is therefore probabilistic because the decision forms part of model inference.

Conceptually:

```text
User request
    ↓
Moderari
    ↓
Model
    ↓
Model decides whether a tool is required
    ↓
Pontis
    ↓
Tool provider
```

There is no guarantee that the model will invoke an available tool.

That behaviour is appropriate when use of the tool is discretionary.

### 2.2 Moderari-Enforced Tools

Moderari may determine that an external capability is required before the user request can be presented to the model.

In this case, Moderari invokes the required tool deterministically.

Conceptually:

```text
User request
    ↓
Moderari
    ↓
Moderari determines prerequisite capability is required
    ↓
Pontis
    ↓
Tool provider
    ↓
Tool result
    ↓
Pontis
    ↓
Moderari
    ↓
Assistant-context injection
    ↓
Model inference
```

The tool invocation is therefore **guaranteed by orchestration policy** rather than dependent upon model behaviour.

This provides the fundamental distinction:

> **Model tool calls are available. Moderari tool calls are required.**

---

## 3. Hard Architectural Rules

### Rule 1 — Providers Return Information or Semantics, Not Prompts

An external provider must not construct model prompts on behalf of Moderari.

A provider returns canonical information or semantics in a structured form.

For example:

```json
{
  "type": "doubt_point",
  "meaning": "The adjacent claim is unverified",
  "instructions": [
    "Surface uncertainty",
    "Identify missing evidence",
    "Do not treat the claim as established fact",
    "State what evidence would increase confidence"
  ]
}
```

The provider owns the meaning and authoritative representation of the information.

It does not own how that information is presented to a model.

This is a hard architectural boundary.

---

### Rule 2 — Pontis Mediates All External Capability Calls

Moderari does not communicate directly with external tool providers.

All tool invocation continues through Pontis.

Pontis remains responsible for:

- provider discovery;
- capability discovery;
- routing;
- invocation;
- correlation;
- tool/provider identity;
- result transport;
- capability cataloguing.

This preserves a consistent tool architecture regardless of whether the call originated from the model or from Moderari.

---

### Rule 3 — Moderari Owns Orchestration Policy

Moderari determines whether an external capability is mandatory for the current inference.

If orchestration policy says that a capability is required, Moderari invokes it before inference.

The model is not asked to determine whether the prerequisite should be satisfied.

This provides a deterministic boundary around requirements that cannot safely depend upon model choice.

---

### Rule 4 — Moderari-Initiated Tool Results Enter Through Assistant Context

Relevant information returned from a Moderari-initiated tool call is supplied to the model through a standard assistant-context injection mechanism.

The provider does not construct that assistant context.

Moderari converts the provider's structured result into the standard model-context representation.

Conceptually:

```text
External provider result
        ↓
Moderari generic context adapter
        ↓
Assistant-context material
        ↓
Model
```

This creates a consistent mechanism independent of the provider.

---

### Rule 5 — Tool Responses Are Not Necessarily Injected Verbatim

A provider may return:

- semantic content;
- identifiers;
- provenance;
- version information;
- diagnostic metadata;
- routing information;
- internal fields.

Not all of this necessarily belongs in the model context.

Moderari therefore makes the **relevant semantic or informational result** available through its assistant-context mechanism while retaining other information as execution evidence.

The mechanism remains generic and must not contain provider-specific prompt construction.

---

### Rule 6 — Required Means Required

A Moderari-enforced capability is an execution prerequisite.

Once Moderari determines that the capability is required, the inference must not silently proceed without successful resolution of that requirement.

This distinction is fundamental:

> **A model may choose whether to use an optional capability. Moderari guarantees the use of a required capability.**

---

## 4. Why Assistant Context

A Moderari-enforced tool result represents information established by the orchestration environment for a particular inference.

It is neither:

- part of the user's original request; nor
- necessarily a permanent system-level behavioural rule.

Assistant context therefore provides a natural location for request-specific externally resolved information.

For example:

```text
System:
General Lumen/model operating instructions.

Assistant context:
Tool-derived information required for this request.

User:
Original user request.
```

This avoids continuously adding provider catalogues or definitions to the system prompt when they are not required.

Only information relevant to the current inference consumes context.

The effectiveness of assistant-context placement should remain experimentally testable rather than being assumed. System, assistant and explicit user-context placement can later be compared using the same underlying semantics.

---

## 5. Motivating Example — MEG

MoniGarr Engineering Glyphs provides semantic markers such as:

```text
:doubt_point:
```

A user might write:

```text
The increased latency is being caused by MongoDB. :doubt_point:

Investigate the problem and recommend what we should do.
```

The glyph represents semantic intent, but Moderari should not contain the definition of `:doubt_point:`.

Nor should the complete MEG catalogue need to reside permanently in the model system prompt.

Instead:

```text
User / Rogare
    │
    │ "MongoDB is causing the latency. :doubt_point:"
    ▼
Moderari
    │
    │ Determines semantic resolution is required
    ▼
Pontis
    │
    │ Routes required capability
    ▼
MEG Tool Provider
    │
    │ Returns canonical semantics
    ▼
Pontis
    │
    ▼
Moderari
    │
    │ Converts relevant semantics into
    │ assistant-context information
    ▼
Model
```

The external MEG provider might expose a capability such as:

```text
meg.resolve(":doubt_point:")
```

and return:

```json
{
  "marker": ":doubt_point:",
  "semantic_type": "doubt_point",
  "meaning": "Adjacent claim is unverified",
  "llm_instructions": [
    "Surface uncertainty",
    "Identify missing evidence",
    "Do not treat the claim as established fact",
    "State what evidence would increase confidence"
  ],
  "version": "..."
}
```

Moderari then supplies the relevant semantic information to the model as assistant context.

The MEG provider has returned **semantics, not a prompt**.

---

## 6. MEG Remains External to Lumen

MEG should not become embedded into Moderari.

Lumen should not own:

- the MEG glyph catalogue;
- glyph definitions;
- semantic meanings;
- MEG versioning;
- MEG-specific prompt construction.

MEG remains an external capability provider.

Lumen requires only the generic ability to:

1. recognise that orchestration requires an external capability;
2. invoke that capability through Pontis;
3. receive its result;
4. insert relevant information into model context;
5. record the complete resulting execution evidence.

This ensures that the architecture remains useful independently of MEG.

MEG is an example of the mechanism, not the reason for the mechanism.

---

## 7. Generalisation Beyond MEG

The same mechanism can support many future external requirements.

### Governance

```text
Moderari
    ↓
policy.resolve
    ↓
Pontis
    ↓
External governance provider
    ↓
Applicable policy constraints
    ↓
Pontis
    ↓
Moderari assistant context
```

### Environment

```text
Moderari
    ↓
environment.resolve
    ↓
Pontis
    ↓
Environment provider
    ↓
Authoritative runtime information
    ↓
Pontis
    ↓
Moderari assistant context
```

### Security / Authority

```text
Moderari
    ↓
authority.resolve
    ↓
Pontis
    ↓
Authority provider
    ↓
Permitted boundaries
    ↓
Pontis
    ↓
Moderari assistant context
```

### Configuration

```text
Moderari
    ↓
configuration.resolve
    ↓
Pontis
    ↓
Configuration provider
    ↓
Request-specific settings
    ↓
Pontis
    ↓
Moderari assistant context
```

### Specifications

```text
Moderari
    ↓
specification.resolve
    ↓
Pontis
    ↓
Specification provider
    ↓
Canonical requirement
    ↓
Pontis
    ↓
Moderari assistant context
```

The architectural pattern remains identical.

---

## 8. Pontis Capability Catalogue

This design provides a natural extension to Pontis.

Pontis can evolve from bridging tool providers toward maintaining a catalogue of externally available capabilities.

For example:

```text
Provider: MEG
Capabilities:
  - semantic.resolve
  - semantic.describe
```

```text
Provider: Governance
Capabilities:
  - policy.resolve
```

```text
Provider: Environment
Capabilities:
  - runtime.describe
```

Moderari can request a capability without owning the implementation or provider-specific protocol.

Pontis remains the authoritative bridge between Lumen orchestration and external capability providers.

This also allows the same capability catalogue to support both model-selected and Moderari-enforced tool invocation while preserving the distinction between their origins.

---

## 9. Determinism Boundary

This architecture introduces an important conceptual boundary.

### Probabilistic Decision

```text
Model:
"I think I need this tool."
```

The tool may or may not be invoked.

### Deterministic Orchestration Requirement

```text
Moderari:
"This capability must be resolved before inference."
```

The tool call must occur.

This distinction is important because requirements concerning governance, authority, semantics, configuration or other execution prerequisites should not rely upon a model choosing correctly.

The orchestration layer exists partly to enforce such boundaries.

---

## 10. Failure Semantics

A Moderari-enforced tool invocation is a prerequisite.

Consequently, failure must not silently degrade into normal inference.

Conceptually:

```text
Required capability
    ↓
Tool invocation fails
    ↓
Do NOT silently omit the result
    ↓
Moderari applies explicit failure policy
```

The eventual policy may include:

- fail the request;
- retry;
- surface the unresolved prerequisite;
- use an explicitly configured fallback.

The important rule is:

> **A required capability must never silently become an optional capability because its provider failed.**

Otherwise the guarantee provided by Moderari-enforced invocation would be meaningless.

---

## 11. Evidence and Traceability

Moderari-enforced tool invocation becomes part of the observable execution and therefore part of the evidence captured by Trace.

A trace might record:

```text
tool_invocation_origin = moderari
tool_invocation_mode = required
provider = meg
capability = semantic.resolve
input = :doubt_point:
provider_version = ...
result_hash = ...
context_injection = assistant
```

Model-originated tool activity should similarly identify its origin:

```text
tool_invocation_origin = model
tool_invocation_mode = optional
```

Trace does not interpret these differences.

**Trace records what happened.**

The distinction is captured because it is factual execution evidence that may later be useful to Replay, Assess or other consumers.

---

## 12. Trace, Replay, Assess and Fiducia Responsibilities

The introduction of Moderari-enforced tool invocation must not change the established separation of responsibilities across Lumen's assurance services.

### Trace — Record Everything

Trace records the complete observable execution.

This includes:

- user interaction;
- model interaction;
- Moderari activity;
- model-originated tool calls;
- Moderari-originated tool calls;
- provider identity;
- capability identity;
- tool inputs and results;
- assistant-context injection;
- relevant version and provenance information;
- execution events;
- resulting outcomes.

Trace does not decide whether an event was important.

It records the evidence.

> **Trace records.**

---

### Replay — Reproduce and Compare

Replay retains its existing responsibility.

It replays recorded execution and compares meaningful observable behaviour, including execution that involved Moderari-enforced tool calls.

Replay may identify divergence between the recorded execution and the replayed execution.

It does not determine why that divergence occurred or whether the divergence was significant to the outcome.

Replay should not acquire assessment policy simply because new types of evidence become available.

> **Replay compares.**

---

### Assess — Interpret and Evaluate

Assess is responsible for determining what the recorded and replayed evidence means.

For example, Assess may decide that Moderari-originated tool calls are irrelevant to a particular behavioural comparison and filter them out.

For another assessment, those same calls may be the primary variable under investigation.

Assess may evaluate:

- whether a divergence matters;
- whether a tool-derived semantic instruction changed behaviour;
- whether provider-version differences are relevant;
- whether context placement affected the result;
- whether outcome quality changed;
- whether apparently different execution paths are functionally equivalent;
- whether a behavioural difference should influence assurance.

This filtering and interpretation belongs in Assess rather than Trace or Replay.

> **Assess evaluates.**

---

### Fiducia — Coordinate Assurance Activity

Fiducia coordinates repeated assurance activity.

It may schedule or orchestrate:

- repeated runs;
- Replay activity;
- Assess activity;
- experimental variations;
- evidence gathering over time.

Fiducia does not itself determine why two executions differed.

That interpretation remains the responsibility of Assess.

> **Fiducia coordinates.**

---

## 13. Evidence Preservation Before Interpretation

A central consequence of this separation is that Lumen should preserve evidence before deciding whether that evidence matters.

For example, Trace may record:

```text
Run A:
  tool_invocation_origin = moderari
  provider = meg
  capability = semantic.resolve
  marker = :doubt_point:

Run B:
  tool_invocation_origin = moderari
  provider = meg
  capability = semantic.resolve
  marker = :doubt_point:
```

Assess might choose to exclude those identical calls while comparing subsequent model behaviour.

Alternatively:

```text
Run A:
  provider_version = 1.0

Run B:
  provider_version = 1.1
```

Assess might determine that the provider change is relevant to the behavioural difference.

The evidence remains available in both cases because Trace recorded it without prematurely deciding its significance.

This preserves a fundamental Lumen principle:

> **Capture the observable evidence first. Interpret its significance later.**

---

## 14. Replay Implications

Moderari-enforced tool calls become part of the execution path that Replay observes and reproduces.

Replay should continue to operate according to its existing behavioural replay and divergence model.

The existence of a deterministic Moderari call does not require Replay to understand the semantics of that call.

Replay needs sufficient recorded information to reproduce and compare the observable execution.

Questions such as:

- whether a provider-version change matters;
- whether a semantic change explains divergence;
- whether a tool call should be excluded from an assessment;
- whether two different tool results are functionally equivalent;
- whether a divergence affected outcome quality;

belong to **Assess**.

Replay remains deliberately concerned with reproduction and comparison rather than interpretation.

---

## 15. MEG Experimental Opportunity

MEG provides a useful first demonstration because its behavioural proposition can be compared against explicit natural-language instructions.

### Control — Explicit Natural Language

```text
The increased latency is being caused by MongoDB.

However, this claim has not been verified. Treat it as uncertain,
identify what evidence is missing, do not assume it is true, and
explain what evidence would increase confidence.

Investigate the problem and recommend what we should do.
```

### MEG Semantic Marker

```text
The increased latency is being caused by MongoDB. :doubt_point:

Investigate the problem and recommend what we should do.
```

The MEG version would trigger:

```text
Moderari
    ↓
Pontis
    ↓
MEG provider
    ↓
Semantic resolution
    ↓
Pontis
    ↓
Moderari assistant-context injection
    ↓
Model
```

Repeated executions can then investigate whether the semantic shorthand produces behaviour equivalent to, better than, or different from the explicit natural-language control.

Additional experiments could compare:

- explicit natural language;
- system-prompt definitions;
- assistant-context injection;
- model-selected tool resolution;
- Moderari-enforced tool resolution;
- different context depths;
- different models;
- different semantic markers;
- conflicting natural-language and semantic instructions.

The architecture should not assume that any approach performs better.

Lumen provides the evidence required to determine that experimentally.

Trace records the complete executions.

Replay reproduces and identifies behavioural divergence.

Assess determines which differences matter and evaluates the resulting behaviour and outcomes.

Fiducia can coordinate repeated experiments and assessments.

---

## 16. Architectural Outcome

The important result is not a MEG-specific feature.

It is a generic Lumen capability:

> **Moderari can deterministically invoke externally provided capabilities through Pontis and inject their relevant results into model context before inference.**

This establishes a clear distinction between:

> **Tools the model may use**

and

> **Tools the orchestration layer requires**

MEG provides a compelling initial use case, but the mechanism is deliberately provider-agnostic and can support future semantic, governance, configuration, authority, specification, environmental and other externally supplied capabilities.

The resulting responsibilities remain clear:

```text
External Provider
    Owns authoritative information or semantics
            │
            ▼
Pontis
    Discovers, catalogues, routes and mediates capabilities
            │
            ▼
Moderari
    Determines mandatory orchestration requirements
    and constructs model context
            │
            ▼
Model
    Performs inference and optionally selects its own tools

Trace
    Records the complete observable execution

Replay
    Reproduces and compares execution

Assess
    Interprets evidence and evaluates significance

Fiducia
    Coordinates repeated assurance activity
```

The underlying principle is:

> **Where an external capability is an execution requirement, Lumen should guarantee its use rather than rely upon the model to probably use it.**

And the evidence principle remains:

> **Trace records. Replay compares. Assess evaluates. Fiducia coordinates.**