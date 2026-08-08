# Milestone 7 Acceptance — Traffic Classification and Replay Planning

## Objective

Derive an immutable, reviewable replay plan from a validated Trace recording while preserving every captured message unchanged.

## Accepted capabilities

- Every validated request/response exchange is assigned exactly one category:
  - Conversation
  - Continuity
  - Control
  - Observation
  - Heartbeat
- The default replay policy is explicit:
  - Conversation: included
  - Continuity: excluded pending experiment choice
  - Control: included
  - Observation: excluded
  - Heartbeat: excluded
- Unknown state-changing requests are retained conservatively as Control.
- The Recording Explorer displays category totals, classification reasons, and replay-candidate state.
- The exchange list can be filtered by All traffic, each category, or Replay candidate.
- Search operates within the selected traffic filter.
- `GET /recordings/{recording_id}/plan` exposes the deterministic default plan.
- Trace recordings and messages remain immutable.

## Explicitly excluded

- Operator editing of replay-plan membership
- Continuity-policy selection
- Replay execution
- Request rewriting or identifier remapping
- Target dispatch
- Response capture
- Assessment or scoring
