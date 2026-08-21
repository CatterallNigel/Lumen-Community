# Lumen --- Third-Party Extension Architecture

**Status:** Architectural principle / integration policy\
**Scope:** Lumen services, external capability providers, adapters, and
future third-party integrations\
**Purpose:** Define how Lumen integrates external functionality without
becoming architecturally dependent on any external product, project,
provider, or implementation.

------------------------------------------------------------------------

## 1. Principle

Lumen may integrate with external capabilities, but it must never depend
upon a specific external implementation.

> **Providers plug into Lumen. Lumen does not reshape itself around
> providers.**

Third-party integrations exist to extend Lumen's capabilities. They must
not redefine Lumen's service responsibilities, internal architecture,
core terminology, or operating assumptions.

An external capability should be capable of being added, removed,
replaced, or upgraded without requiring the surrounding Lumen
architecture to be redesigned.

This is a hard architectural constraint, not merely an implementation
preference.

------------------------------------------------------------------------

## 2. Why This Matters

External functionality can enter a system for many reasons: research
collaboration, customer requirements, useful open-source projects,
commercial products, experimental capabilities, or business
opportunities.

Those reasons do not automatically justify architectural change.

A common engineering failure occurs when a feature is committed to
externally before its architectural implications are understood.
Engineering then inherits a requirement shaped around a particular
product or implementation and is forced to accommodate it inside an
architecture that was never designed for it.

Lumen should explicitly resist this pattern.

The existence of an interesting third-party capability establishes a
reason to **investigate an integration**, not a reason to alter Lumen's
architecture around that implementation.

Before functionality becomes a Lumen commitment, engineering must first
determine:

-   what capability is actually being requested;
-   whether that capability belongs within Lumen's scope;
-   which Lumen service owns the relevant responsibility;
-   whether an existing extension contract can support it;
-   whether a new generic contract is justified;
-   what evidence, provenance, lifecycle, and failure semantics are
    required;
-   what happens when the external capability is unavailable or removed.

The architectural abstraction must belong to Lumen.

------------------------------------------------------------------------

## 3. One Job First

Third-party integration does not override Lumen's **one job first**
principle.

Each Lumen service retains its defined responsibility. An external
capability may assist a service in performing that responsibility, but
it must not cause that service to acquire unrelated responsibilities.

For example:

-   Moderari remains responsible for orchestration even when an external
    capability participates in a transaction.
-   Pontis remains responsible for bridging and transport relationships
    rather than becoming the implementation of a particular external
    tool.
-   Praebere remains the model/provider abstraction rather than becoming
    specific to one model provider.
-   Aestimare remains responsible for assessment even when specialist
    evidence originates from an external methodology or capability.

> **An integration extends a responsibility; it does not redefine it.**

If integrating a third party appears to require changing the fundamental
job of a Lumen service, the integration boundary should be reconsidered
before implementation proceeds.

------------------------------------------------------------------------

## 4. Capability, Not Product

Lumen should model the **capability being consumed**, not the product
that currently supplies it.

Examples illustrate the distinction:

  -----------------------------------------------------------------------
  External implementation             Lumen should think in terms of
  ----------------------------------- -----------------------------------
  Pi                                  External tool/capability provision

  A linguistic glyph provider         External semantic resolution

  A live governance system such as    External transaction-governance
  FLOW                                capability

  Ollama                              Model-provider capability
  -----------------------------------------------------------------------

The names above are useful concrete implementations and development
cases. They are not architectural abstractions.

Consequently, Lumen should not develop:

``` text
Pi support
FLOW support
MEG/glyph support
Ollama-shaped architecture
```

where doing so embeds provider-specific assumptions into the core.

Instead, those implementations should help expose the generic contracts
Lumen requires.

> **Use real integrations to discover the abstraction; do not make the
> abstraction specific to the integration.**

------------------------------------------------------------------------

## 5. Real Implementations as Architectural Test Cases

Abstract plugin architectures designed without real integrations
frequently anticipate the wrong abstractions.

Lumen should therefore use genuine external implementations as test
cases when developing its extension model.

Different integrations deliberately exercise different requirements.

A tool provider may require capability discovery and invocation.

A linguistic provider may require semantic resolution and the return of
structured semantics for insertion into model context.

A governance provider may participate in the live transaction, return
structured governance evidence, and potentially request repair,
re-evaluation, or blocking.

A model provider may expose model lifecycle, availability, invocation,
and provider-specific operational state.

These differences are useful. They reveal which parts of the extension
mechanism genuinely belong in a common contract and which belong in
specialised capability contracts.

The objective is not to force every external system through one
universal interface. The objective is to establish a small set of stable
Lumen-owned contracts appropriate to different classes of capability.

------------------------------------------------------------------------

## 6. Generic Extension Properties

Where applicable, Lumen extension contracts should provide common
mechanisms for:

-   provider identity;
-   capability declaration and discovery;
-   protocol and contract version;
-   health and availability;
-   invocation;
-   structured request and response;
-   transaction and session correlation;
-   provenance;
-   timestamps;
-   failure and timeout semantics;
-   lifecycle state;
-   configuration;
-   observability;
-   evidence capture;
-   optional capability negotiation.

Not every extension will require every property.

The common mechanism should remain deliberately small.
Capability-specific semantics belong in the appropriate specialised
contract rather than accumulating in a universal provider interface.

------------------------------------------------------------------------

## 7. Provider-Specific Vocabulary Must Not Leak Into the Core

External systems will naturally have their own terminology and concepts.

Those concepts should cross into Lumen only where they represent a
genuinely useful general abstraction.

For example, an external governance implementation may describe results
using terms such as:

``` text
pass
repair
block
```

Those terms should not automatically become fundamental Moderari states.

Lumen should first determine the underlying generic semantics. A
governance-capability contract might represent concepts such as
disposition, intervention requirement, release decision, evidence, or
reason. An adapter can then map the external provider's vocabulary onto
the Lumen contract.

Similarly, a glyph is not itself the architectural capability. The
capability may be **semantic resolution**.

This protects the Lumen architecture from becoming a historical record
of whichever third-party implementations happened to be integrated
first.

------------------------------------------------------------------------

## 8. Adapters at the Boundary

Where an external implementation does not naturally expose the Lumen
contract, translation should occur at the boundary.

Conceptually:

``` text
Lumen Service
     |
     v
Lumen-owned Capability Contract
     |
     v
Provider Adapter
     |
     v
External Implementation
```

The adapter is responsible for provider-specific translation.

The Lumen service should not need detailed knowledge of the provider
behind that adapter.

This allows:

``` text
Provider A
    |
 Adapter A
    |
    +---- Lumen Capability Contract ---- Lumen

Provider B
    |
 Adapter B
    |
```

Replacing Provider A with Provider B should primarily be an
adapter/configuration concern rather than a redesign of the consuming
Lumen service.

------------------------------------------------------------------------

## 9. Optionality and Replaceability

A third-party extension should normally be:

**Optional** --- Lumen's core architecture does not cease to exist
because the extension is absent.

**Replaceable** --- another implementation of the same capability can be
substituted through the same Lumen-owned contract.

**Independently versioned** --- the provider can evolve without forcing
lock-step Lumen releases wherever contract compatibility is maintained.

**Contract-bound** --- interaction occurs through an explicit,
documented interface rather than provider-specific assumptions
distributed through the codebase.

**Observable** --- use of the external capability can be traced and
correlated with the Lumen transaction.

**Fail-explicit** --- unavailability, timeout, incompatibility,
malformed responses, and other provider failures have defined behaviour.

**Removable** --- removing an integration does not require extracting
its concepts from unrelated Lumen services.

Not every capability must be optional at runtime for every deployment. A
deployment may deliberately require a particular class of governance or
tooling. The architecture, however, should remain independent of the
implementation supplying that capability.

------------------------------------------------------------------------

## 10. Failure Must Respect the Capability's Role

Plug-and-play must not mean silently ignoring a missing provider.

Failure behaviour depends upon the role assigned to the capability.

An optional semantic enrichment provider might permit execution to
continue without enrichment.

A deployment policy might declare a governance capability mandatory, in
which case inability to obtain a governance decision may require the
transaction to fail closed.

A model-provider capability is obviously necessary when that provider is
selected for execution.

The important distinction is:

> **Lumen policy determines the consequence of capability failure; the
> provider does not determine Lumen policy.**

Required/optional status, fallback behaviour, timeout behaviour, and
failure disposition should therefore be explicit configuration or
policy.

------------------------------------------------------------------------

## 11. Provenance Is Part of the Contract

If an external capability materially influences an execution, Lumen
should be capable of establishing that it did so.

Where appropriate, evidence should identify:

-   provider identity;
-   provider/capability version;
-   contract version;
-   invocation identifier;
-   transaction/session correlation;
-   input or relevant input reference;
-   structured result;
-   relevant configuration or policy identity;
-   timing;
-   failure or intervention information;
-   provenance required to interpret the result later.

This is especially important for Aestimare.

Aestimare should not need provider-specific knowledge to know that
external evidence exists. Specialist assessment engines can consume
evidence through defined evidence contracts and assess the particular
property for which they are responsible.

------------------------------------------------------------------------

## 12. Relationship to Aestimare

Third-party capabilities may create new forms of evidence.

They should not cause Aestimare to become aware of every external
implementation.

For example, a governance provider might emit evidence about whether a
transaction remained within a defined governance boundary. A future
Aestimare governance-conformance micro-engine could assess that
evidence.

The engine's job would be defined by the assessment question, not by the
provider that generated the evidence.

Another governance implementation should therefore be capable of
supplying equivalent evidence without requiring a new provider-specific
Aestimare architecture.

This follows Aestimare's own rule:

> **Micro-engines assess one thing. Aestimare synthesises assessments.**

External evidence can justify a new micro-engine when it introduces a
genuinely new assessment question. It should not justify embedding a
third-party implementation inside an existing engine.

------------------------------------------------------------------------

## 13. Extension Discovery Before Implementation

When a potential integration appears, development should not begin with:

> "How do we add Product X?"

The investigation should instead ask:

1.  **What capability does this provide?**
2.  **Which Lumen responsibility consumes that capability?**
3.  **Is the capability already represented by a Lumen contract?**
4.  **If not, is there enough evidence from real implementations to
    justify a new abstraction?**
5.  **What is genuinely common and what is provider-specific?**
6.  **What structured information enters and leaves the capability?**
7.  **What provenance must survive the interaction?**
8.  **What are the failure, timeout, lifecycle, and versioning
    semantics?**
9.  **Can another provider implement the same contract?**
10. **Can this provider be removed without redesigning Lumen?**

If the answer to the final two questions is no, the proposed boundary
should be treated with suspicion.

------------------------------------------------------------------------

## 14. Avoid Premature Generalisation

Avoiding provider lock-in does not mean designing an enormous generic
extension framework before it is required.

Lumen should generalise from evidence.

A sensible progression is:

``` text
Real integration requirement
        |
        v
Identify capability
        |
        v
Identify owning Lumen responsibility
        |
        v
Define smallest useful Lumen contract
        |
        v
Implement provider adapter
        |
        v
Test with real implementation
        |
        v
Compare against second implementation/use case
        |
        v
Refine abstraction where evidence justifies it
```

This allows the architecture to remain extensible without becoming
speculative.

The first implementation should not own the abstraction, but neither
should Lumen attempt to predict every future implementation before
building anything.

------------------------------------------------------------------------

## 15. Commercial and Product Requests

Commercial value is a legitimate reason to investigate functionality. It
is not an architectural specification.

A customer, collaborator, salesperson, or prospective integration
partner may identify a valuable capability before engineering has
considered it. That discovery can be useful.

The engineering process must still translate the request from:

``` text
"We need integration with X."
```

into:

``` text
"What capability is required, and what is the correct Lumen-owned boundary for it?"
```

No external commitment should silently establish:

-   a new service responsibility;
-   a provider-specific core dependency;
-   a new protocol as a de facto internal standard;
-   provider vocabulary as Lumen terminology;
-   an undocumented lifecycle dependency;
-   an architectural requirement that prevents provider replacement.

This is not intended to prevent commercial experimentation. It exists so
that successful experiments do not become permanent architectural
accidents.

------------------------------------------------------------------------

## 16. Current Architectural Examples

Several existing or prospective integrations provide useful cases for
developing this philosophy.

### Pi

Pi has helped exercise Lumen's tool path and Pontis integration.

The architectural lesson is not that Pontis supports Pi. It is that
Lumen requires a way to bridge external tool capabilities.

Pi should remain replaceable.

### Model Providers

Praebere exists specifically to prevent the rest of Lumen from becoming
bound directly to a model-provider implementation.

Ollama is a provider implementation behind that abstraction, not the
definition of Praebere.

### External Linguistic Capabilities

A linguistic/glyph system could provide structured semantic resolution.

Moderari should not become glyph-aware in a provider-specific sense. A
Lumen-owned semantic-resolution capability can allow an external
provider to resolve a representation and return semantics through a
defined contract.

### Live Transaction Governance

A system such as FLOW may participate before, during, or after model
generation and provide a governance disposition and evidence.

Moderari should not become FLOW-specific. Investigation of a real FLOW
integration can instead help determine what a generic live
transaction-governance capability requires.

These cases are deliberately different.

Their value is partly the functionality they may add to Lumen and partly
the architectural pressure they provide for discovering robust extension
boundaries.

------------------------------------------------------------------------

## 17. Architectural Tests

Before accepting a third-party integration design, ask:

> **If this provider disappeared tomorrow, would Lumen's architecture
> still make sense?**

> **Could another implementation provide this capability without
> changing the consuming service?**

> **Are we integrating a capability, or embedding a product?**

> **Has provider-specific vocabulary leaked into Lumen's core
> responsibilities?**

> **Does the owning Lumen service still have one clear job?**

> **Can Vestigare establish that the external capability participated
> and what evidence it produced?**

> **Can Aestimare consume resulting evidence without knowing the
> identity of the implementation that produced it?**

A design that fails these tests should normally be reconsidered before
implementation.

------------------------------------------------------------------------

## 18. Core Rules

The third-party extension philosophy can be reduced to the following
rules:

1.  **Providers plug into Lumen; Lumen does not reshape itself around
    providers.**
2.  **Integrate capabilities, not products.**
3.  **The architectural contract belongs to Lumen.**
4.  **Preserve one job first at every service boundary.**
5.  **Keep provider-specific translation at the edge.**
6.  **External implementations must remain replaceable wherever the
    capability permits it.**
7.  **Provider failure behaviour is determined by Lumen policy.**
8.  **Material external participation must produce traceable
    provenance.**
9.  **External evidence feeds specialist assessment; it does not
    redefine Aestimare.**
10. **Use real integrations to discover abstractions rather than
    designing speculative plugin frameworks.**
11. **Commercial interest can initiate investigation but cannot silently
    define architecture.**
12. **A successful integration must not become an accidental permanent
    dependency.**

------------------------------------------------------------------------

## 19. Summary

Third-party capabilities are valuable precisely because Lumen should not
need to implement every useful idea itself.

That benefit disappears if each integration causes Lumen to absorb the
external system's assumptions, terminology, lifecycle, or architecture.

The objective is therefore not simply to make Lumen extensible. It is to
make extensions **bounded**.

An external implementation may contribute tools, semantics, governance,
evidence, model access, or capabilities not yet imagined. Lumen provides
the stable contracts through which those capabilities participate.

The implementation can change.

The provider can change.

The capability can evolve.

The Lumen architecture remains Lumen.

> **Third-party functionality should extend Lumen without becoming part
> of Lumen's identity.**
