# Pontis — Multiple Tool Providers and Tool Catalogue

**Status:** Future Development  
**Target:** Post-M0.1

## Context

The current Pontis implementation effectively operates with a single tool provider.

This is an acceptable limitation for the initial external research distribution. However, it should not become an architectural assumption.

External research use may involve multiple independent tool providers associated with the same Lumen session.

This requirement has also arisen from consideration of external tools such as Glyphs, which may provide capabilities independently of the primary client tool provider.

## Architectural Principle

Pontis connects Lumen sessions to **tool capabilities**, not to a singular tool provider.

The eventual architecture may therefore resemble:

```text
                    +-- Tool Provider A
                    |
Client -> Pontis ---+-- Tool Provider B
                    |
                    +-- Tool Provider C
                    |
                    +-- External Tool / Glyph
```

Pontis should be capable of discovering the capabilities exposed by these providers and presenting an appropriate unified tool catalogue to the Lumen environment.

## Tool Catalogue

A future Pontis tool catalogue could associate each available tool with its owning provider.

Conceptually:

```text
Provider A
    tool_a
    tool_b

Provider B
    tool_c
    tool_d

External Glyph
    tool_e
```

The model or consuming Lumen component should not necessarily need to understand the underlying provider topology.

Pontis should be capable of resolving a selected tool to the appropriate provider and routing the invocation accordingly.

## Questions for Future Investigation

Multiple providers introduce several architectural questions which are deliberately left unresolved at this stage:

- provider discovery and registration;
- dynamic provider availability;
- provider lifecycle;
- duplicate tool names;
- tool namespaces;
- capability collisions;
- catalogue refresh;
- provider failure;
- tool provenance;
- session-specific provider availability;
- routing and invocation ownership.

These should be investigated before implementation rather than prematurely encoded into the current Pontis architecture.

## M0.1 Position

M0.1 may support a single tool provider.

This should be documented as an **implementation limitation**, not an architectural definition of Pontis.

The architectural position remains:

> **Pontis connects Lumen sessions to tool capabilities and may ultimately aggregate capabilities from multiple independent tool providers.**