# Lumen Nuntius --- Cross-Service Control-Plane Migration Project

**Date:** 2026-08-25\
**Status:** Development Project Specification\
**Scope:** Pontis, Repetere, Moderari, Servire, Nuntius, with later
Praebere integration\
**Purpose:** Complete the Nuntius control plane as a discrete
cross-service project before returning to the main M0.1 roadmap.

------------------------------------------------------------------------

## Document Revision History

  -----------------------------------------------------------------------
  Date              By                Version           Description
  ----------------- ----------------- ----------------- -----------------
  2026-08-25        Nigel Catterall   1.0               Initial versioned
                                                        baseline
                                                        incorporating the
                                                        agreed Nuntius
                                                        common control
                                                        contract and
                                                        revised
                                                        cross-service
                                                        migration
                                                        roadmap.

  2026-08-30        Nigel Catterall   1.1               Reconciled completed
                                                        N1--N7 implementation
                                                        and validation;
                                                        recorded defensive
                                                        compatibility decision
                                                        and N8 as the next
                                                        active phase.

  -----------------------------------------------------------------------

> **Versioning rule:** Multiple edits made on the same calendar day
> remain within the same document version. Increment the document
> version only when a revision is made on a later date than the most
> recent revision entry.

------------------------------------------------------------------------

## 1. Project Decision

Nuntius should not be developed in isolation and then integrated
opportunistically as unrelated work reaches each affected service.

The current `\obt` implementation is distributed across Pontis, Repetere
and Moderari. Completing Nuntius therefore requires a coordinated
migration across those services, together with the Servire topology
changes that make Nuntius authoritative for command routing.

This work will be treated as a **specific cross-service project**.

The project should complete the common `\obt` control plane to a stable
baseline, after which development returns to the main External Research
Distribution M0.1 roadmap.

This differs from the normal Lumen development preference of
incorporating related changes when another service is already being
modified. In this case, completing the cross-service control plane first
avoids leaving Nuntius partially implemented while multiple old and new
command-routing mechanisms coexist.

------------------------------------------------------------------------

## 2. Confirmed Architectural Decision --- `\obt rogare start`

`\obt rogare start` remains **Pontis-owned**.

It must not be routed:

``` text
Rogare -> Pontis -> Nuntius -> Pontis
```

because Pontis is already the service whose own state and
provider/session bindings are being established by that command.

The intended rule is therefore:

> **Pontis consumes commands that Pontis itself owns. Other
> client-originated `\obt` commands are intercepted by Pontis and routed
> to Nuntius.**

Nuntius is the common router for inter-service control commands. It is
not a mandatory loop through which a service must route a command back
to itself.

------------------------------------------------------------------------

# 3. Current `\obt` Implementation --- Investigation Findings

The current implementation predates Nuntius and uses the provider/chat
request path as a lightweight distributed command transport.

Conceptually:

``` text
Client / UI
    |
    v
Pontis
    |
    +--> consumes Pontis-owned command
    |
    v
Repetere
    |
    +--> consumes Repetere-owned command
    |
    +--> forwards commands it does not own
    |
    v
Moderari
    |
    +--> consumes Moderari-owned command
    |
    v
Model
```

The common behaviour is essentially:

> **Inspect the terminal user message. If it contains an `\obt` command
> owned by this service, consume it locally. Otherwise allow it to
> continue through the stack.**

This implementation already contains several useful invariants and
behaviours that should be preserved during migration.

------------------------------------------------------------------------

## 3.1 Moderari

Moderari contains the earliest examined implementation of the `\obt`
convention.

### Current behaviour

Moderari:

-   examines the terminal user message for `\obt`;
-   does not search historical conversation turns for a command to
    execute;
-   executes recognised commands locally;
-   returns a synthetic assistant-style response;
-   does not invoke the model for a consumed command;
-   filters historical `\obt` command exchanges from later model-bound
    context;
-   has access to the current Lumen session when executing its local
    command;
-   does not expose a generic cross-service control endpoint;
-   does not currently use a generic `request_id`, `origin`,
    `session_id`, command envelope or response contract.

The filtering of historical commands is particularly important. Existing
command traffic may have travelled inside the conversational transport,
but Moderari deliberately prevents that operational traffic from
becoming later model context.

### Architectural conclusion

Moderari already demonstrates the required principle:

> **Control activity must remain outside effective model conversation
> context.**

However, the current local interception mechanism is not itself the
final Nuntius control protocol.

### New M0.1 functionality

The proposed session-scoped Moderari system-prompt policy override used
by Replay is not an existing `\obt` capability.

The future Replay prerequisite:

``` text
Repetere -> Nuntius -> Moderari
```

to establish session-scoped `Pass-through` with positive acknowledgement
is therefore new functionality.

------------------------------------------------------------------------

## 3.2 Repetere

Repetere uses the same general pattern but makes command ownership more
explicit.

### Current behaviour

Repetere:

-   inspects the terminal provider-style request for `\obt replay ...`;
-   consumes Replay-owned commands locally;
-   forwards `\obt` commands it does not own;
-   prevents locally handled Replay commands from entering normal model
    execution;
-   returns synthetic assistant-style command results;
-   records command-decision evidence;
-   currently uses the live Lumen stack itself as the ingress path for
    Replay UI commands.

The current Replay UI can construct:

``` text
\obt replay start <replay-id>
```

as a provider-style user request and submit it to the top of the Lumen
stack.

The command then travels through the stack until Repetere recognises and
consumes it.

### Architectural conclusion

Repetere already contains a useful explicit concept:

> **owned command vs command not owned by this service**

That distinction should be retained.

What should disappear is the need to send an operational command through
the conversational/provider request path merely to reach its
authoritative owner.

### New M0.1 functionality

Repetere does not currently establish Moderari `Pass-through` through a
shared control plane before beginning Replay.

The following will be new orchestration:

``` text
create isolated Replay session
        |
        v
Repetere
        |
        | session-scoped Pass-through request
        v
Nuntius
        |
        v
Moderari
        |
        | 200 positive acknowledgement
        v
Nuntius
        |
        v
Repetere
        |
        v
begin Replay
```

Replay must not begin if the prerequisite cannot be positively
confirmed.

------------------------------------------------------------------------

## 3.3 Pontis

Pontis is already the closest service to the intended external
control-plane boundary.

### Current behaviour

Pontis already intercepts a specific command:

``` text
\obt rogare start
```

before normal downstream processing.

That command is Pontis-owned because it establishes
Pontis/provider/session/tool-routing state.

Pontis also already owns and propagates stable Lumen session identity.
Existing mechanisms include Lumen session headers and session
information in request payloads.

Pontis therefore already provides the correct architectural home for:

-   identifying external client/session context;
-   intercepting client-originated control traffic;
-   returning responses to the correct originating session.

### Architectural conclusion

Pontis should become the general external `\obt` interception point,
subject to local ownership.

Conceptually:

``` text
Client / Rogare
      |
      v
    Pontis
      |
      +--> Pontis-owned `\obt`
      |       -> execute locally
      |
      +--> other `\obt`
      |       -> Nuntius
      |
      +--> ordinary model request
              -> normal Lumen execution path
```

Pontis should not acquire knowledge of which non-Pontis service owns
individual commands.

That responsibility belongs to Servire configuration and Nuntius
routing.

------------------------------------------------------------------------

# 4. What the Investigation Changes in the Nuntius Architecture

The original Nuntius architecture remains substantially correct, but the
investigation allows several parts to be made more precise.

------------------------------------------------------------------------

## 4.1 Nuntius Is a Formalisation of Existing Behaviour

Nuntius is not introducing a new control-command concept.

It formalises an existing distributed convention that currently uses the
chat/provider transport.

The migration is therefore:

``` text
CURRENT

provider-style request
    |
    v
consume-or-forward through services
```

to:

``` text
TARGET

explicit control request
    |
    v
Pontis / internal originator
    |
    v
Nuntius
    |
    v
Servire-configured authoritative owner
```

The command language remains `\obt`.

The transport and routing mechanism change.

------------------------------------------------------------------------

## 4.2 Local Command Ownership Remains Valid

Nuntius must not become a rule that every `\obt` command always leaves
the service that received it.

A service may consume a command locally when the command belongs
intrinsically to that service and routing it through Nuntius would
create a pointless loop.

The confirmed example is:

``` text
\obt rogare start
```

which remains Pontis-owned.

The normal rule is therefore:

> **A service may execute commands it owns locally at its own control
> boundary. Commands requiring another service are routed through
> Nuntius.**

For external client traffic, Pontis is responsible for deciding only:

1.  is this a Pontis-owned command?
2.  if not, is this `\obt` traffic?

Pontis must not resolve the owner of non-Pontis commands.

------------------------------------------------------------------------

## 4.3 Existing Session Identity Should Be Reused

The Nuntius protocol should not create a second Lumen session identity
mechanism.

Pontis already extracts and propagates stable originating-session
identity.

That existing session identity should be carried into Nuntius control
requests.

Nuntius still requires command-level request correlation.

A minimal conceptual envelope is:

``` text
request_id
timestamp
session_id
origin
command
```

with a UTC `timestamp`, while routing/listener metadata is derived from
Servire's configured catalogue.

------------------------------------------------------------------------

## 4.4 Synthetic Assistant Responses Are Compatibility Behaviour, Not the Internal Protocol

Moderari and Repetere currently return synthetic assistant messages
because commands travel through provider-style interfaces.

The common Nuntius service contract should instead use explicit control
responses:

``` text
200, no body
    successful execution

200 + body
    successful query/request

204
    command not handled

4xx / 5xx
    recognised command failure

504
    outcome not confirmed before timeout
```

Where an existing client interface still expects provider-compatible
output, Pontis or the relevant UI adapter may translate the
authoritative control response at the boundary.

Nuntius itself should not model control results as assistant
conversation.

------------------------------------------------------------------------

## 4.5 Existing Content Filtering Remains Defensive Protection

After migration, normal client-originated `\obt` traffic should no
longer enter the model execution route.

Nevertheless, existing defensive filtering in Moderari/Repetere should
not be removed immediately merely because the new route exists.

It remains useful for:

-   compatibility during migration;
-   malformed or legacy requests;
-   regression protection;
-   preventing accidentally retained historical control messages from
    entering model context.

The compatibility/filtering path is retained for defensive purposes in M0.1. Its eventual retirement is future development rather than an M0.1 completion requirement.

------------------------------------------------------------------------

# 5. Required Remediation by Service

## 5.1 Servire

Servire becomes authoritative for the active control-plane topology.

Required changes:

-   [x] add `obt_enabled` to relevant service definitions;
-   [x] define each service's `/control/obt` endpoint;
-   [x] define service-level `listeners[]` for successful reserved `select`
    state updates;
-   [x] expose the configured `\obt services` catalogue;
-   [x] support a successful empty catalogue;
-   [x] validate service identities, endpoints and listener references;
-   [x] treat presence in Servire configuration/catalogue as membership of
    the configured topology --- no separate `active` or `is_active` flag
    is required.

Servire does not route individual commands.

Servire defines the topology that Nuntius materialises and uses.

------------------------------------------------------------------------

## 5.2 Nuntius

Nuntius M1 scaffold is complete.

The remaining Nuntius implementation should include:

-   [x] Servire bootstrap;
-   [x] configured catalogue retrieval;
-   [x] in-memory routing dictionary;
-   [x] catalogue refresh/update;
-   [x] common control request model;
-   [x] request correlation;
-   [x] routing by the service token immediately following `\obt`;
-   [x] recognition of the reserved `select` operation;
-   [x] targeted service routing;
-   [x] terminal response handling;
-   [x] configurable timeout;
-   [x] successful `select` response-body distribution to configured
    listeners;
-   [ ] diagnostics;
-   [ ] minimal diagnostics UI;
-   [ ] health state including Servire connectivity;
-   [x] tests proving control traffic remains outside conversational Trace.

Nuntius must remain domain-agnostic. It understands only the destination
service token after `\obt` and the reserved `select` operation required
for listener distribution.

It must not understand Replay semantics, model semantics, prompt
semantics or provider semantics.

------------------------------------------------------------------------
## 5.3 Pontis

Pontis requires the primary external migration.

Required changes:

-   [x] retain `\obt rogare start` as Pontis-owned;
-   [x] generalise terminal-user `\obt` detection;
-   [x] intercept non-Pontis-owned client `\obt` before normal model
    routing;
-   [x] create a command-level `request_id`;
-   [x] attach the already established originating `session_id`;
-   [x] identify the command origin;
-   [x] submit the control request to Nuntius;
-   [x] receive the authoritative terminal outcome;
-   [x] return that outcome to the correct originating external
    session/client;
-   [x] keep live command traffic outside Repetere/Moderari/model execution;
-   [x] before ordinary model routing, remove historical synchronous control
    pairs consisting of a `user` message beginning with `\obt` and its
    immediately following `assistant` response;
-   [x] remove an unpaired historical `\obt` user message even if no
    assistant response follows;
-   [x] preserve existing ordinary model routing unchanged;
-   [x] leave existing downstream Repetere/Moderari filtering in place as
    defence in depth.

Pontis must not acquire a hard-coded map of Repetere, Moderari, Praebere
or other command ownership.

------------------------------------------------------------------------

## 5.4 Repetere

Required changes:

-   [x] expose a standard `\obt` control endpoint;
-   [x] retain Replay command parser/handler semantics where applicable;
-   [x] accept Replay-owned commands through the explicit control path;
-   [x] stop relying on provider/chat traversal as the primary way Replay UI
    commands reach Repetere;
-   [x] preserve existing command-decision evidence where useful;
-   [x] adapt diagnostics so transport lifecycle belongs to Nuntius while
    Replay-domain execution evidence remains with Repetere;
-   [x] maintain compatibility behaviour during transition;
-   [x] later implement session-scoped Moderari `Pass-through` orchestration
    through Nuntius;
-   [x] require positive acknowledgement before Replay model execution.

Replay-specific system-prompt fidelity remains later work on the main
M0.1 roadmap unless needed to validate the control path itself.

------------------------------------------------------------------------

## 5.5 Moderari

Required changes:

-   [x] expose a standard `\obt` control endpoint;
-   [x] adapt existing Moderari-owned command handlers to the common control
    contract;
-   [x] preserve terminal-message-only and command-history filtering as
    defensive behaviour during migration;
-   [x] remove dependence on synthetic assistant responses from the internal
    control contract;
-   [x] retain compatibility handling as defence in depth; mark eventual retirement for future development;
-   [x] add the session-scoped `Pass-through` control command required later
    by Replay;
-   [x] return authoritative positive acknowledgement when the
    session-scoped policy has been applied;
-   [x] ensure the control command itself never becomes model conversational
    context.

------------------------------------------------------------------------

## 5.6 Praebere

Praebere is not part of the historical `\obt` migration investigation
but is an early consumer of the completed control plane.

After the common routing path is stable, Praebere should implement its
M0.1 commands through the same endpoint and response contract,
including:

``` text
\obt praebere providers
\obt praebere models
\obt praebere select model <model>
```

This provides the first clean example of a service adopting Nuntius
without carrying the old consume-or-forward chat-transport heritage.

------------------------------------------------------------------------

# 6. Cross-Service Mini-Roadmap

The Nuntius project should be completed as a bounded development stream
before returning to the main M0.1 roadmap.

## N1 --- Existing Behaviour Investigation

**Status:** COMPLETE.

Deliverables:

-   [x] current command detection understood;
-   [x] local ownership behaviour understood;
-   [x] consume/forward convention understood;
-   [x] existing session identity mechanism identified;
-   [x] current response style identified;
-   [x] model/Trace exclusion mechanisms identified;
-   [x] `\obt rogare start` ownership decision confirmed.

**Exit condition:** sufficient understanding exists to migrate without
creating a competing mechanism.

------------------------------------------------------------------------

## N2 --- Common Control Contract

**Status:** COMPLETE — agreed contract implemented across the migrated control path.

The agreed contract defines:

-   [x] `POST /control/obt` as the common control endpoint;
-   [x] request envelope: `request_id`, UTC `timestamp`, `session_id`,
    `origin`, and complete unchanged `command`;
-   [x] routing by the service token immediately following `\obt`;
-   [x] `select` as a reserved Nuntius control-language operation;
-   [x] all domain content beyond the routing/service token and reserved
    `select` semantics remains owned by the destination service;
-   [x] Servire as topology authority without a command-ownership catalogue;
-   [x] service-level `listeners[]` used only for successful reserved
    `select` state updates;
-   [x] ordinary query/request responses return only to the originator;
-   [x] one structured response representation with human-readable `message`
    and optional machine-readable `data`;
-   [x] Pontis translation of control results to provider-compatible
    assistant responses for external clients;
-   [x] Pontis removal of historical synchronous `\obt` user/assistant
    control pairs before model execution;
-   [x] existing Repetere/Moderari filtering retained as defence in depth;
-   [x] `\obt` control exchanges excluded from Vestigare conversational
    Trace.

**Exit condition:** Servire, Nuntius, Pontis, Repetere, Moderari and
Praebere can implement against one agreed contract.

------------------------------------------------------------------------

## N3 --- Servire Control-Plane Catalogue

**Status:** COMPLETE.

Implement the authoritative configured topology.

Work:

-   [x] `obt_enabled`;
-   [x] `/control/obt` endpoint metadata;
-   [x] service-level `listeners[]`;
-   [x] `\obt services` catalogue;
-   [x] empty healthy catalogue;
-   [x] validation of service identities, endpoints and listener references;
-   [x] presence in Servire configuration/catalogue defines membership of
    the configured topology;
-   [x] no separate `active` or `is_active` field;
-   [x] no command ownership metadata or command-specific response-target
    catalogue.

**Exit condition:** Servire can authoritatively describe every
configured Nuntius-addressable service, its control endpoint, whether
`\obt` handling is enabled, and which services receive successful
reserved `select` state updates.

------------------------------------------------------------------------

## N4 --- Nuntius Routing Core

**Status:** COMPLETE.

Build the actual routing service against the N2/N3 contracts.

Work:

-   [x] bootstrap from Servire;
-   [x] in-memory registry;
-   [x] routing by service token;
-   [x] reserved `select` recognition;
-   [x] targeted dispatch;
-   [x] request correlation;
-   [x] terminal outcome handling;
-   [x] timeout;
-   [x] catalogue update/rebuild;
-   [x] base diagnostics.

**Exit condition:** an internal test originator can issue a command and
receive exactly one authoritative terminal outcome without using the
model request path.

------------------------------------------------------------------------

## N5 --- Pontis External Interception

**Status:** COMPLETE.

Move the external client boundary onto Nuntius.

Work:

-   [x] preserve Pontis-owned `\obt rogare start`;
-   [x] intercept all other client `\obt`;
-   [x] attach stable session identity;
-   [x] generate request correlation;
-   [x] route to Nuntius;
-   [x] return authoritative response to correct external session;
-   [x] prove ordinary requests remain unaffected.

**Exit condition:** external `\obt` commands no longer need to travel
through the model execution stack to reach non-Pontis owners.

------------------------------------------------------------------------

## N6 --- Repetere Migration

**Status:** COMPLETE for the control-plane migration; N6+ Replay-fidelity work is tracked separately.

Move Replay-owned commands onto the explicit control path.

Work:

-   [x] standard control endpoint;
-   [x] existing Replay command handlers behind that endpoint;
-   [x] Servire/Nuntius service registration;
-   [x] migrate Replay UI/operator command path;
-   [x] retain defensive compatibility handling; eventual retirement is future work;
-   [x] preserve Replay-domain logging.

**Exit condition:** Replay commands are routed Pontis -\> Nuntius -\>
Repetere and no longer depend on conversational traversal.

------------------------------------------------------------------------

## N7 --- Moderari Migration

**Status:** COMPLETE, including the subsequent N7+ session-scoped system-prompt work.

Move Moderari-owned commands onto the explicit control path.

Work:

-   [x] standard control endpoint;
-   [x] existing handlers adapted to common responses;
-   [x] Servire/Nuntius service registration;
-   [x] defensive compatibility behaviour retained; eventual retirement is future work;
-   [x] validate historical control traffic cannot leak into model context.

**Exit condition:** Moderari-owned external commands arrive through
Nuntius rather than through ordinary model execution.

------------------------------------------------------------------------

## N8 --- Nuntius Diagnostics and UI

**Status:** NEXT / ACTIVE DEVELOPMENT PHASE.

Complete the Nuntius operational surface.

Work:

-   [ ] request lifecycle records;
-   [ ] origin/session/request correlation;
-   [ ] resolved owner;
-   [ ] terminal result;
-   [ ] elapsed time;
-   [ ] timeout/unconfirmed outcome;
-   [ ] late response;
-   [ ] routing/configuration errors;
-   [ ] health;
-   [ ] Servire connectivity;
-   [ ] active route count;
-   [ ] in-flight count;
-   [ ] bounded recent history;
-   [ ] Servire-exposed Nuntius diagnostics view.

**Exit condition:** command-routing faults can be diagnosed without
adding detailed Nuntius traffic to the normal Servire Operations Log or
Vestigare conversational Trace.

------------------------------------------------------------------------

## N9 --- Praebere First Native Adoption

**Status:** SUBSTANTIALLY COMPLETE — native control adoption and N9.5 are validated;
N9.6.3 bug fixes/final lifecycle validation and compact Praebere UI closeout remain.

Praebere now uses the completed control plane for provider/model discovery and
selection. This validates that a service without legacy conversational `\obt`
transport can use the architecture cleanly.

The full provider-neutral N9.6.4 readiness taxonomy has been moved to
nice-to-have/post-M0.1 and is not required to prove native adoption.

**Exit condition:** provider/model query and selection commands use the same Nuntius
path and response contract; remaining N9 closeout is tracked in the Praebere N9
addendum and lifecycle documents.

------------------------------------------------------------------------

## N10 --- Compatibility Removal and Regression Validation

After all migrated paths are proven:

-   [ ] identify legacy consume-or-forward traversal code that is no longer
    required;
-   [ ] retain defensive `\obt` model-context filtering in
    Repetere/Moderari;
-   [ ] remove obsolete command traversal paths;
-   [ ] run cross-service regressions;
-   [ ] verify `\obt rogare start` remains Pontis-local;
-   [ ] verify no accidental command loops;
-   [ ] verify no command enters model context;
-   [ ] verify session/request correlation;
-   [ ] verify success is never inferred from silence;
-   [ ] verify normal ask/answer paths are unchanged.

**Exit condition:** one coherent control-plane architecture remains.

------------------------------------------------------------------------

# 7. Project Completion Definition

The Nuntius cross-service project is complete when:

- [x] 1. Servire authoritatively defines the configured `\obt` routing
    topology.
- [x] 2. Nuntius obtains and maintains that topology.
- [x] 3. Pontis consumes Pontis-owned commands locally.
- [x] 4. `\obt rogare start` remains Pontis-owned.
- [x] 5. Other client-originated `\obt` commands are diverted to Nuntius
    before model execution.
- [x] 6. Nuntius routes each command using the service token immediately
    following `\obt` and recognises reserved `select` semantics.
- [x] 7. Repetere and Moderari expose the common control endpoint.
- [x] 8. Authoritative responses return to the correct originator.
- [x] 9. Existing Lumen session identity is preserved.
- [x] 10. Each solicited command has one terminal outcome.
- [x] 11. Pontis strips historical synchronous `\obt` user/assistant control
    pairs before normal model execution, and control traffic remains
    outside Vestigare conversational Trace.
- [x] 12. Detailed Nuntius transport activity remains outside the default
    Servire Operations Log.
- [ ] 13. Nuntius diagnostics provide enough evidence to diagnose routing
    failures and timeouts.
- [x] 14. Praebere can use the same completed mechanism for provider/model
    operations.
- [ ] 15. Legacy conversational command traversal is no longer required as the
    normal route.
- [x] 16. Existing ordinary ask/answer execution remains regression-clean.

At that point, Nuntius is considered a completed foundational Lumen
service for M0.1 purposes.

------------------------------------------------------------------------
# 8. Relationship to the Main M0.1 Roadmap

The common control plane is operational. **N1 through N7 are complete; N9 native Praebere adoption is substantially complete. N8 diagnostics/UI and N10 compatibility removal/regression validation remain open project closeout work.**

The sequence is:

``` text
Nuntius M1 scaffold
        |
        v
existing `\obt` investigation
        |
        v
Nuntius cross-service migration project
        |
        v
Nuntius control plane complete
        |
        v
return to main M0.1 roadmap
```

Later roadmap work can then assume that a stable common `\obt` transport
exists rather than repeatedly modifying command infrastructure while
implementing Replay fidelity, system-prompt policy, provider/model
control and other M0.1 requirements.

This reduces architectural churn and gives subsequent M0.1 work one
stable control-plane dependency.

------------------------------------------------------------------------

# 9. Explicit Non-Goals of This Project

This project does not itself attempt to complete:

-   Replay Experiment structure;
-   full Replay effective-system-prompt fidelity *(subsequently advanced substantially under N6+/N7+; remaining acceptance is tracked in the Repetere addendum)*;
-   Moderari saved Custom prompts *(subsequently completed under N7+)*;
-   the full Moderari system-prompt policy UI *(subsequently completed under N7+)*;
-   Fiducia repeated-experiment orchestration;
-   Aestimare;
-   durable messaging;
-   RabbitMQ or Kafka;
-   distributed consensus;
-   multi-host runtime;
-   operational-log user/internal classification;
-   general service lifecycle control through Nuntius.

Those remain governed by the main M0.1 roadmap.

The exception is where a minimal implementation is required to prove the
common control path itself.

------------------------------------------------------------------------

## Core Project Principle

> **The old `\obt` mechanism already established the command language
> and consume-or-forward convention. The Nuntius project extracts that
> behaviour from the conversational transport, formalises service
> routing, reserved `select` distribution and response semantics,
> preserves existing session identity, and gives Lumen one explicit
> control plane before the remainder of M0.1 development continues.**
