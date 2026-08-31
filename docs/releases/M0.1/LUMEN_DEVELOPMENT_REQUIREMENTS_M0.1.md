# Lumen Development Requirements — External Research Distribution M0.1

**Date:** 2026-08-22  
**Status:** Current Development Requirements  
**Components:** Repetere, Fiducia, Moderari, Servire, Pontis, Praebere, Nuntius, Vestigare, Rogare  
**Origin:** External Research Distribution M0.1 planning

---

## Document Revision History

| Date | By | Version | Description |
| --- | --- | --- | --- |
| 2026-08-23 | Nigel Catterall | 1.0 | First reviewed release |
| 2026-08-30 | Nigel Catterall | 1.1 | Defined Pontis session identity as authoritative for Vestigare Trace binding; added active-session selection, no-session Trace-start prevention, pre-first-interaction recording boundary, and multi-client Trace acceptance requirements. |
| 2026-08-30 | Nigel Catterall | 1.2 | Defined N9 Praebere runtime model lifecycle: Ollama startup discovery, optional preferred model, established versus active-execution sessions, external-client explicit selection, Rogare dropdown state, and model locking during active execution. |


## 1. Purpose

Planning for the Lumen External Research Distribution M0.1 has exposed a set of changes and validation requirements in Lumen proper.

The original requirements concerned:

1. representing repeated replay executions as an explicit **Experiment**, including visible replay divergence; and
2. making **Moderari system-prompt handling configurable** rather than always replacing an incoming system prompt.

Subsequent M0.1 design discussion has added four further requirements:

3. saved and reusable custom system prompts;
4. validation that the Lumen stack correctly isolates multiple simultaneous sessions;
5. a common `\obt` control-command path through **Lumen Nuntius**, required as shared control-plane infrastructure for Replay, Moderari, Praebere and other Lumen services; and
6. replay fidelity for the effective system prompt recorded in the source Trace.

Nuntius is now considered foundational M0.1 infrastructure rather than a later integration convenience. It should be implemented early because several of the remaining M0.1 changes consume `\obt` control commands.

These are Lumen engineering requirements motivated by the External Research Distribution. They should not be implemented as research-distribution-specific behaviour.

Several related ideas have deliberately been kept outside M0.1:

- Rogare human correctness/quality ratings;
- multiple Pontis tool providers and a unified tool catalogue;
- bounded-computational-resource behavioural research;
- Servire single-active-operator enforcement for general research distribution.

The last item is a **pre-general-research-release requirement**, but it is not required to define M0.1 itself.

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

Aestimare will eventually assess the significance, characteristics or implications of accumulated behavioural evidence.

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

# 2.8 Repetere — Effective System Prompt Replay Fidelity

A replay must reproduce the **effective system prompt recorded in the source Trace**, not whatever Moderari happens to be configured to use at replay time.

This is required because the system prompt is part of the execution context and can materially affect model behaviour.

For M0.1, the source Trace is expected to contain the actual system prompt that reached the model. The existing MongoDB research data has been cleared, so compatibility with historical Traces that lack this evidence is not an M0.1 requirement.

Before implementation, a fresh execution should be recorded and inspected to confirm exactly where and how the effective system prompt is represented in the current Trace structure.

## 2.8.1 Replay Behaviour

Before Repetere begins replaying a source Trace, it must instruct Moderari to use **Pass-through** for the replay session.

This is a **blocking prerequisite** for Replay.

Repetere must not begin sending the recorded execution context until Moderari has positively acknowledged that the session-scoped Pass-through change has been applied.

Conceptually:

```text
Source Trace
    |
    | contains effective system prompt
    v
Repetere
    |
    | \obt Moderari: Pass-through
    v
Nuntius
    |
    v
Moderari
    |
    +--> 200 OK
    |       Pass-through applied
    |       for this replay session
    |
    +--> 204 / 4xx / 5xx / timeout
            Pass-through not established
```

Only after `200 OK` does Repetere replay the recorded execution context, including the original effective system prompt.

The replay must not resolve the source Trace back to the **current** Moderari Default prompt or the **current** contents of a saved Custom prompt. The Trace is the authoritative evidence of what was actually supplied to the original execution.

If the required Pass-through state cannot be positively established, Repetere must abort before any model interaction from that replay begins.

For this prerequisite:

```text
200
    Proceed with Replay.

204
    Abort. The expected command owner did not handle the command.

4xx
    Abort. The requested configuration change was rejected.

5xx
    Abort. The required service failed while applying the change.

timeout / unreachable
    Abort. The prerequisite state cannot be confirmed.
```

A failed prerequisite should result in the corresponding Experiment run being recorded as `FAILED / INCOMPLETE`, with the control-plane failure retained as replay/operational evidence.

## 2.8.2 Session Scope

The Pass-through instruction used by Replay must be scoped to the replay session.

It must not change Moderari's global configuration or alter the system-prompt policy of another active session.

This requirement is part of the wider M0.1 concurrent-session isolation requirement.

## 2.8.3 Replay Execution Session Isolation

Each Replay execution must run in a **new, isolated replay session**.

This is required independently of whether the current implementation already behaves this way. Before changing Repetere, M0.1 development must inspect and test the existing Replay/session behaviour to establish the current baseline.

For repeated executions of one source Trace:

```text
Experiment E-001
Source Trace T-001

Run 001 -> Replay Session R-001 -> Trace T-101
Run 002 -> Replay Session R-002 -> Trace T-102
Run 003 -> Replay Session R-003 -> Trace T-103
```

The implementation must not allow a later Replay to inherit conversational context, temporary configuration, tool/session state, or model interaction state produced by an earlier Replay.

The required rule is:

> **Every Replay run within an Experiment executes in a newly created, isolated session whose initial execution context is derived solely from the source Trace and the explicitly defined experimental conditions. No state produced by another Replay run may enter that session.**

The session lifecycle should conceptually be:

```text
create new Replay session
        |
        v
establish session-scoped prerequisites
        |
        | e.g. Moderari Pass-through
        v
receive positive acknowledgement
        |
        v
load/reproduce source Trace context
        |
        v
execute Replay
        |
        v
Vestigare records new Trace
        |
        v
close Replay session
```

The existing session-scoped Moderari Pass-through requirement therefore applies to the newly created Replay session, not merely to the client or operator session that initiated the Replay.

A Replay session must not reuse the preceding Replay session simply because both runs belong to the same Experiment.

This isolation requirement is necessary for controlled repetition: otherwise the execution conditions of later runs may be polluted by evidence produced by earlier runs.

## 2.8.4 Trace Validation and System-Prompt Provenance

M0.1 development must first establish exactly what Vestigare records before any remediation is implemented.

The investigation must compare fresh Traces produced under:

1. current behaviour;
2. Moderari `Pass-through`;
3. Moderari `Custom`, using a deliberately distinct custom system prompt.

The comparison must determine whether Vestigare records:

- the incoming client system prompt;
- the effective system prompt after Moderari processing;
- both;
- or an ambiguous representation that could cause Repetere to reconstruct more than one active system prompt.

Where Moderari replaces an incoming system prompt, the Trace must preserve both facts as provenance:

```text
incoming system prompt
    client-supplied prompt
    replay status: superseded

effective system prompt
    Moderari Default or Custom prompt
    replay status: active
```

The incoming prompt must not be hard-deleted or destructively overwritten. It remains evidence of what the client supplied.

However, from Repetere's point of view, a superseded incoming system prompt is a **soft-deleted replay element**: it remains visible in the Trace but must not be reconstructed as active model context.

The required invariant is:

> **Replay reconstructs exactly one effective system prompt: the system prompt that actually reached the model during the source execution. Superseded incoming system prompts remain provenance but are excluded from replay execution.**

If Vestigare already captures this distinction correctly, no remediation is required.

If it does not, Vestigare must be changed before Replay system-prompt fidelity can be considered complete. A candidate remediation is for Moderari, whenever it replaces the incoming system prompt, to send a correlated `\obt` control message through Nuntius identifying the effective system prompt. Vestigare can then mark the incoming prompt as superseded for Replay and identify the injected prompt as the active effective system prompt.

The `\obt` message itself must remain control-plane traffic and must not become a conversational Trace entry.

---

# 2.9 Trace Model/Provider Provenance and Replay Binding Investigation

Before M0.1 defines any requirement to bind Replay to the provider/model used by a source Trace, or deliberately permits provider/model substitution, the current Vestigare and Repetere behaviour must be established.

The investigation must determine:

- whether Vestigare records the provider used for the source execution;
- whether Vestigare records the model used for the source execution;
- where and in what form provider/model identity is stored;
- whether that information is descriptive provenance only or forms part of the replayable execution context;
- whether Repetere currently reads or acts upon recorded provider/model identity;
- whether provider/model identity appears indirectly in system, assistant or other conversational messages;
- whether an unchanged source Trace can currently be replayed against a different selected provider/model; and
- whether changing provider/model introduces hidden or model-specific context into the reconstructed execution.

A representative investigation should:

1. execute a fresh interaction against a known provider/model;
2. inspect the raw Vestigare Trace and identify all provider/model-related evidence;
3. determine what Repetere reconstructs from that Trace;
4. select a different provider/model through the normal M0.1 control path;
5. attempt Replay of the same source Trace without otherwise changing its recorded conversational context; and
6. inspect both what is sent to the second model and the resulting Vestigare Trace.

The investigation must distinguish:

> **Provider/model provenance** — evidence describing the environment in which the source execution occurred.

from:

> **Provider/model Replay binding** — information that actively causes or requires Repetere to reproduce the source execution using that provider/model.

These are not assumed to be the same thing.

No M0.1 implementation should introduce a provider/model Replay override merely because comparative replay may be desirable. The existing behaviour must first be understood. The result may show that provider/model substitution already occurs naturally and that only provenance, UI clarity or explicit experimental-condition representation is required.

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

Conceptually:

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

---

### Moderari Default

This preserves the existing Lumen behaviour.

Any incoming client system prompt is removed and Moderari injects its standard system prompt.

Conceptually:

```text
Client System Prompt ──► removed

Moderari Default System Prompt
             │
             ▼
            Model
```

This should remain the normal/default operating mode unless deliberately changed.

---

### Custom

Any incoming client system prompt is removed.

Moderari instead injects a user-defined system prompt configured through Servire.

Conceptually:

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


#### Custom Prompt Integrity and Tooling

`Custom` is an **exact researcher-defined experimental condition**.

Moderari currently performs tool-related system-prompt processing as part of its normal behaviour: tool information may be extracted from the incoming client system prompt and incorporated into the system prompt Moderari presents to the model. That behaviour may remain part of `Moderari Default`, but it must not be applied to `Custom`.

When `Custom` is selected, Moderari must:

- remove the incoming client system prompt;
- inject the last explicitly Applied contents of the Custom System Prompt editor exactly as applied;
- not extract tool instructions or descriptions from the removed client prompt and append them to the Custom prompt;
- not inject Moderari-generated tool guidance into the Custom prompt;
- not merge, rewrite, normalise or otherwise augment the Custom prompt.

Tool availability and tool instructions in the system prompt are separate concerns. A tool provider may remain available to the session, but that does not permit Moderari to modify the Custom prompt to advertise or describe those tools.

If a researcher wants tool descriptions or tool-use instructions in a Custom system prompt, those instructions must be explicitly included in the saved or entered Custom prompt. If they are omitted, Moderari must not add them automatically.

> **Custom means the exact researcher-defined system prompt. Tooling information is included in that prompt only when the researcher explicitly puts it there.**

---

# 4. Moderari System Prompt Policy UI and Servire Exposure

The System Prompt Policy is owned by **Moderari**.

Moderari provides the policy selection, Custom prompt editor, saved-prompt operations and active runtime state. Servire does not mediate these operations or send a separate control command to Moderari when the user selects `Apply`.

Servire exposes the Moderari System Prompt Policy UI within the Moderari service tab as part of the operational interface. Whether the Moderari UI is viewed directly or through Servire does not change the ownership or execution path.

Conceptually:

```text
Moderari — System Prompt Policy

Policy

○ Pass-through
● Moderari Default
○ Custom

Saved Prompt
┌─────────────────────────────────────────┐
│ Select saved prompt...                  │
└─────────────────────────────────────────┘

Custom System Prompt
┌─────────────────────────────────────────┐
│                                         │
│                                         │
└─────────────────────────────────────────┘

Editor State: Unchanged / Modified

[ Apply ]  [ Save As… ]  [ Update Saved ]  [ Delete Saved ]

Active Moderari Configuration

Policy: Custom
Applied Prompt: Research Baseline / Current Editor Contents
Status: Applied
```

The saved-prompt selector and Custom prompt editor should only be enabled when `Custom` is selected.

## 4.1 Working Copy and Explicit Apply

Selecting a saved prompt loads its contents into the Custom System Prompt editor as a **working copy**.

The user may use that working copy unchanged or edit it.

Editing the working copy must not:

- alter the stored saved prompt;
- alter Moderari's active runtime configuration;
- implicitly apply the edited contents.

The contents of the editor have no runtime effect until the user explicitly selects `Apply`.

`Apply` makes the complete current contents of the editor Moderari's active Custom system prompt.

The applied prompt is used **verbatim**, subject only to unavoidable transport/serialization representation. Moderari must not add, remove, rewrite, merge, normalise, augment or inject tooling information into it.

The required rule is:

> **Moderari is bound to its last explicitly Applied system-prompt configuration, not to the mutable contents of the Custom System Prompt editor.**

Because the System Prompt Policy UI belongs to Moderari, `Apply` is a local Moderari configuration action. It does not require a Servire → Nuntius → Moderari request/response exchange.

The UI must make the distinction between editor state and active runtime state explicit. For example, after loading `Research Baseline` and modifying the editor without applying it:

```text
Saved Prompt: Research Baseline
Editor State: Modified

Active Moderari Configuration
Policy: Custom
Applied Prompt: Research Baseline
Status: Applied
```

The researcher can therefore see that the editor contains unapplied changes.

After `Apply`, the active configuration must truthfully identify the current editor contents as the applied condition rather than implying that the unchanged saved object is active when the editor has diverged from it.

## 4.2 Saved-Prompt Operations

Persistence and runtime application are separate operations.

The M0.1 UI should provide explicit actions:

```text
Apply
    Make the current editor contents the active Moderari Custom prompt.
    Does not alter the saved-prompt catalogue.

Save As…
    Create a new named saved prompt from the current editor contents.
    Does not overwrite the prompt from which the working copy may have been derived.
    Does not automatically Apply the new saved prompt.

Update Saved
    Explicitly replace the selected saved prompt with the current editor contents.
    Does not automatically alter the active Moderari configuration.

Delete Saved
    Remove the selected saved prompt from the reusable catalogue.
    Does not alter an already-loaded editor working copy.
    Does not alter the active Moderari configuration.
```

There is deliberately no implicit overwrite operation.

A normal `Save` action must not infer that the researcher intends to replace the saved prompt from which the editor contents were derived. Where persistence of edited contents is required, the researcher must explicitly choose either `Save As…` or `Update Saved`.

The conceptual lifecycle is:

```text
Saved Prompt
     |
     | select
     v
Working Copy in Editor
     |
     +-- unchanged --+
     |               |
     +-- edited -----+
                     |
                     +--> Apply
                     |      runtime configuration only
                     |
                     +--> Save As…
                     |      new saved object
                     |
                     +--> Update Saved
                            explicit replacement of selected saved object
```

Deleting or updating a saved prompt must not affect historical Traces that used it. The actual effective system prompt recorded in those Traces remains the authoritative evidence of what was supplied to the model.

## 4.3 Saved-Prompt Persistence

Saved Custom system prompts require persistent storage.

M0.1 should introduce a dedicated MongoDB collection for Moderari saved system prompts.

The exact implementation schema may follow existing Lumen persistence conventions, but the stored object must support at least:

```text
prompt_id
name
content
owner_id
scope
created_at
updated_at
```

`prompt_id` must provide stable object identity independently of the human-readable name.

For M0.1, `owner_id` may identify the installation/default research user because general multi-user operation is not part of the release. The field must nevertheless exist so saved prompts are not modelled as permanently unowned global objects.

`scope` should likewise exist in the M0.1 schema. Its M0.1 semantics may remain deliberately simple, for example a user/default scope, while leaving room for future ownership and organisational policy.

Saved-prompt names should not be required to be globally unique. Where uniqueness is enforced, it should be scoped appropriately to ownership/scope rather than preventing different future users or organisations from using the same human-readable prompt name.

The saved-prompt document must not contain an `is_active` flag merely to represent Moderari runtime state.

Saved-prompt persistence and active Moderari configuration are separate concepts:

```text
Saved Prompt Collection
    reusable prompt definitions

Moderari Runtime State
    last explicitly Applied policy/prompt
```

`Save As…`, `Update Saved` and `Delete Saved` mutate the saved-prompt collection.

`Apply` changes Moderari runtime state and does not mutate the saved-prompt collection.

The selected policy and the last successfully Applied Custom prompt should form part of Moderari's runtime configuration.

## 4.4 Future Ownership and Organisational Policy

M0.1 does **not** define the full semantics of prompt ownership, sharing or organisational policy.

The explicit `owner_id` and `scope` fields are future-compatibility hooks, not an invitation to implement multi-user prompt governance in M0.1.

Future development must separately consider, among other possibilities:

- user-private Custom prompts;
- prompts shared within an organisation;
- installation-wide prompts;
- centrally managed organisational prompts;
- prompts an organisation may require users to use;
- permissions to create, view, update, delete, share, mandate or override prompts.

These questions belong in `development/future` and are explicitly outside the M0.1 scope.

---

# 5. Trace Behaviour

Vestigare must preserve sufficient evidence to distinguish **incoming execution context** from the **effective execution context presented to the model**.

This distinction is required because Moderari may replace an incoming client system prompt.

For example:

```text
Client supplies:
    system = USER-A

Moderari policy:
    Custom

Moderari replaces USER-A with:
    CUSTOM-X

Model receives:
    system = CUSTOM-X
```

Both `USER-A` and `CUSTOM-X` are relevant evidence, but they do not have the same Replay meaning.

Vestigare must preserve the architectural principle:

> **Trace records what actually happened.**

Therefore the client-supplied prompt should remain visible as provenance rather than being hard-deleted or overwritten.

Where Moderari replaces it, however, the Trace must make the transformation unambiguous:

```text
Incoming System Prompt
    USER-A
    status: superseded for Replay

Moderari Policy
    Custom

Effective System Prompt
    CUSTOM-X
    status: active execution context
```

For Repetere, the superseded incoming prompt is effectively soft-deleted: it remains observable evidence but is excluded when the replayable model context is reconstructed.

Replay must never combine the superseded incoming prompt with the effective system prompt.

The required Replay result is:

```text
Source execution:
    Model received CUSTOM-X

Replay:
    Moderari = Pass-through
    Model receives CUSTOM-X
```

and never:

```text
Replay:
    system = USER-A
    system = CUSTOM-X
```

The selected Moderari policy, saved-prompt identity/name/hash and transformation information may be retained as additional provenance, but none may replace the actual effective system-prompt content.

## 5.1 Investigation Before Remediation

No Vestigare implementation change should be assumed until the current behaviour has been tested.

The M0.1 investigation must:

- create a fresh baseline Trace and inspect its system-prompt representation;
- repeat with Moderari in `Pass-through`;
- repeat with Moderari in `Custom` using a clearly distinguishable prompt;
- compare the stored Trace records;
- determine exactly which prompt Repetere currently reconstructs;
- determine whether Replay can currently produce two active system messages.

At least one behavioural test should use deliberately distinguishable or contradictory system-prompt instructions so that an incorrect duplicate-system-prompt replay is likely to produce an observable difference, rather than relying only on simple tasks whose answer may remain unchanged.

## 5.2 Candidate Vestigare Remediation

If the investigation shows that Vestigare records the incoming prompt before Moderari replacement without a reliable representation of the effective prompt, M0.1 should add explicit correction/provenance information.

A candidate control flow is:

```text
Moderari
    |
    | policy = Default or Custom
    | incoming system prompt replaced
    |
    | \obt trace effective-system-prompt
    | request/session correlation
    | effective prompt
    | policy/provenance
    v
Nuntius
    |
    v
Vestigare
```

Vestigare would use the correlated information to:

- retain the incoming system prompt;
- mark it as superseded for Replay;
- retain the actual injected system prompt;
- mark the injected prompt as the effective active system prompt;
- ensure Repetere receives only the effective prompt when reconstructing model context.

This is a candidate remediation, not a pre-judgement of the existing implementation. The exact implementation should follow the evidence produced by the investigation.

---
# 6. Research Significance

Making system-prompt handling configurable turns an existing hidden environmental condition into an explicit experimental variable.

A researcher could deliberately execute equivalent experiments under:

```text
Experiment A — Client system prompt passed through
Experiment B — Moderari default system prompt
Experiment C — Controlled custom system prompt
```

The resulting Traces would contain the actual contexts presented to the model.

This allows behavioural differences associated with system-prompt conditions to be observed and reproduced rather than inadvertently introduced by Lumen itself.

---

# 7. External Research Distribution M0.1 Relevance

Both requirements were identified while defining the first Lumen External Research Distribution.

The proposed research distribution includes:

- Rogare
- Pontis
- Vestigare
- Repetere
- Fiducia
- Moderari
- Praebere
- Servire
- Nuntius
- MongoDB

The distribution is intended to be Dockerised and operate over a private Lumen Docker network.

For M0.1, the complete distribution is a **single-host installation**. All Lumen services and supporting services must run on the same host machine.

Distributed multi-host operation is outside the M0.1 development and validation boundary, even though the service architecture may support network communication between components.

Aestimare is explicitly **not included** in the research distribution.

Consequently, researchers must be able to see the experimental structure and objective divergence evidence without relying upon Aestimare to interpret it.

External researchers may bring their own clients, model providers and system prompts. Moderari must therefore allow the system-prompt policy to be controlled rather than silently altering an externally defined experimental condition.

### External Client Tool Responsibility

When an external client is used, the external client is responsible for its own tool environment.

This includes responsibility for:

- declaring or exposing the tools available to the model;
- providing any tool definitions required by the client/model interaction;
- executing tool calls;
- returning tool results into the continuing client/model interaction.

Lumen does not prescribe or manage the external client's tool environment in M0.1.

From Lumen's perspective, client-managed tools form part of the external client's execution environment.

M0.1 therefore makes no claim of compatibility with, or responsibility for, arbitrary external-client tool implementations.

### Rogare Tool Responsibility

Rogare has no native tool environment.

Where tool use is required, Rogare relies upon an external tool provider.

For M0.1:

- Pi is the only supported and validated tool provider for Rogare;
- Pi must be installed in its expected/default installation location;
- Rogare uses the existing Lumen integration to access Pi;
- alternative tool providers are not supported through Rogare in M0.1.

Broader tool-provider discovery, configurable provider locations, multiple simultaneous providers and alternative tool-provider integrations remain outside the M0.1 scope unless separately validated and brought into the release.

---


# 8. Multi-Session Isolation Validation

## 8.1 Requirement

M0.1 must be tested with more than one active Lumen session.

The purpose of this investigation is to establish that transactions occurring within multiple concurrent sessions remain bounded to their originating session and that no session-specific conversational context, responses, Trace evidence or other session-specific state leaks between sessions.

Testing should verify that interactions initiated within one session remain associated with that session throughout the relevant Lumen execution path.

## 8.2 Execution-Condition Scope

For M0.1, provider/model selection is runtime-global and becomes locked while any
active execution session exists, while Moderari system-prompt policy is session scoped.

A successful Praebere model selection changes the provider/model used by subsequent
model requests from all active sessions. By contrast, each established Moderari
session retains its own active system-prompt policy and, where applicable, Applied
Custom prompt state.

The M0.1 investigation must validate session isolation while preserving this explicit
difference in scope.


---


# 8.3 Vestigare — Active Session Selection and Recording Boundary

Pontis is authoritative for session identity. Vestigare must bind an active Trace to
one explicit Pontis `session_id`; it must not infer the recorded session from creation
order, recency or activity.

M0.1 requires:

- only one active Vestigare recording across the installation;
- Trace Start to be unavailable/rejected when there are no active Lumen sessions;
- the Vestigare UI to obtain/display eligible active Pontis sessions;
- automatic explicit binding when exactly one eligible session exists;
- researcher selection of the session to record when multiple eligible sessions exist;
- the selected Pontis `session_id` to be persisted as the Trace session binding;
- traffic from all unselected sessions to continue normally without entering the active Trace;
- complete-session recording to begin before the selected session's first model interaction;
- Trace Start to be rejected for a session that has already performed model interaction rather than silently creating an incomplete Trace.

A later request may carry earlier conversational messages, but that cannot reconstruct
all execution events and provenance that Vestigare failed to observe. M0.1 must not
present a mid-session capture as a complete research Trace.

The normal supported sequence is:

```text
client connects
    |
    v
Pontis establishes/assigns session_id
    |
    v
researcher starts Trace for that session
    |
    v
Vestigare binds trace_id -> session_id
    |
    v
first model interaction
```

Explicit partial/mid-session Trace support and multiple simultaneous recordings remain
future development.


# 9. Nuntius / Servire — Common `\obt` Control Plane

## 9.1 Requirement

M0.1 development should consolidate Lumen control commands around `\obt` and introduce **Lumen Nuntius** as the lightweight command-routing and response-distribution service.

Nuntius is foundational M0.1 infrastructure because Replay system-prompt fidelity, Praebere discovery/selection and other control operations require a common path.

The control plane should not require callers to know:

- which service owns a command;
- where that service is running;
- whether a successful query response is required by another Lumen service.

The responsibility is divided deliberately:

> **Servire defines the active control-plane topology. Nuntius materialises that topology into a runtime routing dictionary. Domain services execute their commands. Pontis owns external client/session return routing.**

## 9.2 Existing Behaviour Must Be Reused

Before implementing Nuntius, inspect the existing `\obt` behaviour in:

- Repetere;
- Moderari;
- Pontis.

Establish the current command format, injection behaviour, response propagation, request/session correlation and Trace interaction.

Nuntius should consolidate and generalise existing behaviour rather than introduce another incompatible mechanism.

## 9.3 Standard `\obt` Service Capability

Participating Lumen services should expose a common lightweight `\obt` receiving contract.

Servire configuration should include:

```text
obt_enabled
```

If:

```text
obt_enabled = false
```

the service control endpoint may immediately return:

```text
204 No Content
```

If enabled, the request is passed to the service's local `\obt` handler.

An enabled service may still return `204` for an unrecognised or irrelevant command.

## 9.4 Response Contract

The M0.1 control-plane response semantics are:

```text
204 No Content
    Command not handled by this service.

200 OK, no body
    Command executed successfully.

200 OK, with body
    Request/query handled successfully; body is the result.

4xx / 5xx
    Command was recognised/owned but execution failed.
```

The distinction between `200` with and without a body is intentional.

A command such as model selection is an execution and may return `200` with no body.

A command such as model discovery is a request/query and returns `200` with a body.

For execution commands, `200` means the requested operation has been **successfully applied**, not merely that the command was received.

This is especially important for commands that establish prerequisites for subsequent work. A caller may proceed only after the authoritative command owner has returned `200`.

## 9.5 Servire as Control-Plane Configuration Authority

Servire already describes and manages the operational service topology.

M0.1 should extend the Servire service configuration so it can also describe the information Nuntius requires for routing.

For each active `\obt`-capable service, Servire should be able to describe:

- service identity;
- service endpoint;
- `obt_enabled`;
- commands owned by that service;
- consumers of successful query response bodies.

Conceptually:

```yaml
service: praebere
obt_enabled: true

commands:
  models:
    response_targets:
      - originator
      - rogare

  providers:
    response_targets:
      - originator
      - rogare
```

The exact configuration representation should follow existing Servire conventions.

Nuntius consumes this configuration but does not define the policy.

## 9.6 Nuntius Bootstrap and Routing Dictionary

On startup Nuntius sends Servire a direct:

```text
\obt services
```

Servire returns the active control-plane catalogue.

A successful response containing no active services is valid and represents a healthy Nuntius startup.

Nuntius materialises the response into an in-memory routing dictionary.

Conceptually:

```text
replay list
    owner: repetere
    response_targets:
        - originator

models
    owner: praebere
    response_targets:
        - originator
        - rogare
```

Servire remains authoritative.

When Servire starts or stops a service, it sends a control-plane catalogue update through Pontis to Nuntius.

Nuntius must also be able to discard and rebuild its local dictionary from Servire.

## 9.7 Targeted Routing

The normal M0.1 routing path should be targeted rather than broadcast-to-all.

For example:

```text
\obt replay list -> Repetere
\obt models      -> Praebere
```

This avoids unnecessary control-plane traffic and avoids sending potentially large query results to services that do not require them.

`204` remains a useful common/defensive service response but should not be used as the normal mechanism for discovering command ownership.

## 9.8 External Request/Response Routing

Pontis remains the external session bridge.

For a client-originated command:

```text
External Client
      |
      v
    Pontis
      |
      v
    Nuntius
      |
      v
command owner
```

Pontis must preserve the client/session/request correlation.

For a query response:

```text
command owner
      |
      | 200 + body
      v
    Nuntius
      |
      v
    Pontis
      |
      v
originating client/session
```

The boundary is:

> **Pontis owns client/session correlation. Nuntius owns command/service routing and configured response distribution.**

## 9.9 Shared Query Responses

Some query results are required by the originating client and by another Lumen service.

This requirement should be explicit in Servire configuration rather than inferred by Nuntius.

For example:

```text
\obt models

owner: Praebere
response_targets:
    - originator
    - Rogare
```

The flow is:

```text
External Client
      |
    Pontis
      |
    Nuntius
      |
   Praebere
      |
      | 200 + model list
      v
    Nuntius
     /     \
    /       \
Pontis      Rogare
  |
originating client
```

Praebere remains authoritative for the model list.

Nuntius transports the response but does not become an authoritative shared-state store.

Response bodies should be sent only to configured consumers rather than broadcast to every active service.

## 9.10 Trace Boundary

Nuntius traffic is control-plane traffic and must remain outside the normal ask/answer path.

`\obt` commands must not appear as user/model conversational turns in Vestigare Trace records.

Where a command changes an execution condition, the resulting condition remains traceable.

Examples include:

- selected model;
- selected provider;
- effective system prompt.

The rule remains:

> **Trace records the execution context and conditions that actually reached or affected the model, not the internal control message used to establish them.**

## 9.11 Replay System-Prompt Control

Before Repetere replays a source Trace, it must use the common Nuntius path to instruct Moderari to use Pass-through for that replay session.

The override must be session-scoped.

This control command is synchronous from Repetere's perspective.

Nuntius must return Moderari's authoritative outcome to Repetere, and Repetere must receive `200 OK` before beginning the replay.

The effective system prompt from the source Trace is then replayed unchanged.

If Nuntius cannot route the command, Moderari returns `204`, `4xx`, or `5xx`, or the request times out, Repetere must not start the model replay.

This makes Nuntius a dependency of the M0.1 Replay fidelity requirement and makes positive command acknowledgement part of the Replay acceptance criteria.

## 9.12 Praebere Discovery and Selection

Praebere is the first native adopter of the common Nuntius control path.

Expected commands include:

```text
\obt providers
\obt models
\obt model select <model>
```

Praebere must query Ollama on startup and maintain authoritative knowledge of the models actually available locally. A configured model is an optional **preferred model**, not a required installed/default model. Absence of the preferred model must not prevent Praebere or stack startup.

The M0.1 state model distinguishes **available models**, **preferred model**, **selected runtime-global model**, and **selection lock state**.

`providers` and `models` return `200 + body`. Successful model selection must return an explicit authoritative outcome identifying the effective selection.

### Session and Selection Lifecycle

Pontis remains authoritative for session identity.

A newly connected client first has an **established session**: `session_id` exists but no model interaction has occurred. Establishment alone must not lock model selection.

After the first model interaction, the session becomes an **active execution session**. At that point the runtime-global selected model is locked. It remains locked while any active execution session exists.

The intended sequence is:

```text
connect / establish session
        |
        v
model selection allowed if no active execution session
        |
        v
optional Trace start
        |
        v
first model interaction
        |
        v
active execution session / runtime-global model locked
        |
        v
last active execution session ends
        |
        v
model selection available again
```

An external client established while no model is selected must receive the discovered model list and guidance:

```text
\obt praebere model select <model_name>
```

Praebere must not silently apply the installation's preferred model to that external client.

If an authoritative model is already locked by active execution, a newly established external or Rogare session does not receive a model choice; it is informed/displayed which runtime-global model is in use and uses that model.

### Rogare

Rogare should obtain the available-model state through Nuntius/Praebere and display it as a dropdown near the session start/stop controls.

When no active execution session exists, the dropdown is enabled; an available preferred model may be shown as the initial UI choice; and changing the UI choice has no authoritative effect until Praebere acknowledges selection through the common control path.

While any active execution session exists, the dropdown is disabled/locked; Rogare displays the authoritative runtime-global selected model; and Rogare restart/reconnect recovers that state rather than assuming local UI state.

Servire configuration determines whether discovery/state responses are additionally delivered to Rogare or another service.

Per-session model selection is future development and is not required for M0.1.

## 9.13 Operational Logging Boundary

Servire provides an **Operations Log** giving the operator a consolidated view of meaningful operational activity across the Lumen services.

Nuntius command-routing and transport diagnostics must not be written into the normal Servire Operations Log.

Nuntius should maintain its own diagnostic logging, exposed through the **Nuntius UI within Servire**, where command routing, responses, failures, timeouts and related control-plane activity can be inspected when required.

The intended boundary is:

> **Servire Operations Log = operator-relevant Lumen activity.**  
> **Nuntius diagnostics = internal control-command transport activity.**  

One known exception remains in M0.1: Rogare currently polls for status updates, and this internal activity may appear in the Servire Operations Log and obscure more meaningful user activity.

Separating Rogare polling/internal activity from user-session operational activity is not required for M0.1 and remains future development.


## 9.14 M0.1 Development Priority

The preferred dependency sequence is:

```text
1. Inspect existing Repetere / Moderari / Pontis \obt behaviour
2. Define the common 204 / 200 / error contract
3. Extend Servire configuration with obt_enabled, command ownership and response targets
4. Implement Nuntius bootstrap and in-memory routing dictionary
5. Route Pontis \obt traffic to Nuntius and preserve session/request correlation
6. Implement targeted command routing
7. Implement returned query-response routing through Pontis
8. Implement configured shared-response delivery
9. Migrate existing Repetere / Moderari command behaviour
10. Implement Replay system-prompt fidelity through Nuntius, including blocking positive acknowledgement from Moderari
11. Implement Praebere discovery and selection through Nuntius
12. Continue remaining M0.1 consumer changes
```

This establishes the control plane before extending the services that consume it.

---

# 10. Servire — Runtime Authorization and Distribution Security

M0.1 development must implement the trust boundary defined by **M0.1 Runtime Authorization and Code Protection**.

This is a release requirement, not a later commercial-licensing feature.

## 10.1 Installation Identity

Each external installation must have:

- a Distribution ID identifying the issued release artifact;
- a Service Group UUID identifying the installed Lumen stack;
- an installation-specific cryptographic public/private key pair or equivalent established asymmetric identity.

The Service Group UUID must not be accepted as proof of authorization.

The installation private key must remain local to the installation and should use hardware-backed, operating-system-protected or encrypted local storage where available.

Illuminates.One must retain the corresponding registered public identity associated with the Service Group UUID and Distribution ID.

## 10.2 Signed Runtime Authorization

Servire must establish authorization with Illuminates.One over TLS/HTTPS.

The exchange must provide attribution in both directions:

- Servire signs or otherwise cryptographically authenticates the request using the installation identity;
- Illuminates.One verifies the installation identity;
- Illuminates.One returns a signed authorization assertion;
- Servire verifies that assertion using distributed public verification material.

The authorization exchange must include freshness protection, including a cryptographically secure nonce or an equivalent established mechanism, so that a captured successful response cannot simply be replayed.

Established cryptographic standards and libraries must be used. M0.1 must not invent proprietary cryptography.

## 10.3 Authorization Lease and Heartbeat

Successful authorization grants a time-limited runtime lease.

Servire must:

- track the current authorization state;
- renew authorization periodically;
- tolerate bounded temporary connectivity failure;
- enter a defined grace state where policy permits;
- refuse continued authorised operation once the lease and grace period expire.

Deliberate heartbeat suppression and persistent inability to renew authorization must ultimately produce the same security outcome: the installation cannot continue indefinitely as an authorised Lumen runtime.

The exact lease and grace durations should remain server-side/configurable policy rather than hard-coded product constants.

## 10.4 Servire-Controlled Service Lifecycle

For the external M0.1 runtime, Servire is the operational authority for the protected service group.

Normal service startup must occur through Servire.

Protected services must be capable of validating that startup was authorised by Servire, using a short-lived mechanism tied to the installation and service. The exact launch-authorization mechanism remains an implementation decision.

A service must not inherit enough authority to start or authorize other protected services independently.

Direct command-line startup may remain available in the development environment, but it is not a supported authorised external M0.1 execution path.

## 10.5 Protected Runtime Configuration

Protected operational configuration distributed with M0.1 must be encrypted at rest where appropriate.

The distributed installation alone must not contain a permanent standalone secret sufficient to unlock all protected runtime configuration.

Valid Illuminates.One authorization must contribute to making protected configuration usable.

Where practical, decrypted configuration should exist only in process memory and should not be written back to persistent storage as plaintext.

The precise key-derivation/envelope-encryption design requires implementation security review and must use established primitives.

## 10.6 Copy Resistance

M0.1 must validate that copying the release artifact or installed filesystem does not automatically create another authorised installation.

Possession of:

- source code;
- containers;
- encrypted configuration;
- Distribution ID;
- Service Group UUID;
- public verification material;

must not, by itself, be sufficient for normal authorised operation.

Where practical, the installation private identity should be OS- or hardware-bound so filesystem copying does not reproduce it.

## 10.7 Distributed Secrets

The external distribution must not contain secrets whose security depends upon researchers being unable to inspect the distributed code.

In particular, it must not contain:

- the Illuminates.One authorization private signing key;
- a permanent master configuration-decryption key;
- master licensing secrets;
- hidden bypass credentials;
- equivalent hard-coded authorization bypasses.

Public verification material may be distributed.

## 10.8 Security Acceptance

M0.1 security acceptance requires demonstrating that:

- a provisioned installation can authenticate to Illuminates.One;
- Illuminates.One can authenticate the installation independently of its UUID;
- Servire rejects an invalid, stale, mismatched or cryptographically unverifiable authorization;
- a captured authorization response cannot simply be reused for a fresh challenge;
- a valid authorization lease permits normal operation;
- temporary connectivity loss follows the defined lease/grace behaviour;
- expired authorization prevents continued normal authorised operation;
- protected services validate Servire-authorised startup;
- unsupported direct startup does not constitute authorised external operation;
- protected configuration is not normally available in plaintext at rest;
- copied installation material alone does not establish authorization;
- distributed artifacts contain no Illuminates.One private authority or permanent master unlock secret;
- the complete M0.1 distribution operates successfully as a single-host installation.

The implementation must also document the unavoidable boundary that software running on researcher-controlled hardware cannot be made absolutely tamper-proof.

---

# 11. Explicitly Deferred from M0.1

The following work has been identified but is not part of the M0.1 development requirement:

## Rogare Human Assessment

Future Rogare capability to record independent 1–10 user ratings for answer correctness and answer quality, allowing later comparison with Aestimare assessment.

## Pontis Multiple Tool Providers

Future investigation of multiple simultaneous tool providers, provider discovery, tool namespaces and a unified Pontis tool catalogue.

M0.1 may retain a single-tool-provider limitation.

## Bounded Computational Resources Research

Future controlled research into whether resource constraints alter observable model behaviour rather than merely response speed.

## Operational Log Classification

Future Servire/Rogare work should distinguish user-session actions from internal polling, health, discovery and control-plane maintenance so the default operational view is not dominated by internal activity.

This should not add `log_class` or equivalent metadata to the initial M0.1 Nuntius contract.

## Servire Single-Active-Operator Enforcement

Required before unrestricted/general Research Distribution, but not required for M0.1.

The intended mechanism is a single active Servire operator lease per installation, allowing legitimate remote operation while politely declining a second simultaneous operator.

---

# 12. Internal M0.1 Acceptance Criteria

## Repetere / Fiducia

- Introduce **Experiment** as an explicit concept.
- Associate an Experiment with a source Trace.
- Associate replay-created Traces with their Experiment.
- Treat individual replay executions as Experiment runs.
- Allow Fiducia to coordinate multiple runs belonging to an Experiment.
- Display Experiment runs hierarchically in the UI.
- Surface `MATCHED`, `DIVERGED`, and `FAILED / INCOMPLETE` status.
- Surface the first divergence point where available.
- Keep divergence detection separate from behavioural assessment.
- Replay the effective system prompt recorded in the source Trace.
- Instruct Moderari to use Pass-through for the replay session before replay begins.
- Require `200 OK` from Moderari before Replay sends any recorded model interaction.
- Abort Replay on `204`, `4xx`, `5xx`, timeout, or unreachable prerequisite state.
- Record prerequisite failure as `FAILED / INCOMPLETE` with the failure reason retained.
- Ensure the Replay Pass-through override is session-scoped.
- Create a new isolated session for every Replay execution.
- Ensure a Replay session is initialised only from the source Trace and explicitly defined experimental conditions.
- Prevent context or temporary state produced by one Replay from entering another Replay run.
- Verify the effective system prompt is present in a fresh Vestigare Trace.
- Distinguish incoming client system-prompt provenance from the effective system prompt used by the model.
- Preserve superseded incoming prompts as evidence while excluding them from Replay reconstruction.
- Verify Replay reconstructs exactly one active effective system prompt.

## Moderari / Servire

- Replace mandatory system-prompt replacement with a configurable policy.
- Support `Pass-through`.
- Support `Moderari Default`.
- Support `Custom`.
- Treat `Custom` as an exact researcher-defined system-prompt condition.
- Do not add, remove, rewrite, merge, normalise or augment the selected Custom prompt.
- Do not carry tool instructions/descriptions from the removed client system prompt into the Custom prompt.
- Keep tool availability separate from tool instructions contained in the Custom system prompt.
- Require any desired tool-related system instructions to be explicitly included by the researcher in the Custom prompt.
- Preserve current Moderari behaviour, including existing tool-related system-prompt processing, in `Moderari Default`.
- Make the System Prompt Policy UI and runtime state a Moderari responsibility.
- Expose Moderari's System Prompt Policy UI through the Moderari tab in Servire without making Servire the configuration intermediary.
- Add a Custom system-prompt editor with explicit working-copy and applied-state visibility.
- Require explicit `Apply` before editor contents affect Moderari runtime.
- Keep `Apply` separate from saved-prompt persistence.
- Add saved named Custom system prompts in a dedicated MongoDB collection.
- Add stable `prompt_id`, `owner_id` and `scope` fields to the M0.1 saved-prompt schema.
- Provide explicit `Save As…`, `Update Saved` and `Delete Saved` operations.
- Never implicitly overwrite a saved prompt merely because its loaded working copy was edited.
- Ensure saved-prompt persistence operations do not silently alter the active Moderari runtime prompt.
- Ensure Pass-through genuinely leaves client-provided system context unchanged.
- Ensure Vestigare records the effective system prompt that actually reaches the model.
- Where Moderari replaces a client system prompt, preserve the original as superseded provenance rather than active Replay context.

## Multi-Session Validation

- Execute multiple simultaneous Lumen sessions.
- Verify session-state isolation across the stack.
- Verify Trace ownership and execution evidence cannot cross sessions.
- Verify configuration/control state remains correctly scoped.
- Verify sequential and concurrent Replay runs use distinct isolated sessions.
- Verify Pontis provides the authoritative `session_id` used for Vestigare Trace binding.
- Verify Trace Start is unavailable/rejected when no active session exists.
- Verify Vestigare displays/selects the sole eligible session when exactly one exists.
- Verify Vestigare requires explicit session selection when multiple eligible sessions exist.
- Verify Vestigare never selects a session implicitly from recency or activity.
- Verify a Trace can start after session establishment but before that session's first model interaction.
- Verify Trace Start is rejected once the selected session has already performed model interaction.
- With multiple clients active, verify only traffic for the selected `session_id` enters the active Trace.

## Nuntius / Pontis / Praebere / Servire

- Treat Nuntius as foundational M0.1 infrastructure and implement it early.
- Review existing Repetere, Moderari and Pontis `\obt` implementations.
- Establish `\obt` as the common Lumen control-command language.
- Add a common `obt_enabled` service capability.
- Define `204`, `200` without body, `200` with body, and error semantics.
- Extend Servire configuration with command ownership and query-response targets.
- Make Servire authoritative for the active control-plane topology.
- Bootstrap Nuntius directly from Servire using `\obt services`.
- Accept an empty Servire catalogue as healthy.
- Build an in-memory Nuntius routing dictionary from Servire configuration.
- Keep the dictionary current as Servire starts/stops services.
- Route commands directly to configured owners rather than broadcasting normally.
- Preserve external request/session correlation in Pontis.
- Return query results through Nuntius to Pontis and the correct originating client.
- Deliver query response bodies only to additional services configured by Servire.
- Keep Nuntius control traffic out of conversational Trace records.
- Preserve resulting execution conditions in Trace.
- Use the common path for Replay Pass-through control.
- Return the authoritative Moderari execution result to Repetere and require positive acknowledgement before Replay proceeds.
- Use the common path for Praebere provider/model discovery and selection.
- Defer operational-log user/internal classification from the initial M0.1 control-plane implementation.


## Servire / Runtime Authorization Security

- Establish Distribution ID, Service Group UUID and installation cryptographic identity.
- Treat the Service Group UUID as an identifier, not an authentication secret.
- Authenticate Servire authorization requests using the installation identity.
- Verify Illuminates.One signed authorization independently of TLS.
- Protect authorization freshness with nonce/challenge semantics.
- Implement time-limited authorization lease, renewal, grace and expiry.
- Prevent indefinite authorised operation after heartbeat/authorization loss.
- Make Servire the supported external service-lifecycle authority.
- Require protected services to validate Servire-authorised startup.
- Protect operational configuration at rest and couple normal unlock/use to valid authorization.
- Prevent copied release material alone from establishing an authorised installation.
- Keep Illuminates.One private authority and permanent master unlock secrets out of the distribution.
- Validate and document the boundary that source-distributed software on third-party hardware is not absolutely tamper-proof.

## 12.1 Replay Prerequisite Acceptance

Replay system-prompt fidelity is not accepted merely because the Pass-through command can be sent.

Acceptance requires demonstrating that:

- Repetere sends the session-scoped Pass-through command through Nuntius;
- Moderari returns `200 OK` only after the change has been applied;
- Nuntius returns that authoritative result to Repetere;
- Repetere does not begin replay until the `200` is received;
- `204`, `4xx`, `5xx`, timeout, and unreachable cases prevent Replay from starting;
- the failed run is represented as `FAILED / INCOMPLETE`;
- another active session's Moderari policy remains unchanged;
- each Replay run is created in a new session;
- no prior Replay context/state is present in a subsequent Replay session;
- the Replay session contains only source-Trace context and explicitly established execution conditions;
- a replaced incoming system prompt remains visible as provenance but is excluded from Replay model context;
- exactly one effective system prompt is reconstructed for Replay;
- a deliberately system-prompt-sensitive validation demonstrates that duplicate/incorrect system prompts are not being sent to the model.


---

# 13. Architectural Principle

These requirements preserve a broader Lumen principle:

> **Experimental conditions should be explicit, reproducible and observable. Lumen should record what happened without unnecessarily altering the conditions being investigated.**

Repetere and Fiducia establish the relationship between an observation and its controlled repetitions.

Repetere reproduces the effective system context recorded in the source Trace rather than substituting current configuration.

Moderari determines how system-level context is handled.

Praebere determines the model/provider execution environment.

Servire remains authoritative for operational service state and defines the active control-plane routing configuration.

Nuntius materialises that configuration into runtime command and response routing without acquiring domain knowledge.

Vestigare records the execution that actually occurred.

Aestimare, when present in the commercial Lumen distribution, can subsequently assess the behavioural evidence produced by those experiments.
