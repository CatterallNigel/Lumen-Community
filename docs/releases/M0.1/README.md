# Lumen M0.1

**Release:** M0.1  
**Distribution:** Lumen External Research Distribution  
**Status:** In Development

## Overview

M0.1 is the first defined Lumen release.

Its purpose is to establish a coherent external research baseline for observing, reproducing, and repeatedly executing model behaviour under explicit and reproducible experimental conditions.

M0.1 is deliberately bounded. It is not intended to represent the complete Lumen ecosystem or the eventual commercial distribution.

The release concentrates on the evidence-producing path required to support controlled behavioural research:

> **Observe → Reproduce → Repeat**

The evidence produced by this release provides the foundation upon which later assessment and Reasoning Assurance capabilities can be built.

Aestimare is explicitly not part of M0.1.

## Release Documentation

The M0.1 release documentation is divided between the **reviewed release definition** and supporting documents that remain **work in progress**.

Documents in the root of this directory form the current reviewed definition of the M0.1 External Research Distribution.

Documents under `work-in-progress/` remain under active review. They are provided for context and implementation guidance, but should not yet be treated as definitive release documentation. When a work-in-progress document completes review, it may be promoted into the M0.1 release root.

### Definitive M0.1 Documents

#### `LUMEN_DEVELOPMENT_REQUIREMENTS_M0.1.md`

Defines the engineering requirements and investigations identified as necessary for M0.1, together with the internal M0.1 acceptance criteria.

It includes:

- Experiment structure and repeated execution;
- Repetere divergence visibility;
- Fiducia coordination of Experiment runs;
- configurable Moderari system-prompt policy;
- saved and reusable custom system prompts;
- multi-session isolation validation;
- Nuntius and the common `\obt` control-command path;
- Praebere provider/model discovery and selection;
- Trace boundaries for control-plane traffic;
- investigation of provider/model Trace provenance and Replay binding;
- internal acceptance criteria for the M0.1 release candidate.

This document describes **what needs to be developed, investigated or validated before M0.1 is internally accepted**.

#### `LUMEN_EXTERNAL_RESEARCH_DISTRIBUTION_M0.1_REQUIREMENTS_AND_LIMITATIONS.md`

Defines the release boundary.

It records:

- the services included in M0.1;
- functional requirements;
- research requirements;
- required validation;
- known limitations;
- explicitly excluded capabilities;
- work required before wider general research distribution;
- M0.1 completion criteria.

This document describes **what M0.1 is — and what it is not**.

#### `LUMEN_EXTERNAL_RESEARCH_DISTRIBUTION_M0.1_ROADMAP.md`

Orders the M0.1 work into a practical engineering sequence.

The roadmap begins by establishing the behaviour of the existing implementation before introducing changes and concludes with **Internal M0.1 Acceptance Validation**.

This document describes **how the work is expected to progress toward the defined M0.1 release and its internal acceptance**.

### Work-in-Progress Documentation

The following supporting documents remain under active review and are maintained under `work-in-progress/`:

- research licensing;
- runtime authorization and code protection;
- the M0.1 user manual.

These documents remain relevant to M0.1, but their current contents are informational and subject to further review or implementation decisions. Their presence under `work-in-progress/` makes that status explicit.

## Intended M0.1 Stack

The current M0.1 release definition includes:

- Rogare;
- Pontis;
- Vestigare;
- Repetere;
- Fiducia;
- Moderari;
- Praebere;
- Servire;
- Nuntius;
- MongoDB as supporting persistence.

The intended distribution is Dockerised and operates over a private Lumen Docker network.

## Release Boundary

M0.1 is intended to provide researchers with the ability to:

- execute model interactions through Lumen;
- preserve the effective execution context;
- reproduce a recorded execution;
- create controlled repeated executions as an Experiment;
- see objective replay divergence;
- control system-prompt policy;
- reuse saved experimental system prompts;
- discover and select model/provider capabilities through the Lumen control path;
- retain clean execution evidence;
- operate against a stack whose concurrent-session isolation has been validated;
- operate as an authorised installation whose runtime authorization is established and maintained by Servire;
- use protected Lumen services through the supported Servire-controlled lifecycle;
- use protected operational configuration only through the authorised runtime mechanism.

M0.1 does not attempt to assess the significance or quality of the resulting behaviour.

That distinction is intentional.

Repetere can establish that an execution diverged.

M0.1 does not claim whether that divergence is important, acceptable, beneficial, harmful, or evidence of a broader behavioural characteristic.

Those questions belong to later assessment work.

## Explicitly Outside M0.1

The current release boundary excludes:

- Aestimare behavioural assessment;
- Rogare human correctness/quality ratings;
- multiple Pontis tool providers and a unified tool catalogue;
- dedicated bounded-computational-resource behavioural experimentation;
- commercial multi-user/operator entitlement;
- sophisticated shared or versioned prompt libraries;
- durable enterprise messaging for Nuntius.

These may be valid future Lumen capabilities or research areas, but they are not requirements for completing M0.1.

## General Research Distribution

M0.1 and general external research availability are related but not identical milestones.

In particular, wider research distribution is expected to require additional access-control behaviour in Servire so that one installation permits a single active operator session while still allowing legitimate remote browser operation.

That requirement should not be confused with Lumen's underlying session architecture.

M0.1 must still validate that multiple simultaneous Lumen sessions remain correctly isolated.

## Documentation Lifecycle

The documents in this directory are release-specific.

While M0.1 is under development, definitive documents may be updated through review and work-in-progress documents may be promoted into the release root once their contents are considered reviewed and release-defining.

Once M0.1 is released, the documents in the release root should become the historical record of the release baseline rather than being continually rewritten to describe later Lumen versions.

Requirements introduced for subsequent releases should be documented under their corresponding release directory.

For example:

```text
docs/releases/
├── M0.1/
└── M0.2/
```

## Guiding Principle

M0.1 preserves the broader Lumen principle:

> **Experimental conditions should be explicit, reproducible and observable. Lumen should record what happened without unnecessarily altering the conditions being investigated.**

The objective of the first release is not to answer every question about model behaviour.

It is to establish a reliable foundation from which those questions can be investigated.
