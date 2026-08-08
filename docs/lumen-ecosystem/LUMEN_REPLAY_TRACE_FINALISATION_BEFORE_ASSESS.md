# Lumen Replay / Trace Finalisation Before Assess

**Status:** Planning  
**Purpose:** Waypoint document prior to commencing Lumen Assess

---

# Current Position

Servire is now considered operational and development has been intentionally frozen.

The operational control plane for the Lumen++ ecosystem is complete for its original objectives. Future Servire enhancements are documented separately and will be revisited after further operational experience.

The immediate focus now shifts back to Replay and Trace to complete the experimentation workflow before beginning development of **Lumen Assess**.

The goal is to ensure Replay can reliably produce high-quality Trace recordings that Assess can later analyse.

---

# Architectural Responsibilities

The separation of responsibilities remains fundamental:

| Component | Responsibility |
|-----------|----------------|
| **Trace** | Records behaviour |
| **Replay** | Reproduces behaviour |
| **Assess** | Evaluates behaviour |
| **Servire** | Operates the platform |

No changes proposed here should blur those boundaries.

---

# 1. Automatic Trace Recording

## Objective

Replay should automatically manage the lifecycle of Trace recordings during Replay execution.

When a Replay experiment begins it should:

- automatically start a new Trace recording
- automatically generate an appropriate recording name
- associate that recording with the Replay experiment

Example Replay:

```text
Replay-Test-1-SimpleMath
```

Automatically generated Trace recording:

```text
Replay-Test-1-SimpleMath_20260808_153245_ab12cd34
```

The generated name should be:

- human readable
- chronologically sortable
- globally unique
- clearly associated with the originating Replay experiment

## Trace Completion

Trace **must not** stop when Replay forks.

Instead, Trace should continue recording because the purpose is to capture what the model actually does after behavioural divergence.

Trace should stop only when one of the following occurs:

- Replay completes successfully with a full behavioural match.
- Replay forks and the model subsequently returns its response, completing the live conversation after divergence.

This ensures the recording captures the complete behavioural outcome of both matched and divergent executions.

---

# 2. Trace Deletion

The Replay UI currently allows recordings to be inspected but not removed.

Replay development now generates many recordings and requires routine cleanup.

Add the ability to:

- delete individual Trace recordings
- prevent deletion of active recordings
- clean up abandoned recordings
- maintain a manageable recording list

---

# 3. Recovery of Failed or Abandoned Recordings

Occasionally Replay completes but the Trace recording remains permanently in the **recording** state.

Rather than simply deleting these records, the preferred approach is to understand why the recording remained open.

Possible causes include:

- Replay terminated unexpectedly
- Trace terminated unexpectedly
- communication failure
- stop request never sent
- stop acknowledgement lost

The existing logs may already contain sufficient information to determine the underlying cause.

Potential future improvements include:

- startup consistency checking
- stale recording detection
- automatic recovery of orphaned recordings
- richer recording states

Example recording states:

```text
Recording
Completed
Forked
Cancelled
Failed
Abandoned
```

The preferred solution is architectural recovery rather than simply hiding the symptom.

---

# 4. Replay Iteration Support

Replay should support executing the same experiment multiple times.

Example workflow:

```text
Run Replay
    ↓
Complete
    ↓
Pause (configurable)
    ↓
Run Again
    ↓
Repeat
```

Initial requirements:

- configurable iteration count
- configurable delay between iterations
- no teardown of Pi, Lumen or Ollama between runs
- each iteration starts a brand new Trace recording
- each Trace recording closes automatically according to the Trace completion rules above

This capability is expected to become one of the primary inputs into Assess.

---

# 5. Future Discussion — Model Lifecycle

This is **not** immediate implementation.

Future discussion topics include:

- selecting the Ollama model for Replay
- switching models between iterations
- unloading models
- reloading models
- comparing identical Replay experiments across different models

Example future workflow:

```text
Qwen 14B
    ↓
Replay ×10
    ↓
Gemma
    ↓
Replay ×10
    ↓
DeepSeek
    ↓
Replay ×10
```

This would provide Assess with consistent behavioural datasets across multiple models.

---

# Overall Objective

The immediate objective is to evolve Replay into a robust experiment runner capable of repeatedly generating complete and reliable Trace recordings.

Those recordings will become the primary evidence consumed by Lumen Assess for behavioural evaluation, consistency analysis and comparison.

This document marks the transition from completing the operational platform (Servire) to completing the experimentation platform (Replay + Trace), providing the foundation upon which Lumen Assess will be built.
