# Lumen --- Future Development Register

## Document Revision History

  -----------------------------------------------------------------------
  Version           Date              Author            Description
  ----------------- ----------------- ----------------- -----------------
  1.0               2026-08-23        Nigel Catterall   First reviewed
                                                        release

  1.1               2026-08-26        Nigel Catterall   Added Nuntius
                                                        control/event
                                                        consolidation and
                                                        Rogare polling
                                                        migration

  1.2               2026-08-26        Nigel Catterall   Added future
                                                        multi-session
                                                        Trace recording
                                                        capability

  1.3               2026-08-26        Nigel Catterall   Added Rogare control
                                                        error presentation
                                                        polish and optional
                                                        \obt Easter Eggs

  1.4               2026-08-27        Nigel Catterall   Added explicit legacy
                                                        Moderari/backchannel
                                                        compatibility retirement
                                                        to Nuntius consolidation

  1.5               2026-08-27        Nigel Catterall   Recorded M0.1 decision
                                                        to disable the Moderari
                                                        informational command
                                                        announcement and defer
                                                        its future purpose

  1.6               2026-08-27        Nigel Catterall   Deferred direct \obt
                                                        Custom policy selection
                                                        while preserving generic
                                                        policy status reporting

  1.7               2026-08-28        Nigel Catterall   Added Rogare dynamic
                                                        \obt command discovery
                                                        and command-picker UX

   1.8               2026-08-30        Nigel Catterall   Defined M0.1 provider/model
                                                        selection as runtime-global
                                                        and retained per-session
                                                        provider/model selection as
                                                        future development

   1.9               2026-09-04        Nigel Catterall   Recorded the disabled legacy
                                                        Moderari provider/model startup
                                                        validation and deferred the
                                                        removal decision
  -----------------------------------------------------------------------

**Status:** Living Document\
**Purpose:** Index of accepted future development that is intentionally
outside the current release scope

## Purpose

This document is a concise register of future Lumen development already
identified during architecture, research and release planning.

It is **not** a roadmap and should not duplicate detailed design
documentation.

Each entry should identify the relevant area, point to the document
where the requirement or design is described, and provide only enough
description to understand why the item exists.

When a future release is planned, this register can be reviewed to
decide which items should be promoted into that release's roadmap,
requirements and acceptance criteria.

------------------------------------------------------------------------

## Fiducia --- Experiment-Scoped Moderari Execution Policy

**Area:** Fiducia / Moderari / Repetere / Nuntius\
**Document:**
`Fiducia/FIDUCIA_EXPERIMENT_SCOPED_MODERARI_EXECUTION_POLICY.md`

Allow Fiducia to establish and positively acknowledge the required
Moderari execution policy once for an Experiment, rather than requiring
every individual Replay to re-establish the identical policy. The policy
must be explicitly Experiment-scoped and have a safe release/expiry
lifecycle.

## Runtime Authorization --- Release and Update Availability

**Area:** Servire / Illuminates.One Authorization / Distribution\
**Document:** M0.1 runtime authorization architecture (future extension)

Allow the periodic runtime-authorization/heartbeat exchange to report
that a newer Lumen release is available for the authorised installation.
The future mechanism should distinguish an update that is merely
available from one that is recommended or required, and may also
indicate when updated licence terms apply.

The exact update-notification protocol, version policy, acceptance flow,
upgrade mechanics, and enforcement behaviour are intentionally not
defined yet.

## Moderari --- Custom System Prompt Ownership and Scope

**Area:** Moderari / Saved System Prompts / Multi-User Configuration\
**Document:** M0.1 Development Requirements --- Moderari Saved System
Prompts / future extension

Extend saved Custom system prompts beyond the simple M0.1 ownership
model to define user-private, organisation-shared, installation-wide and
potentially organisation-mandated prompts. Future design must define
ownership, visibility and permissions for creating, updating, deleting,
sharing, mandating and overriding saved prompts without changing the
M0.1 principle that the applied prompt is an explicit Moderari runtime
condition.

## Repetere / Moderari --- System-Prompt Substitution Experiments

**Area:** Repetere / Moderari / Vestigare / Fiducia / Aestimare\
**Document:** Future experiment-variant design

Allow a researcher to derive an experimental execution from an existing
Trace while deliberately replacing the Trace's recorded effective system
prompt with a different explicitly Applied Custom system prompt. The
source Trace must remain immutable; the substitution is a declared
experimental override, and the resulting Trace must record the actual
substituted prompt used. This would allow comparison of behavioural
deviation caused by changing the system-prompt condition while retaining
the remaining recorded execution conditions as far as practicable.

## Repetere / Fiducia --- Controlled Experimental Variants

**Area:** Repetere / Fiducia / Vestigare / Aestimare / Moderari /
Praebere\
**Document:** Future controlled-experiment architecture

Extend Replay beyond exact reproduction so that a source Trace can
become the baseline for controlled experimental variants in which one or
more execution conditions are deliberately changed and explicitly
recorded. Potential variables include system prompt, model/provider,
tool availability or semantics, context treatment and bounded
computational resources. Ordinary Replay must remain semantically
distinct: **Replay reproduces; a Variant Experiment deliberately changes
declared conditions.** This should be revisited as Lumen develops from
replay and divergence observation toward a broader controlled
experimental framework for behavioural assessment.

## Multi-Session --- Independent Execution Conditions

**Area:** Moderari / Praebere / Session Management / Servire / Nuntius\
**Document:** M0.1 Development Requirements --- Multi-Session Isolation
Validation / future extension

Extend concurrent-session support so that individual active sessions may
independently select and retain their own provider and model without affecting
other sessions. Moderari system-prompt policy is already session scoped in M0.1
and therefore does not need to wait for this future provider/model work.

Future development must define how these execution conditions are
scoped, bound to session identity, changed safely during session
lifecycle, and released when a session ends. It should also define the
required configuration ownership, locking or isolation semantics across
Moderari, Praebere and the wider Lumen control path.

The M0.1 rule remains deliberately simpler for provider/model state:
Praebere maintains one runtime-global provider/model selection shared by all
active sessions. A model change therefore affects subsequent model requests
from every session. Moderari system-prompt policy is a separate execution
condition and is already session scoped in M0.1.

## Nuntius --- Unified Internal Control, Event and Status Messaging

**Area:** Nuntius / Pontis / Rogare / Servire / All Services\
**Document:** Future Nuntius control-plane consolidation

Consolidate Lumen's remaining bespoke backchannel, control, event and
status communication onto Nuntius. The normal model/data path should
remain separate, while internal operational chatter is carried through
the dedicated Nuntius control plane. This includes progressively
retiring historical Pontis backchannel mechanisms where Nuntius provides
the equivalent responsibility.

Rogare's current polling/backchannel traffic should also move to
Nuntius. Rather than repeatedly polling Pontis and contributing routine
status traffic to Servire Operational Logs, Rogare should receive or
retrieve the relevant control/status information through Nuntius. A
major objective is to keep Servire Operational Logs focused on
meaningful service operation and exceptional events rather than
high-frequency UI coordination chatter.

The future design should define event delivery, subscriptions or
retrieval semantics, ownership and correlation, reconnect/recovery
behaviour, and what minimal Nuntius activity---if any---should remain
visible in Servire Operational Logs. The migration should be incremental
and should not alter the ordinary ask/answer model path.

As part of this consolidation, retain the historical Moderari compatibility
handling defensively while supported callers may still depend upon it, but treat
Nuntius as the authoritative path for explicit external `\obt` control traffic.
Once remaining historical backchannel/status traffic has migrated to Nuntius,
remove the obsolete Moderari conversational/backchannel interception and
compatibility implementation as part of the same cleanup rather than as an
isolated change.

Before retirement, validate that retained compatibility handling cannot cause
duplicate command execution and cannot leak historical `[Lumen Command]` or
other operational/backchannel traffic into external-client conversational
output, model context, or conversational Trace. The compatibility path is not an
M0.1 usability blocker provided these safety properties hold.

## Moderari --- Informational Command Announcement

**Area:** Moderari / External Clients / Nuntius / Control UX\
**Document:** N7+ --- Moderari System-Prompt Policy / future Moderari control UX

The historical Moderari informational announcement that identifies the Moderari
version and explains that commands beginning with `\obt` are handled by Lumen is
deliberately **disabled for M0.1**.

Testing during N7+ showed inconsistent client-visible presentation: the announcement
could appear in Pi after a clean Moderari start while Rogare did not present it, and
later sessions did not necessarily receive it. The important control-boundary
validation found no `\obt` command leakage into ordinary model execution. Continuing
to investigate the presentation behaviour is therefore not justified for M0.1.

For M0.1 the preferred behaviour is deterministic absence: external clients should
not receive this unsolicited informational announcement. The underlying implementation
should be disabled rather than treated as a required M0.1 feature.

Future development may reconsider whether an equivalent message has a useful,
well-defined purpose, such as explicit client capability discovery, session metadata,
or control-path guidance. Any reintroduction must define its lifecycle and scope
clearly and must preserve the architectural rule that operational/control information
cannot become model prompt or conversational context.

This item is therefore **closed for M0.1 by deliberate disablement**, with any
redesign or reintroduction deferred to future development.

## Moderari --- Legacy Provider and Model Startup Validation Removal

**Area:** Moderari / Praebere / Provider Readiness / Startup Lifecycle\
**Document:** Praebere N9.6 Runtime and Readiness Reconciliation Mini-Roadmap

Moderari historically checked Ollama availability, enumerated the configured model
and optionally sent a live chat-completion probe during startup. This originated in
the earlier OBT architecture, when Moderari was responsible for validating and warming
its configured model. The live probe could load that model with long-lived residency
before any Lumen session performed an ask.

N9.6 assigns provider availability, model discovery and model residency to Praebere.
Moderari's provider/model startup checks are therefore deliberately disabled for M0.1.
The implementation remains present behind the disabled
`startup_validation.provider_model_checks_enabled` compatibility flag, while Moderari
continues to validate only its configuration, filesystem and MongoDB dependencies.

A future development decision must determine whether to remove the retained provider
and model validation code, its configuration fields and related tests completely, or
whether a provider-neutral diagnostic use remains valid outside the normal startup
lifecycle. Any retained diagnostic must be explicitly invoked, must not establish
provider authority, and must not load, warm or alter model residency implicitly.

The earlier orphaned `llama-server` observations may have been contributed to by this
legacy live startup probe and its long-lived model residency. That historical cause no
longer affects the M0.1 architecture because Ollama is externally operated and Moderari
no longer probes it during startup.

## Moderari --- Direct `\\obt` Selection of Custom System-Prompt Policy

**Area:** Moderari / Nuntius / System-Prompt Policy / Control UX\
**Document:** Future Moderari system-prompt control design

M0.1 provides direct `\\obt` switching between the existing **Default** system-prompt
policy and **Pass-through**, together with `\\obt moderari prompt status` to report the
currently active policy mode.

Direct command selection of **Custom** is intentionally deferred. A future command may
provide an equivalent capability such as:

```text
\\obt moderari prompt custom
```

but it should only be introduced once Custom prompt selection, saved-prompt identity,
working-copy state and application semantics are sufficiently defined that a command
cannot select an ambiguous or unintended prompt.

The policy-status contract should not need to change when this capability is added.
`\\obt moderari prompt status` should always report only the active **mode** ---
`default`, `pass-through`, or `custom` --- and should not expose which saved Custom
prompt is selected or reveal the Custom prompt's contents.

## Vestigare --- Concurrent Session-Bound Trace Recordings

**Area:** Vestigare / Pontis / Rogare / Repetere / Session Management /
Nuntius\
**Document:** Future multi-session Trace architecture

Extend Vestigare beyond the M0.1 single-active-recording model so that
multiple independent Lumen sessions may be traced concurrently. Each
active Trace must remain bound to exactly one Lumen session and must
contain only the model-path traffic belonging to that session;
concurrent traffic from another session must never be merged into the
Trace.

The M0.1 correctness rule should remain the foundation: a Trace
represents one model conversation/session suitable for later Replay.
M0.1 may therefore deliberately permit only one active recording at a
time, with explicit recording ownership and session binding, rather than
allowing a global recording window to combine otherwise isolated
conversations.

Future multi-session tracing should allow separate simultaneous
recordings, for example Trace A for session A and Trace B for session B,
with independent recording identity, owner, start/stop lifecycle and
status. Rogare, Vestigare and any later control surface should expose
only the controls permitted for the recording it owns, while Nuntius can
carry authoritative recording-state and ownership changes between
services.

The future design should define concurrent recorder storage and locking,
session-to-recording mapping, ownership and permissions, lifecycle
recovery after client/service failure, UI representation of multiple
active recordings, and how Repetere selects and replays each resulting
session-bound Trace independently.


## Rogare / Nuntius --- Dynamic `\obt` Command Discovery and Picker

**Area:** Rogare / Nuntius / `\obt` Services / Control UX\
**Document:** Future Rogare control-command discovery design

Add a lightweight `\obt` command picker to Rogare so researchers and
operators do not need to remember the complete command syntax exposed by
every Lumen service. Selecting an entry should insert the corresponding
command into Rogare's chat composer for review or completion; it must
**not execute the command automatically**.

The command list must be discovered dynamically rather than duplicated
as a hard-coded Rogare or Nuntius catalogue. Nuntius already knows the
active `\obt`-capable services and enough routing information to address
them, while each service remains authoritative for the commands it owns.
Discovery should therefore use the service control path: Nuntius requests
the authoritative help/command catalogue from each relevant active
service, analogous to the existing `\obt moderari help` behaviour,
aggregates those responses, and returns the available command set to
Rogare.

The future discovery response should be machine-readable and should
support at least the service, command/template and a concise description.
Parameterized commands may be presented as editable templates such as
`\obt session resume <number|session_id>` or
`\obt model select <model>`. Rogare may group or filter discovered
commands by service and availability, but it must not become a second
source of truth for command ownership or syntax.

This remains a usability layer over the existing control architecture:
Rogare inserts ordinary `\obt` command text into the composer and the
user explicitly sends it through the normal Pontis/Nuntius path. Command
discovery must not create a parallel execution mechanism or cause control
traffic to enter the model conversation.

## Rogare --- Friendly Control-Path Error Presentation

**Area:** Rogare / Nuntius / Control Responses\
**Document:** Future Rogare control-response presentation polish

Improve Rogare's presentation of structured non-success responses returned
through the Nuntius control path. Where a service returns a useful human-readable
`message` together with structured error data, Rogare should present the message
cleanly to the user rather than exposing the complete HTTP status and serialized
response body as the primary conversational output.

The underlying HTTP status, structured error code, request correlation and
diagnostic data should remain available for debugging and observability. This is
presentation polish only; it must not weaken the explicit control-path error
semantics or cause failed `\obt` commands to enter model execution.

## Nuntius / Services --- `\obt` Easter Eggs

**Area:** Nuntius / Rogare / Pontis / Moderari / Repetere / Other `\obt` Services\
**Document:** Future control-plane interaction polish

Consider adding a very small set of deliberate Easter Eggs to the explicit
`\obt <service> <text>` command space. Engineers and researchers are likely to
probe service-addressed command combinations naturally while exploring Lumen;
an intentionally playful response to selected otherwise-invalid combinations
could acknowledge that behaviour without exposing additional control
capabilities.

Easter Eggs must remain subordinate to the control architecture. They must not
masquerade as real commands, reveal hidden or privileged controls, weaken
authorization, make command discovery ambiguous, or ever fall through into the
model path. Unknown commands should continue to fail normally unless they match
an explicit Easter Egg owned by the addressed service.

This is optional future polish, not an M0.1 requirement. If implemented, the
responses should be sparse enough that discovering one feels intentional rather
than turning the `\obt` namespace into a novelty interface.

## Rogare --- Conversation Copy and Export

**Area:** Rogare / Research UX / Conversation Records\
**Document:** Future Rogare conversation export design

Add explicit copy/export functionality to Rogare's Conversation window so that a
researcher or engineer can capture the complete human-readable conversation without
manually selecting individual messages. The initial useful capability should include
**Copy Conversation** and **Export as Markdown**. A structured **Export as JSON** may
also preserve roles, session identity, timestamps and other appropriate conversational
metadata for later analysis.

This export represents the conversation as presented through Rogare and must remain
semantically distinct from a Vestigare Trace. Rogare exports the human-readable
interaction record; Vestigare preserves execution evidence suitable for inspection,
Replay and behavioural analysis. Exporting a Rogare conversation must therefore not be
treated as exporting or reconstructing the authoritative Trace.

This is future research/usability functionality rather than an M0.1 requirement. It
should only be promoted into M0.1 if manual conversation extraction becomes a material
impediment during external research testing.
