# Lumen v3.2.9 Release Notes

## Completion Integrity and Final Answer Realisation

v3.2.9 focuses on the boundary between retained understanding and successful task completion.

### Changes

- Final Cognitive Checkpoints now require verified EOF rather than merely a valid-looking final tool result.
- Read continuation detection supports the actual Pi output format: `Use offset=N to continue`.
- Invalid non-positive read limits are removed and continuation offsets are enforced by Lumen.
- Final answer synthesis is explicitly grounded in the authoritative Final Cognitive Checkpoint.
- A one-time completion recovery handles empty, very short, or generic-refusal responses after a final checkpoint.
- Completion diagnostics report EOF, checkpoint, outstanding-read, answer-size, recovery, persistence and delivery states.
- Final result persistence now uses a valid dedicated MongoDB results collection accessor.

### Validation

- 133 automated tests passing.
