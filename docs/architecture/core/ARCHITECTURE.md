# Lumen Architecture

**Status:** Canonical  
**Document role:** Architectural principles, system boundaries and responsibility model  
**Last reviewed:** 2026-08-21

---

## Purpose

This document is the canonical architectural description of **Lumen as a Reasoning Assurance Service**.

It defines why Lumen is structured as a family of bounded services, the responsibilities that must remain separate, the major execution and evidence planes, and the architectural principles that should govern future development.

For the current service topology, ports, maturity and concrete service relationships, see:

- [Lumen Ecosystem — Current Architecture and Topology](../ecosystem/LUMEN_ECOSYSTEM_CURRENT_ARCHITECTURE.md)
- [Lumen Service Matrix](../ecosystem/LUMEN_SERVICE_MATRIX.md)

Detailed implementation behaviour remains in the architecture documents within each service folder.

The earlier `ARCHITECTURE.md` and its two appendices are retained under `core/history/` as historical records of Lumen's architectural evolution.

---

## 1. Architectural Position

Lumen is not a model, agent, prompt framework or model provider.

Lumen is an **independent Reasoning Assurance Service** around model execution. It coordinates execution, records evidence, reproduces behaviour, repeats controlled executions and assesses accumulated behavioural evidence without requiring the underlying model or external capability provider to become Lumen-aware.

The architectural objective is not to make probabilistic systems deterministic. It is to make their behaviour **observable, reproducible, assessable and increasingly trustworthy through evidence**.

A useful progression is:

```text
Observe -> Reproduce -> Repeat -> Assess
```

In current service terms:

```text
Vestigare -> Repetere -> Fiducia -> Aestimare
```

A single execution is an observation. Repeated controlled executions become an experiment. Multiple experiments begin to characterise behaviour. Longitudinal evidence allows behavioural change to be detected.

---

## 2. Lumen Is the Product

The name **Lumen** refers to the complete ecosystem.

The named components are independently bounded Lumen services. No individual service is "Lumen itself", and no service should absorb responsibilities simply because it is convenient to do so.

The current service family is:

| Service | Primary responsibility |
|---|---|
| **Moderari** | Model-request orchestration and controlled intervention |
| **Pontis** | Client/session bridging and protocol correlation |
| **Praebere** | Model-provider abstraction, readiness and lifecycle |
| **Vestigare** | Execution trace and evidence capture |
| **Repetere** | Replay and divergence reproduction |
| **Fiducia** | Controlled repeated Replay orchestration |
| **Aestimare** | Behavioural assessment of accumulated evidence |
| **Rogare** | Human conversational and operational client |
| **Servire** | Operational control plane |
| **Audire** | Proposed distributed operational transport |

The canonical current service matrix contains maturity, dependencies and ports.

---

## 3. Core Architectural Rule: One Bounded Responsibility

Each Lumen service has one primary job.

Services cooperate through explicit interfaces, but responsibility must not migrate across boundaries merely to simplify a local implementation.

Examples:

- Vestigare records what happened; it does not decide what the evidence means.
- Repetere reproduces an execution; it does not own repeated experiment scheduling.
- Fiducia schedules and coordinates repeated Replay activity; it does not perform behavioural judgement.
- Aestimare assesses evidence; it does not become the trace recorder or replay engine.
- Servire operates the stack; it does not become the conversational execution path.
- Praebere abstracts provider lifecycle and readiness; it does not become the model or the orchestrator.
- Pontis bridges protocols and sessions; it does not reinterpret provider semantics.
- Moderari orchestrates requests and may enforce defined interventions; external providers return capability/semantics rather than Lumen prompts.

This separation allows each capability to mature independently and keeps evidence about one responsibility from being confused with another.

---

## 4. Architectural Planes

Lumen deliberately separates concerns into several planes.

### 4.1 Conversational execution plane

The principal model interaction path is:

```text
User
  |
Rogare
  |
Pontis
  |
Vestigare
  |
Repetere
  |
Moderari
  |
Model-provider boundary
  |
External model provider
```

The services in this path are intentionally transparent where their responsibility requires transparency.

Pontis bridges. Vestigare records. Repetere remains transparent during normal execution and becomes active when controlled replay is requested. Moderari owns orchestration at the model boundary.

The model provider remains external to Lumen.

### 4.2 Provider and capability plane

Lumen must remain agnostic to particular model and tool providers.

**Praebere** isolates provider-specific model lifecycle and readiness behaviour from the rest of Lumen.

**Pontis** isolates protocol/session bridging for external capabilities and tool providers.

Where Moderari invokes an external capability, the capability returns its semantics or result. It must not become responsible for constructing Lumen's prompts or orchestration policy. This preserves the boundary between external capability and Lumen behaviour.

### 4.3 Evidence plane

The evidence architecture is deliberately progressive:

```text
Execution
   |
Vestigare
   |  recorded observation
Repetere
   |  controlled reproduction / divergence evidence
Fiducia
   |  repeated controlled executions
Aestimare
   |  behavioural assessment
Reasoning Assurance evidence
```

These stages are related but not interchangeable.

Lumen's assurance claims should arise from accumulated evidence, not from a single successful answer or a supplier benchmark.

### 4.4 Operational control plane

**Servire** is separate from the conversational execution path.

Its responsibility is to operate the Lumen stack: dependency validation, service lifecycle, operational state and access to logs/operational interfaces.

This distinction matters because operating Lumen and executing a model request are different concerns.

**Audire** is the architectural direction for distributed operational transport. Its purpose is to change how operational commands, acknowledgements and process output move between Servire and services when those services no longer share one host. Audire must not absorb Servire's control-plane responsibility.

---

## 5. Evidence Before Judgement

Lumen distinguishes **capture**, **reproduction**, **experimentation** and **assessment**.

That distinction is fundamental to Reasoning Assurance.

### Observation

Vestigare captures the observable execution and its relevant metadata.

An individual execution can establish what happened once. It cannot by itself characterise model behaviour.

### Reproduction

Repetere re-executes a recorded interaction under controlled conditions and records whether the result matches or diverges.

Divergence is evidence, not automatically failure.

### Repetition

Fiducia coordinates repeated Replay executions so that behaviour can be examined across a population of controlled runs rather than inferred from isolated examples.

### Assessment

Aestimare interprets accumulated evidence.

Aestimare's role is to assess **behaviour, not intelligence**. It may ultimately coordinate multiple specialist internal assessment engines, each independently examining a bounded behavioural dimension, while Aestimare combines their evidence into a wider assessment.

The architecture should therefore support plural assessment methods rather than forcing Reasoning Assurance into one monolithic scoring algorithm.

---

## 6. Behaviour Is Contextual

A model response cannot be assessed independently of the conditions that produced it.

Relevant conditions may include:

- model and provider;
- system and assistant instructions;
- accumulated conversation context;
- tool availability and tool use;
- Moderari intervention;
- external capability results;
- model parameters and provider state;
- prior ambiguity or contradiction in the session;
- the evidence and provenance available at execution time.

This is particularly important in long-running sessions. The model may retain a much larger explicit context than the human user is actively holding in working memory. Ambiguity or contradiction introduced earlier can therefore materially affect later behaviour even when the immediate exchange appears straightforward.

Lumen must preserve enough provenance to distinguish behavioural change from changed execution conditions.

---

## 7. Reproducibility Does Not Mean Determinism

Probabilistic model execution means that identical conditions need not produce identical outputs.

Lumen therefore does not define successful Replay as "the model must always emit the same text."

Replay exists to make differences observable and attributable.

Assessment can then ask more useful questions:

- Did the answer remain semantically sufficient?
- Did reasoning behaviour materially change?
- Did tool use change?
- Did the model reach an equivalent acceptable outcome by another path?
- Did divergence expose instability, ambiguity or a changed dependency?
- Is observed variation acceptable for this task and consequence level?

This distinction prevents deterministic software expectations from being incorrectly imposed on probabilistic systems.

---

## 8. Sufficiency Rather Than a Single Ideal Answer

Many engineering and reasoning tasks do not have one uniquely correct solution.

Different languages, architectures, responsibility divisions or reasoning paths may all be valid provided they satisfy the relevant constraints.

Aestimare should therefore support assessment of **sufficiency** and behavioural characteristics rather than assume that every task has one canonical output.

Where a definitive answer does exist, it can be used as stronger experimental ground truth. Where it does not, assessment must consider the requirements and evidence appropriate to that class of task.

---

## 9. Provenance Is Architectural, Not Administrative

Reasoning Assurance depends on knowing the conditions under which evidence was produced.

Trace, Replay and assessment evidence must remain correlatable to the execution that produced it.

Lumen should preserve provenance sufficiently to answer questions such as:

- Which model/provider was used?
- What context and orchestration conditions applied?
- Which tools or external capabilities were available or invoked?
- Which recorded execution was replayed?
- Which repeated runs belong to the same experiment?
- Which assessment was derived from which evidence?
- What changed between two otherwise comparable executions?

Provenance is therefore part of the architecture of assurance, not merely logging metadata.

---

## 10. External Independence

Lumen should not become structurally dependent on one client, model, model provider, tool provider or deployment topology.

Current implementations such as Ollama or Pi are valuable integrations and test cases, but they do not define the architecture.

The architectural boundary should permit replacement or addition of providers without requiring unrelated Lumen services to adopt provider-specific concepts.

The same principle applies to deployment. HTTP services may run on one host today and on distributed hosts later. Deployment transport may evolve without changing service responsibility.

---

## 11. Operational Independence

The architecture separates **service responsibility** from **process location**.

A Lumen service is defined by its contract and responsibility, not by:

- sharing a machine with another service;
- being started by the same process;
- writing to a locally visible stdout stream;
- using a particular broker;
- using a fixed development port.

This is why Audire can later replace local operational transport without redefining Servire, and why ports in the ecosystem topology are configuration rather than architectural identity.

---

## 12. Canonical Architectural Boundaries

The following boundaries should be treated as hard architectural constraints unless deliberately revised in this document.

1. **Lumen is model-agnostic and provider-agnostic.**
2. **Lumen services have bounded primary responsibilities.**
3. **Evidence capture is separate from evidence assessment.**
4. **Single Replay execution is separate from repeated experiment orchestration.**
5. **Operational control is separate from conversational execution.**
6. **Provider-specific lifecycle belongs behind Praebere.**
7. **Protocol/session bridging belongs behind Pontis.**
8. **External capability providers return capability results/semantics, not Lumen orchestration prompts.**
9. **Historical evidence and provenance are preserved rather than rewritten to fit current architecture.**
10. **A behavioural assessment must remain traceable to the evidence and execution conditions from which it was derived.**
11. **Probabilistic divergence is evidence to assess, not automatically an error.**
12. **Deployment topology and transport may change without changing service responsibility.**

---

## 13. Current and Emerging Architecture

The architecture is sufficiently stable for this document to act as the canonical architectural source, but not every service has the same implementation maturity.

At the time of this review:

- the principal execution stack is implemented and operational;
- Servire provides the operational control plane;
- Fiducia has an implemented initial repeated-Replay capability;
- Aestimare is the active assessment design/development area;
- Audire is documented architectural direction for distributed operational transport.

Maturity changes belong in the service matrix and ecosystem topology. They do not require this architectural document to become a release-status ledger.

---

## 14. Relationship to Historical Architecture

Lumen's earlier architecture documents described a system centred more heavily on continuity, projects, sessions, checkpoints, knowledge branches and a unified console concept.

Those documents are retained because they explain how the present architecture developed. They are not deleted or rewritten to make them appear current.

The current architecture has solidified around a wider Reasoning Assurance service family with explicit separation between:

- orchestration;
- provider integration;
- protocol bridging;
- evidence capture;
- replay;
- repeated experimentation;
- behavioural assessment;
- human interaction;
- operational control.

Historical documents should be read as provenance, not as current implementation authority.

---

## 15. Documentation Authority

For architectural questions, use this order:

1. **This document** — architectural principles and responsibility boundaries.
2. **Current ecosystem topology** — current service relationships, flows and maturity.
3. **Service matrix** — concise responsibility/dependency/interface index.
4. **Individual service architecture documents** — detailed service-specific behaviour.
5. **Current supporting documents** — focused runtime/design material.
6. **Historical, proposal and research documents** — provenance and exploration, not current authority.

If a current implementation change contradicts this document, the discrepancy should be resolved deliberately rather than silently allowing architecture and implementation to diverge.

---

## Architectural Summary

> **Lumen surrounds probabilistic model execution with independently bounded services that observe, reproduce, repeat and assess behaviour while preserving the provenance required to explain what changed and under what conditions.**

The architecture exists to make Reasoning Assurance an evidence-producing engineering process rather than an assumption about model capability.
