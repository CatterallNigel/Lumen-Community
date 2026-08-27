# Lumen Market Capability Matrix

**Status:** Living research matrix  
**Initial review:** 2026-08-26

## Purpose

This matrix compares Lumen with relevant AI observability, evaluation and experimentation systems.

It is not intended to prove that Lumen is unique. Its purpose is the opposite: to make overlap explicit so that Lumen's genuine differentiation can be identified and tested.

Ratings are preliminary and should be revised whenever deeper research contradicts them.

## Relationship Summary

| System | Primary observed category | Relationship to Lumen | Overall functional overlap | Current conceptual overlap |
|---|---|---|---|---|
| Opik | Observability, evaluation, behavioural/regression testing, optimisation | Partial competitor / adjacent | High | Medium–High |
| LangSmith | Agent observability, evaluation, testing, deployment | Partial competitor / adjacent | High | Medium–High |
| Langfuse | Observability, evaluation, experiments, prompt management | Partial competitor / adjacent | High | Medium |
| Arize Phoenix | Observability, evaluation, experiments, replay, prompt engineering | Partial competitor / adjacent | High | Medium–High |

"Conceptual overlap" is deliberately more cautious than feature overlap. It asks whether the product appears to pursue the same assurance objective, not whether it contains similar mechanisms.

## Capability Matrix

Legend: **● strong/current capability**, **◐ partial/adjacent or requires qualification**, **○ no equivalent established in initial review**, **? requires deeper research**.

| Capability / responsibility | Lumen | Opik | LangSmith | Langfuse | Arize Phoenix |
|---|:---:|:---:|:---:|:---:|:---:|
| LLM/agent tracing | ● | ● | ● | ● | ● |
| Tool/intermediate-step capture | ● | ● | ● | ● | ● |
| Multi-turn/session grouping | ● | ◐ | ● | ● | ◐ |
| Dataset/test-case management | ◐ | ● | ● | ● | ● |
| Repeatable experiments | ● | ● | ● | ● | ● |
| Automated evaluation/scoring | ◐ | ● | ● | ● | ● |
| Intermediate/trajectory evaluation | ◐ | ● | ● | ◐ | ◐ |
| Prompt/model comparison | ● | ● | ● | ● | ● |
| Production quality monitoring | ◐ | ● | ● | ● | ● |
| Prompt optimisation | ○ | ● | ◐ | ◐ | ● |
| Replay of recorded model invocation | ● | ? | ? | ? | ● |
| Full historical interaction reproduction | ● | ? | ? | ? | ? |
| Divergence/fork lineage from replay | ● | ? | ? | ? | ? |
| Preserve original orchestration/system-prompt context for replay | ● | ? | ? | ? | ? |
| Provider abstraction/control | ● | ◐ | ◐ | ◐ | ● |
| Explicit objective/fact/decision continuity | ● | ○* | ○* | ○* | ○* |
| Unresolved-question/assumption/correction continuity | ● | ○* | ○* | ○* | ○* |
| Provenance across evolving reasoning/context | ● | ? | ? | ? | ? |
| Behavioural characterisation across repeated executions | ◐ | ◐ | ◐ | ◐ | ◐ |
| Longitudinal behavioural change evidence | ◐ | ◐ | ◐ | ◐ | ◐ |
| Independent specialist assessors/evidence providers | Planned | ◐ | ◐ | ◐ | ◐ |
| External evidence accepted without owning source | Architectural direction | ? | ? | ? | ? |
| Reasoning Assurance as primary product objective | ● | ○* | ○* | ○* | ○* |

`○*` means **no equivalent was established in this initial documentation review**, not a claim that the product cannot provide it.

## What the Matrix Already Tells Us

### 1. Tracing is commodity capability

Every initial system provides substantial tracing. Vestigare remains necessary to Lumen's architecture, but tracing alone cannot define Lumen's market position.

### 2. Experiments and evaluation are also crowded

All four systems support datasets and repeatable experiments. All provide evaluation/scoring mechanisms. Aestimare therefore cannot be differentiated merely by saying that it evaluates repeated model executions.

### 3. "Behaviour" is not unique terminology

Opik explicitly supports behavioural test suites and agent trajectory evaluation. Langfuse describes repeatable checks of application behaviour. LangSmith evaluates agent quality and intermediate steps. Lumen needs a more precise meaning for **behavioural characterisation** than generic behavioural testing.

### 4. Replay is not unique

Phoenix explicitly supports Span Replay, and several platforms support backtesting or re-running datasets/application versions. Repetere's differentiation must come from its replay semantics and its relationship to provenance, trace lineage, divergence and assurance evidence.

### 5. The candidate whitespace is a combination, not an isolated feature

The strongest current candidate for Lumen differentiation is the combination of:

**continuity + provenance + controlled historical reproduction + divergence lineage + repeated behavioural experiments + longitudinal evidence + assurance interpretation.**

No conclusion should yet be drawn that no competitor provides this combination. That is what this research stream must test.

## Candidate Lumen Market Definition

The current landscape suggests the following working distinction:

> **Observability systems preserve executions so engineers can inspect what happened. Evaluation systems score executions so teams can determine whether applications meet defined criteria. Optimisation systems use those results to improve performance. Lumen is being developed to preserve and reproduce the evidence required to characterise how model behaviour and reasoning context develop, vary, and change over time — so confidence can be based on accumulated evidence rather than a single successful execution.**

This is a **research hypothesis**, not approved marketing language.

## Engineering Consequence

Before adding a generic observability, evaluation, experiment-management, prompt-management or replay feature, Lumen development should ask:

1. Is this capability already well served by mature external systems?
2. Does Lumen require ownership of it to preserve assurance semantics?
3. Could the external system instead become an evidence provider?
4. What Lumen-specific provenance or continuity requirement would be lost through integration?

This prevents Lumen from expanding into a generic AI observability platform and concentrates engineering effort on the Reasoning Assurance layer.

## Next Research Priorities

The initial matrix exposes four areas worth deeper comparison when development time permits:

1. **Replay semantics** — especially Phoenix and LangSmith backtesting versus Repetere.
2. **Session/thread continuity** — especially LangSmith threads and Langfuse sessions versus Lumen continuity.
3. **Behavioural evaluation** — Opik behavioural test suites versus Aestimare behavioural characterisation.
4. **External evidence interoperability** — whether OpenTelemetry traces and external experiment data can become Aestimare evidence without losing provenance.

These are research tasks, not current M0.1 release requirements.
