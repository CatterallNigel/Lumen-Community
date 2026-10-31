# Lumen Moderari --- IP-01 Implementation Start Strategy

**Date:** 2026-10-08\
**Status:** Agreed implementation starting point\
**Related:** IP-01.1 SIE Foundation Architecture Decisions

## 1. Implementation Principle

Implementation will proceed incrementally, but the initial components
will not be throwaway or deliberately trivial substitutes for the
production architecture.

The governing principle is:

> **Start small in scope, but not simplistic in behaviour.**

Testing follows the same principle: use controlled scenarios, but
exercise the real production package, real manifests, schema-valid EMD,
real CIM and real Adapter/Filter implementations.

The first implementations should be legitimate production foundations
that can be extended as IP-01 progresses. They must contain enough real
protocol and Filter complexity to provide meaningful validation of the
SIE architecture.

A simplistic synthetic pipeline proving only that SIE can call several
Python objects in sequence is insufficient. The initial implementation
should prove canonicalisation, cross-protocol materialisation, Filter
execution, discovery and execution flow using real architectural
components.

## 2. Implement Existing Manifests First

The currently designed Adapter and Filter manifest definitions should be
implemented rather than redesigned speculatively before coding.

Implementation pressure will determine whether the manifests contain too
much, too little, or the wrong information.

The first discovery foundation should support:

``` text
Discover
   ↓
Read Manifest
   ↓
Validate
   ↓
Register
   ↓
Determine Availability
```

Manifest and discovery code is production foundation, not test
scaffolding.

The current version rule remains:

``` text
component identity → one installed implementation → one version
```

Component versions and CIM version are provenance and compatibility
facts; they are not currently runtime selection dimensions.

### 2.1 Base Adapter Manifest

A Base Adapter is a Lumen-owned complete implementation of one external interaction interface to and from CIM. Its manifest owns the architectural facts that specialised implementations must not redefine.

Conceptually:

``` yaml
id: openai
type: adapter
version: 1.0.0
implementation: moderari.interaction.adapters.openai:OpenAIBaseAdapter
interface: openai-chat-completions
cim_version: "1.0"
```

`interface` replaces the earlier `protocol` terminology. The interface alias is a Lumen-defined canonical identity for the externally observable interaction interface. For example, `openai-chat-completions` can be grounded by Praebere against the provider endpoint that implements that interface.

Base Adapters are Lumen components. Third-party Adapter extension is through Specialised Adapters rather than creation of arbitrary Base interfaces.

### 2.2 Specialised Adapter Manifest

A Specialised Adapter (SpAd) is a complete effective Adapter at runtime. It normally reuses or inherits the Base Adapter implementation and overrides only the behaviour required for a particular provider/model combination. The inheritance relationship is implementation and metadata inheritance; it is **not** an SIE execution chain.

SIE therefore executes:

``` text
SIE → resolved Specialised Adapter → CIM
```

and never:

``` text
SIE → Specialised Adapter → Base Adapter → CIM
```

The Specialised Adapter manifest declares only the facts that distinguish the specialisation:

``` yaml
base_adapter: openai
provider: ollama
model: qwen2.5-coder
```

The Base Adapter supplies the interface, CIM version and other fixed architectural metadata. Discovery resolves the Specialised Adapter against its Base Adapter and constructs the effective Adapter definition used by SIE.

`provider` is required because Moderari observes the representation produced by the provider's mapping between its exposed interface and the underlying model. That mapping may itself be model-specific, opinionated or lossy. The same model served by two providers must therefore not be assumed to require the same specialisation.

`model` may identify either a full model identifier or a model family. Resolution precedence is:

``` text
base interface + provider + exact model
        ↓
base interface + provider + model family
        ↓
Base Adapter
```

The SpAd implementation is instantiated and invoked as one complete Adapter. Internal Python inheritance or use of `super()` is an implementation detail invisible to SIE.

## 3. Define the EMD Structural Contract

Before SIE availability and resolution can be tested meaningfully, the
Effective Model Definition (EMD) requires an authoritative structural
contract.

SIE does not own EMD semantics or EMD provenance. Praebere will
eventually supply the current EMD. However, SIE must be able to consume
a structurally valid EMD and determine whether the attributes required
by discovered components are present.

The EMD schema must therefore define **how model characteristics are
represented without defining a closed vocabulary of all possible
characteristics**.

Conceptually:

``` text
EMD
 ├── schema/version
 ├── provider identity
 ├── model identity
 ├── provider/model interface
 └── characteristics
       ├── context_window: ...
       ├── tools: ...
       └── arbitrary future characteristic: ...
```

This is illustrative rather than the final EMD schema.

The provider/model interface is identified by a canonical Lumen interface alias rather than by an open-ended protocol label. For example:

``` yaml
provider: ollama
model: qwen2.5-coder:14b
interface: openai-chat-completions
characteristics:
  context_window: 32768
  tools: true
```

Praebere is ultimately responsible for supplying the provider, model and effective interface identity in the EMD. SIE consumes these facts; it does not classify providers or infer protocol families.

The governing rule is:

> **EMD schema defines the structure through which model characteristics
> are supplied. It does not define the complete vocabulary or semantic
> meaning of those characteristics.**

A Filter may therefore declare a requirement for a characteristic
unknown to SIE. If that characteristic exists in the supplied EMD, SIE
can determine structural availability without acquiring semantic
knowledge of the characteristic.

The EMD schema and representative EMD documents become production
contracts and test fixtures rather than ad-hoc Python dictionaries
created only for pytest.

## 4. Define CIM 1.0 Against OpenAI and Anthropic

CIM 1.0 should not be derived solely from OpenAI and then expanded later
when Anthropic is introduced.

Instead, OpenAI and Anthropic should both be examined while defining the
initial CIM schema:

``` text
OpenAI representation ─┐
                       ├── identify interaction semantics ──→ CIM 1.0
Anthropic representation┘
```

The objective is not to model every feature or API field from either
protocol. The objective is to identify a meaningful initial set of
interaction semantics required to support real interactions across both
protocols.

Likely semantic areas include, subject to implementation analysis:

-   system instruction;
-   ordered interaction messages;
-   participant/role semantics;
-   content and text parts;
-   tool definitions;
-   tool calls;
-   tool results;
-   interaction parameters that Lumen chooses to preserve.

This list is not the CIM schema. The schema must emerge from examination
of the actual protocol representations and the interaction semantics
Lumen intends to preserve.

The normative CIM definition remains:

> **CIM is a versioned, schema-defined canonical interaction data model.
> The schema is authoritative; language-specific representations may be
> generated from it. Adapters map external protocol representations to
> and from CIM. Adding support for a protocol requires a new Adapter,
> but requires a CIM change only when that protocol introduces
> interaction semantics that the current CIM cannot represent and Lumen
> chooses to preserve.**


### 4.0 CIM 1.0 Coverage Scope and Semantic-Fidelity Invariant

CIM 1.0 should begin with a deliberately broad interaction scope drawn from
the principal capabilities exposed by OpenAI Chat Completions and Anthropic
Messages. The purpose of this scope is to avoid constructing a minimal chat
model that immediately requires fundamental CIM changes as real Adapters,
Filters and SIE scenarios are implemented.

The initial coverage checklist is:

| Interaction element | OpenAI Chat Completions | Anthropic Messages |
| --- | --- | --- |
| System instructions | `system` or `developer` message | Top-level `system` |
| Conversation | `messages[]` | `messages[]` |
| Message roles | `system`, `developer`, `user`, `assistant`, `tool` | `user`, `assistant` |
| Message content | `content`: string or structured parts | `content`: string or structured blocks |
| Text content | String or `text` part | `text` block |
| Images | `image_url` content part | `image` block |
| Documents | No equivalent general-purpose document block in standard Chat Completions | `document` block |
| Tool definitions | `tools[]` | `tools[]` |
| Tool parameters | `function.parameters` | `input_schema` |
| Tool selection | `tool_choice` | `tool_choice` |
| Tool invocation | Assistant `tool_calls[]` | Assistant `tool_use` blocks |
| Tool call identity | `tool_call.id` | `tool_use.id` |
| Tool results | Separate `tool` role message | `tool_result` block within a `user` message |
| Multiple tool calls | Multiple entries in `tool_calls[]` | Multiple `tool_use` blocks |
| Generation temperature | `temperature` | `temperature` |
| Maximum output tokens | `max_completion_tokens` | `max_tokens` |
| Stop sequences | `stop` | `stop_sequences` |
| Completion status | `finish_reason` | `stop_reason` |
| Token usage | `usage` | `usage` |
| Streaming | Streaming request semantics; SSE transport | Streaming request semantics; SSE transport |
| Structured output | `response_format` | `output_config.format` |
| Extended thinking | Model-specific reasoning controls and outputs | `thinking` configuration and thinking blocks |
| Model selection | `model` | `model` |

This table is a **CIM 1.0 semantic coverage checklist**, not the final CIM
schema and not a requirement to copy either external API structure. Each area
must be analysed to determine the canonical semantic representation required
to preserve the interaction information carried by the supported interfaces.

#### Adapter Semantic-Fidelity Invariant

The central Adapter invariant is:

> **Adapter A IN → CIM → Adapter B OUT must preserve the complete interaction
> semantics of the source representation wherever the destination interface is
> capable of representing them.**

Adapters change representation. They do not intentionally change the meaning
of the interaction.

Conceptually, without a Filter:

```text
External A
    ↓
Adapter A IN
    ↓
CIM
    ↓
Adapter B OUT
    ↓
External B
```

the desired relationship is:

```text
semantics(External A) == semantics(CIM) == semantics(External B)
```

subject only to an explicitly evidenced representational limitation in CIM or
the destination interface.

This means that a field must not be omitted from CIM merely because equivalent
information may also exist elsewhere in Lumen. For example, if an interaction
request explicitly contains a model selection, that selection is information
carried by the interaction and must be considered for canonical
representation. The Session Execution Plan or EMD may independently contain
model information for execution and resolution purposes; that does not make
the interaction's own model selection semantically nonexistent.

The same principle applies to generation parameters, tool-selection
instructions, structured-output requests, reasoning controls and other
supported interaction information.

#### Transport Mechanism Versus Interaction Semantics

Transport implementation and interaction semantics must still be separated.

For example, Server-Sent Events (SSE) is a transport mechanism and is not
itself a CIM semantic. However, if the source interaction explicitly requests
streaming, that request is interaction information and must be considered for
canonical representation.

Conceptually:

```text
External:
    stream = true
        ↓
CIM:
    canonical streaming/delivery intent
        ↓
Destination Adapter:
    destination-equivalent streaming request
```

The fact that both interfaces may implement that request using SSE does not
require CIM to model SSE itself.

The governing distinction is:

> **CIM preserves interaction meaning, including explicit interaction
> requests that affect how the interaction is performed. It does not preserve
> incidental wire-format or transport implementation details that do not
> change that interaction meaning.**

#### CIM Coverage Gaps Are Fidelity Gaps

The existing `X, Y, Z` rule applies at the CIM IN boundary as well as at the
destination OUT boundary.

The ideal canonicalisation is:

```text
Source X,Y,Z
      ↓
Adapter IN
      ↓
CIM X,Y,Z
```

If the source interface validly supplies `Z` but CIM cannot represent it:

```text
Source X,Y,Z
      ↓
Adapter IN
      ↓
CIM X,Y
```

then semantic fidelity has already been reduced before the destination
interface is considered. The unmapped-supported-field evidence defined in
Section 4.8 therefore identifies a potential CIM coverage gap and must remain
visible.

By contrast, if CIM preserves `Z` but the destination cannot represent it:

```text
Source X,Y,Z
      ↓
CIM X,Y,Z
      ↓
Destination X,Y
```

the loss occurs at the destination boundary and follows the established
`PASS + SEMANTIC-LOSS WARNING` rule.

CIM 1.0 should therefore seek to represent the complete interaction semantics
within its declared supported-interface scope. It should not deliberately
discard a source semantic simply because the initial destination Adapter,
Filter or execution path does not currently require it.

#### Adapter Transformation Versus Filter Transformation

This semantic-fidelity invariant also establishes the fundamental distinction
between Adapters and Filters:

```text
Adapter
    → representation may change
    → semantics should not

Filter
    → semantics may deliberately change
```

For a filtered interaction:

```text
External A
    ↓
Adapter A IN
    ↓
CIM₁
    ↓
Filter
    ↓
CIM₂
    ↓
Adapter B OUT
    ↓
External B
```

the expected relationships are:

```text
semantics(External A) == semantics(CIM₁)

semantics(CIM₂) == semantics(External B)
```

but:

```text
semantics(CIM₁) == semantics(CIM₂)
```

is **not** required, because the Filter is the explicitly selected component
permitted to transform interaction semantics.

The resulting architectural rule is:

> **CIM 1.0 should seek to represent the complete interaction semantics within
> its declared supported-interface scope. Adapters canonicalise and
> materialise those semantics without intentionally changing them. Intentional
> semantic transformation belongs to Filters, not Adapters. Where semantic
> equivalence cannot be maintained because CIM or the destination interface
> cannot represent a source semantic, that loss must be explicit evidence
> rather than an implicit design choice.**


### 4.1 CIM Preserves Semantics, Not External Protocol Structure

External interface schemas already define the structural placement of their
fields. CIM is therefore primarily concerned with the canonical semantic
meaning carried by those fields rather than reproducing either external
document structure.

For example:

``` text
OpenAI                         CIM                    Anthropic
function.parameters  ───→  tool_parameters  ←───  input_schema
```

The OpenAI Base Adapter knows where `function.parameters` occurs in an OpenAI
representation. The Anthropic Base Adapter knows where `input_schema` occurs in
an Anthropic representation. CIM needs only the canonical semantic
`tool_parameters` and the relationships required to preserve its meaning.

Adapter IN is therefore conceptually **extract + canonicalise**:

``` text
external representation
        ↓
Base Adapter
        ↓
locate external field/structure
        ↓
identify canonical semantic + value
        ↓
CIM
```

Adapter OUT is conceptually **materialise the destination structure from CIM**:

``` text
CIM semantic + value
        ↓
destination Base Adapter
        ↓
place value into destination-defined structure
        ↓
valid external representation
```

The destination Adapter owns knowledge of its interface schema. It should
materialise only the structures required to represent available CIM semantics.
Optional destination structures for which CIM contains no value may be omitted.
Required destination structures must not be invented merely to make a document
appear complete.

CIM will still contain structure where structure is necessary to preserve
semantic relationships, for example ordering, association of a tool result with
a tool call, and grouping of related content. Such structure exists because the
relationship is semantically significant, not because an external protocol uses
the same document layout.

### 4.2 Adapter OUT Materialisation Outcomes

Cross-interface materialisation must distinguish semantic loss from inability
to construct a valid destination representation.

#### Destination Cannot Represent a Supplied CIM Semantic

Given:

``` text
CIM/source:       X, Y, Z
Destination:      X, Y
```

if the destination interface has no representation for `Z`, the Adapter may
still be able to construct a valid destination representation containing `X`
and `Y`. `Z` is lost at the destination boundary.

``` text
X ───────────────→ X
Y ───────────────→ Y
Z ────── X          unsupported by destination
```

This is **successful materialisation with semantic loss**:

``` text
PASS + SEMANTIC-LOSS WARNING
```

The warning must identify the semantic information that could not be
represented. It is architectural evidence of a known lossy transformation, not
merely a diagnostic log message. Future policy may choose to make particular
losses fatal, but the structural fact remains that a valid destination document
was constructible.

#### Destination Requires a Semantic CIM Cannot Supply

Given:

``` text
CIM/source:       X, Y
Destination:      X, Y, Z
                         ↑ required
```

the Adapter cannot construct a valid destination representation if `Z` is
required and no corresponding CIM semantic/value exists.

``` text
X ───────────────→ X
Y ───────────────→ Y
? ───────────────→ Z REQUIRED
```

This is a **materialisation failure**:

``` text
FAIL
```

The Adapter must not invent `Z`, silently substitute an arbitrary value, or emit
a representation that violates the destination contract.

#### Destination Has an Optional Semantic CIM Does Not Supply

Given:

``` text
CIM/source:       X, Y
Destination:      X, Y, Z
                         ↑ optional
```

the Adapter materialises `X` and `Y` and omits `Z`. No semantic information has
been lost and the destination contract is satisfied:

``` text
PASS
```

The Base Adapter contract must therefore support three fundamental OUT results:

| Condition | Result |
| --- | --- |
| Destination can represent the supplied CIM semantics | **PASS** |
| CIM supplies semantics the destination cannot represent | **PASS + semantic-loss warning** |
| Destination requires semantics CIM cannot supply | **FAIL** |

These outcomes are interface-independent and form part of the Base Adapter
contract. The semantic-loss result is also evidence relevant to later
Vestigare/Aestimare assurance because it identifies a known point at which an
interaction changed meaning or lost representable information during
translation.



### 4.2.1 Structural Representability, Value Compatibility and the Limit of Adapter Knowledge

The Adapter outcome rules above describe loss that Moderari can establish
objectively from the supported source semantics, CIM mappings and destination
interface structure. They must not be extended into an attempt to prove
semantic equivalence of arbitrary values where that equivalence is not known.

For example, two interfaces may expose conceptually corresponding controls:

```text
Protocol A                 CIM                 Protocol B

reasoning = "light"   →   reasoning = "light"   →   thinking = "light"
```

The Base Adapter mappings may legitimately establish that Protocol A's
`reasoning` structure and Protocol B's `thinking` structure both represent the
canonical CIM reasoning semantic. That establishes a structural/semantic
mapping for the field. It does **not** establish that every possible value in
one interface has an equivalent meaning in the other.

For example:

```text
Protocol A values:
    light
    medium
    high

Protocol B values:
    thinking
    deep
    ultra
```

Moderari cannot generically infer:

```text
light  == thinking
medium == deep
high   == ultra
```

and must not invent such a translation.

#### Known Structural Loss

If CIM contains a semantic for which the destination Adapter has no structural
representation, the loss is objectively known:

```text
Protocol A:
    reasoning = "light"
        ↓
CIM:
    reasoning = "light"
        ↓
Protocol B:
    <no reasoning/thinking structure>
```

The established result applies:

```text
PASS + SEMANTIC-LOSS WARNING
```

The Adapter can prove that supplied CIM information was not represented in the
destination.

#### Structure Exists but Value Compatibility Is Unknown

A different case occurs when the destination has a mapped structure but the
compatibility or exact meaning of the value is not known:

```text
Protocol A:
    reasoning = "light"
        ↓
CIM:
    reasoning = "light"
        ↓
Protocol B:
    thinking = "light"
```

Unless an explicit Adapter transform exists, the value is preserved. The Base
Adapter does not guess that `light` should become `minimal`, `thinking`, `deep`
or any other destination value.

The provider/model may subsequently:

```text
accept the value
reject the value
ignore the value
normalise the value
interpret it differently
return an error
or exhibit other provider/model-specific behaviour
```

Those outcomes are downstream observable evidence. Moderari can record the
Adapter output and the subsequent provider/model response rather than
pretending to know in advance how the provider/model will interpret an
otherwise mapped value.

The governing rule is:

> **Lumen records semantic loss it can observe or establish from its declared
> mappings and interface structures. It does not attempt to infer semantic
> loss or value equivalence that it cannot know.**

#### External Schema Validation Does Not Prove Semantic Equivalence

An external-interface schema may provide additional structural knowledge. For
example, an enum may establish that a destination accepts only:

```text
thinking
deep
ultra
```

If the Adapter attempts to materialise:

```text
light
```

and destination schema validation rejects that value, the Adapter has an
objective validation failure and must not claim successful materialisation.

However, schemas cannot be assumed to provide complete value vocabularies, and
even matching enum labels do not prove identical meaning. Therefore:

> **Schema validity establishes only what the declared interface contract is
> capable of validating. It does not establish semantic equivalence between
> arbitrary source and destination values.**

The architecture must not depend on every provider or interface schema
declaring enough information to predict provider/model behaviour.

#### Value Preservation Is the Default

Where a canonical semantic mapping exists and no explicit value transform is
defined, the Adapter preserves the value.

Conceptually:

```text
source semantic/value
        ↓
Adapter IN
        ↓
CIM semantic/same value
        ↓
Adapter OUT
        ↓
destination mapped structure/same value
```

An Adapter transform is appropriate where the representation conversion is
known and deterministic, for example decoding a JSON-encoded tool argument
into a structured CIM object and encoding it again for an interface that
requires a string.

It is not appropriate for the Base Adapter to invent a value substitution
merely because two vocabularies appear conceptually similar.

#### Filters Own Intentional Semantic Correction

If Lumen has an intentional policy that a canonical value should be changed
for a particular execution path, that is a semantic transformation and belongs
in a Filter.

For example:

```text
Protocol A
reasoning = "light"
      ↓
Adapter A IN
      ↓
CIM
reasoning = "light"
      ↓
Filter
      ↓
CIM
reasoning = "deep"
      ↓
Adapter B OUT
      ↓
Protocol B
thinking = "deep"
```

The Filter makes the semantic change explicit and observable. This preserves
the architectural distinction:

```text
Base Adapter
    representation translation
    preserve mapped values by default

Specialised Adapter
    provider/model-specific representation compatibility correction

Filter
    intentional semantic transformation
```

A Specialised Adapter may therefore correct a known provider/model-specific
representation peculiarity of the Base interface. It must not become a hidden
policy layer for changing interaction intent. If the desired change is
semantic -- for example choosing a different reasoning level because it is
considered more appropriate for a model -- the change belongs in a Filter.

The combined governing principle is:

> **Adapters preserve mapped interaction semantics and values unless a known
> representation transform is required. Specialised Adapters correct known
> provider/model-specific representation compatibility behaviour. Filters are
> the explicit mechanism for intentional semantic change. Unknown value
> compatibility is not guessed; the emitted representation and subsequent
> provider/model behaviour become evidence.**


### 4.3 Semantic Paths, Concrete Paths and Collection Indices

CIM and Adapter mappings should distinguish a **schema/mapping path** from a
**concrete interaction path**.

A schema or mapping path uses `[*]` to describe a repeatable semantic
collection:

```text
tools[*].name
tools[*].description
tools[*].parameters

messages[*].tool_calls[*].id
messages[*].tool_calls[*].name
messages[*].tool_calls[*].arguments
```

A real interaction contains concrete collection instances:

```text
tools[0].name
tools[1].name

messages[2].tool_calls[0].id
messages[2].tool_calls[0].name
messages[2].tool_calls[0].arguments
```

The wildcard is therefore part of the schema/mapping definition; the numeric
index belongs to an actual interaction instance.

The index is significant during canonicalisation because it preserves
ordering, grouping and the identity of values belonging to the same collection
member. It is **not**, however, a protocol-independent semantic value and must
not be assumed to be the same numeric position in the destination
representation.

For example, an OpenAI `tool_calls[0]` object might be materialised as an
Anthropic content block whose destination array position is not `[0]`. The
destination Adapter determines its own structural placement.

The governing rule is:

> **Indices identify instances and preserve semantic relationships while an
> interaction is canonicalised. They are not destination coordinates.**

### 4.4 Base Adapter Generic Tree-Walk and Mapping Model

The Base Adapter abstraction should own the generic traversal and mapping
machinery wherever practical. A concrete Base Adapter supplies knowledge of
its external interface structure and its mapping to CIM semantics.

For IN, the generic walker traverses the actual external representation and
produces concrete structural paths such as:

```text
tools[0].function.name
tools[0].function.description
tools[0].function.parameters
messages[2].tool_calls[0].id
messages[2].tool_calls[0].function.name
messages[2].tool_calls[0].function.arguments
```

These concrete paths are matched against Adapter mapping patterns:

```text
tools[*].function.name
tools[*].function.description
tools[*].function.parameters
messages[*].tool_calls[*].id
messages[*].tool_calls[*].function.name
messages[*].tool_calls[*].function.arguments
```

The captured collection instances are then used to construct the corresponding
CIM semantic objects. Conceptually:

```text
External mapping path                         CIM mapping path

tools[*].function.name              ↔         tools[*].name
tools[*].function.description       ↔         tools[*].description
tools[*].function.parameters        ↔         tools[*].parameters

messages[*].tool_calls[*].id        ↔         messages[*].tool_calls[*].id
messages[*].tool_calls[*].function.name
                                      ↔       messages[*].tool_calls[*].name
messages[*].tool_calls[*].function.arguments
                                      ↔       messages[*].tool_calls[*].arguments
```

Thus, for an actual interaction:

```text
tools[0].function.name  →  tools[0].name
tools[1].function.name  →  tools[1].name
```

The concrete Adapter does not need to reimplement generic tree traversal for
each interface. Its principal responsibility becomes declaration of where the
interface carries each CIM semantic, plus exceptional transforms where simple
path mapping is insufficient.

The same mapping knowledge can support OUT in the reverse direction. The
generic Base Adapter machinery walks the concrete CIM instance, matches CIM
semantic paths against the mapping definitions, and asks the destination
Adapter mapping to materialise the appropriate destination structure.

Conceptually:

```text
                 Abstract Base Adapter
                         │
             ┌───────────┼───────────┐
             │           │           │
         Tree Walker    Mapper    Materialiser
             │           │           │
             └───────────┼───────────┘
                         │
                  Mapping Definitions
                         ↑
                Concrete Base Adapter
```

The intended implementation model is therefore:

```text
OpenAI Base Adapter
    = generic Base Adapter engine
      + OpenAI structure ↔ CIM mappings

Anthropic Base Adapter
    = generic Base Adapter engine
      + Anthropic structure ↔ CIM mappings
```

A Specialised Adapter may inherit those mappings and override only the
provider/model-specific mappings or transforms that differ.

### 4.5 Structural Context Is Required to Identify Semantics

The generic tree walker does not make the Adapter protocol-blind. Field names
alone are insufficient to determine semantics because the same field name may
appear at different structural locations with different meanings.

For example, `name` may identify a tool definition, a tool invocation, or some
other named object. The mapping therefore operates on structural paths rather
than bare field names.

Similarly, some interfaces encode meaning through structural discriminators,
for example a typed content block. Mapping rules may therefore require path
conditions or predicates in addition to a simple path.

The governing distinction is:

> **The Base Adapter provides generic traversal and materialisation mechanics.
> The concrete Adapter provides knowledge of where its interface represents
> each CIM semantic.**

CIM does not preserve the external protocol structure, but the IN Adapter must
understand that structure sufficiently to extract both the semantics and the
semantic relationships represented by it.

### 4.6 Mapping Transforms and Non-Trivial Representations

Not every mapping will be a direct value-for-value path mapping. Some
interfaces encode the same semantic value differently.

For example, a tool-call argument may be represented externally as a
JSON-encoded string while CIM represents the arguments as a structured object.
Such a mapping requires directional transforms:

```text
IN:
external encoded value
        ↓
decode / normalise
        ↓
CIM semantic value

OUT:
CIM semantic value
        ↓
encode / materialise
        ↓
external value
```

A mapping definition may therefore conceptually contain:

```text
external path
CIM path
optional IN transform
optional OUT transform
optional structural predicate
```

Simple mappings require only the two paths. Exceptional transforms should be
declared or overridden by the concrete Adapter without moving generic
traversal, mapping or materialisation logic out of the Base Adapter.

This also provides a natural implementation boundary for Specialised Adapters:
a provider/model specialisation can inherit the Base Adapter mappings and
override only those mappings, predicates or transforms where the observed
provider/model representation differs.



### 4.7 Cardinality Differences Between External Interfaces and CIM

External interfaces may represent the same semantic concept with different
cardinality. For example, one interface may permit a collection:

```text
tools[*]
```

while another permits only a single item:

```text
tool
```

CIM should represent the **semantic cardinality Lumen intends to preserve**,
rather than adopting the structural restriction of whichever external
interface is narrower. If an interaction can semantically contain multiple
tool definitions, the CIM representation remains a collection:

```text
tools[*]
```

#### Scalar External Value to CIM Collection

An external interface that permits only one item can canonicalise that item as
one member of the CIM collection:

```text
External:

tool.name = weather

        ↓

CIM:

tools[0].name = weather
```

This is a valid IN canonicalisation and does not imply semantic loss.

#### CIM Collection to Scalar Destination

OUT materialisation must consider the actual CIM cardinality.

If CIM contains one item and the destination permits one item:

```text
CIM tools[0] → destination tool
```

the result is:

```text
PASS
```

If CIM contains multiple items but the destination permits only one:

```text
CIM                     Destination

tools[0] ───────────┐
tools[1] ───────────┼──→ tool   (one item permitted)
tools[2] ───────────┘
```

the destination cannot represent the full CIM semantic cardinality.

The generic Base Adapter must not silently choose an item merely because one
valid destination document could be produced. Choosing `tools[0]`, for
example, is itself a semantic decision.

The default outcome where no explicit reduction policy exists is therefore:

```text
FAIL — cardinality mismatch
```

If a concrete Adapter or explicitly selected policy legitimately defines a
reduction rule, for example:

```text
multiple → first
```

then materialisation may instead produce:

```text
PASS + SEMANTIC-LOSS WARNING
```

The warning must identify both the cardinality mismatch and the CIM instances
that were not represented.

The governing rule is:

> **Cardinality is structural, but a cardinality mismatch can cause semantic
> loss. CIM preserves the semantic cardinality; Adapters reconcile external
> structural cardinality with it and must not invent a reduction decision.**

This extends the general OUT outcome rules:

| Cardinality condition | Result |
| --- | --- |
| CIM cardinality fits destination cardinality | **PASS** |
| CIM exceeds destination cardinality and an explicit valid reduction policy exists | **PASS + semantic-loss warning** |
| CIM exceeds destination cardinality and no reduction policy exists | **FAIL — cardinality mismatch** |
| Destination requires more items than CIM supplies | **FAIL** if its minimum cardinality cannot be satisfied |

#### Mapping Metadata

The Adapter mapping model must therefore be capable of expressing or deriving
three distinct concerns:

```text
path
transform
cardinality
```

Conceptually:

```text
External mapping              CIM mapping             Cardinality

tools[*]              ↔       tools[*]                many ↔ many
tool                  ↔       tools[*]                one  ↔ many
```

The exact mapping-definition syntax remains an implementation decision. The
architectural requirement is that cardinality constraints are known during
materialisation and handled explicitly rather than being hidden inside
procedural Adapter code.

For IN, the generic tree walker remains applicable to both scalar and
collection structures. A scalar external item becomes a single CIM collection
member; an external collection produces the corresponding sequence of CIM
members.

For OUT, the materialiser uses the destination mapping and cardinality
constraint to determine whether the CIM instances can be represented without
loss, can be represented only through an explicit lossy reduction, or cannot
be validly materialised.



### 4.8 IN Schema Validation, Mapping Coverage and Unmapped Supported Fields

The generic tree walker has no responsibility for deciding whether an observed
field is meaningful, supported or part of the external interaction interface.
Its responsibility is mechanical: traverse the received representation and
report the concrete structural paths and values that exist.

For example:

```text
tools[0].function.name       = "weather"
tools[0].function.parameters = {...}
foo.bar                      = 123
```

The concrete Base Adapter's supported external-interface schema then determines
whether each observed path belongs to the interface that the Adapter claims to
support.

Conceptually:

```text
Incoming representation
        ↓
Generic Tree Walker
        ↓
concrete path + value
        ↓
supported external-interface schema
        │
        ├── path is not part of supported schema
        │       ↓
        │    DROP SILENTLY
        │
        └── path is part of supported schema
                ↓
          CIM mapping exists?
                │
                ├── YES → map/canonicalise into CIM
                │
                └── NO  → drop + record UNMAPPED SUPPORTED FIELD
```

For example, an observed path:

```text
tools[0].function.name
```

may match the supported external-schema path:

```text
tools[*].function.name
```

and the Adapter mapping:

```text
tools[*].function.name → CIM tools[*].name
```

so the value is canonicalised normally.

By contrast, an observed artifact such as:

```text
foo.bar
```

that does not appear anywhere in the supported interface schema is not treated
as part of the supported protocol representation and is silently discarded.
It does not create a CIM-loss event merely because it happened to be present in
the received document.

If, however, the external interface schema defines:

```text
messages[*].some_supported_parameter
```

and the received representation contains that field, but the Adapter/CIM
mapping set contains no corresponding CIM mapping, the field is deliberately
not preserved and that fact must be recorded.

The IN outcomes are therefore:

| Observed external path | CIM mapping | Outcome |
| --- | --- | --- |
| Not valid in supported external-interface schema | N/A | **DROP SILENTLY** |
| Valid in supported external-interface schema | Exists | **MAP / PASS** |
| Valid in supported external-interface schema | Does not exist | **DROP + LOG UNMAPPED SUPPORTED FIELD** |

The third outcome is objective mapping-coverage evidence. It establishes that
a field belonging to the supported external interface crossed the IN boundary
but is not represented by the current CIM/Adapter mapping.

It should not automatically be labelled semantic loss merely because no CIM
mapping exists. The architectural fact known at this boundary is narrower:

> **The supported external interface supplied a recognised field that the
> current CIM mapping does not preserve.**

Subsequent analysis may determine that the field is irrelevant to the
interaction semantics Lumen intends to preserve, or that it exposes a semantic
gap requiring a CIM change. This directly supports the existing CIM evolution
rule: CIM changes when a supported protocol introduces interaction semantics
that CIM cannot represent and Lumen chooses to preserve.

The governing assurance rule is:

> **A recognised field belonging to a supported external interface is never
> silently lost at the CIM boundary. An artifact that is not part of that
> supported interface schema is not treated as protocol interaction content
> and may be silently discarded.**

This makes the external-interface schema a formal part of each Base Adapter's
implementation contract. Conceptually, a Base Adapter therefore consists of:

```text
Base Adapter
    │
    ├── Supported External-Interface Schema
    │       └── defines what is valid interface structure
    │
    ├── CIM Mapping Definitions
    │       └── defines what supported structure is preserved in CIM
    │
    └── Generic Base Adapter Engine
            ├── tree walk
            ├── external-schema matching
            ├── mapping-pattern matching
            ├── transforms / predicates
            ├── cardinality handling
            └── destination materialisation
```

This separation is intentional. The tree walker observes structure; the
external-interface schema establishes whether that structure belongs to the
supported interface; the mapping definitions establish whether and how the
recognised structure is represented in CIM.


## 5. Generate the Python CIM Representation

The CIM 1.0 schema is authoritative.

The Python representation used by SIE, Adapters and Filters should be
generated from that schema where practical rather than maintained as an
independently authoritative hand-written model.

Conceptually:

``` text
CIM 1.0 Schema
      ↓
Generated Python CIM Representation
      ↓
Adapters + Filters + SIE Flow
```

The initial implementation should therefore validate both the schema and
the generation approach early.

IP-01 supports one active CIM version. The Session Execution Plan
records that exact CIM version.

## 6. First Base Adapters Are Real Protocol Adapters

The first Base Adapters should be simple but legitimate implementations
of the production Adapter contract:

1.  OpenAI Base Protocol Adapter
2.  Anthropic Base Protocol Adapter

They do not need to implement every capability of their respective
protocols initially. They must implement the defined CIM 1.0 semantic
subset correctly and be structured so that further protocol coverage
extends the same implementation rather than replacing it.

The Base Adapter contract remains conceptually:

``` text
IN:  external protocol representation → CIM
OUT: CIM → external protocol representation
```

The abstract Base Adapter should implement the reusable mechanics for both
directions: tree traversal, supported external-schema matching,
mapping-pattern matching, concrete collection instance handling, mapping
coverage reporting, destination materialisation, and the common PASS /
semantic-loss / FAIL outcome model. Concrete Base Adapters supply the
supported external-interface schema together with interface-specific semantic
path mappings, predicates and exceptional transforms.


The first Adapter validation matrix is:

``` text
OpenAI    → CIM → OpenAI
Anthropic → CIM → Anthropic
OpenAI    → CIM → Anthropic
Anthropic → CIM → OpenAI
```

The cross-protocol cases are especially important because they validate
that interoperability is being provided through CIM rather than through
pairwise protocol knowledge.

Neither Adapter should know that the other exists.

## 7. First Filter Is the Real System Prompt Filter

The first transformational Filter should be the actual System Prompt
Filter rather than a dummy message-mutating Filter.

It is sufficiently simple to understand and test, while exercising real
CIM semantics.

Its initial behaviour remains:

-   insert the configured Lumen System Prompt at the beginning of
    applicable context; or
-   replace the existing System Prompt at the beginning of applicable
    context.

It operates only on CIM:

``` text
CIM
 ↓
System Prompt Filter
 ↓
CIM
```

This makes System Prompt the initial production template for future
transformational Filters.

## 8. First Meaningful SIE Validation

The first meaningful SIE implementation milestone should prove a
complete cross-protocol path using discovered production components:

``` text
               Manifest Discovery
                      ↓
              Component Registry
                      ↓
OpenAI ──→ OpenAI Base Adapter
                      ↓
                    CIM 1.0
                      ↓
              System Prompt Filter
                      ↓
                    CIM 1.0
                      ↓
            Anthropic Base Adapter
                      ↓
                  Anthropic
```

The reverse path must also be demonstrated:

``` text
Anthropic
    ↓
Anthropic Base Adapter
    ↓
CIM 1.0
    ↓
System Prompt Filter
    ↓
CIM 1.0
    ↓
OpenAI Base Adapter
    ↓
OpenAI
```

This should execute through SIE rather than through a special-purpose
test pipeline.

The milestone therefore validates together:

-   manifest parsing;
-   component discovery;
-   component registration;
-   CIM version compatibility;
-   Base Adapter IN and OUT contracts;
-   generated CIM representation;
-   Filter discovery and invocation;
-   System Prompt transformation;
-   SIE execution ordering;
-   cross-protocol materialisation;
-   absence of direct Adapter-to-Adapter knowledge.

## 9. Initial Components Become Development Templates

The first implementations are intended to become templates for
subsequent development.

### Base Adapter Template

``` text
OpenAI Base Adapter
       ↓
Anthropic Base Adapter validates portability
       ↓
Future Base Protocol Adapters
```

The second Adapter is important because it tests whether the Base
Adapter contract and CIM were accidentally shaped around OpenAI.

### Transformational Filter Template

``` text
System Prompt Filter
       ↓
Future Transformational Filters
```

### Private-Model / Observational Filter Template

Cognition will subsequently establish the second major Filter pattern:

``` text
Cognition Filter
       ↓
Private Ask / Response through SIE + Core
       ↓
Future Observational / Private-Model Filters
```

The templates should be documented and structured for extension rather
than replaced after the architectural tests pass.

## 10. SIE Validation Standard

The first implementation should demonstrate substantially more than:

> SIE can call three Python objects in sequence.

The required proof is:

> **A discovered OpenAI Adapter can canonicalise a real interaction, a
> discovered protocol-independent Filter can transform that interaction,
> and a separately discovered Anthropic Adapter can materialise it into
> a different external protocol without any of those components knowing
> about one another.**

If this succeeds in both directions, the implementation has provided
meaningful evidence for the central CIM/SIE architecture.

## 11. Implementation Sequence

The initial implementation sequence is therefore:

### Stage 1 --- Manifest Foundation

Implement the currently defined Adapter and Filter manifests,
validation, discovery and registries.

Do not redesign the manifest model unless implementation demonstrates a
concrete deficiency.

### Stage 2 --- CIM 1.0 Semantic Analysis

Use the CIM 1.0 coverage checklist in Section 4.0 to compare the OpenAI and
Anthropic interaction representations required for the initial implementation.

For each covered interaction element, identify the canonical interaction
semantics required to preserve the source meaning across the Adapter IN → CIM
→ Adapter OUT boundary. Do not omit a supported source semantic merely because
the same fact exists elsewhere in Lumen or because an initial destination does
not require it.

Separate interaction semantics from incidental transport implementation, while
preserving explicit interaction requests such as streaming intent where they
affect the meaning of the interaction.

During semantic analysis, distinguish structural representability from value
compatibility. A destination structure that maps to a CIM semantic does not
imply that Moderari can prove equivalence for every possible source and
destination value. Preserve mapped values by default, define explicit
representation transforms only where justified, and leave intentional semantic
correction to Filters.

### Stage 3 --- CIM 1.0 Schema

Create the authoritative versioned CIM 1.0 schema.

Establish schema validation and the initial language-generation
approach.

### Stage 4 --- Generated Python CIM

Generate the Python CIM representation and prove that Adapter and Filter
contracts can consume it without protocol-specific assumptions leaking
into SIE.

### Stage 5 --- OpenAI Base Adapter

Implement the defined CIM 1.0 subset for OpenAI IN and OUT.

Validate:

``` text
OpenAI → CIM → OpenAI
```

### Stage 6 --- Anthropic Base Adapter

Implement the same CIM 1.0 semantic subset for Anthropic IN and OUT.

Validate:

``` text
Anthropic → CIM → Anthropic
```

Then validate cross-protocol conversion in both directions.

### Stage 7 --- System Prompt Filter

Implement the real System Prompt Filter against CIM 1.0.

Validate its behaviour independently of either external protocol.

### Stage 8 --- SIE Execution Flow

Execute the complete discovered/resolved path through SIE:

``` text
OpenAI → Adapter → CIM → Filter → CIM → Adapter → Anthropic
```

and:

``` text
Anthropic → Adapter → CIM → Filter → CIM → Adapter → OpenAI
```

SIE must perform flow control only. Any protocol or message
transformation appearing inside SIE is an architectural failure signal.

### Stage 9 --- Specialised Adapter / Qwen

Once the Base Adapter + CIM cross-interface topology is proven, introduce
the Qwen Specialised Adapter to validate provider/model specialisation of
a Base Adapter. The first SpAd should exercise the manifest relationship
`base_adapter + provider + model` and prove that the resolved SpAd is one
complete effective Adapter at runtime.

This must not alter the established cross-interface Base Adapter topology
or introduce a runtime SpAd → Base Adapter execution chain.

### Stage 10 --- Cognition

Introduce Cognition to validate private-model/observational Filter
execution, including `filter_execution_id` correlation and the SIE →
Moderari Core → provider/model → Core → SIE → Filter round trip.

## 11. Test the Production Moderari Package Through the SIE Entry Point

The SIE test environment must exercise the **production Moderari
package**, not a separate miniature SIE application.

The complete Moderari Python package should be built/installed normally.
Pytest then imports and invokes the production SIE entry point directly.

Initial SIE tests deliberately do **not** start the complete Moderari
service and do not require:

-   HTTP/FastAPI;
-   MongoDB;
-   Praebere;
-   Pontis;
-   Nuntius;
-   Ollama;
-   OpenAI API access;
-   Anthropic API access.

The distinction is:

> **Test the real SIE code inside the real Moderari package, while
> substituting controlled fixtures for the services and runtime
> environment that will eventually surround it.**

Conceptually:

``` text
                  pytest
                     │
             SIE Scenario Harness
                     │
       ┌─────────────┼──────────────┐
       │             │              │
 Scenario        EMD Fixture    Component Fixtures
       │             │              │
       └─────────────┼──────────────┘
                     ↓
             Production SIE API
                     ↓
          Production SIE Components
          ┌──────────┼──────────┐
          ↓          ↓          ↓
      Adapters      CIM       Filters
```

Full Moderari service integration tests are a later layer and should
prove Core/SIE, Praebere/EMD and real provider integration without
repeating the complete combinatorial SIE scenario suite.

## 12. Create a File-Driven SIE Scenario Harness

The Scenario Harness is part of the IP-01 foundation and should be
created early enough that every subsequent Adapter, Filter and resolver
feature is added together with executable scenarios.

A scenario should provide the same classes of input SIE will eventually
receive from the surrounding Lumen services:

``` text
session configuration
EMD
client interface/representation
raw client interaction
Filter configuration
discovered component manifests
```

A representative layout is:

``` text
tests/
└── sie/
    ├── scenarios/
    │   ├── openai_to_anthropic/
    │   │   ├── scenario.yaml
    │   │   ├── emd.yaml
    │   │   ├── input.json
    │   │   ├── expected_cim_in.json
    │   │   ├── expected_cim_filtered.json
    │   │   └── expected_output.json
    │   │
    │   ├── anthropic_to_openai/
    │   │   └── ...
    │   └── ...
    │
    └── test_sie_scenarios.py
```

Exact filenames and serialization formats may change during
implementation, but the scenario-driven principle should remain.

The harness is not a replacement SIE and must contain no protocol,
Filter or resolution logic that belongs in production code.

## 13. Validate at Three Levels

A successful scenario must validate more than the final external JSON.

### 13.1 External Result

For an OpenAI-to-Anthropic scenario:

``` text
actual Anthropic representation == expected Anthropic representation
```

This proves the externally observable result.

### 13.2 CIM Boundaries

The harness should be able to validate the canonical interaction at
significant component boundaries:

``` text
OpenAI Adapter IN
        ↓
expected CIM before Filter

System Prompt Filter
        ↓
expected CIM after Filter
```

This makes failures attributable. If the final Anthropic output is
wrong, the test can distinguish failure in:

-   OpenAI → CIM mapping;
-   Filter transformation;
-   CIM → Anthropic mapping.

These checkpoints validate public architectural boundaries, not private
SIE implementation details.

### 13.3 Resolution and Execution Decisions

The harness should validate the production resolution result / Session
Execution Plan information that is architecturally significant, for
example:

``` text
CIM version
Adapter A identity/version
Filter identities/versions/order
Adapter B identity/version
availability outcome
```

This ensures that a correct final payload was produced through the
intended resolved architecture rather than accidentally through an
incorrect path.

The production resolution result should therefore be deterministic and
inspectable without pytest reaching into private SIE state.

## 14. EMD and Negative Scenarios

The harness must exercise real EMD documents conforming to the EMD
schema.

This enables deterministic availability tests such as:

``` text
Required Filter characteristics present
        ↓
AVAILABLE
```

and:

``` text
Required Filter characteristic absent
        ↓
NOT AVAILABLE
```

It should also validate incompatible foundation contracts, for example:

``` text
Adapter requires CIM 1.0
Filter requires CIM 1.1
        ↓
configuration rejected
```

Later Specialised Adapter scenarios should prove both sides of its
eligibility rule:

``` text
OpenAI client interface
provider = ollama
OpenAI provider/model interface
matching Qwen model selector
        ↓
Ollama/Qwen Specialised Adapter eligible
```

versus:

``` text
Anthropic client interface
provider = ollama
OpenAI provider/model interface
matching Qwen model selector
        ↓
Specialised Adapter NOT eligible
Base Anthropic + Base OpenAI path resolved
```

Negative scenarios are part of the executable architecture specification
rather than secondary error tests.

## 15. Harness Outputs

For a scenario execution, the harness should be able to capture and
assert:

``` text
availability/resolution result
Session Execution Plan
CIM boundary representations
final external representation
architecturally relevant execution ordering
```

This must be achieved through legitimate production outputs or a clean
observation boundary, not by embedding test-only routing behaviour
inside SIE.

If an execution observer is required, its purpose is to expose
component-boundary events for diagnostics/testing; it must not become
part of the transformation or routing semantics.

## 16. Revised Foundation Sequence

The implementation prerequisites are now:

``` text
1. Manifest schemas/contracts
        ↓
2. EMD schema
        ↓
3. CIM 1.0 schema shaped by OpenAI + Anthropic
        ↓
4. Generated/internal CIM representation
        ↓
5. Component discovery and registry
        ↓
6. SIE availability/resolution API
        ↓
7. Scenario Harness
        ↓
8. OpenAI Base Adapter
   Anthropic Base Adapter
   System Prompt Filter
        ↓
9. Cross-protocol SIE scenario validation
```

The Scenario Harness is intentionally introduced before the production
components are complete. It becomes the continuing executable
specification through which those components are developed.

## 17. Praebere Multi-Interface Discovery — Future Companion Change

This work is outside the present Moderari IP-01 implementation scope, but the new Adapter interface identity creates a future Praebere requirement that must be retained.

Praebere is presently implemented around Ollama. As additional providers and interaction interfaces are supported, discovery should not be treated as generic protocol-name classification inside SIE. Praebere provider implementations will need to determine which known interaction endpoints/interfaces a configured provider exposes and map those observed endpoints to Lumen's canonical interface aliases.

Conceptually:

``` text
Praebere Provider Implementation
        ↓
known/discoverable provider endpoints
        ↓
probe / identify available interaction interfaces
        ↓
map endpoint/interface → Lumen interface alias
        ↓
EMD reports effective interface alias
```

For example, a provider may expose more than one usable interface:

``` text
Ollama
 ├── native chat interface
 └── OpenAI-compatible chat-completions interface
```

Praebere may therefore eventually report multiple available interfaces or resolve the effective interface selected for a session. The exact multi-interface discovery and selection design belongs to the future Praebere work and is deliberately not designed as part of IP-01.

The architectural boundary for the present work is:

> **Praebere discovers or establishes the provider/model interaction interface and supplies its canonical alias in the EMD. SIE consumes that alias and resolves an Adapter; SIE does not discover provider endpoints or classify protocols.**

This future change should be carried into the Praebere companion work when multi-provider/multi-interface discovery is implemented.

## 18. Implementation Discipline

Throughout IP-01:

> **Every new piece should prove or extend an existing architectural
> contract. A new architecture should be introduced only when
> implementation evidence demonstrates that the existing contract is
> insufficient.**

Existing Moderari code is evidence and a source of behaviour to migrate;
it is not automatically the architecture of the new SIE.

Likewise, the first implementation should contain enough real complexity
to challenge the architecture. A successful simplistic test does not
prove that discovery, CIM, Adapter and Filter boundaries remain valid
under realistic interaction semantics.

## 19. First Implementation Checkpoint

The first substantial checkpoint is complete when all of the following
are true:

-   Adapter and Filter manifests are implemented and discoverable;
-   one active CIM 1.0 schema is authoritative;
-   a Python CIM representation is generated/derived from that schema;
-   OpenAI Base Adapter maps its supported subset to/from CIM;
-   Anthropic Base Adapter maps the same semantic subset to/from CIM;
-   System Prompt Filter transforms CIM without protocol knowledge;
-   SIE directs the resolved component flow without transforming
    interaction content;
-   OpenAI → CIM → System Prompt → CIM → Anthropic succeeds;
-   Anthropic → CIM → System Prompt → CIM → OpenAI succeeds;
-   each Adapter remains unaware of the opposing protocol;
-   the Filter remains unaware of either external protocol;
-   the components are retained as production foundations/templates
    rather than discarded as test implementations.

At that point IP-01 has moved beyond a structural proof and demonstrated
the core interoperability claim on which the Standalone Interaction
Engine is based.
