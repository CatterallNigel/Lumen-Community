# Moderari Milestone 5P — Versioned Mapping and Compatibility Foundation

**Status:** Implemented foundation; full 5P semantic coverage review remains open.
**Base:** 5O CIM Response Envelope Foundation.
**Mapping revision:** `2026-10-12` (Lumen mapping snapshot identifier, **not** a claim that the providers publish this schema version).
**CIM:** `1.0` with additive `extensions` property.

## Implemented

- Base OpenAI and Anthropic Adapters declare a mapping revision, checked against their YAML mapping manifests.
- CIM has structured `extensions[]` entries with `protocol`, `revision`, `path`, and original JSON `value`.
- Additional fields unknown to a declared supported schema are extracted recursively at the Base Adapter boundary before strict validation and preserved as opaque CIM extensions. This is not semantic interpretation.
- On same-protocol OUT, extensions are restored to their source paths if the output shape permits. On cross-protocol OUT, opaque extensions are **not** injected into a foreign protocol and produce `SemanticLoss` records. Unrestorable same-protocol paths also produce loss records.
- Request and completed-response compatibility-aware entry points return `PASSTHROUGH_UNSUPPORTED_PROTOCOL` for structurally incompatible known fields instead of guessing a mapping. Existing strict `canonicalise*` entry points continue to raise on incompatible representations for backward compatibility.
- Qwen SpAd no longer discards surrounding prose when it finds a fenced or embedded tool-call payload. Only complete tool-call-only content is eligible for the correction.
- Regression coverage added for unknown request/response fields, cross-protocol extension loss, incompatible envelopes, and Qwen mixed prose.

## Critical distinction: preservation is not understanding

Unknown fields are carried as opaque data. They cannot be assumed safe to translate into another protocol. A future extension may have semantics that affect execution, so a caller must decide whether to permit execution with opaque fields; simply preserving unknown fields is not equivalent to supporting their semantics. Pass-through is safe only if the destination accepts the original envelope and **no CIM-dependent Filter is run**.

## Explicit remaining 5P work

**Full protocol coverage is not yet certified.** The existing request and non-streaming response schemas are historically supported subsets. Known-but-unmapped provider features may currently enter `extensions` just like newly introduced fields. We must audit the complete supported OpenAI Chat Completions and Anthropic Messages protocol definitions against the published provider schemas, then extend explicit CIM semantics, IN/OUT mappings, and conformance fixtures accordingly. This work must not be represented as complete merely because opaque extensions can preserve raw values.

**Streaming remains 5Q.** The new compatibility contract is for JSON request and completed response envelopes; it does not yet parse streaming event sequences. Canonical stream event vocabulary and lifecycle conformance are 5Q.

**Production pass-through routing is not wired.** The new return outcome is a boundary contract; Moderari/SIE must enforce destination protocol compatibility, prevent CIM Filters, and preserve evidence of the decision before live use. The existing strict API still fails closed.

**Opaque extension rebinding needs hardening.** Extensions are currently restored using source JSON paths. If a Filter or translation changes array ordering or structural shape, restoration may be unsafe. Before production use, bind extensions to stable semantic identities or fail closed rather than attach them to a different message/choice.

**Opaque preservation is JSON-semantic rather than byte-for-byte transport preservation.** Framing, whitespace, key ordering and raw byte-level signatures are not represented. Exact transport pass-through must retain the original envelope separately.

**No Vestigare persistence yet.** Loss reporting exists, but final evidence persistence remains 5S.

## Invariants

1. Adapter.IN must never silently discard supported interaction semantics.
2. Unknown fields are opaque extensions, not asserted CIM semantics.
3. Known fields with incompatible structures are not best-matched.
4. `PASSTHROUGH_UNSUPPORTED_PROTOCOL` is not equivalent to successful canonicalisation.
5. Cross-protocol opaque extensions must never be injected into an unrelated destination schema.
6. Only final effective OUT semantic losses should be persisted by Moderari.
7. SIE remains protocol agnostic.

## Validation

- `PYTHONPATH=src pytest -q`: **500 passed** in the build environment.
- `python -m compileall -q src tests`: verified.
- Ruff and mypy are unavailable in this environment; run them locally before promoting the milestone.
