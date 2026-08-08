# Lumen v3.2.12 Release Notes

## Purpose

v3.2.12 corrects session identity and the end-of-read completion lifecycle. The previously planned dependency-validation work moves to v3.2.13, and the UI work moves to v3.2.14.

## Changes

- Generate a fresh UUID for every new conversation root.
- Preserve that UUID across Pi tool-call turns using observed conversation lineage.
- Never derive session identity from the model name or opening prompt.
- Start a clean session when lineage cannot be proven rather than contaminating historical records.
- Preserve resumed-session ancestry separately from the active run identity.
- Remove the model-generated Final Cognitive Checkpoint between the final source chunk and the answer.
- Treat EOF as resource exhaustion only; it is not proof that Qwen has completed reading or synthesis.
- Allow Qwen to process the final chunk and produce the user-facing answer normally.
- Persist a non-disruptive terminal-result checkpoint only after a normal assistant answer is available.
- Link the persisted result to that terminal checkpoint.
- Add lifecycle logging for session creation, unresolved lineage, completion detection and terminal persistence.

## Roadmap movement

- Former v3.2.12 dependency validation: v3.2.13.
- Former v3.2.13 UI work: v3.2.14.
