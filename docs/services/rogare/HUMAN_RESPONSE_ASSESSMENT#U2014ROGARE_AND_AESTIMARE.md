# Human Response Assessment — Rogare and Aestimare

**Status:** Future Development / Research  
**Target:** Post-M0.1

## Purpose

Lumen Rogare should eventually allow a user to record their own assessment of a model response.

The initial proposal is to capture two independent user ratings on a 1–10 scale:

- **Answer Correctness** — how correct the user believes the answer to be.
- **Answer Quality** — how good the user considers the answer overall.

These measures should remain separate. A response may be factually correct while being incomplete, unclear or otherwise poor quality. Conversely, a response may appear authoritative, useful and well structured while containing factual or reasoning errors.

## Relationship to Aestimare

Human assessment should not be treated as ground truth for Lumen Aestimare.

Instead, it represents an additional observation associated with an execution.

This creates the opportunity to compare:

- user-perceived correctness;
- user-perceived quality;
- Aestimare behavioural assessment;
- execution evidence captured by Trace;
- repeated behaviour observed through Replay.

Agreement between human and machine assessment is useful evidence, but disagreement may be equally or more valuable.

For example, a user may rate an answer highly while Aestimare identifies unsupported assertions, uncertainty or behavioural anomalies.

Alternatively, Aestimare may find strong evidence supporting an answer which the user considers poor or incorrect.

These disagreements themselves become subjects for investigation.

## Architectural Principle

Human assessment should be associated with the execution being assessed rather than incorporated into the execution itself.

Conceptually:

```text
Execution
    |
    +-- Trace evidence
    |
    +-- User correctness rating
    |
    +-- User quality rating
    |
    +-- Aestimare assessment
```

This preserves the independence of the different observations.

## Initial Scope

A future first implementation should remain deliberately simple:

- correctness rating from 1–10;
- quality rating from 1–10;
- ratings optional;
- ratings associated with the relevant execution;
- sufficient provenance to identify who or what supplied an assessment where identity is available.

Additional qualitative feedback could be considered later but is not required for the initial capability.

## Research Value

The capability creates a potentially valuable longitudinal dataset comparing human perception with behavioural assessment.

Questions that could eventually be investigated include:

- How closely does perceived correctness correlate with Aestimare assessment?
- Does perceived response quality influence perceived correctness?
- Are confidently presented incorrect responses consistently rated more highly?
- Do user assessments change as trust in a model develops?
- Can disagreement between human and Aestimare assessment identify useful classes of reasoning-assurance problem?

The purpose is not to establish whether the human or Aestimare is inherently "right."

The purpose is to capture independent evidence about the same model behaviour.