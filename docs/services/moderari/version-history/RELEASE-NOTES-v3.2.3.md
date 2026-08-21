# Lumen v3.2.3 — Auditable Cognitive Deltas and Crash Recovery

## Purpose

This release strengthens long-running source inspection after the overnight `dashboard.py` run exposed ambiguity in checkpoint growth, model-selected read overlap, and crash-recovery progress.

## Changes

- Checkpoints now require three explicit artefacts:
  - **Checkpoint Delta** — what was learned from newly retired source ranges.
  - **Perception Change** — how that material expanded, corrected, confirmed, or did not change the model's prior understanding.
  - **Updated Distilled Continuity** — a compact fixed-schema current cognitive state.
- Every new source range must be acknowledged in the delta.
- Unchanged perception must be explained rather than represented by a silent duplicate.
- Distilled Continuity is revised in place and bounded; the output budget is explicitly a ceiling, not a target.
- Only the `Updated Distilled Continuity` section is re-injected as working state. The complete checkpoint, including its delta, remains visible and the delta is persisted in the immutable checkpoint record.
- Sequential read offsets are canonicalised by Lumen from the exact continuation marker returned by the previous successful read. Qwen decides whether to continue; Lumen decides where.
- Every completed read is now represented in runtime source-progress metadata independently of checkpoint completion.
- Session snapshots persist completed read action/result pairs for crash recovery.
- Restore reconstructs latest Distilled Continuity, checkpoint-retained raw material, and completed-but-not-yet-distilled reads before continuing from the runtime next offset.
- Checkpoint prompt version updated to `cognitive-delta-v3`.
- Version updated to 3.2.3.

## Deferred

The separate full-payload audit log remains planned for the next release so this run can isolate checkpoint cognition, deterministic reading, and recovery behaviour.
