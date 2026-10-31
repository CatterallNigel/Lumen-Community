# Lumen Moderari --- OUT Materialisation, Semantic Loss, and Specialised Adapter Architecture

**Date:** 2026-10-10\
**Status:** Architecture companion --- IP-01.1\
**Scope:** Moderari SIE Adapter OUT materialisation, semantic-loss
outcomes, and Specialised Adapter (SpAd) behaviour.

## 1. Purpose

This document defines how Moderari handles interaction semantics when a
Canonical Interaction Model (CIM) instance is materialised into a
destination interface, particularly where the destination cannot
represent every CIM semantic.

It also defines how Specialised Adapters (SpAds) may provide
deterministic provider/model-specific representation compatibility
without moving semantic policy into the Adapter layer.

The governing separation remains:

> **SIE is flow control. Adapters map representations. Filters transform
> or observe CIM. SIE itself transforms nothing.**

> **IN canonicalisation is lossless. Filters may intentionally transform
> semantics. OUT materialisation may be lossy where the destination
> interface cannot represent the resulting CIM.**

## 2. IN and OUT Are Deliberately Asymmetric

### 2.1 IN

An Adapter declaring support for an external interface must preserve
every declared supported interaction semantic on IN.

``` text
External Interface → Adapter IN → CIM
```

A supported external interaction semantic with no CIM representation is
a conformance defect:

``` text
SUPPORTED INTERACTION SEMANTIC
        +
NO CIM REPRESENTATION
        ↓
CIM_COVERAGE_FAILURE
```

There is therefore no `PASS_WITH_SEMANTIC_LOSS` for declared supported
IN semantics.

### 2.2 OUT

OUT has a different contract:

``` text
CIM → Adapter OUT → Destination Interface
```

CIM is the union of interaction semantics Lumen has chosen to preserve.
A destination interface may represent only a subset. OUT is therefore
the legitimate point at which representational semantic loss may occur,
provided that loss is explicit.

## 3. OUT Outcomes

### PASS

`PASS` means a valid destination representation was produced and all
applicable CIM interaction semantics were faithfully represented.

### PASS_WITH_SEMANTIC_LOSS

> **PASS_WITH_SEMANTIC_LOSS means that an Adapter OUT operation has
> produced a valid destination-interface representation, but one or more
> CIM interaction semantics could not be represented by that destination
> interface and were therefore omitted or deterministically reduced.
> Every such loss MUST be explicitly identified and recorded. The
> Adapter MUST NOT invent, approximate, reinterpret, or silently
> transform the missing semantic.**

``` text
CIM semantics: A B C
        ↓
Destination represents: A B
        ↓
Valid destination representation
        +
C cannot be represented
        ↓
PASS_WITH_SEMANTIC_LOSS
```

This is not a CIM failure. It records the representational boundary of
the selected destination.

### FAIL

`FAIL` means a valid destination representation cannot be produced
without violating the Adapter contract, for example because
destination-required information is absent, information would have to be
invented, the destination contract would be violated, or semantic
transformation would be required.

A failure must never be converted into apparent success by guessing.

## 4. OUT Decision Rule

``` text
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
Is the missing semantic objectively unrepresentable
by the destination contract?
        │
        ├── YES → PASS_WITH_SEMANTIC_LOSS
        └── NO / UNKNOWN → FAIL
```

> **Lumen records semantic loss it can observe or establish. It does not
> guess semantic equivalence, incompatibility, or loss.**

## 5. Loss Is Not Transformation

`PASS_WITH_SEMANTIC_LOSS` does not permit an Adapter to replace an
unsupported semantic with something it considers similar.

``` text
Adapter
    → representation may change
    → semantics should not intentionally change

Filter
    → semantics may deliberately change
```

A deterministic mapping that changes representation while preserving
meaning remains Adapter behaviour. A deliberate change of meaning
belongs in a Filter.

## 6. Filters Remain Intentional

Destination loss does not cause Moderari to find, select, or invoke a
Filter.

``` text
OUT cannot represent X
        ↓
Find a Filter to change X
        ↓
       NO
```

A Filter participates only because it was intentionally selected in the
Session Execution Plan:

``` text
CIM₁ → Selected Filter → CIM₂ → Adapter OUT
```

The Adapter then independently returns `PASS`,
`PASS_WITH_SEMANTIC_LOSS`, or `FAIL`.

## 7. Specialised Adapters

A Specialised Adapter provides deterministic representation
compatibility for a known provider/model implementation of an interface.

Typical resolution identity is:

``` text
interface + provider + model/model-family
```

There is no probabilistic judgement by SIE.

### Direction-independent resolution

> **Adapter resolution is independent of interaction direction. The
> interface/provider/model identity resolves one effective Adapter. SIE
> invokes that Adapter's IN or OUT operation according to its position
> in the interaction path.**

``` text
interface + provider + model/family
              ↓
       Effective Adapter
          ┌────┴────┐
         IN()      OUT()
```

The manifest does not need `specialises: in/out`.

### Override IN, OUT, or both

A SpAd is a complete effective Adapter runtime object, normally based on
the relevant Base Adapter. It may inherit either operation unchanged and
override only what is required.

``` text
Base Adapter
   ├── IN()
   └── OUT()
        ↑
Specialised Adapter
   ├── IN()   inherited or overridden
   └── OUT()  inherited or overridden
```

## 8. Qwen --- Same Interface, Specialised Representation

A provider/model may nominally use the same interface while requiring
specialised representation behaviour:

``` text
Qwen provider/model
        ↓
Qwen/OpenAI SpAd IN
        ↓
       CIM
        ↓
Qwen/OpenAI SpAd OUT
        ↓
OpenAI-compatible client
```

And in reverse:

``` text
OpenAI-compatible client
        ↓
Qwen/OpenAI SpAd IN
        ↓
       CIM
        ↓
Qwen/OpenAI SpAd OUT
        ↓
Qwen provider/model
```

The same effective SpAd resolves because the
interface/provider/model-family identity matches. Direction selects the
operation, not the Adapter.

## 9. Different Interfaces Resolve Independently

``` text
Anthropic client
       ↓
Anthropic Adapter IN
       ↓
      CIM
       ↓
Qwen/OpenAI SpAd OUT
       ↓
Qwen provider/model
```

Each endpoint resolves its own effective Adapter. SIE does not need to
understand why specialised behaviour exists.

## 10. SpAds Can Increase Representational Fidelity

A Base Adapter may legitimately encounter loss:

``` text
CIM
 ↓
Base Adapter OUT
 ↓
Destination subset cannot represent semantic X
 ↓
PASS_WITH_SEMANTIC_LOSS
```

Known provider/model behaviour may allow a SpAd to represent X:

``` text
CIM
 ↓
Provider/model SpAd OUT
 ↓
Known deterministic representation
 ↓
PASS
```

This is a principal value of SpAds: deterministic compatibility
knowledge can improve fidelity without putting provider/model
peculiarities or probabilistic decisions into SIE.

## 11. SpAds Do Not Become Filters

> **A SpAd may change how a semantic is represented. It may not silently
> change what the semantic means.**

If supporting a provider/model requires an intentional semantic change,
that behaviour belongs in a Filter.

A SpAd may use a provider/model-specific representation that genuinely
preserves the chronology of a later system instruction. It may not
silently move that instruction to the beginning if doing so changes when
it becomes effective.

## 12. Worked Example --- Interleaved System Instructions

CIM may contain:

``` text
sequence 0: system — "You are helpful."
sequence 1: user — "Hello."
sequence 2: assistant — "Hello!"
sequence 3: system — "From now on, respond in French."
sequence 4: user — "How are you?"
```

If the Base destination contract can represent the initial system
instruction and conversation but not the later system instruction, and
omission still permits a valid request:

``` text
initial system → represented
conversation   → represented
later system   → not representable
                    ↓
          PASS_WITH_SEMANTIC_LOSS
```

The Adapter records the lost later instruction. It does not move,
combine, reinterpret, or substitute it.

If a provider/model SpAd knows a valid deterministic representation for
the later instruction, it can produce `PASS`.

If the user intentionally selects a Filter that moves or consolidates
instructions, the semantic transformation occurs before OUT:

``` text
CIM₁ → selected Filter → CIM₂ → Adapter OUT
```

## 13. Worked Example --- Cardinality

Suppose CIM contains:

``` text
items = [A, B, C]
```

If the destination represents all items, the result is `PASS`.

If the destination can represent only a deterministically valid subset
and its contract supplies a legitimate reduction rule, the Adapter may
produce a valid destination, explicitly report omitted semantics, and
return `PASS_WITH_SEMANTIC_LOSS`.

The generic Adapter must not arbitrarily choose the first item.

If no valid deterministic reduction exists, the result is `FAIL`.

A Filter may intentionally reduce the collection before OUT only when
deliberately selected.

## 14. Loss Evidence

A `PASS_WITH_SEMANTIC_LOSS` result must make the loss explicit enough to
establish:

-   the CIM semantic not represented;
-   its location in the CIM instance;
-   the destination interface;
-   why it could not be represented;
-   whether it was omitted or deterministically reduced;
-   the effective Adapter identity and version.

Illustrative only:

``` yaml
outcome: PASS_WITH_SEMANTIC_LOSS

losses:
  - cim_path: instructions[1]
    semantic: system_instruction
    reason: destination_contract_cannot_represent_position
    disposition: omitted

adapter:
  identity: anthropic-messages
  version: ...
```

This is not a frozen evidence schema.

## 15. What PASS_WITH_SEMANTIC_LOSS Does Not Mean

It does not mean that the model will probably understand the request,
that the Adapter found an approximate equivalent, that Lumen selected a
Filter, that the lost semantic was judged unimportant, or that behaviour
will be equivalent.

It means only:

> **A valid destination representation was produced, and Lumen can
> objectively identify one or more CIM semantics that the destination
> representation did not preserve.**

## 16. Representation, Provider/Model Compatibility, and Behaviour

Three questions remain separate:

**Representational compatibility:** Can the destination representation
express the CIM semantic? This is principally Adapter territory.

**Provider/model compatibility:** Does a known provider/model
implementation require or permit a specialised deterministic
representation? This may be encoded in a SpAd selected from
provider/model identity.

**Behavioural correctness:** Will the model interpret and follow the
interaction as intended? Adapter conformance does not establish this.

The Adapter architecture must not turn behavioural uncertainty into
guessed representation rules.

## 17. Conformance Expectations

The IP-01.1 harness should verify all three OUT outcomes.

For `PASS`, the supported CIM fixture is fully materialised and accepted
by the destination supported-interface schema.

For `PASS_WITH_SEMANTIC_LOSS`, the harness verifies that the destination
representation is valid, the loss is objectively caused by destination
representational limits, every omitted/reduced semantic is reported, no
unreported semantic disappears, and no substitute semantic is invented.

For `FAIL`, the harness verifies that a valid destination representation
cannot be constructed under the declared contract without invention or
semantic transformation.

A currently hard-failing case must therefore be reconsidered if a valid
partially faithful destination can still be produced.

## 18. Normative Architectural Rules

1.  **IN is lossless for all declared supported interaction semantics.**
2.  **OUT may be lossy only where the destination cannot represent a
    resulting CIM semantic.**
3.  **Every OUT semantic loss is explicit.**
4.  **Semantic loss is never silently converted into semantic
    transformation.**
5.  **Adapters do not guess value or semantic equivalence.**
6.  **Filters are intentional and are never automatically selected to
    repair Adapter loss.**
7.  **A valid partially faithful destination representation may return
    `PASS_WITH_SEMANTIC_LOSS`.**
8.  **If a valid destination cannot be produced without invention or
    semantic transformation, OUT returns `FAIL`.**
9.  **Adapter resolution is independent of IN/OUT direction.**
10. **Interface/provider/model identity resolves one effective
    Adapter.**
11. **A SpAd may inherit or override IN, OUT, or both without
    directional manifest selection.**
12. **A SpAd may encode deterministic provider/model representation
    compatibility.**
13. **A SpAd remains an Adapter and must not silently perform
    intentional semantic transformation.**
14. **SIE selects and invokes the resolved Adapter; it does not infer
    why specialised behaviour is required.**
15. **Behavioural correctness is not established by representational
    conformance.**

## 19. Consequence for IP-01.1

The conformance work has demonstrated the need to distinguish:

``` text
PASS
PASS_WITH_SEMANTIC_LOSS
FAIL
```

The next implementation work should formalise these outcomes before CIM
1.0 is frozen.

In particular:

> **If a valid destination representation can still be produced without
> inventing or transforming semantics, materialise it, explicitly record
> every unrepresented CIM semantic, and return
> `PASS_WITH_SEMANTIC_LOSS`.**

Only cases where valid materialisation is impossible under the
destination contract remain `FAIL`.

This does not require a CIM schema change merely because semantic loss
occurs on OUT. A CIM change is required only if CIM itself cannot
represent an interaction semantic Lumen has chosen to preserve.
