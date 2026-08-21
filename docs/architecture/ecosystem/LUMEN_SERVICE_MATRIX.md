# Lumen Service Matrix

**Status:** Canonical  
**Document role:** Current service responsibility and integration index  
**Last reviewed:** 2026-08-21

---

## Purpose

This matrix provides a concise current index of the Lumen services. It is intentionally shorter than the service architecture documents and should be updated whenever a service's primary responsibility, dependency boundary, interface or maturity changes.

| Service | One job | Primary inputs | Primary outputs | Key dependencies / relationships | Canonical local port | Current maturity | Primary documentation |
|---|---|---|---|---|---:|---|---|
| **Fiducia** | Schedule and orchestrate repeated Replay execution | Prepared Replay definitions, schedules | Replay execution references and orchestration history | Repetere, Pontis, MongoDB; managed by Servire | `11430` | **Implemented / stable initial capability** | [`../../services/fiducia/README.md`](../../services/fiducia/README.md) |
| **Praebere** | Abstract model-provider lifecycle and readiness | Provider/model configuration; lifecycle/readiness requests | Provider status, model state, lifecycle actions | External model provider; Servire; Moderari boundary | `11431` | **Implemented / integrated** | [`../../services/praebere/README.md`](../../services/praebere/README.md) |
| **Rogare** | Provide the human conversational and operations client | User interaction | Conversational requests, operational actions, rendered responses/status | Pontis; service operational APIs; Servire | `11432` | **Implemented / integrated** | [`../../services/rogare/README.md`](../../services/rogare/README.md) |
| **Pontis** | Bridge client/session traffic and external capability-provider protocols | Client HTTP; ACP/provider traffic | Opaque downstream HTTP; correlated provider/session responses | Rogare, Vestigare, external tool providers, Servire | `11435` | **Implemented / operational** | [`../../services/pontis/README.md`](../../services/pontis/README.md) |
| **Moderari** | Orchestrate model requests and Lumen policy/context intervention | OpenAI-compatible model requests | Provider-bound model requests and orchestrated responses | Repetere upstream; Praebere/provider boundary; external capabilities where invoked | `11436` | **Implemented / operational** | [`../../services/moderari/README.md`](../../services/moderari/README.md) |
| **Repetere** | Reproduce recorded executions and identify divergence | Recorded/prepared Replay evidence; live replay traffic | Replay runs, match/divergence evidence | Vestigare evidence; Moderari downstream; Fiducia scheduler | `11437` | **Implemented / mature initial capability** | [`../../services/repetere/README.md`](../../services/repetere/README.md) |
| **Vestigare** | Transparently record execution evidence | Model-request HTTP traffic | Persisted trace/evidence | Pontis upstream; Repetere downstream; MongoDB | `11438` | **Implemented / operational** | [`../../services/vestigare/README.md`](../../services/vestigare/README.md) |
| **Servire** | Operate the Lumen service stack | Service configuration, dependency state, operator actions | Lifecycle actions, operational state, aggregated logs/navigation | Managed Lumen services; infrastructure dependencies | `11439` | **Implemented / operational control plane** | [`../../services/servire/README.md`](../../services/servire/README.md) |
| **Aestimare** | Assess accumulated behavioural evidence | Trace/Replay/Fiducia evidence and specialist assessment results | Consolidated behavioural assessments | Vestigare, Repetere, Fiducia evidence; future internal specialist engines | — | **Active design / development** | [`../../services/aestimare/README.md`](../../services/aestimare/README.md) |
| **Audire** | Carry distributed operational commands and service output without changing Servire's responsibility | Servire lifecycle commands; local service stdout/stderr/status | Brokered operational events, acknowledgements and service output | Servire; local service process; proposed message broker such as RabbitMQ | — | **Architectural direction / not implemented** | [`../../services/audire/README.md`](../../services/audire/README.md) |

---

## External Dependencies and Providers

These are important to the runtime topology but are **not Lumen services**.

| Dependency / provider | Role |
|---|---|
| **MongoDB** | Persistent storage used by evidence, replay, continuity and orchestration capabilities. |
| **Redis** | Infrastructure dependency where required by a service/configuration; not part of the Lumen service identity. |
| **Ollama** | Current principal development model-provider implementation. Access/lifecycle is abstracted through Lumen boundaries; Ollama is not architecturally mandatory. |
| **Pi** | Current important ACP/tool-provider implementation and architectural test case; not the definition of Pontis or the tool-provider contract. |
| **RabbitMQ** | Candidate future transport for Audire's distributed operational messaging; not currently a required Lumen runtime dependency. |

---

## Responsibility Chain

For reasoning-assurance evidence, the intentionally separated responsibilities are:

```text
Vestigare  -> records what happened
Repetere   -> reproduces what happened
Fiducia    -> repeats under controlled scheduling
Aestimare  -> assesses what the accumulated evidence means
```

For live model interaction:

```text
Rogare -> Pontis -> Vestigare -> Repetere -> Moderari -> model provider
```

For operations:

```text
Operator -> Servire -> managed Lumen services / dependencies
```

Audire is intended eventually to change the **transport** of the operational relationship, not Servire's responsibility.

---

## Maturity Labels

The maturity column is deliberately coarse and describes current engineering state rather than product readiness:

- **Implemented / operational** — executable and integrated in the current stack.
- **Implemented / stable initial capability** — first bounded capability completed and stabilised; further development expected.
- **Active design / development** — architecture and development work exists, but the service must not yet be represented as an equivalent operational capability.
- **Architectural direction / not implemented** — documented future architecture without current implementation evidence.
