# Lumen Moderari --- IP-01.1 CIM/Adapter Conformance Harness Specification

**Date:** 02026-10-08\
**Status:** Implementation Specification\
**Scope:** IP-01.1 --- CIM 1.0 and Base Adapter Mapping Conformance\
**Component:** Moderari Standalone Interaction Engine (SIE)

------------------------------------------------------------------------

## 1. Purpose

The CIM/Adapter Conformance Harness exists to validate the Canonical
Interaction Model (CIM) and protocol-to-CIM mapping specifications
**before** the production Base Adapter execution engine is implemented.

The harness is not a prototype Adapter runtime. Its purpose is to
provide deterministic evidence that:

1.  every supported interaction semantic accepted on an Adapter IN
    boundary can be represented in CIM without loss;
2.  protocol-specific mapping specifications correctly canonicalise
    external representations into CIM;
3.  same-interface round trips preserve canonical interaction semantics;
4.  cross-interface materialisation either preserves canonical semantics
    or reports an explicit and expected OUT semantic loss; and
5.  CIM 1.0 is sufficiently complete to be frozen as the canonical
    contract for the supported interfaces.

The initial conformance scope is:

-   OpenAI Chat Completions;
-   Anthropic Messages;
-   CIM 1.0.

------------------------------------------------------------------------

## 2. Governing Architectural Invariant

The harness shall enforce the following invariant:

> **IN canonicalisation is lossless. Filters may intentionally transform
> semantics. OUT materialisation may be lossy where the destination
> interface cannot represent the resulting CIM.**

Conceptually:

``` text
External A: X,Y,Z
       ↓
Adapter A IN
       ↓
CIM₁: X,Y,Z
       ↓
Filter(s)
       ↓
CIM₂: X,Y,Z'
       ↓
Adapter B OUT
       ↓
External B: X,Y
```

If `Z'` cannot be represented by External B, the loss occurs at the
**OUT boundary**, not at the IN boundary.

A supported interaction semantic that cannot be represented in CIM is
therefore not an acceptable runtime loss condition. It is a **CIM
coverage failure**.

------------------------------------------------------------------------

## 3. CIM Scope

CIM represents canonical **interaction semantics**, independently of the
external interface from which they originated or to which they may later
be materialised.

CIM is not intended to preserve wire or transport artefacts that do not
carry interaction semantics.

Examples outside CIM include:

-   HTTP headers that do not themselves constitute interaction
    semantics;
-   SSE framing;
-   JSON whitespace;
-   JSON formatting;
-   transport chunk boundaries where those boundaries have no
    interaction meaning;
-   field ordering where ordering has no semantic significance.

Where ordering, identity, grouping, authority, content type, delivery
intent, or another characteristic **does** carry interaction meaning,
that semantic must be preserved by CIM.

The practical scope of CIM 1.0 is therefore the union of the interaction
semantics Lumen declares supported on its initial IN interfaces, rather
than merely the intersection of semantics shared by those interfaces.

------------------------------------------------------------------------

## 4. Harness Boundary

The Conformance Harness shall test contracts and mappings without
requiring the production Moderari execution environment.

It shall not require:

-   FastAPI;
-   MongoDB;
-   Pontis;
-   Praebere;
-   Nuntius;
-   Vestigare;
-   Ollama;
-   Anthropic or OpenAI network access;
-   a running model;
-   the production SIE execution engine.

The harness may contain a deliberately small deterministic mapping
executor sufficient to exercise fixture data against:

-   the supported external-interface schema;
-   the protocol mapping specification;
-   `cim-1.0.schema.json`.

This executor must remain test infrastructure. It shall not acquire
provider routing, session management, Filter execution, model execution,
transport, retry, or production orchestration responsibilities.

------------------------------------------------------------------------

## 5. Authoritative Inputs

The initial harness shall consume three classes of specification
artifact.

### 5.1 CIM Schema

``` text
cim-1.0.schema.json
```

The CIM schema is authoritative for the structure of valid CIM 1.0
instances.

### 5.2 Protocol Mapping Specifications

``` text
openai-chat-completions-cim-1.0.mapping.yaml
anthropic-messages-cim-1.0.mapping.yaml
```

Each mapping specification defines how a supported external interface
maps interaction semantics to and from CIM.

A mapping specification must not depend on knowledge of another
protocol.

### 5.3 Supported External-Interface Schemas

Each Base Adapter under test must have a supported external-interface
schema describing the interface structures that Lumen claims to support.

The external schema answers:

> Is this observed external structure part of the supported interface?

The mapping specification answers:

> If it carries interaction semantics, where is that semantic
> represented in CIM?

The CIM schema answers:

> Is the resulting canonical representation valid CIM?

These are separate responsibilities.

------------------------------------------------------------------------

## 6. Fixture Structure

Fixtures shall be small and semantic-focused rather than large
all-purpose examples.

Initial structure:

``` text
tests/
└── fixtures/
    └── adapters/
        ├── openai-chat-completions/
        │   ├── 001-minimal-text/
        │   │   ├── external.json
        │   │   └── expected-cim.json
        │   ├── 002-system-instruction/
        │   │   ├── external.json
        │   │   └── expected-cim.json
        │   └── ...
        │
        └── anthropic-messages/
            ├── 001-minimal-text/
            │   ├── external.json
            │   └── expected-cim.json
            └── ...
```

A fixture may additionally contain:

``` text
expected-out.json
expected-loss.json
fixture.yaml
```

where required by a particular conformance test.

------------------------------------------------------------------------

## 7. Fixture Metadata

Where metadata is required, `fixture.yaml` should describe the test
intent rather than duplicate protocol data.

Example:

``` yaml
id: openai-006-tool-definition
interface: openai-chat-completions
cim_version: "1.0"

purpose: >
  Verify that an OpenAI function tool definition is canonicalised
  into the CIM tool definition without semantic loss.

assertions:
  in_lossless: true
  cim_valid: true
```

For an expected OUT loss:

``` yaml
id: anthropic-to-openai-document-loss
source_interface: anthropic-messages
destination_interface: openai-chat-completions
cim_version: "1.0"

assertions:
  in_lossless: true
  out_loss_expected: true
```

The expected loss itself belongs in `expected-loss.json`, not as prose
hidden inside the test implementation.

------------------------------------------------------------------------

## 8. Initial Fixture Catalogue

The initial suite should cover individual semantic features before
combinations of features.

### 8.1 OpenAI Chat Completions

Initial fixtures should include, where they are within the declared
supported interface:

-   minimal text interaction;
-   system instruction;
-   developer instruction;
-   multi-turn conversation;
-   structured content;
-   image content;
-   tool definitions;
-   tool choice;
-   single tool call;
-   multiple tool calls;
-   tool result;
-   generation controls;
-   stop sequences;
-   structured format controls;
-   reasoning controls and reasoning content where supported;
-   explicit model selection;
-   streaming/delivery intent;
-   completion reason;
-   token usage and supported usage detail.

### 8.2 Anthropic Messages

Initial fixtures should include, where they are within the declared
supported interface:

-   minimal text interaction;
-   system instruction;
-   multi-turn conversation;
-   structured content blocks;
-   image content;
-   document content;
-   citations;
-   tool definitions;
-   tool choice;
-   single tool use;
-   multiple tool uses;
-   tool result;
-   generation controls;
-   thinking controls;
-   thinking content;
-   thinking signatures;
-   redacted thinking;
-   explicit model selection;
-   streaming/delivery intent;
-   completion reason;
-   token usage and supported usage detail;
-   cache-related interaction semantics where included in the supported
    interface.

The catalogue is not itself authoritative. The supported
external-interface schema and mapping specification determine the actual
coverage requirement.

------------------------------------------------------------------------

## 9. IN Conformance

For every fixture, the harness shall perform:

``` text
External fixture
      ↓
supported-interface validation
      ↓
tree walk
      ↓
mapping match
      ↓
canonicalisation
      ↓
actual CIM
      ↓
CIM schema validation
      ↓
compare with expected CIM
```

### 9.1 IN Success Conditions

An IN fixture passes only when:

1.  the external representation is valid for the declared supported
    interface;
2.  every supported interaction semantic encountered is mapped;
3.  no supported interaction semantic is silently discarded;
4.  the produced CIM validates against the declared CIM schema;
5.  the produced CIM is semantically equal to `expected-cim.json`.

### 9.2 IN Coverage Classification

Every externally observed item relevant to coverage shall resolve to one
of:

``` text
MAPPED_SEMANTIC
EXCLUDED_NON_SEMANTIC
```

There shall be no normal third state for a supported interaction
semantic.

If the harness encounters:

``` text
SUPPORTED_INTERACTION_SEMANTIC
+
NO_CIM_MAPPING
```

the result is:

``` text
CIM_COVERAGE_FAILURE
```

The fixture fails.

This indicates that either:

-   CIM is incomplete;
-   the mapping specification is incomplete; or
-   the supported-interface classification is incorrect.

It is not an acceptable semantic-loss warning.

------------------------------------------------------------------------

## 10. Explicit Non-Semantic Exclusions

A field or structure may be excluded only when it is explicitly
classified as outside canonical interaction semantics.

The harness shall distinguish:

``` text
not mapped because non-semantic
```

from:

``` text
not mapped because nobody created a mapping
```

The first may pass. The second must fail.

Exclusions should be reviewable specification data rather than
hard-coded silent behaviour in the walker.

------------------------------------------------------------------------

## 11. Wildcards, Collections and Instance Identity

Mapping specifications may use wildcard paths such as:

``` text
tools[*].function.name
    →
tools[*].name
```

An actual fixture may produce:

``` text
tools[0].function.name
tools[1].function.name
```

The harness shall capture collection instance identity while
canonicalising the interaction so that values belonging to the same
semantic object remain grouped.

Captured source indices preserve:

-   instance identity;
-   grouping;
-   source order where order is semantically relevant.

They are not destination coordinates.

The OUT side must materialise destination collections according to
destination structure and cardinality, not by blindly replaying source
indices.

------------------------------------------------------------------------

## 12. Mapping Transforms

The harness shall support deterministic transforms required solely
because equivalent semantics use different representations.

Examples include:

``` text
JSON string containing tool arguments
    ↔
structured CIM JSON value
```

Transforms shall not invent semantic equivalence between arbitrary
values.

A mapping transform may change representation.

It must not silently change interaction intent.

Intentional semantic change belongs to a Filter, not an Adapter mapping.

------------------------------------------------------------------------

## 13. Same-Interface Round-Trip Conformance

After IN conformance is established independently, each interface shall
be tested for semantic round-trip stability:

``` text
External₁
   ↓ IN
CIM₁
   ↓ OUT
External₂
   ↓ IN
CIM₂
```

The primary assertion is:

``` text
CIM₁ == CIM₂
```

where equality is canonical semantic equality.

The harness shall not require byte-for-byte or structural identity
between `External₁` and `External₂` where the interface permits more
than one semantically equivalent representation.

This test proves that the Adapter can materialise a representation
which, when canonicalised again, preserves the original CIM semantics.

------------------------------------------------------------------------

## 14. Cross-Interface Conformance

Once same-interface round trips pass, the harness shall test both
initial cross-interface paths:

``` text
OpenAI IN
    ↓
CIM₁
    ↓
Anthropic OUT
    ↓
Anthropic IN
    ↓
CIM₂
```

and:

``` text
Anthropic IN
    ↓
CIM₁
    ↓
OpenAI OUT
    ↓
OpenAI IN
    ↓
CIM₂
```

Three outcomes are valid.

### 14.1 Full Preservation

``` text
CIM₁ == CIM₂
```

Result:

``` text
PASS
```

### 14.2 Known Destination Loss

If the destination interface cannot represent a CIM semantic:

``` text
CIM₁: X,Y,Z
destination can represent: X,Y
```

the OUT Adapter may produce:

``` text
External: X,Y
```

provided it also emits explicit semantic-loss evidence identifying `Z`.

Result:

``` text
PASS_WITH_SEMANTIC_LOSS
```

The subsequent destination IN should then produce the expected reduced
CIM:

``` text
CIM₂: X,Y
```

and the difference between CIM₁ and CIM₂ must correspond exactly to the
declared OUT loss.

### 14.3 Undeclared Difference

If:

``` text
CIM₁ != CIM₂
```

and the difference is not fully explained by declared OUT loss, the
result is:

``` text
FAIL
```

------------------------------------------------------------------------

## 15. OUT Failure Conditions

OUT loss and OUT failure are distinct.

### 15.1 Representable With Loss

If valid destination output can be constructed while omitting an
unsupported CIM semantic:

``` text
CIM X,Y,Z
destination supports X,Y
```

result:

``` text
PASS_WITH_SEMANTIC_LOSS
```

### 15.2 Required Destination Semantic Missing

If the destination requires a semantic that is not present in CIM and
cannot be deterministically derived:

``` text
CIM X,Y
destination requires X,Y,Z
```

the Adapter must not invent `Z`.

Result:

``` text
FAIL
```

### 15.3 Optional Destination Semantic Missing

If the destination supports optional `Z`, but CIM contains only X,Y:

``` text
CIM X,Y
destination optionally supports Z
```

the Adapter omits `Z`.

Result:

``` text
PASS
```

No semantic-loss warning is required because CIM did not contain `Z`.

------------------------------------------------------------------------

## 16. Value Compatibility

Structural representability and value compatibility are separate
concerns.

If:

``` text
Protocol A reasoning = "light"
       ↓
CIM reasoning.effort = "light"
```

and Protocol B has a structurally corresponding reasoning field, the
Adapter shall preserve the mapped value by default.

It shall not infer:

``` text
light == thinking
medium == deep
high == ultra
```

unless an explicit deterministic representation rule establishes such a
mapping.

Unknown value compatibility is not itself semantic loss.

If a declared destination schema objectively rejects the preserved
value, the output fails destination validation.

Otherwise, subsequent provider/model acceptance, rejection,
normalisation, reinterpretation or error is downstream observable
evidence and is not predicted by the conformance harness.

------------------------------------------------------------------------

## 17. Expected Semantic-Loss Artifact

An OUT fixture that expects semantic loss shall declare that loss
explicitly.

Example:

``` json
{
  "losses": [
    {
      "cim_path": "messages[0].content[1].signature",
      "reason": "destination_not_representable"
    }
  ]
}
```

The exact production evidence schema may be defined later.

For IP-01.1, the requirement is that expected losses are:

-   explicit;
-   path-identifiable;
-   deterministic;
-   machine-comparable;
-   produced only at OUT.

------------------------------------------------------------------------

## 18. Conformance Results

The harness should report results at three levels.

### 18.1 Fixture Result

``` text
PASS
PASS_WITH_SEMANTIC_LOSS
FAIL
```

### 18.2 Coverage Result

For each supported interface:

``` text
supported semantic paths
mapped semantic paths
explicit non-semantic exclusions
unmapped supported semantic paths
```

The last value must be zero for IN conformance.

### 18.3 Suite Result

A suite passes only when every required fixture satisfies its expected
outcome and there are no undeclared IN coverage gaps.

------------------------------------------------------------------------

## 19. Failure Categories

The initial harness should use a small, explicit failure vocabulary:

``` text
EXTERNAL_SCHEMA_FAILURE
CIM_SCHEMA_FAILURE
CIM_COVERAGE_FAILURE
MAPPING_FAILURE
TRANSFORM_FAILURE
CARDINALITY_FAILURE
OUT_REQUIRED_SEMANTIC_MISSING
UNDECLARED_SEMANTIC_LOSS
ROUND_TRIP_MISMATCH
EXPECTED_LOSS_MISMATCH
```

This vocabulary is for conformance reporting and may later be aligned
with production Moderari evidence/error types.

------------------------------------------------------------------------

## 20. CIM 1.0 Freeze Gate

CIM 1.0 shall not be considered frozen until all of the following are
true:

1.  OpenAI Chat Completions has 100% mapping coverage for all declared
    supported IN interaction semantics.
2.  Anthropic Messages has 100% mapping coverage for all declared
    supported IN interaction semantics.
3.  Every produced CIM fixture validates against `cim-1.0.schema.json`.
4.  Every expected-CIM comparison passes.
5.  OpenAI same-interface round trips preserve CIM semantics.
6.  Anthropic same-interface round trips preserve CIM semantics.
7.  OpenAI → Anthropic cross-interface tests either preserve CIM
    semantics or produce exactly the expected OUT semantic-loss
    evidence.
8.  Anthropic → OpenAI cross-interface tests either preserve CIM
    semantics or produce exactly the expected OUT semantic-loss
    evidence.
9.  No supported IN interaction semantic is silently discarded.
10. No unresolved CIM coverage failure remains.

The freeze decision therefore has objective test evidence rather than
relying solely on architectural review.

------------------------------------------------------------------------

## 21. Protocol C and Future CIM Evolution

The same conformance process shall be used when a new interface is
introduced.

For a future Protocol C:

``` text
Protocol C
    ↓ IN coverage analysis
existing CIM
```

If every supported Protocol C interaction semantic maps losslessly into
the existing CIM, no CIM version change is required.

If Protocol C introduces semantic `W` which Lumen chooses to preserve
and CIM 1.0 cannot represent:

``` text
Protocol C: W
      ↓
CIM 1.0: no representation
```

the conformance result is:

``` text
CIM_COVERAGE_FAILURE
```

That failure provides objective evidence for extending the canonical
model.

A new CIM version should therefore be driven by demonstrated semantic
requirements rather than speculative protocol accommodation.

------------------------------------------------------------------------

## 22. Relationship to Filters

Filters are not required to prove initial Adapter IN conformance.

The baseline Adapter invariant is:

``` text
External
   ↓
Adapter IN
   ↓
CIM
```

with no semantic loss.

Once that invariant is proven, Filter fixtures may later test:

``` text
CIM₁
   ↓
Filter
   ↓
CIM₂
```

where semantic change is intentional and expected.

Adapter conformance must not rely on a Filter to repair information lost
during IN canonicalisation.

A Filter can transform anything represented by CIM. This is one of the
reasons CIM must preserve the complete supported IN interaction
semantics rather than only semantics shared by the initial destination
protocols.

------------------------------------------------------------------------

## 23. Relationship to the Production Base Adapter Engine

The Conformance Harness precedes implementation of the production Base
Adapter engine.

Its output is intended to stabilise:

-   CIM 1.0;
-   supported-interface schemas;
-   mapping specifications;
-   transform requirements;
-   wildcard/cardinality behaviour;
-   loss semantics;
-   expected Adapter behaviour.

After CIM 1.0 passes the freeze gate, the production Base Adapter engine
can be implemented against these established contracts.

The production engine should then be run against the same fixture suite.

The fixture suite therefore becomes a permanent Adapter conformance
suite rather than disposable pre-implementation test code.

------------------------------------------------------------------------

## 24. Initial Implementation Sequence

``` text
1. Establish fixture directory structure

2. Implement CIM schema validation

3. Implement external supported-interface validation

4. Implement deterministic fixture tree walker

5. Implement mapping-path and wildcard matching

6. Implement minimal canonicalisation executor

7. Implement coverage accounting

8. Add OpenAI atomic IN fixtures

9. Add Anthropic atomic IN fixtures

10. Resolve every discovered CIM/mapping coverage failure

11. Implement minimal OUT materialisation required by tests

12. Run same-interface round trips

13. Run cross-interface round trips

14. Resolve undeclared loss/mismatch

15. Freeze CIM 1.0

16. Begin production Base Adapter execution engine
```

------------------------------------------------------------------------

## 25. Non-Goals for IP-01.1

The Conformance Harness is not intended to:

-   execute a model;
-   test model quality;
-   determine whether two arbitrary provider-specific values have
    equivalent meaning;
-   infer undocumented provider behaviour;
-   test network transport;
-   test HTTP/SSE implementation;
-   replace production Adapter code;
-   implement session orchestration;
-   implement Filters;
-   implement Specialised Adapters;
-   test Praebere model discovery;
-   test provider health;
-   perform live-provider integration testing.

Those concerns belong to later implementation and integration stages.

------------------------------------------------------------------------

## 26. Completion Criterion

IP-01.1 Conformance Harness work is complete when the harness can
provide reproducible evidence that:

> **Every declared supported OpenAI Chat Completions and Anthropic
> Messages IN interaction semantic is losslessly represented by CIM 1.0,
> and every tested OUT difference is either semantically equivalent or
> explicitly accounted for as destination-interface loss.**

At that point CIM 1.0 may be frozen and implementation of the production
Base Adapter execution engine can begin.
