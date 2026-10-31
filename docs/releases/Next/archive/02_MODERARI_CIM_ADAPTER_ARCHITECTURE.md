# Lumen Moderari CIM and Adapter Architecture

**Status:** Current authoritative architecture  
**Date:** 02026-10-10  
**Scope:** CIM, Base Adapters, materialisation, Specialised Adapters and representation fidelity

## 1. Canonical Interaction Model

> **CIM is a versioned, schema-defined canonical interaction data model. The schema is authoritative; language-specific representations may be generated from it. Adapters map external protocol representations to and from CIM. Adding support for a protocol requires a new Adapter, but requires a CIM change only when that protocol introduces interaction semantics that the current CIM cannot represent and Lumen chooses to preserve.**

CIM is not itself an action or mapping mechanism. It is the canonical semantic representation between external interfaces.

## 2. Semantic Scope

CIM represents interaction semantics, not wire or transport artefacts. HTTP headers, SSE framing, JSON whitespace and semantically insignificant field ordering do not belong in CIM merely because they appear on the wire.

CIM is the **union of supported interaction semantics across supported IN interfaces**, not the lowest common denominator between protocols.

If a supported IN semantic has no CIM representation, that is a **CIM coverage failure**.

## 3. Semantic Fidelity

The governing invariant is:

> **Adapter IN is lossless for all declared supported interaction semantics. Filters may intentionally transform semantics. Adapter OUT may be lossy only where the destination interface cannot represent the resulting CIM, and that loss must be explicit.**

Without a Filter:

```text
semantics(External A) == semantics(CIM)
```

for every supported IN semantic.

On OUT:

```text
semantics(CIM) == semantics(External B)
```

where the destination can represent the CIM semantics.

With a Filter, intentional semantic difference occurs only between the pre-Filter and post-Filter CIM.

## 4. Base Adapter

A Base Adapter describes a declared external interface such as:

- `openai-chat-completions`
- `anthropic-messages`

IN performs extraction and canonicalisation. OUT materialises destination structure from CIM.

The generic Adapter foundation owns validation, mapping/coverage mechanics, tree/path handling and destination contract enforcement. Concrete protocol Adapters own protocol-specific structural knowledge and exceptional deterministic representation transforms.

## 5. Mapping Model

Mappings describe external-interface structure ↔ CIM semantic paths. Collection mappings use wildcard definitions such as:

```text
tools[*].function.name → tools[*].name
```

Observed paths use concrete indices such as `tools[0].function.name`.

Indices identify instances and preserve ordering/grouping while canonicalising. They are not destination coordinates.

A mapping language is not required to express every possible provider/model oddity. Base Adapter mappings describe the declared interface.

## 6. IN Coverage

Every observed field within the supported external-interface subset must be classified as either:

- `MAPPED_SEMANTIC`; or
- `EXCLUDED_NON_SEMANTIC`.

A supported interaction semantic without a CIM mapping is `CIM_COVERAGE_FAILURE`.

Non-semantic exclusions must be explicit specification data rather than silent walker behaviour.

## 7. OUT Materialisation

IN and OUT are deliberately asymmetric.

IN is lossless for the supported semantic subset. OUT may encounter a destination incapable of representing some CIM semantics.

OUT has three architectural outcomes:

### PASS

A valid destination representation is produced and all applicable CIM semantics are faithfully represented.

### PASS_WITH_SEMANTIC_LOSS

> **An Adapter OUT operation has produced a valid destination-interface representation, but one or more CIM interaction semantics could not be represented by that destination interface and were therefore omitted or deterministically reduced. Every such loss MUST be explicitly identified and recorded. The Adapter MUST NOT invent, approximate, reinterpret, or silently transform the missing semantic.**

### FAIL

A valid destination representation cannot be produced without invention, undeclared semantic transformation, or violation of the destination contract.

Decision rule:

```text
Can OUT produce a valid destination representation?
        │
        ├── NO → FAIL
        │
        └── YES
              ↓
Can every applicable CIM semantic be faithfully represented?
        │
        ├── YES → PASS
        │
        └── NO
              ↓
Is missing semantic objectively unrepresentable by destination contract?
        │
        ├── YES → PASS_WITH_SEMANTIC_LOSS
        └── NO / UNKNOWN → FAIL
```

## 8. Cardinality

CIM represents semantic cardinality rather than the narrowest external restriction.

A singular source may canonicalise to a one-member CIM collection without loss.

If CIM contains multiple values and a destination is singular:

- a destination-defined deterministic reduction may produce `PASS_WITH_SEMANTIC_LOSS`, with omitted semantics recorded;
- without a legitimate deterministic rule, OUT fails rather than arbitrarily selecting an item.

## 9. Value Compatibility

Field correspondence does not prove arbitrary value equivalence.

Adapters preserve mapped values unless a known deterministic representation transform is required. Unknown value compatibility is not guessed. If the destination schema explicitly rejects a value, that is objective incompatibility; otherwise emitted representation and subsequent provider/model behaviour become evidence.

## 10. Specialised Adapters

A Specialised Adapter (SpAd) handles deterministic provider/model-specific representation compatibility behaviour while remaining within the same declared interface.

A SpAd is not a cross-protocol Adapter and is not a Filter.

Resolution is direction-independent:

```text
interface + provider + model → one effective Adapter
```

The same effective SpAd may be invoked for `IN` or `OUT`. Through normal inheritance/composition it may override either operation or both; unmodified operations inherit Base Adapter behaviour.

There is no `specialises: in/out` manifest property.

## 11. SpAds and Outliers

A provider/model deviation from a published interface is not automatically a new protocol and does not automatically justify a CIM field or Base Adapter mapping.

> **Base Adapter mappings describe the declared external interface. A Specialised Adapter corrects known deviations from that interface. Deviations do not become part of the Base Adapter mapping or CIM merely because a provider/model exhibits them.**

A SpAd may contain arbitrary deterministic compatibility code required to normalise an outlier. It must not silently change semantic intent.

## 12. Mandatory SpAd Boundary Conformance

> **A Specialised Adapter may perform whatever deterministic representation correction is required for its provider/model, but the representation leaving the SpAd MUST conform either to the published schema of its declared external interface or, where the SpAd operation produces CIM directly, to the active CIM schema. A SpAd may not introduce a private intermediate representation into the SIE execution path.**

More generally:

> **Adapter internals are extensible; Adapter boundaries are not. Every Adapter operation MUST emit a representation conforming to its declared destination contract.**

This rule applies equally to built-in and third-party/user SpAds.

## 13. Qwen Tool SpAd

The first production SpAd addresses the known Qwen/Ollama tool-call representation outlier.

A Qwen response may encode an intended tool call in assistant content rather than the OpenAI `tool_calls` structure. The SpAd deterministically recognises the supported outlier and normalises it to a valid OpenAI-compatible representation.

Conceptually:

```text
Qwen/Ollama non-conforming response
              ↓
         Qwen Tool SpAd
              ↓
OpenAI-compliant response
              ↓
normal OpenAI Adapter/CIM processing
```

The SpAd changes representation, not meaning.

It is intentionally scoped to the provider/model combinations for which the behaviour is known. It does not assert that every Qwen model or provider requires the same correction.

## 14. SpAds Versus CIM and Filters

Use the following classification:

```text
Provider/model odd representation
        → SpAd

Genuinely new interaction semantic
        → CIM evolution

Intentional change of semantic meaning
        → Filter
```

A user-written SpAd is code. Lumen does not require an ever-growing declarative mapping language capable of describing every provider/model peculiarity.

## 15. SpAds and Semantic Loss

A Base Adapter may report `PASS_WITH_SEMANTIC_LOSS` because its base destination contract cannot represent a CIM semantic.

A provider/model-specific SpAd may know a deterministic, valid representation that preserves that semantic and therefore return `PASS`.

This is representational compatibility knowledge, not hidden semantic policy.

## 16. Evidence

Semantic-loss evidence should identify at minimum:

- affected CIM path / semantic;
- destination interface;
- reason;
- disposition such as omitted or deterministically reduced;
- effective Adapter identity/version.

The exact evidence persistence schema remains integration work.

## 17. Current Production Foundation

Implemented through Milestone 5E:

- BaseAdapter boundary contract;
- production OpenAI Chat Completions Adapter;
- production Anthropic Messages Adapter;
- Adapter registry and deterministic resolution;
- exact-model, model-family and Base fallback;
- SpAd inheritance;
- Qwen Tool SpAd;
- external and CIM schema validation;
- `PASS`, `PASS_WITH_SEMANTIC_LOSS`, and failure semantics;
- conformance harness retained as independent authority.

The production BaseAdapter is a solid boundary/orchestration foundation. Some protocol-specific canonicalisation/materialisation remains explicit rather than being reduced to a completely generic declarative execution engine.

## 18. Architectural Invariants

1. Protocols do not define CIM; interaction semantics define CIM.
2. Supported IN semantics are lossless into CIM.
3. OUT loss is explicit.
4. Adapters do not intentionally alter semantic meaning.
5. SpAds correct same-interface provider/model representation deviations.
6. Filters perform intentional semantic transformation.
7. SpAds are resolved independently of direction.
8. One effective Adapter operates at each boundary.
9. Unknown compatibility is not guessed.
10. Every Adapter/SpAd output conforms to its declared destination schema.
