# Lumen Vestigare — System-Prompt Trace Investigation and Possible Remediation

**Status:** M0.1 Investigation / Conditional Development  
**Service:** Vestigare  
**Related Services:** Moderari, Repetere, Nuntius  
**Release:** External Research Distribution M0.1

## 1. Purpose

This document defines the investigation required to determine how Vestigare currently records system prompts when Moderari changes the model execution context, and the remediation required if the current Trace representation is insufficient for faithful Replay.

The investigation exists because an incoming client system prompt and the system prompt actually presented to the model may be different.

A Replay must reproduce the latter without losing the provenance of the former.

## 2. Risk

Moderari currently removes an incoming client system prompt when using its own Default prompt, and the planned `Custom` policy will do the same for a researcher-defined prompt.

Conceptually:

```text
Client:
    system = USER-A

Moderari:
    policy = Custom
    replace USER-A with CUSTOM-X

Model:
    system = CUSTOM-X
```

If Vestigare records `USER-A` before the transformation, while another mechanism records `CUSTOM-X`, Repetere could potentially reconstruct both as active system messages.

For simple or deterministic test asks this may not produce an obvious output difference, so a replay can appear to match while the execution conditions are incorrect.

This is an M0.1 replay-fidelity risk.

## 3. Required Evidence Model

The Trace must be capable of representing two distinct facts:

```text
Incoming System Prompt
    USER-A
    provenance: client supplied
    replay status: superseded

Effective System Prompt
    CUSTOM-X
    provenance: Moderari Custom
    replay status: active
```

The incoming prompt must not be hard-deleted or destructively overwritten.

It remains evidence of what entered Lumen.

However, when Moderari replaces it, the incoming prompt becomes a **soft-deleted Replay element**: visible for provenance and later assessment, but excluded from the active context reconstructed by Repetere.

The invariant is:

> **Exactly one effective system prompt is active in Replay: the system prompt that actually reached the model in the source execution.**

## 4. Investigation Sequence

No remediation should be implemented until the current behaviour is established.

### Test 1 — Current/Baseline Behaviour

Perform a fresh ask using a known client system prompt.

Record:

- client request;
- Moderari behaviour;
- resulting Vestigare Trace;
- exact Trace ordering and representation;
- system prompt reconstructed by Repetere.

### Test 2 — Pass-through

Configure Moderari to `Pass-through`.

Use:

```text
Client system prompt = USER-A
```

Expected model condition:

```text
system = USER-A
```

Inspect the resulting Trace and Replay reconstruction.

### Test 3 — Custom

Configure Moderari to `Custom`.

Use:

```text
Client system prompt = USER-A
Moderari Custom = CUSTOM-X
```

Expected model condition:

```text
system = CUSTOM-X
```

Inspect whether the Trace contains:

- USER-A only;
- CUSTOM-X only;
- both with an explicit relationship;
- both without an explicit relationship.

Then inspect what Repetere reconstructs.

### Optional Confirmation — Moderari Default

Repeat using `Moderari Default` if required to confirm that Default and Custom replacement follow the same Trace path.

## 5. Structural Comparison

For each test, capture and compare:

- Trace record sequence;
- incoming system prompt;
- effective system prompt;
- Moderari policy provenance;
- request/session correlation;
- which records are considered replayable;
- actual Replay request sent toward the model.

The investigation must answer:

1. Where does Vestigare observe the incoming request?
2. Does it observe the post-Moderari model context?
3. Does it currently distinguish incoming from effective system context?
4. Can Repetere currently reconstruct more than one active system prompt?
5. Does Replay currently reproduce the prompt that actually reached the model?

## 6. Behavioural Validation

Structural inspection alone is insufficient.

At least one test should make incorrect system-prompt reconstruction behaviourally visible.

For example:

```text
Client system prompt:
    Prefix every answer with CLIENT:

Moderari Custom:
    Prefix every answer with CUSTOM:
```

The original Custom execution should follow the Custom condition.

Replay should reproduce the same effective condition.

The exact test wording is not important; the requirement is that the prompts are deliberately distinguishable, preferably contradictory, so that accidentally supplying both is more likely to expose the defect.

This test complements Trace inspection; it does not replace it.

## 7. Decision Point

### If Vestigare Already Records Correct Effective Context

If the Trace already:

- preserves incoming prompt provenance where required;
- identifies the effective prompt;
- allows Repetere to reconstruct only that effective prompt;

then no Vestigare remediation is required.

The behaviour should be formalised with regression tests.

### If Vestigare Does Not Distinguish the Contexts

Vestigare requires M0.1 remediation before Replay fidelity is accepted.

## 8. Candidate Remediation

If Moderari's transformation occurs after Vestigare has recorded the incoming prompt, Moderari can explicitly notify Vestigare of the effective system prompt.

Conceptually:

```text
Moderari
    |
    | incoming prompt replaced
    |
    | \obt trace effective-system-prompt
    | request/session id
    | effective prompt
    | Moderari policy
    v
Nuntius
    |
    v
Vestigare
```

This uses the common M0.1 control plane rather than introducing a private Moderari/Vestigare mechanism.

The exact command name and payload should be defined during implementation.

## 9. Vestigare Remediation Behaviour

On receiving the correlated effective-system-prompt information, Vestigare should be able to:

1. locate the relevant execution/Trace;
2. retain the original client system prompt;
3. mark that prompt as superseded for Replay;
4. retain the actual Moderari-injected prompt;
5. mark the injected prompt as the active effective system prompt;
6. retain Moderari policy/provenance where useful;
7. expose only the effective prompt to Repetere as active Replay context.

The `\obt` control message itself must not become a user/model conversational Trace record.

## 10. Repetere Contract

Repetere must not attempt to infer which of multiple system prompts is authoritative.

Vestigare must expose an unambiguous replayable representation.

Repetere then reconstructs:

```text
effective system prompt
+ replayable conversational context
```

and excludes:

```text
superseded incoming system prompt
```

The original incoming prompt remains inspectable evidence.

## 11. Ordering and Correlation

Any remediation must correctly handle ordering.

The Moderari notification must be correlated to the exact execution using the appropriate request/session identifiers.

Vestigare must not apply an effective-system-prompt update to another session or another Trace.

If the correction cannot be correlated unambiguously, the Trace must not be treated as replay-fidelity-complete.

## 12. Trace and Nuntius Boundary

The control path and evidence path remain separate:

> **Nuntius diagnostics record how the control message was transported.**

> **Vestigare records the resulting execution context and provenance.**

The internal `\obt` command must not appear as a conversational model turn.

## 13. M0.1 Acceptance

This investigation/remediation is complete when:

- baseline, Pass-through and Custom cases have been inspected;
- the incoming and effective system prompts can be distinguished where Moderari replaces the prompt;
- the original incoming prompt remains visible as provenance;
- a superseded prompt is excluded from Replay;
- Repetere reconstructs exactly one effective system prompt;
- Replay uses the prompt that actually reached the model;
- request/session correlation prevents cross-Trace correction;
- internal `\obt` traffic remains outside conversational Trace;
- a system-prompt-sensitive behavioural test confirms the expected execution condition;
- regression tests protect the resulting behaviour.

## 14. Architectural Principle

> **Trace records both what entered Lumen and what actually reached the model when those differ, without confusing provenance with replayable execution context.**

This preserves evidence while allowing Replay to reproduce the actual model execution conditions faithfully.
