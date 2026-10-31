# Lumen Moderari --- Adapter Response Protocol Coverage: Known Gaps and Development Work

**Status:** Active development backlog\
**Context:** Milestone 5O --- CIM Response Envelope Foundation\
**Scope:** CIM, Base Adapters, protocol mapping schemas, conformance
tests, and SIE response traversal

## Purpose

Milestone 5O establishes response-envelope canonicalisation for the
explicitly supported **non-streaming** OpenAI and Anthropic response
structures.

This is not intended to define the final coverage boundary of the Base
Adapters.

The preferred direction is to treat currently unsupported protocol forms
as **further Adapter/CIM development work**, rather than accepting them
as permanent limitations, wherever the protocol semantics can reasonably
be represented by Lumen's Canonical Interaction Model (CIM).

## Architectural invariant

The following rule remains normative:

> **Any semantic loss introduced during Adapter.OUT MUST be explicitly
> reported by the Adapter and persisted by Moderari as interaction
> evidence. Semantic loss during Adapter.IN is not permitted.**

Accordingly:

-   Adapter.IN must unpack the supported source protocol representation
    into CIM without semantic loss.
-   CIM must be capable of carrying all supported interaction semantics
    required for lossless IN canonicalisation.
-   Adapter.OUT repacks CIM into the destination protocol
    representation.
-   `PASS_WITH_SEMANTIC_LOSS` is valid only when the final destination
    representation cannot express one or more CIM semantics.
-   An applicable Specialised Adapter (SpAd) may eliminate a Base
    Adapter representational loss.
-   Semantic-loss evidence is recorded only when the **final
    Adapter/SpAd OUT result** remains `PASS_WITH_SEMANTIC_LOSS`.
-   SIE must not acquire protocol-specific knowledge to compensate for
    incomplete Adapter mappings.

## Current supported response boundary

Milestone 5O explicitly supports the implemented **non-streaming
response-envelope structures** for the OpenAI and Anthropic Base
Adapters.

The current implementation should therefore be described as:

> **Response-envelope foundation with explicit non-streaming protocol
> coverage.**

It must not be described as exhaustive OpenAI or Anthropic protocol
coverage.

## Development gap 1 --- Streaming response envelopes

Streaming responses use a materially different representation from
completed non-streaming responses.

For example, a streaming protocol may deliver an interaction as a
sequence of partial events/deltas rather than one complete response
envelope.

The Adapter architecture therefore needs explicit streaming support that
preserves the same invariants as non-streaming processing:

``` text
Provider stream
      ↓
Provider Base Adapter.IN
      ↓
canonical streaming representation / accumulated CIM semantics
      ↓
SIE / Filters
      ↓
Client Base Adapter.OUT
      ↓
Client protocol stream
```

Required future work includes:

-   Define how streaming events are represented canonically.
-   Determine whether CIM represents individual stream events, an
    evolving interaction, the completed interaction, or a defined
    combination of these.
-   Map OpenAI streaming response events/deltas into the canonical
    representation.
-   Map Anthropic streaming events into the canonical representation.
-   Preserve tool-call arguments and other values that may arrive
    incrementally.
-   Preserve stop/completion semantics.
-   Preserve protocol-supported usage/accounting information.
-   Define cross-protocol streaming translation.
-   Define the relationship between streaming evidence and Vestigare.
-   Add same-protocol and cross-protocol streaming conformance tests.
-   Ensure no Adapter.IN semantic loss occurs merely because information
    is delivered incrementally.

Until this work is completed, streaming remains an **explicit Adapter
capability limitation**, not an acceptable source of silent semantic
loss.

## Development gap 2 --- Additional protocol variants and fields

The initial response schemas cover the response structures deliberately
implemented in Milestone 5O. OpenAI and Anthropic protocols contain, and
may introduce, additional response fields, content forms, events, and
protocol variants.

For each additional protocol feature, development must classify it as
one of:

1.  **Interaction semantic** --- must have a CIM representation and
    lossless Adapter.IN mapping.
2.  **Protocol representation/mechanic** --- handled by the Adapter but
    need not survive as an interaction semantic.
3.  **Execution/provenance metadata** --- must be preserved through the
    appropriate Lumen evidence/execution mechanism where required.
4.  **Unsupported destination semantic** --- may result in
    `PASS_WITH_SEMANTIC_LOSS` on Adapter.OUT only.

The existence of an unknown or currently unmapped semantic field must
not result in silent Adapter.IN loss.

## Development gap 3 --- Protocol mapping completeness

OpenAI and Anthropic mapping schemas should ultimately describe the
complete supported interaction representation in both directions:

``` text
Protocol request envelope
        ↕
       CIM
        ↕
Protocol response envelope
```

The mappings therefore need continuing conformance review against the
protocol structures Lumen claims to support.

A useful completeness test is:

``` text
Protocol A envelope
        ↓ Adapter A.IN
       CIM
        ↓ Adapter A.OUT
Protocol A envelope
```

With no Filters applied, the result must be semantically equivalent to
the input.

Cross-protocol testing should then establish:

``` text
Protocol A
   ↓
  CIM
   ↓
Protocol B
```

Any loss here must arise only because **Protocol B cannot express a CIM
semantic**, and must be explicitly reported as
`PASS_WITH_SEMANTIC_LOSS`.

## Development gap 4 --- Qwen SpAd review

The Qwen SpAd remains compatible with the Milestone 5O response-envelope
foundation, but its behaviour must be reviewed against the strengthened
losslessness invariant.

In particular, any Qwen-specific tool-call correction must preserve
surrounding semantics rather than replacing or discarding content that
entered through the OpenAI representation.

The Qwen SpAd may legitimately convert a Base Adapter
`PASS_WITH_SEMANTIC_LOSS` into `PASS` where Qwen has a deterministic
non-standard representation that preserves the CIM semantic.

It must not hide genuine residual semantic loss.

## Development gap 5 --- Persisting final OUT semantic losses

Milestone 5O defines Adapter-level semantic-loss reporting. The
remaining integration work must ensure that a final:

``` text
PASS_WITH_SEMANTIC_LOSS
```

causes Moderari to persist the reported losses through Vestigare as
interaction evidence.

No semantic-loss evidence should be persisted when an applicable SpAd
successfully resolves the representational problem and the final OUT
result is `PASS`.

## Completion criteria

This area should be considered mature when:

-   supported OpenAI non-streaming responses round-trip through CIM
    without semantic loss;
-   supported Anthropic non-streaming responses round-trip through CIM
    without semantic loss;
-   supported OpenAI and Anthropic streaming forms have defined CIM
    mappings and conformance tests;
-   cross-protocol mappings explicitly identify destination-only
    representational loss;
-   no supported Adapter.IN path can silently discard interaction
    semantics;
-   final `PASS_WITH_SEMANTIC_LOSS` reports are persisted by Moderari as
    Vestigare interaction evidence;
-   SpAds can correct provider/model-specific representational
    deficiencies without altering SIE or CIM semantics; and
-   SIE remains entirely protocol-agnostic.

## Working position

The current limitation:

> "The current response schemas cover explicitly supported non-streaming
> structures. Streaming and additional protocol variants still need
> mapping and conformance work."

is therefore recorded as **known unfinished Adapter/CIM development**,
not as the desired permanent architectural boundary.

The default engineering preference is to extend CIM and the relevant
Base Adapter mappings whenever the source protocol contains meaningful
interaction semantics that Lumen intends to support.
