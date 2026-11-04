# Lumen Development Requirements

**Date:** 2026-08-21  
**Status:** Superseeded Development Requirements  
**Components:** Lumen Repetere, Lumen Fiducia, Lumen Moderari, Lumen Servire  
**Origin:** External Research Distribution M0.1 planning

---

## 1. Purpose

Planning for the Lumen External Research Distribution M0.1 has exposed two requirements in the existing Lumen implementation that should be addressed independently of the research distribution itself.

They concern:

1. The representation of repeated replay executions as an **Experiment**, including clear visibility of replay divergence.
2. Making **Moderari system-prompt handling configurable** rather than always replacing an incoming system prompt.

These are changes to Lumen proper. The External Research Distribution provides the immediate motivation, but neither requirement should be implemented as research-distribution-specific behaviour.

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

---

# 4. Servire Configuration

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

# 5. Trace Behaviour

No special modification to Vestigare is required merely to capture the effective system prompt.

Vestigare records the execution context.

Therefore, whichever system prompt actually passes through Moderari and reaches the model should naturally form part of the recorded Trace.

This preserves the existing architectural responsibility:

> **Trace records what actually happened.**

Vestigare should not need to understand the semantics of Moderari's system-prompt configuration merely to duplicate configuration information.

The selected Moderari policy may be retained as execution/configuration provenance where appropriate, but the effective system prompt itself should not be redundantly recorded merely because the policy has become configurable.

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
- MongoDB

The distribution is intended to be Dockerised and operate over a private Lumen Docker network.

Aestimare is explicitly **not included** in the research distribution.

Consequently, researchers must be able to see the experimental structure and objective divergence evidence without relying upon Aestimare to interpret it.

Similarly, external researchers may bring their own clients, model providers, system prompts and tool environments. Moderari must therefore allow the system-prompt policy to be controlled rather than silently altering an externally defined experimental condition.

---

# 8. Development Requirements Summary

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

## Moderari / Servire

- Replace mandatory system-prompt replacement with a configurable policy.
- Support `Pass-through`.
- Support `Moderari Default`.
- Support `Custom`.
- Preserve current Moderari behaviour as the default mode.
- Add system-prompt policy configuration to Servire.
- Add a custom system-prompt editor to Servire.
- Ensure Pass-through genuinely leaves the client-provided system context unchanged.
- Allow Vestigare to continue recording the resulting effective context normally.

---

## 9. Architectural Principle

These changes preserve a broader Lumen principle:

> **Experimental conditions should be explicit, reproducible and observable. Lumen should record what happened without unnecessarily altering the conditions being investigated.**

Repetere and Fiducia establish the relationship between an observation and its controlled repetitions.

Moderari determines how system-level context is handled.

Vestigare records the resulting execution.

Aestimare, when present in the commercial Lumen distribution, can subsequently assess the behavioural evidence produced by those experiments.