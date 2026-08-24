# Lumen Research

This directory contains the research foundations, research questions, experimental themes, and cross-cutting research concerns that inform the development of Lumen.

Lumen's research work is concerned primarily with the observable behaviour of model-based systems and the evidence required to understand, reproduce, compare, and assess that behaviour.

The research material is deliberately separated from implementation documentation. A research question may influence future Lumen development without implying that a particular feature, mechanism, or conclusion has already been adopted.

## Research Areas

### Decision Quality

`decision-quality/`

Research concerned with the quality and sufficiency of model decisions under bounded conditions.

This includes questions around satisficing, evidence, uncertainty, resource constraints, and the distinction between an acceptable decision and an assumed ideal or optimal answer.

### Model Behaviour

`model-behaviour/`

Research concerned with how models behave across executions, contexts, conditions, and time.

This includes repeatability, divergence, behavioural change, context effects, prompt effects, and investigation into whether computational resource constraints can produce observable behavioural differences.

### Model Capability

`model-capability/`

Research concerned with what a model is capable of doing and how that capability should be distinguished from the quality, reliability, or trustworthiness of an individual response.

This area supports the broader Lumen distinction between capability and demonstrated behaviour.

## Cross-Cutting Research

Some research documents do not belong exclusively to one research domain.

These remain at the root of `docs/research/` until a sufficiently coherent body of work exists to justify a dedicated research area.

Examples include research infrastructure, experimental operation, external research distribution, and concerns that span several research domains.

## Research Ecosystem

`LUMEN_RESEARCH_ECOSYSTEM.md` describes the wider relationship between Lumen's research themes and should be used as the principal orientation document for the research programme.

The individual research directories and notes develop specific questions within that broader framework.

## Relationship to Lumen Development

Research documentation may:

- identify questions that Lumen should make experimentally observable;
- propose future experiments;
- identify variables that should be captured as evidence;
- expose architectural questions;
- inform future Aestimare assessment work;
- motivate later service capabilities.

Research documentation does not, by itself, define a release requirement.

Where research produces a concrete engineering requirement, that requirement should also be represented in the appropriate service, architecture, development, or release documentation.

## Current Structure

```text
docs/research/
├── README.md
├── LUMEN_RESEARCH_ECOSYSTEM.md
├── MULTI-SESSION_OPERATION_AND_RESEARCH_DISTRIBUTION_ACCESS_CONTROL.md
├── decision-quality/
├── model-behaviour/
└── model-capability/
```

The structure should evolve only when the research material justifies a new coherent area. Individual notes should not create new top-level categories unnecessarily.

## Guiding Principle

Lumen research is intended to build evidence about model behaviour rather than assume conclusions about model intelligence, correctness, or trustworthiness.

A single execution is an observation. Repeated controlled executions become an experiment. Multiple experiments begin to characterise behaviour. Longitudinal evidence allows us to detect behavioural change.
