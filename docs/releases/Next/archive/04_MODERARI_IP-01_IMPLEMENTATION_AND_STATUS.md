# Lumen Moderari IP-01 / IP-01.1 Implementation Plan and Status

**Status:** Living implementation record  
**Date:** 02026-10-10  
**Branch:** `research-foundation-next`  
**Baseline:** `main` remains the M0.1 Research Foundation Build

## 1. Objective

IP-01 establishes the standalone Moderari interaction-processing foundation before reconnecting it to the wider Lumen stack.

The implementation strategy deliberately validates the semantic architecture before integrating SIE into service runtime.

## 2. Governing Implementation Order

The implementation has followed:

```text
Architecture decisions
      ↓
CIM 1.0 candidate
      ↓
OpenAI + Anthropic mapping candidates
      ↓
Conformance harness
      ↓
CIM/mapping adversarial tests
      ↓
OUT/loss semantics
      ↓
Production BaseAdapter foundation
      ↓
Production protocol Adapters
      ↓
Adapter resolution + SpAd inheritance
      ↓
Real Qwen Tool SpAd
      ↓
Production BaseFilter + System Prompt Filter
      ↓
SIE discovery
      ↓
SIE execution integration
```

## 3. Completed Work

### IP-01.1 Milestone 1–4D.1 — Conformance foundation

Completed:

- isolated CIM/Adapter conformance harness;
- deterministic tree walking and wildcard/index capture;
- mapping loading;
- Draft 2020-12 schema validation;
- IN coverage accounting;
- OpenAI fixtures;
- Anthropic fixtures;
- adversarial instruction chronology;
- multimodal/image cases;
- tool definitions, tool calls and tool results;
- tool-choice cases;
- Anthropic thinking/signed/redacted cases;
- cross-protocol semantic comparison;
- one-way cross-protocol materialisation;
- full return-journey testing;
- formal `PASS`, `PASS_WITH_SEMANTIC_LOSS`, `FAIL` architecture.

### Milestone 5A — BaseAdapter foundation

Completed:

- production `adapters` package independent of the conformance package;
- BaseAdapter boundary/orchestration contract;
- external schema validation;
- CIM validation;
- lossless-IN coverage enforcement;
- production materialisation result/loss types;
- production-independent mapping/path/tree utilities.

### Milestone 5B — Production protocol Adapters

Completed:

- `OpenAIChatCompletionsAdapter`;
- `AnthropicMessagesAdapter`;
- explicit Adapter/CIM version checks;
- production cross-interface conformance tests;
- PASS, semantic-loss and genuine failure cases.

### Milestone 5C / 5C.1 — Adapter resolution and SpAd inheritance

Completed:

- Adapter registry;
- Base Adapter registration;
- Specialised Adapter registration;
- exact provider/model resolution;
- provider/model-family resolution;
- Base fallback;
- ambiguity failure;
- direction-independent resolution;
- normal inheritance producing one effective Adapter.

### Milestone 5D / 5D.1 — Qwen Tool SpAd

Completed:

- first genuine production SpAd;
- scoped Qwen/Ollama tool-response compatibility correction;
- deterministic normalisation of supported assistant-content tool-call forms;
- preservation of existing native OpenAI `tool_calls`;
- deterministic correlation IDs where required;
- no mutation of the original response;
- same-interface compatibility correction without CIM expansion.

The Qwen SpAd establishes the rule that provider/model outliers belong in SpAd code rather than in an ever-growing Base Adapter mapping dictionary.

### Milestone 5E — System Prompt Filter

Completed:

- production BaseFilter contract;
- production SystemPromptFilter;
- CIM input/output validation;
- insert initial system prompt;
- replace existing initial system prompt;
- preserve later system instructions;
- no persistence/provider/protocol knowledge inside the Filter;
- deterministic transformation evidence.

## 4. Current Architectural Baseline

The implementation now proves three distinct extension concepts with production examples:

```text
Base Adapter
   ├── OpenAI Chat Completions
   └── Anthropic Messages

Specialised Adapter
   └── Qwen Tool SpAd

Filter
   └── System Prompt Filter
```

This is intentionally the point at which SIE discovery becomes useful: discovery will operate over real production component contracts rather than speculative abstractions.

## 5. Next Milestone — SIE Component Discovery

The next implementation target is unified discovery/registration for:

- Base Adapters;
- Specialised Adapters;
- Filters.

Discovery must preserve category-specific contracts rather than flattening all components into one generic plugin type.

It should provide deterministic identity/version registration and startup failure on ambiguous/duplicate identities.

SpAd discovery must retain interface/provider/model applicability. Filter discovery must retain CIM version, execution mode and EMD requirements.

## 6. Mandatory Component Boundary Validation

A key rule established during Milestone 5D is:

> **Adapter internals are extensible; Adapter boundaries are not. Every Adapter operation MUST emit a representation conforming to its declared destination contract.**

For SpAds this means output must conform either to the published/supported external-interface schema or, when producing CIM directly, the active CIM schema.

For Filters, output must conform to the active CIM schema.

This should become enforceable conformance/discovery metadata rather than relying on convention.

## 7. Work After Discovery

Following discovery, the intended sequence is:

- construct SIE execution resolution around discovered components;
- materialise immutable Session Execution Plan/Execution Instance;
- integrate the request path;
- define the complete provider-response Adapter/CIM path without distorting CIM for provider/model outliers;
- integrate EMD applicability;
- connect Moderari configuration;
- integrate evidence;
- connect companion-service changes;
- exercise end-to-end interoperability.

## 8. Explicitly Deferred

Not required before discovery:

- generic mapping support for arbitrary SpAd oddities;
- complete provider-response CIM path;
- dynamic arbitrary third-party Python loading/security model;
- multi-CIM-version runtime negotiation;
- Cognition Filter;
- Compaction Filter;
- live Praebere/Pontis/Rogare integration.

## 9. Acceptance Direction

IP-01 foundation is successful when Moderari can resolve a session's exact interaction-processing components, execute them deterministically through SIE, preserve/transform semantics according to the Adapter/Filter contracts, produce explicit evidence, and do so without embedding protocol/model peculiarities into SIE itself.

## 10. Documentation Status

The previous 13 architecture/implementation documents have been retained unchanged under `archive/original-13/`.

The authoritative current set is now:

1. `01_MODERARI_SIE_ARCHITECTURE.md`
2. `02_MODERARI_CIM_ADAPTER_ARCHITECTURE.md`
3. `03_MODERARI_FILTER_ARCHITECTURE.md`
4. `04_MODERARI_IP-01_IMPLEMENTATION_AND_STATUS.md`
5. `05_MODERARI_CONFORMANCE_SPECIFICATION.md`
6. `06_LUMEN_COMPANION_SERVICE_CHANGES.md`

The archive remains useful as design provenance but is no longer the primary source of current architectural truth.
