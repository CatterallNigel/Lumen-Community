# Lumen v3.2.4 — Working-State Reconstruction and Instruction Observability

## Purpose

This release improves the quality and clarity of Qwen's persisted working state while keeping the existing rolling-read and recent raw-chunk policy unchanged. It is deliberately scoped so checkpoint quality and instruction interpretation can be evaluated without simultaneously changing source-retention behaviour.

## Changes

- The generated system prompt now requires Qwen to continuously build, test, and revise a working model of the complete subject while reading.
- Source chunks are explicitly treated as parts of the same file or body of work rather than unrelated documents.
- Presentation constraints are distinguished from internal analysis requirements.
- Ambiguous, overlapping, or contradictory instructions must be identified explicitly rather than resolved through an unreported compromise.
- Checkpoints now include an **Instruction State** section recording:
  - current task phase;
  - active controlling instructions and their practical interpretation;
  - ambiguities or conflicts;
  - the resolution being applied;
  - behavioural effects on analysis, tools, synthesis, and presentation.
- Checkpoints now distinguish expanded, corrected, confirmed, reorganised, and unchanged perception.
- Updated Distilled Continuity is richer and self-contained, including architectural boundaries, evidence, inference, assumptions, task strategy, instruction interpretation, and question lifecycle.
- Questions now use explicit lifecycle states: `OPEN`, `PARTIALLY RESOLVED`, `RESOLVED`, `SUPERSEDED`, and `INVALID`.
- The checkpoint prompt requires a complete current-state reconstruction rather than a diary or incremental append.
- The model-facing continuity window is limited to exactly the two most recent checkpoints:
  - the immediate previous checkpoint is comparison and omission-recovery evidence only;
  - the current checkpoint is explicitly authoritative.
- Older checkpoints remain stored immutably for audit and Continuity Inspector use but are not returned to Qwen.
- Checkpoint output capacity increased from a 2,000-token baseline / 4,096-token maximum to a 3,000-token baseline / 6,144-token maximum.
- Checkpoint prompt version updated to `working-state-v4`.
- Recent raw-source retention remains unchanged at two chunks.
- Version updated to 3.2.4.

## Intentionally Deferred

- Removing recent raw chunks after reading completes.
- Task-phase-specific context construction modes.
- Reducing the active checkpoint window from two to one.
- Automated validation and promotion of a checkpoint to canonical continuity.
- Model calibration and profile-generation suites.
