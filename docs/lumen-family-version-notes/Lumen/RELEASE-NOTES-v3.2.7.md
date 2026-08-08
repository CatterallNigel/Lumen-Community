# Lumen v3.2.7 Release Notes

## Primary experiment

v3.2.7 begins the evidence-oriented checkpoint redesign. It changes one deliberately bounded facet: the former **Inferences and confidence** section is replaced by **Evidence-linked observations and conclusions**.

Each conclusion must now identify:

- the observation;
- supporting source evidence;
- the supported conclusion;
- confidence;
- what remains unconfirmed.

This tests whether reducing ambiguity improves checkpoint usefulness without redesigning every weak section simultaneously.

## Final Cognitive Checkpoint lifecycle

Final Cognitive Checkpoint generation is now task-scoped and one-way:

- it can run only when the latest turn is a valid terminal tool result;
- EOF and invalid-range tool errors cannot trigger it;
- it is captured at most once for a user task;
- after capture, Lumen injects an explicit transition from reading to synthesis;
- a later user request cannot be intercepted by stale completion state.

## Operational changes

- Version updated to 3.2.7.
- Added `python app.py --clear-logs` to remove configured current and rotated log files before startup.
- Existing session commands remain available.
