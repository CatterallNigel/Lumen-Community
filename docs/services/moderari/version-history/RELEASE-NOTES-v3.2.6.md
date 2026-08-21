# Lumen v3.2.6

## Checkpoint optimisation

- Revised the checkpoint prompt to favour architectural synthesis over a flat function inventory.
- Functions and routes are now organised into architectural capability groups.
- Data and control flow must be examined through representative request, state-change, rendering, and dependency paths.
- Uncertainty sections may no longer return an unexplained bare `None`; the model must state what was examined.
- Added a checkpoint evolution assessment using explicit classifications:
  - `STATIC FOR EVIDENCED REASON`
  - `REFINED`
  - `EXTENDED`
  - `CORRECTED`
  - `SUPERSEDED`
  - `STILL UNCERTAIN`
- Advanced the checkpoint prompt version to `working-state-v5`.

## Final Cognitive Checkpoint

- Added automatic detection of a completed source-reading phase.
- When the final read result has no continuation marker and no requested source remains, Lumen creates a Final Cognitive Checkpoint before asking the model to execute the requested task.
- The Final Cognitive Checkpoint retires all remaining raw source chunks, including the normally retained recent chunks.
- Final checkpoints are persisted with `checkpoint_type: final_cognitive` and are visually distinguished in the UI.
- The final checkpoint closes the provenance gap between the last rolling checkpoint and the final answer.

## Result persistence and UI

- Final model responses are now first-class session artefacts.
- Results are associated with the session, request, model profile, final checkpoint, source progress, timing, and finish reason.
- Added JSON endpoints:
  - `GET /api/results`
  - `GET /api/results/latest`
- The checkpoint page now includes Checkpoints and Results views.

## Logging separation

- Added dedicated interaction, UI, and audit log files.
- Pi → Lumen → model traffic remains in the interaction stream.
- Dashboard and API page activity is written to the UI stream.
- Checkpoint and result persistence events are written to the audit stream.

Default files:

```yaml
logging:
  interaction_file: logs/interaction.log
  ui_file: logs/ui.log
  audit_file: logs/audit.log
```

## Validation

- The complete automated test suite passes.
