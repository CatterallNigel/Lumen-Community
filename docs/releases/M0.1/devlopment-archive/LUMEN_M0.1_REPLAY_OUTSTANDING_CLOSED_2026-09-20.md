# Lumen M0.1 Replay — Outstanding Work

**Date:** 2026-09-19  
**Status:** CLOSED for M0.1 — A1, A2, A3 and A6 closed; A4 and A5
deferred to post-M0.1 hardening/acceptance  
**Companion:** `LUMEN_M0.1_REPLAY_SIGN_OFF_2026-09-13.md`

## Purpose

This document contains only work that should remain open after reviewing
the nine documents in `development-in-progress-2026-09-12.zip` against
the implementation completed through Repetere 0.20.18 and Fiducia 0.7.3.

It intentionally separates **M0.1 completion work** from **post-M0.1
future development**.

# A. M0.1 completion work

## 1. Fiducia scheduled Replay end-to-end acceptance — CLOSED

Fiducia can now discover a staged Repetere Experiment using the current
contract. Complete the live scheduling path and verify that a Fiducia
trigger:

1.  targets the intended `experiment_id`;
2.  creates a genuinely new Repetere Run;
3.  receives a fresh Pontis Replay session;
4.  creates/correlates the Replay child Trace;
5.  records Repetere acceptance separately from terminal outcome;
6.  records the authoritative terminal result (`MATCHED`, `DIVERGED`, or
    `FAILED / INCOMPLETE`);
7.  retains required/effective model and failure/result provenance;
8.  leaves the Experiment reusable after clean terminal cleanup.

**Status update — 2026-09-19: CLOSED.** The scheduled Replay path has
now been proven end-to-end, including Fiducia scheduling, creation of a
new Repetere Run, fresh Replay/session lifecycle, child Trace
correlation, terminal result evidence, and clean lifecycle completion
across the participating services and persisted evidence.

## 2. Fiducia / Servire managed PID lifecycle — CLOSED

Validate and, if still required, correct the Phase 9 managed-process
requirements:

- genuine running Fiducia instance blocks duplicate startup;
- stale/mismatched PID is removed/replaced once;
- clean Servire stop/start does not repeatedly report stale-PID
  recovery;
- PID/lock/transient runtime files are not treated as durable
  application data.

**Status update — 2026-09-20: CLOSED for M0.1.** The Fiducia/Servire
managed-process lifecycle, including stale-PID handling, was exercised
during the earlier Fiducia/Servire lifecycle development. The later
clean M0.1 installation and managed Stack startup also completed
normally. This document did not retain the original stale-PID test
evidence, but repeating that historical destructive test solely for
release-checklist closure is not required for M0.1.

## 3. Fiducia Clear Logs contract — CLOSED

Verify Servire **Clear Logs** against the current Fiducia CLI/service
package.

Required result:

- command succeeds;
- does not start a second Fiducia instance;
- does not corrupt/remove the live PID incorrectly;
- subsequent health and scheduling operation remain normal.

The historical evidence showed Servire invoking a `clear-logs` form that
Fiducia rejected with exit code 2.

**Status update — 2026-09-19: CLOSED.** The current Servire/Fiducia
Clear Logs contract has now been explicitly verified: Clear Logs
succeeds without starting a second Fiducia instance or damaging the live
PID/lifecycle, and subsequent service operation remains normal.

## 4. Explicit Replay prerequisite negative-path acceptance — DEFERRED POST-M0.1

Run the remaining controlled failure matrix against the current
Pontis/Praebere/Repetere implementation:

- source Trace missing model provenance;
- recorded model unavailable in Praebere catalogue;
- incompatible selected/reserved/locked model;
- model selection/reservation failure where practical to induce;
- activation/load failure where practical to induce;
- Vestigare recording contention/failure.

For each case verify:

- no Replay inference occurs when prerequisites fail;
- one failed/incomplete Run is preserved with a specific reason;
- no unintended active Pontis session, ACP binding, model execution or
  reservation remains;
- existing unrelated model/session state is unchanged;
- Experiment returns to `STAGED` after clean failure, or
  `RECOVERY_REQUIRED` only when cleanup genuinely cannot be confirmed.

**M0.1 disposition — 2026-09-20:** DEFERRED. This deliberate
negative-path failure-injection matrix is no longer a release blocker
for M0.1. It remains useful post-M0.1 hardening/acceptance work. The
M0.1 release decision instead relies on the accumulated Replay
integration evidence and the successful final clean-install baseline.

## 5. Recovery-path acceptance — DEFERRED POST-M0.1

Induce at least one genuine cleanup-boundary failure and verify:

- UI shows `Recovery Required` only for an actual failed cleanup
  boundary;
- another Run is blocked while recovery is required;
- guarded recovery is idempotent;
- recovery changes cleanup/reconciliation state only, not the immutable
  terminal scientific result/evidence;
- after recovery the Experiment becomes actionable again.

Normal cleanup/finalisation has already been exercised; this item is
specifically the genuine failure/recovery path.

**M0.1 disposition — 2026-09-20:** DEFERRED. Deliberately manufacturing
a cleanup-boundary failure solely to exercise `RECOVERY_REQUIRED` and
guarded recovery is not required for the M0.1 external research release.
Retain this as post-M0.1 resilience/hardening acceptance.

## 6. Final cross-service M0.1 acceptance matrix — CLOSED

Perform one clean-start final pass across the current packages for:

- source Trace model provenance;
- stage / unstage;
- manual Run;
- Run Again with a fresh session;
- matched Replay;
- divergent Replay;
- failed/incomplete Replay;
- child Trace correlation and inspection;
- child deletion and Experiment cleanup semantics;
- model release and Pontis session closure;
- Fiducia scheduled Run;
- no manual browser refresh required for active/terminal UI convergence.

Run component quality gates for every package changed in the final pass:

``` text
pytest
ruff check .
mypy src tests
```

Repetere and Fiducia have already had extensive green local runs during
this work; this is the final release-candidate matrix rather than a
request to reopen completed implementation.

**Status update — 2026-09-20: CLOSED for M0.1.** The matrix is accepted
on the accumulated integration evidence across Repetere, Fiducia,
Pontis, Praebere, Vestigare, Rogare and Servire together with the final
clean-install release-candidate baseline. The clean run removed prior
M0.1 Docker state, rebuilt the distribution, created a fresh
installation identity, completed registration and signed authorization,
and brought the managed Stack up successfully. Previously proven Replay
behaviours are not being reopened solely to reproduce historical
acceptance evidence.

# B. Post-M0.1 carry-forward

In addition to the future-development items below, the deliberate
failure-injection work retained in **A4** and **A5** is now part of
post-M0.1 hardening/acceptance and is not an M0.1 release blocker.

The following are intentionally **not blockers for M0.1**. They remain
future-development items from
`REVIEWED_DOCUMENTS_CARRY_FORWARD_NOT_IN_M0.1_ROADMAP.md`.

## Provider-neutral readiness

Implement the full provider-neutral readiness contract beyond the
concrete M0.1 Ollama
availability/residency/reservation/lock/reconciliation states.

## Per-session model selection and multi-model ownership

Move beyond the M0.1 runtime-global model to independent session-owned
model selection, including concurrent different-model residency,
reference counting, resource failure handling and safe independent
release.

## Rogare recovery and presentation

Add automatic discovery/reconnection to an existing Pontis session and
restore local conversation-history presentation after reconnect.

## Pontis shutdown quiescing

Add an admission/quiescing boundary so late external requests are
rejected cleanly while the stack is stopping. Continue to treat the
earlier shutdown hang as a watch item unless it recurs.

## Legacy traversal source hygiene

During a future Repetere/Moderari source pass, confirm obsolete
consume-or-forward traversal code is gone while retaining defensive
filtering that prevents control commands entering model context or
Replay input.

## Additional providers

Add providers beyond Ollama only through Praebere; do not distribute
provider-specific lifecycle/discovery logic into the other Lumen
services.

# M0.1 completion criterion

**M0.1 Replay acceptance is CLOSED as of 2026-09-20.**

Sections **A1, A2, A3 and A6** are closed on accumulated
implementation/integration evidence and the final clean-install
release-candidate baseline. Sections **A4 and A5** have been explicitly
moved out of the M0.1 release gate and retained as post-M0.1
hardening/acceptance work.

Section **B** remains post-M0.1 future development and does not block
M0.1.
