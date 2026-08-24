# Periti Assessores

## Overview

**Periti Assessores** are the specialist assessors within **Lumen Aestimare**.

Aestimare coordinates assessment, but it should not become a single monolithic assessment algorithm. Different questions about model behaviour require different evidence, methodologies and interpretations. Those concerns are separated into specialist assessors so that each can develop independently while contributing evidence to the wider Aestimare assessment.

An individual specialist assessor is a **Peritus**. Collectively, they are the **Periti Assessores** — Expert Assessors.

## Architectural Principle

A Peritus should exist because a **distinct assessment question has been identified**, not because the architecture requires another assessor.

There is deliberately no predetermined number or complete taxonomy of Periti.

New assessors should emerge as Lumen research identifies assessment concerns that:

- represent a distinct question about model behaviour, performance or fitness for purpose;
- require their own evidence or interpretation;
- can be evaluated meaningfully as a specialised concern; and
- provide findings useful to Aestimare's broader assessment.

This allows the assessment architecture to grow from evidence and research rather than from an arbitrary classification designed in advance.

## Responsibilities

Each Peritus is responsible for a specialised assessment domain.

A Peritus may:

- consume evidence captured elsewhere in the Lumen ecosystem;
- define domain-specific observations and measurements;
- apply its own assessment methodology;
- distinguish measured evidence from inference or estimation;
- produce findings within its defined area of responsibility; and
- evolve independently as its methodology matures.

A Peritus should not attempt to become a complete assessment of model quality.

Its findings represent one specialised dimension of the evidence available to Aestimare.

## Aestimare's Responsibility

Aestimare remains the coordinating assessment service.

Its role is to make specialist assessment available as part of a broader Reasoning Assurance process rather than requiring every concern to be implemented inside one assessment mechanism.

Conceptually:

```text
Aestimare
    |
    +-- Periti Assessores
            |
            +-- Oeconomia
            +-- future Peritus
            +-- future Peritus
```

Not every execution or assessment necessarily requires every Peritus. Which specialist assessments are relevant may depend upon the task, available evidence and purpose of the assessment.

## Independence

Periti should remain independently understandable and independently evolvable.

Two Periti may examine evidence from the same execution while asking different questions and reaching findings within different domains.

For example, economic efficiency is not equivalent to behavioural stability, task sufficiency or another future assessment concern. Correlation between findings does not require those concerns to be collapsed into a single assessor.

This separation allows Aestimare to preserve the distinction between different forms of evidence while still bringing them together when a wider assessment is required.

## Naming Convention

Specialist assessors follow the naming convention:

**Aestimare `<Latin Domain Name>` — `<English Role>`**

The Latin name identifies the assessment domain within the Lumen family. The English description makes the responsibility explicit.

The implementation/documentation directory uses the lowercase Latin domain name.

For example:

```text
periti_assessores/
├── README.md
└── oeconomia/
    └── AESTIMARE_OECONOMIA—ECONOMIC_...
```

## Defined Periti

### Aestimare Oeconomia — Economic Efficiency Assessor

**Oeconomia** is the first defined Peritus.

It examines the economics and efficiency of model use beyond simple token pricing. Its scope includes the relationship between satisfactory task outcomes and the total cost of obtaining and consuming those outcomes.

This can include:

- input and output token cost;
- prompt and context burden;
- clarification and retry cost;
- output efficiency;
- human comprehension and consumption burden;
- execution and tool cost; and
- comparative fitness for purpose across models or task classes.

Its central premise is that computational cost alone does not describe the economics of model use. A model that is inexpensive per token may still be expensive to use if it requires substantial human effort, while a more expensive model may be economically preferable when it reaches a satisfactory result with less interaction.

See the `oeconomia/` directory for the detailed Oeconomia design material.

## Evolution

The current Periti structure is intentionally small.

Oeconomia establishes the first concrete specialist assessor and demonstrates the architectural pattern. Additional Periti should be added only when research or observed model behaviour establishes a sufficiently distinct assessment concern.

This keeps Aestimare evidence-led:

> **Identify the assessment question first; define the Peritus second.**
