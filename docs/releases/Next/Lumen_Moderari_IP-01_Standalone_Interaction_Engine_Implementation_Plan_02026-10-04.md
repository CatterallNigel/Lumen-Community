# Lumen Moderari Redesign --- IP-01 Implementation Plan

## Moderari Standalone Interaction Engine: CIM, Adapter and Filter Foundations

**Date:** 02026-10-04\
**Status:** Implementation planning baseline\
**Scope:** Moderari standalone development, before Lumen Stack
integration

------------------------------------------------------------------------

# 1. Purpose

IP-01 establishes the new Moderari interaction-engine foundations
outside the running Lumen Stack so that the Canonical Interaction Model
(CIM), Adapter Framework and Filter Framework can be implemented,
exercised and verified independently before integration with Praebere,
Pontis, Rogare, Vestigare, Repetere or Nuntius.

The work package is deliberately implementation-focused. It does not
reopen the consolidated Moderari architecture.

The principal objective is to produce a working, testable interaction
engine that can accept synthetic session/model facts and interaction
requests and deterministically:

1.  recognise a supported client interaction representation or return
    `Unknown`;
2.  resolve the normal client-to-provider Adapter path;
3.  translate between supported published representations through CIM
    when required;
4.  apply provider/model-specific Specialised Adapter behaviour without
    introducing model-specific logic into Moderari Core;
5.  discover, validate and execute Filters;
6.  evaluate Filter requirements against supplied Effective Model
    Definition facts;
7.  materialise Filter output in the recognised client representation
    using mandatory Adapter loopback when required;
8.  preserve execution-path provenance and generic execution evidence
    needed by later Moderari integration; and
9.  reject unsupported or unsafe execution paths explicitly rather than
    silently compensating.

IP-01 is complete when these behaviours can be demonstrated through an
isolated test harness without requiring any other running Lumen service
or a live model provider.

------------------------------------------------------------------------

# 2. Architectural Baseline

The consolidated architecture documents are authoritative for IP-01:

-   `Lumen_Moderari_Redesign_Proposal_02026-10-03_Consolidated.md`
-   `Lumen_Moderari_Adapter_Architecture_02026-10-03_Consolidated.md`
-   `Lumen_Moderari_Filter_Architecture_02026-10-03_Consolidated.md`
-   `Lumen_Moderari_Canonical_Interaction_Model_Architecture_02026-10-03_Consolidated.md`
-   `Lumen_Praebere_Rogare_Pontis_Vestigare_Repetere_Companion_Changes_02026-10-03_Consolidated.md`

The previous high-level implementation plan remains useful background,
but IP-01 deliberately changes the implementation sequence by developing
the Adapter, CIM and Filter foundations before Praebere integration.

The following boundaries are fixed:

> **Praebere Model Profile = knowledge**\
> **Moderari Adapter = compatibility**\
> **Moderari Filter = executable intentional transformation**\
> **Moderari Filter Profile = user configuration convenience**

For IP-01, Praebere knowledge is represented by synthetic test inputs.
IP-01 must not implement competing provider/model knowledge.

------------------------------------------------------------------------

# 3. Implementation Principles

## 3.1 Standalone First

The interaction engine must be executable and testable without:

-   Praebere;
-   Pontis;
-   Rogare;
-   Nuntius;
-   Vestigare;
-   Repetere;
-   Servire;
-   MongoDB;
-   Ollama;
-   Colibri; or
-   any live model.

External-service facts are supplied through explicit test fixtures or
harness inputs.

## 3.2 Preserve M0.1

The current `main` branches remain the M0.1 Research Foundation Build.

IP-01 is developed on the Moderari redesign branch. Equivalent redesign
branches may exist in the other Lumen services, but IP-01 should not
require changes to them.

## 3.3 No Architecture-by-Implementation

Implementation convenience must not alter established responsibility
boundaries.

In particular:

-   Filters must not use CIM.
-   CIM must not contain provider/model quirks.
-   Praebere concepts must not be reimplemented inside Moderari.
-   Pontis must not become a representation classifier.
-   Adapter translation must not become intentional interaction
    transformation.
-   Filter transformation must not be treated as representation
    translation.

## 3.4 Explicit Failure

Unsupported, ambiguous or incompatible paths must produce explicit typed
results/errors.

The implementation must not silently:

-   guess an interaction representation;
-   discard documented semantics;
-   substitute an Adapter;
-   substitute a Filter;
-   bypass mandatory Filter loopback;
-   introduce model-specific compensation into Core; or
-   reinterpret missing provider-representation knowledge as client
    `Unknown`.

## 3.5 Deterministic Resolution

Given the same:

-   installed extension set;
-   client representation result;
-   provider representation;
-   provider identifier;
-   model identifier;
-   Adapter Enabled/Disabled state;
-   selected Filters; and
-   Effective Model Definition,

the engine must resolve the same execution path.

------------------------------------------------------------------------

# 4. IP-01 Scope

IP-01 contains six implementation areas:

``` text
Moderari Standalone Interaction Engine
│
├── Representation Framework
│   ├── representation identities
│   ├── request recognition
│   └── Unknown handling
│
├── CIM
│   ├── schema/version
│   ├── semantic objects
│   └── validation
│
├── Adapter Framework
│   ├── Base Representation Adapters
│   ├── Specialised Adapters
│   ├── registry/discovery
│   ├── deterministic resolver
│   ├── normal execution translation
│   └── Filter loopback translation
│
├── Filter Framework
│   ├── registry/discovery
│   ├── requirement evaluation
│   ├── configuration validation
│   ├── execution
│   └── Final Filter Output
│
├── Execution Evidence
│   ├── resolved component identity/version
│   ├── filter_execution_id
│   ├── provenance
│   └── loss/incompatibility evidence
│
└── Standalone Harness
    ├── synthetic model/session facts
    ├── request fixtures
    ├── extension fixtures
    └── acceptance scenarios
```

------------------------------------------------------------------------

# 5. Explicitly Out of Scope

IP-01 does **not** implement:

-   Praebere session reservations;
-   Praebere Effective Model Definition construction;
-   provider lifecycle or model activation;
-   Pontis session routing;
-   Nuntius `\obt` transport;
-   Rogare session configuration UI;
-   Servire administration UI;
-   Vestigare transport or persistence;
-   Repetere replay orchestration;
-   complete Moderari session lifecycle;
-   immutable production Session Execution State;
-   production MongoDB persistence;
-   live Ollama/Colibri execution;
-   hot loading/unloading of Python extensions;
-   third-party Base Representation Adapters;
-   arbitrary CIM extension by plugins;
-   response-direction Filters;
-   Filter Profile administration UI; or
-   final policy for every possible lossy published-representation
    conversion.

IP-01 may define data structures that later services will consume, but
it must not prematurely implement those services.

------------------------------------------------------------------------

# 6. Proposed Internal Package Boundary

The exact filenames may change during implementation, but the code
should preserve the following conceptual separation:

``` text
moderari/
└── interaction/
    ├── representations/
    │   ├── types.py
    │   ├── resolver.py
    │   ├── registry.py
    │   └── recognition.py
    │
    ├── cim/
    │   ├── schema.py
    │   ├── models.py
    │   ├── validation.py
    │   └── version.py
    │
    ├── adapters/
    │   ├── contracts.py
    │   ├── registry.py
    │   ├── discovery.py
    │   ├── resolver.py
    │   ├── execution.py
    │   ├── loopback.py
    │   ├── openai/
    │   ├── anthropic/
    │   └── specialised/
    │       └── qwen/
    │
    ├── filters/
    │   ├── contracts.py
    │   ├── registry.py
    │   ├── discovery.py
    │   ├── requirements.py
    │   ├── execution.py
    │   └── builtins/
    │
    ├── execution/
    │   ├── facts.py
    │   ├── decisions.py
    │   ├── provenance.py
    │   └── errors.py
    │
    └── harness/
        ├── fixtures/
        ├── scenarios/
        └── runner.py
```

The purpose of this structure is separation of responsibility, not
directory structure for its own sake.

------------------------------------------------------------------------

# 7. Core Standalone Input Model

The standalone engine requires explicit synthetic inputs corresponding
to facts that the integrated Stack will later supply.

A minimum execution input should distinguish:

``` text
Session Configuration Facts
├── provider identifier
├── provider interaction representation
├── provider model identifier
├── Effective Model Definition
├── Adapter execution state: Enabled / Disabled
└── selected Filter configuration, or None

First-Ask Facts
└── raw client request boundary
```

The engine must not derive provider/model facts from the request.

The client representation is established only from the request boundary
by the Representation Resolver.

Provider interaction representation is supplied as authoritative
configuration input.

A missing provider representation is therefore an incomplete
configuration error in the standalone harness, not the
`Unknown client representation` path.

------------------------------------------------------------------------

# 8. Work Stage IP-01.1 --- Foundation Types and Execution Results

Implement the common typed objects required by the standalone engine
before implementing translation behaviour.

Initial types should cover:

-   representation identity;
-   known/Unknown client representation result;
-   provider/model execution facts;
-   Effective Model Definition fixture;
-   Adapter state;
-   Adapter identity/version;
-   Filter identity/version;
-   Filter configuration;
-   execution-path decision;
-   translation result;
-   Filter execution result;
-   Final Filter Output;
-   incompatibility/failure result;
-   provenance/origin; and
-   semantic-loss reporting.

Avoid large generic dictionaries where a stable domain type is already
known.

### Acceptance

-   Core types can be instantiated without any Lumen service.
-   Known representation and `Unknown` are distinct states.
-   Provider representation cannot accidentally be treated as client
    recognition.
-   Execution decisions are inspectable and serialisable for tests.
-   Failure conditions have machine-testable identities rather than
    relying on message strings.

------------------------------------------------------------------------

# 9. Work Stage IP-01.2 --- Representation Registry and Client Recognition

Implement the representation-recognition boundary independently from CIM
translation.

Each Lumen-controlled Base Representation Adapter should be able to
contribute recognition/validation knowledge for its published external
representation.

The Representation Resolver must return:

``` text
Known(<representation>)
```

or:

``` text
Unknown
```

It must not guess when evidence is insufficient or ambiguous.

Recognition must remain separate from CIM mapping.

### Initial recognition targets

-   OpenAI-compatible published request representation;
-   Anthropic published request representation;
-   deliberately ambiguous fixture;
-   unsupported/unknown fixture.

### Acceptance

-   OpenAI fixture is recognised as OpenAI.
-   Anthropic fixture is recognised as Anthropic.
-   ambiguous fixture returns `Unknown`.
-   unsupported fixture returns `Unknown`.
-   recognition does not invoke CIM.
-   recognition does not inspect provider/model identity to decide
    client representation.
-   a Specialised Adapter cannot make an Unknown representation known.

------------------------------------------------------------------------

# 10. Work Stage IP-01.3 --- CIM 1.0

Define the first concrete CIM schema from the published OpenAI and
Anthropic interaction semantics required by the initial Adapter use
cases.

The schema must be deliberately smaller than the union of both APIs.

Initial candidate semantic concepts are:

``` text
Interaction
├── system context
├── messages
│   ├── role
│   └── content
├── tool definitions
│   ├── name
│   ├── description
│   └── input schema
├── tool calls
│   ├── call identity
│   ├── tool identity
│   └── arguments
├── tool results
│   ├── call identity
│   └── result content
└── compatibility-critical request options
```

The final CIM 1.0 content must be derived during implementation from the
concrete OpenAI/Anthropic mappings rather than treating the illustrative
architecture schema as final.

CIM must have:

-   schema identity;
-   explicit version;
-   typed semantic objects;
-   validation;
-   clear optional/required fields; and
-   an explicit extension/evolution boundary.

### Acceptance

-   CIM instances can be constructed and validated independently.
-   CIM contains no Qwen/provider-specific concepts.
-   CIM contains no Filter concepts.
-   CIM is not exposed as a network protocol.
-   schema version is explicit.
-   unsupported semantic material cannot disappear silently during
    conversion.

------------------------------------------------------------------------

# 11. Work Stage IP-01.4 --- Base Representation Adapter Contract

Implement the Lumen-controlled Base Representation Adapter contract.

A Base Adapter must support the operations required to:

``` text
external representation → CIM
CIM → external representation
```

and declare:

-   identity;
-   implementation version;
-   Lumen extension API version;
-   external representation identity;
-   compatible CIM version(s);
-   supported direction(s); and
-   recognition/validation capability where applicable.

Base Representation Adapters are Lumen-controlled. Third-party extension
discovery must not permit a plugin to introduce a new Base
Representation Adapter.

### Acceptance

-   a Base Adapter can map external material to CIM;
-   a Base Adapter can materialise CIM into its external representation;
-   compatibility with CIM version is checked;
-   the contract contains no provider/model-specific matching;
-   the contract can be exercised without the resolver.

------------------------------------------------------------------------

# 12. Work Stage IP-01.5 --- OpenAI Base Representation Adapter

Implement the first concrete Base Representation Adapter against the
published OpenAI interaction representation.

The first implementation should cover only the semantic concepts
required for CIM 1.0 and the acceptance scenarios.

Tests should include round-trip cases where semantically representable
content survives:

``` text
OpenAI → CIM → OpenAI
```

Round-trip equality should be semantic rather than dependent on
irrelevant JSON field ordering.

### Acceptance

-   supported OpenAI request material maps to valid CIM;
-   CIM material maps back to valid OpenAI representation;
-   tool definitions/calls/results required by CIM 1.0 survive
    semantically;
-   unsupported documented semantics are reported rather than silently
    discarded.

------------------------------------------------------------------------

# 13. Work Stage IP-01.6 --- Anthropic Base Representation Adapter

Implement Anthropic as the second Base Representation Adapter.

This stage is important because it proves that CIM is a genuine semantic
bridge rather than an OpenAI-shaped intermediate object.

Required paths include:

``` text
Anthropic → CIM → Anthropic
OpenAI → CIM → Anthropic
Anthropic → CIM → OpenAI
```

### Acceptance

-   supported Anthropic material maps to valid CIM;
-   CIM material maps to valid Anthropic representation;
-   OpenAI ↔ Anthropic translation operates without either Adapter
    containing direct knowledge of the other;
-   interoperability loss is surfaced explicitly.

------------------------------------------------------------------------

# 14. Work Stage IP-01.7 --- Adapter Registry and Discovery

Implement restart-based Adapter discovery and the internal Adapter
Registry.

The registry contains:

``` text
Lumen Base Representation Adapters
Specialised Adapters
```

Discovery should support external extension packages through a
configured extension location.

Initial extension manifest validation should cover:

-   extension type;
-   identity;
-   implementation version;
-   extension API version;
-   entry point;
-   base representation specialised;
-   provider;
-   model/family matching key; and
-   configuration metadata where required.

Duplicate Specialised Adapter provider+model keys must cause
registry/startup failure.

### Acceptance

-   valid extensions are discovered;
-   malformed manifests are rejected;
-   incompatible extension API versions are rejected;
-   duplicate Specialised Adapter keys fail deterministically;
-   third-party manifests cannot register a new Base Representation
    Adapter;
-   discovery requires restart only;
-   no provider/model-specific extension is hardcoded into Core.

------------------------------------------------------------------------

# 15. Work Stage IP-01.8 --- Deterministic Adapter Resolver

Implement the architectural resolution order exactly.

For normal execution:

``` text
1. recognise client representation
2. if Unknown:
      No Filters       → No Adapter / pass-through
      Filters selected → reject
3. require configured provider representation
4. exact provider+model Specialised Adapter
5. provider+derived-family Specialised Adapter
6. same known representation → No Adapter
7. Base Representation Adapter path through CIM
8. otherwise reject known incompatibility
```

Family derivation is mechanical:

``` text
qwen2.5-coder:14b
→ qwen2.5-coder
```

No priority, scope, specificity or stacking system is to be introduced.

### Adapter Disabled

The normal execution Adapter Enabled/Disabled control must be
represented explicitly.

Disabled means no normal client↔provider translation is performed.

It does **not** disable mandatory Filter loopback.

The exact integrated user-control behaviour belongs later, but IP-01
must prove this distinction.

### Acceptance

Every resolution branch above has a deterministic unit test.

In particular:

-   exact beats family;
-   family beats same-representation pass-through;
-   same representation short-circuits CIM when no Specialised Adapter
    applies;
-   Specialised Adapters do not stack;
-   known differing representations use Base/CIM path when available;
-   known incompatible path rejects;
-   Unknown client + no Filters passes unchanged;
-   Unknown client + Filters rejects;
-   missing provider representation is an incomplete-configuration
    failure;
-   Disabled normal Adapter processing does not disable loopback.

------------------------------------------------------------------------

# 16. Work Stage IP-01.9 --- Specialised Adapter Contract

Implement the Specialised Adapter extension contract.

A Specialised Adapter:

-   specialises an existing Lumen Base Representation Adapter;
-   does not define a new interaction representation;
-   does not extend CIM;
-   declares provider/model matching;
-   may match exact model or derived family;
-   owns correction of provider/model deviations;
-   owns any loss introduced by its override behaviour.

The initial contract should allow a Specialised Adapter to intercept the
appropriate conversion/materialisation point without requiring
model-specific branches in Core.

The exact override mechanism was deliberately deferred by the
architecture and is therefore a genuine implementation design decision
inside this stage.

The chosen mechanism must be documented before the Qwen migration is
considered complete.

### Acceptance

-   a Specialised Adapter can alter behaviour for a known provider/model
    deviation;
-   Core contains no `if qwen`, `if ollama`, etc.;
-   the Specialised Adapter cannot register a new representation;
-   the Specialised Adapter cannot modify the CIM schema;
-   exact/family matching uses registry metadata only;
-   only one effective Specialised Adapter is used for an execution.

------------------------------------------------------------------------

# 17. Work Stage IP-01.10 --- Qwen Specialised Adapter

Migrate the existing Qwen compatibility behaviour into the Specialised
Adapter architecture.

Before migration:

1.  inspect the current Moderari Qwen translator;
2.  identify the exact deviation(s) it compensates for;
3.  separate published OpenAI semantics from Qwen-specific behaviour;
4.  create regression fixtures from current known behaviour.

Then implement the Qwen Specialised Adapter without carrying old
architectural coupling into the new framework.

### Acceptance

-   existing required Qwen compatibility behaviour is reproduced;
-   Qwen-specific logic exists only in the Specialised Adapter;
-   removing the Qwen extension removes Qwen-specific compensation
    without affecting Base OpenAI behaviour;
-   the Qwen Adapter resolves by provider/model metadata;
-   family matching works for applicable Qwen model variants;
-   no Qwen concept appears in CIM.

------------------------------------------------------------------------

# 18. Work Stage IP-01.11 --- Normal Adapter Execution Path

Implement execution of the resolved normal client→provider path.

Possible effective paths are:

``` text
No Adapter / pass-through

Specialised Adapter path

Base Adapter:
Client Representation
    → CIM
    → Provider Representation
```

The execution result must carry generic metadata describing:

-   client representation;
-   provider representation;
-   Adapter state;
-   effective Adapter identity/version where applicable;
-   CIM version where used;
-   Specialised Adapter identity/version where applicable;
-   loss/incompatibility information; and
-   final provider-bound representation.

This metadata is an internal result in IP-01. Vestigare integration is
later.

### Acceptance

-   pass-through does not introduce CIM;
-   same-representation No Adapter does not normalise through CIM;
-   Base translation uses CIM;
-   Specialised path reports its effective component identity/version;
-   execution output and decision metadata can be inspected
    independently.

------------------------------------------------------------------------

# 19. Work Stage IP-01.12 --- Filter Contract and Registry

Implement the self-describing Filter contract and restart-based Filter
discovery.

A Filter declares:

-   identity;
-   implementation version;
-   extension API version;
-   description;
-   entry point;
-   execution stage;
-   output representation;
-   positive model requirements;
-   configuration schema.

Filters operate only in the to-model direction.

Filters do not:

-   consume CIM;
-   produce CIM;
-   own model truth;
-   classify the client representation;
-   transform the client-visible response.

### Acceptance

-   independently packaged Filters can be discovered;
-   invalid manifests are rejected;
-   unavailable Filters can be identified;
-   Filter code is not hardcoded into the framework;
-   Filter contract contains an explicit external output representation;
-   Filter API contains no CIM dependency.

------------------------------------------------------------------------

# 20. Work Stage IP-01.13 --- Filter Requirement Evaluation

Implement generic Filter requirement evaluation against a supplied
synthetic Effective Model Definition.

The evaluator must preserve the architectural rule:

> `Unknown` does not satisfy a positive requirement.

Requirements are owned semantically by Moderari Filters; model facts are
supplied as external authoritative input.

The initial implementation should avoid inventing a broad
model-characteristic ontology inside IP-01. Use only characteristics
required by the initial Filters/tests.

### Acceptance

-   satisfied positive requirement → applicable;
-   false requirement → not applicable;
-   Unknown requirement value → not applicable;
-   Filter does not query provider/model runtime;
-   Filter does not modify model facts;
-   requirement evaluation is generic and not Filter-specific Core
    logic.

------------------------------------------------------------------------

# 21. Work Stage IP-01.14 --- Filter Execution Identity and Provenance

Implement generic Filter invocation ownership.

For every Filter invocation the Filter Framework creates:

``` text
filter_execution_id
```

The framework, not the Filter, owns this identity.

Execution evidence should distinguish origin from:

-   client;
-   Filter; and
-   model/private Filter interaction where applicable.

Origin must be known from execution-path state, not inferred from role,
content or protocol fields.

Provenance must not be inserted into the external interaction
representation.

### Acceptance

-   every Filter invocation receives a unique execution identity;
-   execution identity is stable across its associated artefacts;
-   origin is explicit internal metadata;
-   external protocol material remains free of Lumen provenance fields.

------------------------------------------------------------------------

# 22. Work Stage IP-01.15 --- Filter Execution and Mandatory Loopback

Implement the complete Filter execution path.

``` text
Filter Framework
    │
    ├── create filter_execution_id
    │
    ▼
Filter
    │
    └── Filter-produced material
        in Filter native output representation
    │
    ▼
Filter Framework
    │
    ├── output representation == client representation
    │       → Final Filter Output
    │
    └── differs
            → Adapter Framework loopback
            → Final Filter Output
               in client representation
```

Loopback translation is mandatory when representations differ.

It must operate even when normal client→provider Adapter processing is
Disabled.

Only the **Final Filter Output** is treated as the Filter result.

Transient Filter-produced representation and CIM intermediates are
implementation mechanics.

### Acceptance

-   Filter output already in client representation requires no loopback;
-   OpenAI Filter output + Anthropic client uses OpenAI→CIM→Anthropic
    loopback;
-   loopback remains active with normal Adapter state Disabled;
-   Final Filter Output is always in the recognised client
    representation;
-   Unknown client representation + selected Filter rejects before
    Filter execution;
-   transient CIM is not exposed as Filter output;
-   pre-loopback material is not confused with Final Filter Output.

------------------------------------------------------------------------

# 23. Work Stage IP-01.16 --- Initial Test Filter

Before migrating the real System Prompt Filter, implement a deliberately
simple test Filter.

Its purpose is to prove:

-   discovery;
-   configuration;
-   requirement evaluation;
-   execution identity;
-   output representation declaration;
-   transformation;
-   loopback;
-   Final Filter Output; and
-   provenance.

The test Filter should perform an unmistakable but simple intentional
transformation so framework defects are easy to diagnose.

It should not depend on a live model.

### Acceptance

The complete Filter framework can be tested without System Prompt
semantics or private model calls.

------------------------------------------------------------------------

# 24. Work Stage IP-01.17 --- System Prompt Filter Migration

Once the generic Filter framework is proven, migrate existing System
Prompt behaviour into the new Filter architecture.

Moderari Core must have no implicit default System Prompt.

System Prompt behaviour must exist only when the Filter is explicitly
configured.

The migration should begin with an inspection of current Moderari
behaviour to distinguish reusable implementation from legacy coupling.

### Acceptance

-   no Filter selected → no Lumen default System Prompt;
-   System Prompt Filter selected → configured prompt material is
    produced;
-   output is materialised in the client representation;
-   System Prompt-specific logic does not appear in Filter Framework
    Core;
-   behaviour is testable without a live model.

------------------------------------------------------------------------

# 25. Work Stage IP-01.18 --- Cognition Filter Contract Skeleton

The architecture identifies Cognition as the second current built-in
Filter, but its full behaviour need not block the Adapter/CIM
foundation.

IP-01 should at minimum prove that the framework can represent a Filter
that:

-   has positive model/context requirements;
-   may perform private model interaction;
-   produces external-representation interaction material;
-   associates private Ask/Response artefacts with one
    `filter_execution_id`.

If existing Cognition behaviour is sufficiently stable and reusable,
migration may proceed in IP-01. Otherwise a contract-level test
implementation is sufficient and full migration can become a subsequent
work package.

This decision should be made after inspecting the existing
implementation rather than assumed now.

------------------------------------------------------------------------

# 26. Work Stage IP-01.19 --- Published-Semantic Loss Handling

Implement an explicit result mechanism for published-representation
interoperability loss.

The architecture intentionally leaves the final policy open. IP-01
therefore should not prematurely define one universal policy.

It must, however, make loss impossible to hide.

Minimum implementation:

``` text
TranslationResult
├── success
├── incompatibility
└── semantic_loss
       ├── source semantic
       ├── target representation
       └── explanation/identity
```

Tests may initially treat semantic loss as a rejected translation unless
an individual scenario explicitly permits it.

Provider/model-specific override loss remains the responsibility of the
Specialised Adapter and must be reported separately from Base/CIM
interoperability loss.

### Acceptance

-   documented source semantics cannot silently disappear;
-   Base/CIM loss and Specialised Adapter override loss are
    distinguishable;
-   tests can assert the exact loss boundary.

------------------------------------------------------------------------

# 27. Work Stage IP-01.20 --- Standalone Scenario Harness

Build a lightweight harness around the interaction engine.

The harness is not a replacement client or temporary Lumen service.

It exists to make architectural scenarios easy to execute and inspect.

A scenario should be able to declare:

``` yaml
provider:
  id: ollama
  representation: openai
  model: qwen2.5-coder:14b

model_facts:
  ...

adapter_state: enabled

filters:
  ...

client_request:
  fixture: anthropic/tool-call.json
```

and return an inspectable result containing:

-   recognition result;
-   resolved Adapter path;
-   Filter applicability;
-   Filter execution(s);
-   loopback path(s);
-   Final Filter Output(s);
-   normal provider translation path;
-   final provider-bound interaction;
-   component identities/versions;
-   provenance;
-   loss/failure information.

The harness should be usable both from automated tests and a simple
developer-facing command line.

------------------------------------------------------------------------

# 28. Minimum Acceptance Scenario Matrix

The following scenarios form the minimum IP-01 acceptance suite.

  -----------------------------------------------------------------------
  Scenario                            Expected result
  ----------------------------------- -----------------------------------
  OpenAI client → OpenAI provider, no No Adapter
  Specialised Adapter                 

  Anthropic client → Anthropic        No Adapter
  provider                            

  OpenAI client → Anthropic provider  Base translation through CIM

  Anthropic client → OpenAI provider  Base translation through CIM

  Unknown client → provider, no       Pass through unchanged
  Filters                             

  Unknown client + Filter             Reject

  Known differing representations     Reject
  with missing Base path              

  Missing provider representation     Incomplete configuration

  Exact Specialised Adapter match     Exact Adapter used

  Exact missing, family Specialised   Family Adapter used
  Adapter match                       

  Exact + family both exist           Exact used

  Specialised Adapter + same          Specialised Adapter used
  representation                      

  Same representation, no Specialised No Adapter; no CIM
  Adapter                             

  Duplicate Specialised Adapter       Startup/registry failure
  registry key                        

  Normal Adapter Disabled, no Filters No normal translation

  Normal Adapter Disabled + Filter    Loopback still executes
  requiring loopback                  

  OpenAI-native Filter + OpenAI       No loopback
  client                              

  OpenAI-native Filter + Anthropic    Loopback through CIM
  client                              

  Filter positive requirement = true  Applicable

  Filter positive requirement = false Not applicable

  Filter positive requirement =       Not applicable
  Unknown                             

  Filter unavailable                  Profile/config invalid

  Published semantic cannot map to    Explicit loss/incompatibility
  target                              

  Qwen exact/family case              Specialised behaviour, no Core Qwen
                                      branch

  Filter invocation                   unique `filter_execution_id`

  Filter-produced material after      Final output in client
  loopback                            representation

  Client-origin and Filter-origin     provenance distinct internally
  material                            
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 29. Testing Layers

## 29.1 Unit Tests

Cover:

-   representation recognition;
-   family derivation;
-   registry uniqueness;
-   manifest validation;
-   CIM validation;
-   each Base Adapter mapping;
-   requirement evaluation;
-   resolver branches;
-   Filter execution identity;
-   provenance;
-   loss reporting.

## 29.2 Contract Tests

Every Base Adapter must run against the same Base Adapter contract
suite.

Every Specialised Adapter must run against the same Specialised Adapter
contract suite.

Every Filter must run against the same Filter contract suite.

This makes third-party extension compliance testable.

## 29.3 Translation Tests

Use fixture pairs and semantic assertions for:

``` text
OpenAI → CIM
CIM → OpenAI
Anthropic → CIM
CIM → Anthropic
OpenAI → CIM → Anthropic
Anthropic → CIM → OpenAI
```

## 29.4 Architectural Negative Tests

Negative tests are first-class requirements.

Examples:

-   Filter imports/depends upon CIM contract;
-   third-party Base Adapter registration;
-   duplicate specialised key;
-   ambiguous representation guessed as known;
-   Specialised Adapter selected while client representation is Unknown;
-   Filters allowed with Unknown client representation;
-   loopback bypassed because normal Adapter state is Disabled;
-   silent semantic loss;
-   model-specific Core branch.

Where practical these should fail mechanically rather than depend solely
on code review.

## 29.5 Regression Tests

Before migrating current Qwen/System Prompt behaviour, capture existing
required behaviour as fixtures.

The new implementation must reproduce required behaviour without
reproducing obsolete architecture.

------------------------------------------------------------------------

# 30. Implementation Order

Recommended coding order:

``` text
IP-01.1   Foundation types/results
    ↓
IP-01.2   Representation recognition
    ↓
IP-01.3   CIM 1.0
    ↓
IP-01.4   Base Adapter contract
    ↓
IP-01.5   OpenAI Base Adapter
    ↓
IP-01.6   Anthropic Base Adapter
    ↓
IP-01.7   Adapter registry/discovery
    ↓
IP-01.8   Deterministic resolver
    ↓
IP-01.9   Specialised Adapter contract
    ↓
IP-01.10  Qwen migration
    ↓
IP-01.11  Normal execution path
    ↓
IP-01.12  Filter contract/registry
    ↓
IP-01.13  Filter requirements
    ↓
IP-01.14  Filter identity/provenance
    ↓
IP-01.15  Filter execution + loopback
    ↓
IP-01.16  Test Filter
    ↓
IP-01.17  System Prompt migration
    ↓
IP-01.18  Cognition contract/migration decision
    ↓
IP-01.19  Explicit loss handling
    ↓
IP-01.20  Complete standalone scenario harness
```

This order may be implemented in small vertical slices where useful, but
later stages must not force architectural coupling back into earlier
layers.

------------------------------------------------------------------------

# 31. Existing Moderari Code Inspection Points

Existing Moderari code should be inspected selectively rather than
treated as the design source.

The first inspection should identify:

1.  current Qwen translation behaviour;
2.  current System Prompt injection/replacement behaviour;
3.  current Cognition behaviour;
4.  existing context manipulation useful to Filters;
5.  existing request/response models that may be reusable;
6.  existing provider/model conditionals that must disappear;
7.  existing tests/fixtures that capture useful behaviour.

Each piece should be classified as:

``` text
RETAIN
    implementation already fits the new responsibility

ADAPT
    useful implementation but coupled to old boundaries

MIGRATE
    behaviour belongs in a new Adapter/Filter extension

DELETE
    obsolete under the new architecture

DEFER
    belongs to later Stack integration
```

No existing code is retained merely to preserve the old path.

------------------------------------------------------------------------

# 32. Deliverables

IP-01 should produce:

1.  standalone interaction-engine package;
2.  Representation Resolver and registry;
3.  CIM 1.0 schema and typed implementation;
4.  OpenAI Base Representation Adapter;
5.  Anthropic Base Representation Adapter;
6.  Adapter registry/discovery;
7.  deterministic Adapter resolver;
8.  Specialised Adapter extension contract;
9.  migrated Qwen Specialised Adapter;
10. normal Adapter execution engine;
11. Filter registry/discovery;
12. Filter requirement evaluator;
13. Filter execution framework;
14. mandatory client-representation loopback;
15. execution identity/provenance structures;
16. simple framework Test Filter;
17. migrated System Prompt Filter;
18. Cognition Filter migration decision/contract proof;
19. explicit interoperability-loss result mechanism;
20. standalone developer harness;
21. automated unit, contract, regression and architectural acceptance
    tests;
22. concise developer documentation for creating a Specialised Adapter;
23. concise developer documentation for creating a Filter; and
24. an integration contract describing the facts future
    Praebere/Moderari session integration must supply.

------------------------------------------------------------------------

# 33. Definition of Done

IP-01 is complete when all of the following are true.

### Independence

The entire acceptance suite runs without the Lumen Stack or a live
provider.

### Representation

Moderari can positively recognise supported client representations and
return `Unknown` without guessing.

### CIM

OpenAI and Anthropic published interaction semantics required by the
supported scenarios can be represented and translated through a
versioned CIM.

### Adapter Resolution

All architectural resolution branches are deterministic and tested.

### Specialisation

Qwen-specific compatibility is implemented as a Specialised Adapter with
no Qwen-specific Core logic.

### Filters

A Filter can be independently discovered, validated, checked for
applicability and executed without CIM knowledge.

### Loopback

Filter output can be materialised into the recognised client
representation through mandatory Adapter loopback independently of the
normal Adapter Enabled/Disabled state.

### Evidence

The engine exposes component identity/version, `filter_execution_id`,
provenance and translation/loss decisions generically without depending
on Vestigare.

### Failure Behaviour

Unknown, incompatible, unavailable, duplicate and lossy cases are
explicit and testable.

### Extension Boundary

A new compliant Specialised Adapter or Filter can be added without
changing Moderari Core.

### Integration Readiness

The standalone engine has a clear input/output contract suitable for
subsequent Praebere and full Moderari session integration.

------------------------------------------------------------------------

# 34. Exit Point and Next Work Package

IP-01 deliberately stops before production Stack integration.

At completion, the next major work package should begin with
**Praebere**, using the proven standalone engine to define exactly which
authoritative session/model facts Moderari requires.

That work should establish:

-   per-session provider/model reservation;
-   provider interaction representation;
-   Effective Model Definition;
-   characteristic provenance;
-   model lifecycle/readiness; and
-   the configuration-time exchange through which Moderari evaluates
    Filter applicability.

The important change from the earlier high-level plan is that Praebere
will then be implemented against a **working Moderari interaction
contract**, rather than against an interaction framework that exists
only on paper.

------------------------------------------------------------------------

# 35. IP-01 Guiding Test

A concise test of whether IP-01 has maintained the architecture is:

> Given only synthetic provider/model facts, installed Adapter/Filter
> extensions and a raw client request, can the standalone Moderari
> interaction engine determine what it safely knows, what it must not
> guess, what representation path is required, what intentional Filter
> transformation is requested, what compatibility translation is
> required, and produce an inspectable result explaining exactly what it
> did --- without knowing anything about the rest of the Lumen Stack?

If yes, the foundation is ready for Praebere and Stack integration.
