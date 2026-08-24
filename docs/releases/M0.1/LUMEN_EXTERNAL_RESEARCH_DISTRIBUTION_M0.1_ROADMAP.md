# Lumen External Research Distribution M0.1 — Development Roadmap

**Date:** 2026-08-22  
**Status:** Proposed M0.1 Roadmap

> **“Plans are worthless, but planning is everything.”**  
> — Dwight D. Eisenhower


## Document Revision History

| Date | By | Version | Description |
| --- | --- | --- | --- |
| 2026-08-23 | Nigel Catterall | 1.0 | First reviewed release |

## Purpose

This roadmap orders the work currently identified as necessary to reach the Lumen External Research Distribution M0.1 milestone.

It is intentionally an engineering sequence rather than a calendar commitment.

M0.1 is intended to establish a coherent, reproducible research baseline. The sequence below is dependency-driven: the shared control plane is established before the M0.1 features that consume it.

All M0.1 development and acceptance validation assumes a **single-host installation**. Distributed multi-host deployment is outside the M0.1 release boundary.

## Phase 1 — Confirm Existing Behaviour

Before changing command or replay behaviour, establish the current implementation baseline.

- Inspect existing Repetere `\obt` handling and injection.
- Inspect existing Moderari `\obt` handling and injection.
- Document exactly how Moderari currently extracts, removes, rewrites or reinserts tool-related information from an incoming client system prompt.
- Confirm the effective system prompt currently presented to the model when tools are available.
- Inspect Pontis recognition/routing of `\obt`.
- Confirm current Praebere provider/model discovery and selection paths.
- Confirm current Trace representation of system prompts, model/provider context and replay executions.
- Establish whether Vestigare records provider identity and model identity, where that information is stored, and whether it is descriptive provenance or part of the replayable execution context.
- Determine whether Repetere reads or acts upon any recorded provider/model identity.
- Determine whether provider/model identity appears indirectly in system, assistant or other conversational messages.
- Test whether the same source Trace can currently be replayed against a different selected provider/model without otherwise changing its recorded conversational context.
- Inspect what is actually sent to the alternate model and what the resulting Vestigare Trace records.
- Compare fresh baseline, `Pass-through`, and `Custom` executions to determine whether Trace contains the incoming system prompt, the post-Moderari effective prompt, or both.
- Determine which system-prompt record Repetere currently reconstructs and whether Replay can send two active system prompts.
- Confirm current Servire knowledge of configured/running services.
- Confirm how request/session correlation currently works for control responses.
- Inspect current Repetere session creation/reuse behaviour.
- Determine whether sequential Replay runs currently create fresh sessions or inherit context/state from a preceding Replay.
- Confirm current Moderari context-compaction behaviour at the approximately 63% utilisation threshold, including checkpoint creation, use of the two most recent checkpoints, and what effective post-compaction context Vestigare records.

**Exit condition:** existing behaviour and integration points are understood well enough to change them without introducing parallel mechanisms, including a documented understanding of whether provider/model identity is Trace provenance, Replay-binding information, or both.

## Phase 2 — Servire Control-Plane Configuration

Extend Servire's existing service descriptions so they can define the active `\obt` topology required by Nuntius.

- Add/configure `obt_enabled` per service.
- Define command ownership metadata.
- Define successful query-response targets.
- Extend `\obt services` so Nuntius can obtain the active control-plane catalogue.
- Accept a valid empty active catalogue.
- Define runtime service-start/service-stop catalogue updates.
- Validate duplicate/conflicting exclusive command ownership.

Do not add `log_class` or operational-log user/internal classification in this phase.

**Exit condition:** Servire can authoritatively describe which active service owns each configured control command and where successful query results must be delivered.

## Phase 3 — Nuntius Control Plane

Implement Nuntius against the Servire-defined topology.

- Implement Nuntius service skeleton and health.
- Bootstrap directly from Servire using `\obt services`.
- Treat a successful empty catalogue as healthy.
- Build an in-memory routing dictionary.
- Keep the dictionary current from Servire topology updates.
- Standardise the service `\obt` endpoint.
- Implement:
  - `204` — not handled;
  - `200` with no body — successful execution;
  - `200` with body — successful query/request;
  - `4xx/5xx` — recognised command failure.
- Route commands directly to configured owners rather than broadcasting normally.
- Preserve request correlation while commands execute.
- Return query responses to Pontis.
- Deliver response bodies only to additional consumers configured by Servire.
- Verify control traffic remains outside the ask/answer Trace path.
- Implement Nuntius diagnostic logging for command routing, responses, failures, timeouts and related control-plane activity.
- Expose Nuntius diagnostics through the Nuntius UI within Servire.
- Keep Nuntius diagnostic/control-plane traffic out of the normal Servire Operations Log.

**Exit condition:** a caller can issue a Lumen control command without knowing the owning service, query results are returned/distributed according to the Servire-defined topology, and Nuntius transport diagnostics are available through its Servire-exposed UI without polluting the Servire Operations Log.

## Phase 4 — Pontis Request/Response Integration

Complete the external control-command path.

- Recognise client-originated `\obt`.
- Route it directly to Nuntius.
- Keep it outside normal model ask/answer processing.
- Preserve client/session/request correlation.
- Receive returned query results from Nuntius.
- Deliver each result to the correct originating client/session.

**Exit condition:** an external client can issue a request/query command and receive the authoritative response through the common Lumen control plane.

## Phase 5 — Repetere / Moderari Control Migration and Replay Fidelity

Move existing control behaviour onto Nuntius and establish faithful system-context replay.

- Migrate existing Repetere/Moderari `\obt` behaviour to the common path.
- Perform the defined Vestigare system-prompt investigation using baseline, `Pass-through`, and `Custom` executions.
- Preserve an incoming system prompt as provenance if Moderari supersedes it, but exclude the superseded prompt from Replay reconstruction.
- Ensure Trace unambiguously identifies the effective system prompt that reached the model.
- Correct Vestigare if the distinction is not currently represented; candidate remediation is a correlated Moderari `\obt` effective-system-prompt notification through Nuntius.
- Ensure any remediation control message remains outside conversational Trace.
- Implement a session-scoped Moderari Pass-through override.
- Require each Replay execution to create a new isolated Replay session.
- Have Repetere issue the Pass-through override through Nuntius for that new Replay session before replay.
- Require positive Moderari acknowledgement before model interaction begins.
- Initialise the Replay session only from the source Trace and explicitly defined experimental conditions.
- Replay the effective system prompt captured in the source Trace.
- Verify sequential Replay runs cannot inherit context or temporary state from preceding runs.
- Verify Replay never substitutes the current Moderari Default or current saved Custom prompt.
- Verify Replay reconstructs exactly one active effective system prompt and never combines a superseded incoming prompt with its Moderari replacement.
- Run at least one deliberately system-prompt-sensitive behavioural validation capable of exposing duplicate/incorrect system context.

**Exit condition:** Replay reproduces exactly the recorded effective system prompt in a newly created isolated Replay session; superseded incoming system prompts remain provenance but are not replayed, and no prior Replay state is inherited. Replay control behaviour must remain consistent with the M0.1 shared concurrent-session execution-condition rules validated in Phase 10.

## Phase 6 — Praebere Discovery and Model Selection

Move provider/model control onto the common command path.

- Finalise `\obt providers`.
- Finalise `\obt models`.
- Finalise `\obt model select <model>`.
- Make Praebere the executor of provider/model operations.
- Return provider/model query results to the originating caller.
- Deliver discovery results to Rogare/Moderari where Servire configuration requires it.
- Avoid broadcasting provider/model lists to unrelated services.
- Verify selected model/provider state is represented correctly in subsequent execution evidence.

- Confirm the M0.1 tool-responsibility boundary: an external client owns its own tool declaration, execution and result handling.
- Confirm Rogare has no native tool environment and uses an external tool provider.
- Validate Pi as the supported M0.1 tool provider for Rogare, installed in its expected/default installation location.
- Do not claim Rogare support for alternative tool providers in M0.1.

**Exit condition:** Rogare and external clients use the same Nuntius path for provider/model discovery and selection.

## Phase 7 — Moderari System-Prompt Policy

Make system-prompt handling an explicit experimental condition.

- Implement `Pass-through`.
- Preserve `Moderari Default`.
- Implement `Custom` as an exact researcher-defined system-prompt condition.
- In `Custom`, remove the incoming client system prompt without carrying its tool instructions/descriptions into the selected Custom prompt.
- In `Custom`, do not add, remove, rewrite, merge, normalise or otherwise augment the selected Custom prompt.
- Keep tool availability separate from tool instructions contained in the Custom system prompt.
- Require any desired tool descriptions or tool-use instructions to be explicitly present in the researcher-defined Custom prompt.
- Implement the System Prompt Policy UI as a Moderari-owned operational surface.
- Expose that Moderari UI within Servire's Moderari tab without making Servire a control intermediary.
- Add explicit editor working-copy versus active-runtime-state visibility.
- Require explicit `Apply` before Custom editor contents become Moderari's active prompt.
- Treat `Apply` as a local Moderari configuration action rather than a Servire/Nuntius round trip.
- Verify Pass-through does not alter client system context.
- Verify Custom reaches the model as the last explicitly Applied editor contents, subject only to unavoidable transport encoding/serialization; the Applied prompt need not correspond to a stored saved prompt.
- Verify a Custom prompt without tool instructions is not silently augmented when tools are available.
- Verify a Custom prompt containing researcher-supplied tool instructions does not acquire a second Moderari-generated tool block.
- Verify the effective system prompt appears in Trace.

**Exit condition:** system-prompt behaviour is explicit, selectable and reproducible; `Custom` is demonstrably the exact researcher-defined prompt rather than a Moderari-generated or tool-augmented derivative.

## Phase 8 — Saved System Prompts

Complete the M0.1 Custom-prompt persistence and reuse workflow.

- Add a dedicated MongoDB collection for Moderari saved system prompts.
- Define a minimal schema containing stable `prompt_id`, human-readable `name`, `content`, `owner_id`, `scope`, `created_at` and `updated_at`.
- Use a simple/default M0.1 owner identity while retaining `owner_id` as an explicit future-compatibility field.
- Retain an explicit `scope` field without implementing multi-user/organisational policy in M0.1.
- List/select saved prompts.
- Load a selected prompt into the Custom editor as a working copy.
- Ensure editing the working copy does not mutate the saved object or active runtime configuration.
- Implement `Apply` independently of persistence.
- Implement `Save As…` to create a new saved prompt without overwriting the source prompt.
- Implement explicit `Update Saved` for replacement of an existing saved prompt.
- Implement explicit `Delete Saved`.
- Ensure `Save As…`, `Update Saved` and `Delete Saved` do not silently alter Moderari's active runtime prompt.
- Ensure `Apply` does not silently create or update a saved prompt.
- Retain the actual applied prompt in Trace regardless of later saved-prompt changes.
- Do not store active runtime state as an `is_active` property of a saved-prompt document.

Do not add prompt sharing, organisational prompt libraries, mandatory organisational prompts, permission policy or sophisticated prompt version history in M0.1. Those are future development concerns.

**Exit condition:** researchers can safely reuse and modify controlled system-prompt conditions while saved definitions, editor working copies and Moderari's active runtime configuration remain explicit and independent.


## Phase 9 — Experiment Structure

Implement the Experiment relationship required for controlled repeated execution.

- Introduce Experiment as an explicit concept.
- Associate an Experiment with its source Trace.
- Associate replay-created Traces with Experiment runs.
- Have Fiducia coordinate repeated runs against an Experiment.
- Ensure every Experiment run receives its own Replay session rather than reusing the session of another run.
- Surface run state as `MATCHED`, `DIVERGED`, or `FAILED / INCOMPLETE`.
- Surface first divergence point where available.
- Present Experiment/run relationships clearly in the UI.
- Preserve the distinction between Repetere divergence detection and future Aestimare assessment.

**Exit condition:** a researcher can create repeated controlled executions and see the resulting evidence as one coherent Experiment.

## Phase 10 — Multi-Session Isolation Validation

Investigate concurrent-session transaction isolation under the M0.1 shared execution-condition model.

- Run at least two simultaneous active sessions.
- Verify transactions initiated within each session remain bounded to their originating session throughout the relevant Lumen execution path.
- Verify session-specific conversational context, responses, Trace evidence and other session-specific state do not leak between sessions.
- Verify Pontis/Nuntius request-response correlation remains bound to the originating session.
- Validate that the first active session establishes the shared provider, model and Moderari system-prompt condition for concurrent sessions.
- Verify a subsequent concurrent session cannot change the provider, selected model, Moderari system-prompt policy or Applied Custom system-prompt content while another session remains active.

Any demonstrated session leakage is an M0.1 blocker.

**Exit condition:** concurrent transactions remain bounded to their originating sessions, with no session-specific leakage, and all concurrent sessions operate under the provider/model and Moderari system-prompt conditions established by the first active session.

## Phase 11 — Runtime Authorization and Distribution Security

Implement the external-distribution trust boundary defined by **M0.1 Runtime Authorization and Code Protection**.

- Define Distribution ID and Service Group UUID handling.
- Provision an installation-specific cryptographic identity.
- Register the installation public identity with Illuminates.One.
- Implement Servire → Illuminates.One authorization over TLS/HTTPS.
- Authenticate the Servire request using the installation identity.
- Implement Illuminates.One signed authorization responses and Servire verification.
- Implement nonce/challenge freshness protection.
- Implement authorization lease, renewal, grace and expiry states.
- Verify deliberate heartbeat suppression cannot provide indefinite normal operation.
- Define and implement short-lived Servire-authorised protected-service startup.
- Ensure protected services reject unsupported/unauthorised startup as an authorised external runtime.
- Encrypt protected operational configuration at rest.
- Couple normal configuration unlock/use to valid runtime authorization without embedding a permanent master unlock secret in the distribution.
- Prefer in-memory use of decrypted protected configuration.
- Validate that copying release/install material does not automatically reproduce an authorised installation.
- Inspect final distributed artifacts for private Illuminates.One authority, master secrets or hard-coded bypass material.
- Document the explicit non-goal of absolute tamper resistance on researcher-controlled hardware.

**Exit condition:** a provisioned installation can establish signed runtime authorization, operate under a bounded lease, start protected services through Servire, use protected configuration only through the authorised runtime path, and fail closed after authorization expiry; copied distribution material alone is insufficient to create another authorised installation.

## Phase 12 — Internal M0.1 Acceptance Validation

This phase is Lumen's **internal acceptance testing** for the M0.1 release candidate. It is performed before the distribution is supplied for external research use and is not intended to replace or predetermine external researcher evaluation.

Run the complete M0.1 research workflow as an internal acceptance exercise.

A representative validation should demonstrate:

1. client/Rogare connection;
2. provider/model discovery through Nuntius;
3. model selection;
4. selectable system-prompt policy through the Moderari-owned UI, including when exposed in Servire;
5. saved Custom prompt selection and loading into the editor as a working copy;
6. verification that editing the working copy does not alter the saved prompt or active Moderari configuration;
7. explicit Apply of the current editor contents;
8. verification that Save As creates a new prompt while Update Saved is the only explicit replacement path;
9. verification that saved-prompt persistence and active runtime state remain independent;
10. verification that Custom reaches the model without Moderari tool extraction, merging or augmentation;
11. verification that tool availability does not silently alter a Custom system prompt;
12. normal model execution and Trace capture;
13. source Trace selection;
14. session-scoped Replay Pass-through;
15. faithful replay of the source effective system prompt;
16. Experiment creation;
17. Fiducia-coordinated repeated executions;
18. Repetere divergence reporting;
19. clean Trace evidence without internal `\obt` chatter;
20. query responses delivered only to the correct originator/configured consumers;
21. simultaneous-session isolation, including confirmation that the first active session establishes provider/model and Moderari system-prompt behaviour for all concurrent sessions and later sessions cannot change those shared conditions;
22. a fresh isolated session for every Replay run;
23. no context/state inheritance between repeated Replay runs;
24. incoming versus effective system-prompt provenance is distinguishable;
25. exactly one effective system prompt is reconstructed during Replay;
26. a system-prompt-sensitive test confirms replaced and duplicate prompts cannot silently contaminate Replay;
27. installation cryptographic identity and signed Illuminates.One authorization;
28. nonce/freshness rejection of replayed authorization responses;
29. authorization lease renewal, grace and expiry behaviour;
30. protected-service startup through Servire authorization;
31. protected configuration availability only through the authorised runtime path;
32. copied installation material failing to establish another authorised installation;
33. absence of Illuminates.One private signing authority, permanent master unlock secrets and hard-coded authorization bypasses from distributed artifacts;
34. external-client tool declaration/execution remains client-owned, while Rogare tool use is validated through Pi in its expected/default installation location;
35. Moderari context compaction is exercised across the approximately 63% utilisation threshold and the effective checkpoint/compacted context remains observable in Vestigare;
36. Nuntius diagnostics are available through the Nuntius UI within Servire and do not enter the normal Servire Operations Log;
37. Rogare status polling remains a documented M0.1 Operations Log limitation;
38. the complete Dockerised Lumen distribution operates successfully on a single host machine;
39. Vestigare provider/model provenance and Repetere provider/model Replay-binding behaviour have been established by inspection and controlled cross-model/provider Replay testing, with no unsupported binding or substitution guarantee left implicit.
40. a Git-based release feedback and issue-reporting route is available and documented for the external researcher, providing a persistent place for comments, bug reports and feature requests rather than relying on LinkedIn or other informal messaging.

Regression tests for existing Lumen behaviour should remain green.

**Exit condition:** the M0.1 release candidate passes Lumen's internal acceptance validation, the workflow is coherent end-to-end, its known limitations are documented, and an appropriate Git-based channel is available for external research comments, bug reports and feature requests.

## After M0.1 — Before General Research Release

The following work is not required to define M0.1 but is required or expected before wider external research distribution:

- Servire single-active-operator enforcement using a renewable operator lease;
- graceful refusal of a second simultaneous Servire operator;
- explicit release/exit of the active operator session;
- recovery from abandoned/crashed operator sessions;
- final packaging/distribution hardening and documentation, including provisioning/revocation operating procedures where required.

## Explicitly Deferred Beyond M0.1

- Rogare human correctness/quality ratings;
- Aestimare;
- multiple Pontis tool providers and unified tool catalogue;
- bounded-computational-resource behavioural experiments;
- Servire Operations Log user/internal classification and default filtering, including removal/filtering of Rogare status-polling noise;
- commercial multi-user/operator entitlement;
- saved-prompt ownership and organisational policy, including user-private, organisation-shared, installation-wide and mandatory organisational prompt scopes and associated permissions;
- sophisticated prompt libraries/versioning;
- durable enterprise messaging for Nuntius;
- distributed multi-host deployment, including the required service discovery, transport, operational control, security, authorization and logging validation.
