# Lumen Documentation

This directory contains the working architectural, engineering, research and release documentation for **Lumen — the Reasoning Assurance Service**.

Lumen preserves the evidence needed to understand, reproduce and assess model behaviour across sessions. The current M0.1 External Research Distribution is centred on:

> **Observe → Reproduce → Repeat**

The documentation is organised by **purpose first**, then by service where appropriate.

## Start here

- **architecture/** — canonical core, ecosystem, installation, security and extension architecture.
- **services/** — current documentation for each Lumen service, together with historical release and milestone notes.
- **research/** — model behaviour, capability, decision quality, behavioural research and future assessment work.
- **roadmaps/** — the current development roadmap plus dated and historical plans.
- **engineering/** — the Engineering Diary, implementation narratives, carry-over documents and development digests.
- **standards/** — engineering, documentation, UI and operational standards.
- **vision/** — product, service and UI vision documents.
- **evidence/** — test logs, screenshots and other captured engineering evidence.
- **publications/** — outward-facing articles and long-form explanatory material.
- **brand/** — Lumen visual identity and brand guidance.
- **site/** — planning and content for the Lumen microsite.
- **archive/** — retained historical material that is no longer part of the active navigation path.

## Current architectural boundaries

The following responsibilities should be used when reading or updating the documentation:

| Area | Current authority |
| --- | --- |
| Operational control | **Servire** is the operational authority and supported control plane for starting, stopping and supervising the Lumen service group. |
| Common control path | **Nuntius** provides the shared control and diagnostic path used by the operator-facing interfaces. |
| Client and execution sessions | **Pontis** owns session lifecycle, selected-model enforcement, request correlation and the bridge into execution. |
| Model use | **Moderari** determines how the selected model is used, including orchestration, prompting and tool behaviour. It does not own provider readiness or model lifecycle. |
| Provider and model lifecycle | **Praebere** owns provider discovery, model discovery, runtime-global model selection, reservation, readiness and model residency caused by Lumen. |
| Trace evidence | **Vestigare** records execution evidence and correlated traces, including multiple active sessions. |
| Reproduction and experiments | **Repetere** owns experiment/run persistence, fresh replay-session creation and reproducible execution. |
| Scheduling and assurance workflow | **Fiducia** coordinates scheduled and assurance-related work. |
| User interaction | **Rogare** provides the research and session interface, including model selection during session establishment. |

## Current provider lifecycle

Ollama is **external infrastructure**. Lumen verifies its availability but does not start or stop the Ollama server.

- Selecting a model records runtime intent; it does not load the model.
- The first ask demand-loads the selected model when required.
- The selected model is reserved while active execution sessions depend on it.
- Praebere tracks only model residency initiated by Lumen.
- When the final dependent session ends, Praebere may unload a model it caused to become resident.
- A model that was already resident before Lumen used it is left untouched.

These rules supersede older documents that describe Lumen as owning the Ollama process or eagerly loading a model during stack or session startup.

## M0.1 distribution boundary

The M0.1 External Research Distribution contains Rogare, Pontis, Vestigare, Repetere, Fiducia, Moderari, Praebere, Servire, Nuntius and MongoDB on a private Docker network. **Aestimare is not included in M0.1.**

The supported runtime is Servire-controlled. Installation identity uses a Service Group UUID and cryptographic identity; protected configuration is unlocked through runtime authorisation over TLS/HTTPS with Illuminates.One. Directly starting individual protected services is not the supported operating model.

## Documentation rule

Each significant topic should have one canonical current engineering document. Other material must be identified and treated as one of:

- implementation history;
- captured evidence;
- a summary or carry-over note;
- a future-development proposal; or
- superseded context.

Where a historical document conflicts with the current canonical architecture, the canonical document and the boundaries stated above take precedence. Do not silently rewrite historical evidence to match the present design; mark or register it as superseded instead.

## Canonical current indexes

- [Canonical Lumen architecture](architecture/core/ARCHITECTURE.md)
- [Current ecosystem architecture and topology](architecture/ecosystem/LUMEN_ECOSYSTEM_CURRENT_ARCHITECTURE.md)
- [Canonical service matrix](architecture/ecosystem/LUMEN_SERVICE_MATRIX.md)
- [Documentation status and lifecycle convention](standards/DOCUMENT_STATUS_AND_LIFECYCLE.md)
- [Superseded document register](standards/SUPERSEDED_DOCUMENT_REGISTER.md)

When an architectural or ownership decision changes, update the relevant canonical document, service matrix and superseded-document register before relying on milestone notes or the Engineering Diary as the current definition.
