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

Extend concurrent-session support beyond the M0.1
shared-execution-condition model so that individual active sessions may
independently select and retain their own provider, model and Moderari
system-prompt condition without affecting other sessions.

Future development must define how these execution conditions are
scoped, bound to session identity, changed safely during session
lifecycle, and released when a session ends. It should also define the
required configuration ownership, locking or isolation semantics across
Moderari, Praebere and the wider Lumen control path.

The M0.1 rule remains deliberately simpler: concurrent sessions are
investigated for transaction and state isolation, while the first active
session establishes the provider, model and Moderari system-prompt
behaviour shared by all concurrent sessions.

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
