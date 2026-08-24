# Lumen Development Roadmap Overview

**Date:** 2026-08-22  
**Status:** Forward-looking development overview  
**Location:** `development/future/ROADMAP_OVERVIEW_2026-08-22.md`

## Purpose

This document captures the current high-level development direction for Lumen following the preparation of the M0.1 external research release.

It is intentionally an **overview rather than a committed delivery schedule**. The order and scope may change as M0.1 is exercised externally and as research evidence becomes available.

The current direction can broadly be described as:

**M0.1 → External Evaluation → Release Generalisation → Full Research Release → Lumen Website → Aestimare**

Aestimare development does not need to wait until the end of this sequence. Its foundational architecture can develop alongside the other work as evidence and requirements emerge.

---

## 1. M0.1 — First External Research Release

The immediate priority is M0.1.

M0.1 is intended to establish that somebody outside the Lumen development environment can:

- install the distribution;
- configure it;
- start and stop the required services;
- understand the operational state through Servire;
- conduct useful experiments;
- use Trace and Replay;
- understand the principal workflows through documentation;
- operate within the intended licensing and security model.

M0.1 does **not** need to demonstrate every aspect of Lumen's eventual provider, model and tool independence.

Its primary purpose is to prove that Lumen can move successfully from an internal development system to a controlled external research distribution.

---

## 2. External Evaluation

The first external use of M0.1 should be treated as an important part of development rather than merely as validation of a completed release.

External use is expected to expose issues that are difficult to identify from within the development environment, including:

- installation friction;
- configuration assumptions;
- unclear workflows;
- documentation gaps;
- operational usability issues;
- missing diagnostics;
- unexpected failure modes;
- assumptions embedded in individual services;
- research workflow requirements.

These findings should feed directly into the development of the subsequent research distribution.

### Research Evidence

Where practical and with the researcher's agreement, the underlying experimental evidence generated during external evaluation could be particularly valuable.

This may include:

- traces;
- replay executions;
- divergence evidence;
- repeated experimental runs;
- model responses;
- configuration and execution metadata;
- observations and research conclusions.

This evidence could provide an important empirical foundation for the later development of **Lumen Aestimare**.

Rather than designing behavioural assessors entirely from hypothetical examples, Aestimare can increasingly be developed against real experimental evidence.

---

## 3. From M0.1 to a Repeatable Release Model

M0.1 should also be considered an experiment in **how Lumen itself should be released**.

Following the first external deployment, the lessons learned should be used to establish a repeatable release model.

This is likely to include further work around:

- reproducible builds;
- release branches and tagging;
- packaging;
- Dockerisation and deployment;
- installation and configuration;
- upgrade and migration procedures;
- licensing lifecycle;
- runtime authorisation;
- heartbeat behaviour;
- security hardening;
- support and issue reporting;
- feature-request handling;
- release documentation;
- user documentation;
- operational diagnostics.

The objective is to move from:

> "We can produce an external build."

to:

> "We have a repeatable process for producing, supporting and evolving external Lumen releases."

---

## 4. Full Research Release

The full research release should go beyond demonstrating that the current implementation can operate externally.

It should provide evidence that the **architecture itself is genuinely portable and extensible**.

A useful distinction is:

> **M0.1 proves the implementation works externally.**

> **The full research release should prove the architecture works externally.**

Several areas are particularly important.

### 4.1 Pontis — Multiple Tool Providers

Pontis should demonstrate that its abstraction is not dependent upon the current Pi/ACP development environment.

Future work should investigate:

- multiple tool providers;
- alternative tool-provider implementations;
- provider discovery and configuration;
- potentially using more than one tool provider within a session;
- capability representation;
- isolation of provider-specific behaviour within the appropriate integration boundary.

The objective is that downstream Lumen services operate against the Pontis abstraction rather than against assumptions about a particular tool provider.

### 4.2 Praebere — Multiple Model Providers

Ollama is the first model-provider implementation for Praebere, but it should not become an architectural dependency.

A second substantially different provider should therefore be implemented.

**LLM Studio** is an obvious candidate.

The desired architecture remains:

`Moderari → Praebere → Provider`

For example:

`Moderari → Praebere → Ollama`

or:

`Moderari → Praebere → LLM Studio`

Moderari should **not** acquire LLM-Studio-specific behaviour.

Provider-specific lifecycle, discovery and integration concerns belong within Praebere and its provider adapters.

Successfully changing model providers without modifying Moderari would provide strong evidence that the Praebere boundary is functioning as intended.

### 4.3 Multiple Models

The full research release should also demonstrate operation against different model families.

This helps identify assumptions that may inadvertently have arisen from development primarily against Qwen and Ollama.

Lumen should remain concerned with model behaviour and evidence rather than requiring characteristics unique to a particular model.

### 4.4 Configuration and Discovery

Development assumptions should progressively become explicit configuration or discoverable capabilities.

Examples include:

- model providers;
- provider endpoints;
- available models;
- tool providers;
- provider capabilities;
- lifecycle capabilities;
- service dependencies.

This should reduce the amount of environment-specific knowledge embedded within individual services.

### 4.5 Servire

Servire should become the operational surface through which these choices remain understandable.

For example, if an installation uses LLM Studio rather than Ollama, Servire should still be capable of showing:

- what provider is configured;
- what models are available;
- what model is selected;
- whether the provider is healthy;
- whether the required model is available;
- what Lumen is currently using;
- whether dependencies required by the stack are satisfied.

The same principle should apply as additional tool providers and other configurable integrations are introduced.

---

## 5. Proving Abstractions

A useful engineering principle for future Lumen development is:

> **A second implementation proves an abstraction.**

An abstraction backed by only one implementation may represent little more than architectural intent.

For example:

`Praebere → Ollama`

demonstrates that Praebere can abstract Ollama.

But:

`Praebere → Ollama`

and

`Praebere → LLM Studio`

provide substantially stronger evidence that Praebere actually represents a model-provider abstraction.

The same reasoning applies to Pontis and tool providers.

Where an architectural boundary is important to Lumen's provider-agnostic design, implementing a genuinely different second integration should be considered an important validation exercise.

---

## 6. Lumen Website

Once the external distribution and release model have matured sufficiently, development of:

`lumen.illuminates.one`

can become a significant focus.

Waiting until this point has advantages.

The website can describe an externally usable system rather than a rapidly changing development environment, and material generated during the release process can be reused.

This may include:

- architecture documentation;
- service descriptions;
- screenshots;
- installation material;
- user documentation;
- research methodology;
- research results;
- diagrams;
- examples;
- release information.

The site should therefore represent the system that researchers can actually obtain and use.

---

## 7. Aestimare Development

Aestimare is expected to become a major development focus after the release infrastructure and external research environment have matured.

However, Aestimare development does not need to remain completely dormant until then.

There is an important distinction between:

**building Aestimare's assessment infrastructure**

and:

**determining what its assessors should actually assess.**

### 7.1 Early Aestimare Infrastructure

Foundational work can be undertaken opportunistically alongside M0.1 and full-release development.

Potential early work includes:

- Aestimare service structure;
- evidence input contracts;
- assessor registration and discovery;
- assessor execution lifecycle;
- assessment result schema;
- provenance requirements;
- evidence references;
- assessor isolation;
- aggregation and presentation mechanisms;
- reference or dummy assessor implementations.

This work establishes the environment into which specialised assessors can later be introduced.

### 7.2 Periti Assessores

Aestimare's specialised internal assessment engines are currently referred to as:

**Periti** — singular **Peritus**

and collectively:

**Periti Assessores — Expert Assessors**

Each Peritus should assess a specialised dimension of model behaviour independently.

Aestimare coordinates these assessors and their evidence rather than becoming one monolithic assessment algorithm.

Conceptually:

`Evidence / Experiments`

↓

`Aestimare`

↓

`Peritus A | Peritus B | Peritus C | ...`

↓

`Individual evidence-based assessments`

↓

`Aestimare coordination / presentation`

### 7.3 Let Evidence Inform the Assessors

The specific Periti should not necessarily all be predetermined.

External research may expose behavioural characteristics, relationships and failure modes that are difficult to anticipate beforehand.

A useful development question when examining research evidence is therefore:

> **What independent forms of assessment would have helped explain this evidence?**

Those questions can inform the creation of specialised Periti.

This allows Aestimare's assessment capabilities to emerge partly from observed behaviour rather than solely from theoretical decomposition.

---

## 8. Evidence Progression

The broader Aestimare direction continues to follow the principle:

> **A single execution is an observation. Repeated controlled executions become an experiment. Multiple experiments begin to characterise behaviour. Longitudinal evidence allows us to detect behavioural change.**

Trace, Replay and the external research environment provide the mechanisms through which this evidence can accumulate.

Aestimare can then operate upon that evidence rather than attempting to infer behavioural characteristics from isolated model responses.

---

## 9. Parallel Development Tracks

The roadmap should therefore not be interpreted as a completely sequential process.

Two tracks can progress alongside one another.

### Primary Release Track

`M0.1`

→ `External Evaluation`

→ `Feedback / Corrections`

→ `Provider and Tool Generalisation`

→ `Release Engineering`

→ `Full Research Release`

→ `lumen.illuminates.one`

→ `Major Aestimare Development`

### Aestimare Foundation Track

`Aestimare Skeleton`

→ `Evidence Model`

→ `Periti Contract`

→ `Assessor Infrastructure`

→ `External Research Evidence`

→ `Identification of Useful Assessment Dimensions`

→ `Real Periti`

→ `Mature Aestimare`

The amount of effort applied to the second track can vary according to available time and what is learned during M0.1 development and external evaluation.

---

## 10. Overall Direction

The immediate objective is not to rush from M0.1 directly into Aestimare.

M0.1 provides an opportunity to establish something equally important: a genuinely external Lumen research environment.

That environment can then:

1. validate the current implementation;
2. expose usability and operational weaknesses;
3. establish the release process;
4. test Lumen's architectural abstractions;
5. generate real experimental evidence;
6. provide material for the Lumen website;
7. provide empirical input into Aestimare.

By the time Aestimare becomes the principal development focus, Trace, Replay, Pontis, Moderari, Praebere, Servire and the external distribution should already provide much of the operational and evidential foundation that it requires.

Aestimare can therefore emerge as the assessment layer over an established experimental system rather than being developed simultaneously with the infrastructure needed to support it.

---

## Status

This document records the development direction as understood on **2026-08-22**.

It is deliberately exploratory.

M0.1 external use, researcher feedback, experimental evidence and subsequent architectural findings should be expected to modify this roadmap.