# Reviewed Documents — Carry-Forward Work Outside the M0.1 Roadmap

**Date:** 2026-09-06  
**Status:** Active carry-forward record  
**Source:** Final reconciliation of the N6–N10, Praebere and associated reviewed-document chain

## Purpose

This document preserves work found in the reviewed documents that is not complete and
is intentionally not part of the active
`LUMEN_EXTERNAL_RESEARCH_DISTRIBUTION_M0.1_ROADMAP`.

It prevents historical implementation checklists from remaining open merely to retain
future work. The original N6–N10/Praebere documents are closed and retained as evidence.

## Future Development

### Provider-Neutral Readiness

- Implement the full N9.6.4 provider-neutral readiness contract, including capability
  distinctions beyond the concrete M0.1 Ollama availability, residency, reservation,
  lock and reconciliation states.

### Per-Session Model Selection

- Replace the M0.1 runtime-global model selection with explicit per-session model
  selection and ownership.
- Permit a Repetere replay to use a replay-private model concurrently with interactive
  sessions using a different model.
- Add safe concurrent multi-model residency, reference counting, memory/resource
  failure handling and independent release.
- Support deliberate different-model replay as an explicitly labelled experimental
  variation rather than a faithful same-model replay.

### Rogare Recovery and Presentation

- Add automatic discovery/reconnection to an existing Pontis session.
- Restore Rogare conversation-history presentation after reconnect. M0.1 preserves
  execution context but requires manual entry of the Pontis session ID and does not
  restore prior local chat display.
- Consider friendlier diagnostics for internal Pontis-local commands if they are ever
  exposed beyond internal use.

### Shutdown Quiescing

- Add a Pontis shutdown quiescing/admission guard so late external requests are
  rejected cleanly while the stack is stopping.
- Continue to treat the previously observed Servire shutdown hang as a watch item;
  investigate shutdown ordering and process waits only if it recurs.

### Legacy Traversal Source Cleanup

- During the next Repetere/Moderari source pass, confirm that obsolete legacy
  consume-or-forward command traversal code has been removed.
- Retain defensive filtering that prevents control commands entering model context or
  replay input.
- This is source hygiene rather than an unresolved live-routing defect: N10 live
  validation confirmed direct routing, no loops, preserved correlation and no control
  leakage.

### Future Providers

- Add model-provider adapters beyond the M0.1 Ollama boundary only through Praebere.
- Do not distribute provider-specific lifecycle or discovery knowledge into Pontis,
  Moderari, Rogare, Repetere, Fiducia or Nuntius.

## M0.1 Work Deliberately Excluded from This Record

The following newly clarified work is now present in the canonical M0.1 roadmap and is
therefore not duplicated as an outstanding item here:

- authoritative provider/model identity in Vestigare `trace_recordings`;
- Repetere replay-start recorded-model validation and normal selection/reservation;
- controlled rejection when the active global model differs from the recorded model;
- Fiducia deterministic scheduled-replay failure for missing, unavailable or
  conflicting recorded models;
- failed staged-Replay recovery and fresh-session retry behaviour.

## Disposition

Move an item from this record only when it enters an approved roadmap or is completed.
Do not reopen the closed N6–N10/Praebere implementation addenda to track it.
