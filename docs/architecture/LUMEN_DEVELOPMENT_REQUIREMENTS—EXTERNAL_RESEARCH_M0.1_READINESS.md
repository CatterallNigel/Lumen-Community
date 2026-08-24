# Lumen Development Requirements — External Research M0.1 Readiness

**Date:** 2026-08-21  
**Status:** Proposed Development Requirements  
**Components:** Lumen Repetere, Lumen Fiducia, Lumen Moderari, Lumen Praebere, Lumen Rogare, Lumen Servire, Lumen Pontis, Lumen Vestigare  
**Origin:** External Research Distribution M0.1 planning

---

## 1. Purpose

Planning for the Lumen External Research Distribution M0.1 has exposed several requirements in the existing Lumen implementation that should be addressed independently of the research distribution itself.

They concern:

1. The representation of repeated replay executions as an **Experiment**, including clear visibility of replay divergence.
2. Making **Moderari system-prompt handling configurable** rather than always replacing an incoming system prompt.
3. Removing hard-coded model assumptions from **Praebere** and **Moderari**.
4. Making Praebere authoritative for provider, protocol, endpoint and model discovery.
5. Providing model selection through **Rogare** when Rogare is the active console.
6. Providing provider/model discovery to external consoles through the existing explicit `\obt` mechanism.

These are changes to Lumen proper.

The External Research Distribution provides the immediate motivation, but the requirements should not be implemented as research-distribution-specific behaviour.

---

# 2. Repetere — Experiment Structure and Divergence Visibility

## 2.1 Background

Repetere currently allows a recorded Trace to be replayed and records the resulting execution through Vestigare.

Repeated replay executions are therefore related to the original Trace, but that relationship is not presently represented clearly enough in the UI.

Replay-created Traces appear substantially as independent Traces rather than as executions belonging to a controlled experiment.

In addition, although Repetere detects divergence during replay, the resulting Traces are not currently presented with a clear matched/divergent status.

This becomes particularly important when Fiducia is used to execute multiple repetitions of the same recorded Trace.

---

## 2.2 Experiment as a First-Class Concept

A recorded Trace selected as the basis for controlled repeated execution should become the **source Trace of an Experiment**.

The original Trace remains unchanged.

An Experiment references that Trace and groups the replay executions generated from it.

Conceptually:

```text
Experiment E-001
Source Trace: T-143

├── Run 001 — Trace T-151 — MATCHED
├── Run 002 — Trace T-152 — MATCHED
├── Run 003 — Trace T-153 — DIVERGED
├── Run 004 — Trace T-154 — MATCHED
└── Run 005 — Trace T-155 — DIVERGED
```

The replay-created Traces remain normal Vestigare Traces.

The Experiment provides the relationship between:

- the source observation;
- the controlled repetitions;
- the resulting evidence.

The Trace data model should not be distorted simply to create a hierarchical UI representation.

---

## 2.3 Experimental Model

This formalises the existing Lumen research principle:

> A single execution is an observation. Repeated controlled executions become an experiment. Multiple experiments begin to characterise behaviour. Longitudinal evidence allows us to detect behavioural change.

The Experiment therefore becomes the natural unit connecting Repetere, Fiducia and, eventually, Aestimare.

---

## 2.4 Fiducia Relationship

Fiducia should operate against the Experiment rather than treating repeated Repetere executions as unrelated replay requests.

A researcher should be able to:

1. Select a source Trace.
2. Create an Experiment.
3. Specify the required number of repetitions.
4. Have Fiducia coordinate those repetitions.
5. Have Repetere execute each replay.
6. Have Vestigare capture each resulting execution.
7. Review all resulting runs as members of the Experiment.

This keeps responsibilities separated:

**Fiducia** coordinates repeated execution.

**Repetere** reproduces the recorded execution and detects replay divergence.

**Vestigare** records what actually occurred.

The Experiment associates the evidence.

---

## 2.5 Repetere Divergence Visibility

Repetere already identifies divergence during replay.

That information should be surfaced explicitly for every Experiment run.

At minimum, the UI should distinguish:

```text
MATCHED
DIVERGED
FAILED / INCOMPLETE
```

Where existing Repetere evidence permits, a divergent run should also expose the first detected divergence point.

For example:

```text
Run 003
Status: DIVERGED
First divergence: Interaction 7
Trace: T-153
```

This information should be visible without requiring the researcher to manually inspect and compare every resulting Trace.

---

## 2.6 Repetere Is Not Performing Assessment

The distinction between **divergence detection** and **behavioural assessment** must remain explicit.

Repetere may state:

> This replay diverged from the recorded execution.

Repetere should not state:

> This divergence is significant.

or:

> This behaviour is unacceptable.

Those are assessment questions.

Repetere reports an observed property of the reproduction process.

Aestimare may eventually assess the significance, characteristics or implications of accumulated behavioural evidence.

This distinction is particularly important because Aestimare is not part of the External Research Distribution.

---

## 2.7 UI Representation

The preferred UI representation should make the experimental relationship immediately apparent.

For example:

```text
Trace T-143
Original execution

└── Experiment E-001
    ├── Run 001  MATCHED
    ├── Run 002  MATCHED
    ├── Run 003  DIVERGED
    ├── Run 004  MATCHED
    └── Run 005  DIVERGED
```

This does not imply that the child Traces are physically embedded within the source Trace.

It is a representation of their experimental relationship.

---

# 3. Moderari — Configurable System Prompt Policy

## 3.1 Background

Moderari currently owns the system prompt supplied to the model.

An incoming client system message is removed and replaced with the Moderari system prompt.

This behaviour is appropriate for normal Lumen operation, but becomes problematic for controlled behavioural research.

A system prompt can materially affect model behaviour.

Consequently, silently replacing a researcher-supplied system prompt changes the experimental conditions.

System-prompt handling should therefore become an explicit Moderari configuration policy.

---

## 3.2 Required Modes

Moderari should support three system-prompt modes.

### Pass-through

Moderari does not replace the incoming system prompt.

The client-provided context is passed through with its system prompt unchanged.

If the client supplies no system prompt, Moderari does not create one.

```text
Client System Prompt
        │
        ▼
     Moderari
        │
        │ unchanged
        ▼
       Model
```

### Moderari Default

This preserves the existing Lumen behaviour.

Any incoming client system prompt is removed and Moderari injects its standard system prompt.

```text
Client System Prompt ──► removed

Moderari Default System Prompt
             │
             ▼
            Model
```

This should remain the normal/default operating mode unless deliberately changed.

### Custom

Any incoming client system prompt is removed.

Moderari instead injects a user-defined system prompt configured through Servire.

```text
Client System Prompt ──► removed

Servire
   │
   └── Custom System Prompt
             │
             ▼
          Moderari
             │
             ▼
            Model
```

This allows a researcher to establish a controlled system-prompt condition without requiring the external client to manage it.

---

# 4. Servire — Moderari Configuration

The Moderari configuration UI in Servire should expose the system-prompt policy.

Conceptually:

```text
Moderari

System Prompt Policy

○ Pass-through
● Moderari Default
○ Custom

Custom System Prompt
┌─────────────────────────────────────────┐
│                                         │
│                                         │
└─────────────────────────────────────────┘
```

The custom prompt editor should only be enabled when `Custom` is selected.

The selected policy should form part of Moderari's runtime configuration.

---

# 5. Vestigare — System Prompt Trace Behaviour

No special modification to Vestigare is required merely to capture the effective system prompt.

Vestigare records the execution context.

Therefore, whichever system prompt actually passes through Moderari and reaches the model should naturally form part of the recorded Trace.

This preserves the existing architectural responsibility:

> **Trace records what actually happened.**

Vestigare should not need to understand the semantics of Moderari's system-prompt configuration merely to duplicate configuration information.

The selected Moderari policy may be retained as execution/configuration provenance where appropriate, but the effective system prompt itself should not be redundantly recorded merely because the policy has become configurable.

---

# 6. Praebere — Dynamic Provider and Model Management

## 6.1 Background

Praebere currently contains assumptions tied to the development environment, including a configured/hard-coded Qwen model.

This is suitable for the current development stack but is not appropriate for an external research distribution or for Lumen's longer-term provider-independent architecture.

Praebere should become the authoritative Lumen service for:

- model provider configuration;
- provider endpoint configuration;
- provider protocol;
- provider availability;
- discovery of available models;
- selected/default model;
- model availability;
- model lifecycle where supported by the provider.

Neither Moderari nor a Lumen client should contain hard-coded knowledge of the model currently installed.

---

## 6.2 Initial Provider Scope

The initial supported provider for the External Research Distribution M0.1 should remain:

**Ollama**

The architecture should nevertheless avoid encoding Ollama-specific assumptions outside Praebere.

Additional providers can subsequently be implemented as Praebere adapters.

An immediate candidate is:

**LM Studio**

Both Ollama-native and OpenAI-compatible protocols are relevant to external research environments.

---

## 6.3 Provider Configuration

A Praebere provider configuration should describe at least:

```text
provider
endpoint
protocol
```

Conceptually:

```text
provider: ollama
endpoint: <configured Ollama endpoint>
protocol: ollama-native
```

A future OpenAI-compatible provider could similarly be represented as:

```text
provider: lmstudio
endpoint: <configured LM Studio endpoint>
protocol: openai-compatible
```

Provider endpoints must not be hard-coded into Moderari.

Praebere owns the relationship between a provider, its endpoint and the protocol required to communicate with it.

---

## 6.4 Model Discovery

Praebere should query the configured provider for the models currently available through that provider.

The available model list should therefore reflect the actual runtime environment rather than a model list maintained within Lumen configuration.

Conceptually:

```text
Praebere
    │
    ├── Provider
    ├── Endpoint
    ├── Protocol
    │
    └── Available Models
            ├── Model A
            ├── Model B
            └── Model C
```

This removes the current assumption that Lumen operates against a particular Qwen model.

---

# 7. Rogare — Model Selection

When Rogare is being used as the Lumen console, it is the natural place for the researcher or user to select the model used for an interaction.

Rogare should obtain the available model list from Praebere.

Conceptually:

```text
Provider: Ollama

Model:
┌─────────────────────────────────────────┐
│ qwen2.5-coder:14b-32k                ▼ │
└─────────────────────────────────────────┘
```

Rogare does not discover models itself.

It presents model information supplied by Praebere.

This functionality is specific to Rogare as a Lumen-controlled console. External consoles communicating through Pontis must not depend upon Rogare for model discovery.

---

# 8. External Console — Provider and Model Discovery

An external console may communicate with Lumen through Pontis without using Rogare.

The external console therefore requires a mechanism for discovering the model environment without using the Rogare UI.

For the initial implementation, Praebere should make provider/model information available through the existing explicit `\obt` command mechanism.

For example:

```text
\obt models
```

may return:

```text
Provider: Ollama

Available models:
- qwen2.5-coder:14b-32k
- llama3.1:8b
- mistral:7b
```

Related operational queries may subsequently include:

```text
\obt provider
\obt models
\obt model
```

The precise command vocabulary can be refined during implementation.

The important requirement is that an external console capable of communicating through Pontis can discover the Lumen model environment without requiring a separate Praebere UI.

---

# 9. Moderari — Dynamic Model Resolution

## 9.1 Background

Moderari currently also contains a configured/hard-coded Qwen model assumption.

This should be removed.

Moderari should not independently maintain model identity when Praebere is responsible for the provider/model environment.

Praebere becomes the authoritative source for the active provider and model.

---

## 9.2 Praebere-to-Moderari Communication

Praebere should communicate the active provider/model state to Moderari through the established Lumen communication path.

This can use the explicit `\obt` operational-message mechanism through Pontis.

Conceptually:

```text
Praebere
    │
    │ \obt
    │ provider: ollama
    │ model: qwen2.5-coder:14b-32k
    │ protocol: ollama-native
    │ endpoint: ...
    │
    ▼
Pontis
    │
    ▼
Moderari
```

The exact message representation should follow existing `\obt` conventions rather than introducing an unrelated configuration channel.

---

## 9.3 Information Required

As provider support expands, model identity alone may not be sufficient.

The operational information made available by Praebere should be capable of representing:

```text
provider
model
protocol
endpoint
```

Moderari should consume the information required to conduct the interaction without becoming responsible for discovering or managing the provider itself.

This allows provider-specific behaviour to remain isolated behind Praebere.

---

# 10. Provider and Model Responsibility Boundary

The intended responsibility boundary is:

```text
Praebere
│
├── Provider configuration
├── Provider endpoint
├── Provider protocol
├── Provider availability
├── Model discovery
├── Model availability
├── Selected/default model
└── Model lifecycle where supported
        │
        ▼
    Moderari
        │
        ├── consumes active model information
        ├── controls model interaction
        └── applies system-prompt policy
```

Rogare and external clients consume this capability rather than becoming authorities over provider state.

---

# 11. Research Significance

## 11.1 System Prompt

Making system-prompt handling configurable turns an existing hidden environmental condition into an explicit experimental variable.

A researcher can deliberately execute equivalent experiments under:

```text
Experiment A — Client system prompt passed through
Experiment B — Moderari default system prompt
Experiment C — Controlled custom system prompt
```

The resulting Traces contain the actual contexts presented to the model.

This allows behavioural differences associated with system-prompt conditions to be observed and reproduced rather than inadvertently introduced by Lumen itself.

---

## 11.2 Provider, Runtime and Protocol

Provider/runtime selection is also capable of influencing observed model behaviour.

The same nominal model may behave differently when:

- served through different inference runtimes;
- configured differently;
- exposed through different API protocols;
- provided with different tool environments;
- given different system prompts.

Consequently, provider, model and effective execution context are relevant experimental conditions.

Lumen should make these conditions observable rather than silently embedding development-environment assumptions.

Together, the research environment may involve controlled variation across:

```text
Model
Runtime / Provider
Protocol
System Prompt
User/Input Representation
Tool Environment
Repeated Execution
```

These variables do not necessarily need to be interpreted by Lumen.

The immediate requirement is that they can be controlled, reproduced and observed.

---

# 12. External Research Distribution M0.1 Relevance

The proposed External Research Distribution includes:

- Rogare
- Pontis
- Vestigare
- Repetere
- Fiducia
- Moderari
- Praebere
- Servire
- MongoDB

The distribution is intended to be Dockerised and operate over a private Lumen Docker network.

Existing external model providers and research tooling may remain outside the Docker environment.

Conceptually:

```text
Researcher's Host
│
├── Existing Model Provider
├── Existing Models
├── Python / Jupyter / Research Tools
└── External Console / Tool Environment
        │
        ▼
Docker — Lumen Research Distribution
        │
        ├── Rogare
        ├── Pontis
        ├── Vestigare
        ├── Repetere
        ├── Fiducia
        ├── Moderari
        ├── Praebere
        ├── Servire
        └── MongoDB
```

Praebere connects the Lumen environment to the configured external model provider.

External clients may communicate through Pontis.

Rogare provides the Lumen-native console when an external console is not required.

Aestimare is explicitly **not included** in the External Research Distribution.

Consequently, researchers must be able to inspect experimental structure and objective divergence evidence without relying upon Aestimare to interpret the results.

---

# 13. Consolidated Development Requirements

## 13.1 Repetere / Fiducia

- Introduce **Experiment** as an explicit concept.
- Associate an Experiment with a source Trace.
- Associate replay-created Traces with their Experiment.
- Treat individual replay executions as Experiment runs.
- Allow Fiducia to coordinate multiple runs belonging to an Experiment.
- Display Experiment runs hierarchically in the UI.
- Surface `MATCHED`, `DIVERGED`, and `FAILED / INCOMPLETE` status.
- Surface the first divergence point where available.
- Keep divergence detection separate from behavioural assessment.

## 13.2 Moderari — System Prompt

- Replace mandatory system-prompt replacement with a configurable policy.
- Support `Pass-through`.
- Support `Moderari Default`.
- Support `Custom`.
- Preserve current Moderari behaviour as the default mode.
- Allow the custom system prompt to be configured through Servire.
- Ensure Pass-through genuinely preserves the client-provided system context.
- Allow Vestigare to continue recording the resulting effective context normally.

## 13.3 Praebere — Provider and Model

- Remove the hard-coded Qwen model assumption.
- Make provider configuration explicit.
- Associate provider with endpoint and protocol.
- Initially support Ollama as the reference provider.
- Discover available models dynamically from the provider.
- Maintain the selected/default model.
- Make provider/model information available through `\obt`.
- Supply model choices to Rogare.
- Communicate active provider/model state to Moderari.
- Keep the design open to additional provider adapters and OpenAI-compatible providers.

## 13.4 Rogare

- Obtain available models from Praebere.
- Provide model selection when Rogare is the active console.
- Do not independently discover or manage provider models.

## 13.5 Moderari — Model Resolution

- Remove Moderari's hard-coded Qwen model assumption.
- Make Praebere authoritative for provider/model state.
- Receive active provider/model information from Praebere.
- Use the existing Lumen/Pontis operational path for this communication.
- Do not duplicate provider discovery or model management within Moderari.

## 13.6 Servire

- Expose Moderari system-prompt policy configuration.
- Provide the custom system-prompt editor.
- Continue to provide operational control over the Lumen stack.

## 13.7 Vestigare

- Continue recording the actual execution context.
- Capture the effective system prompt naturally as part of that context.
- Continue recording the resulting executions of Experiment runs.
- Avoid taking responsibility for interpreting experimental conditions.

---

# 14. Architectural Principle

The requirements in this document establish three related controls over experimental conditions.

### Experiment Structure

Repetere and Fiducia establish which executions are controlled repetitions of a source observation.

### System Context

Moderari explicitly controls whether the system prompt is passed through, supplied by Lumen, or deliberately defined by the researcher.

### Model Environment

Praebere explicitly establishes which provider, protocol and model form the inference environment.

Vestigare records what actually occurs through that environment.

Together these support the broader Lumen principle:

> **Experimental conditions should be explicit, reproducible and observable. Lumen should record what happened without unnecessarily altering the conditions being investigated.**

Aestimare, when present in a commercial Lumen distribution, may subsequently assess the behavioural evidence produced under those conditions, but assessment remains separate from observation, reproduction and experimental control.