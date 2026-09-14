# Lumen M0.1 Replay — Consolidated Sign-Off

**Date:** 2026-09-13  
**Status:** Implementation review complete; completed work signed off below  
**Scope reviewed:** all nine documents supplied in `development-in-progress-2026-09-12.zip`, reconciled against the implementation and live validation completed through Repetere 0.20.18 and Fiducia 0.7.3.

## Sign-off basis

This sign-off distinguishes **implemented/observed behaviour** from work that still requires an explicit acceptance test. A requirement is not marked complete merely because the supporting schema or UI exists. Remaining M0.1 acceptance work is moved to `LUMEN_M0.1_REPLAY_OUTSTANDING_2026-09-13.md`.

A small number of older requirements have been superseded by later agreed behaviour. In particular, the earlier absolute rule that Unstage never deletes an Experiment is now refined: an Experiment with no Runs is deleted on Unstage because it contains no experimental evidence; an Experiment with Run history is retained as `UNSTAGED`.

## 1. `LUMEN_M0.1_REPLAY_IMPLEMENTATION_SEQUENCE.md`

### Iteration 1 — Vestigare evidence and model provenance — **SIGNED OFF**

Authoritative recording-level model provenance is present. Replay child correlation supports source Trace, Experiment, Run and execution session identity. Vestigare remains owner of Trace recordings/messages.

### Iteration 2 — Experiment and Run persistence — **SIGNED OFF**

`experiments` and `experiment_runs` are the active persistence model. Stage/list/inspect/unstage, Run creation/history and correlation are implemented. Terminal Run evidence is retained. Deletion semantics have subsequently been strengthened: child deletion coordinates Vestigare evidence deletion with Repetere Run metadata, and an empty unstaged Experiment is removed.

### Iteration 3 — Fresh Pontis session per Run — **SIGNED OFF**

Replay Runs use Pontis-assigned `session-repetere-*` identities. The previous session-reuse/context-reconnection defect has been removed. Terminal cleanup closes the Replay execution session.

### Iteration 4 — Model validation/reservation/activation — **SIGNED OFF for implemented M0.1 path**

The recorded model is treated as the required model; Pontis/Praebere own model preparation and lifecycle. Repetere no longer intentionally begins Replay inference before the prepared Pontis session is active. Model lifecycle timeout support is in place.

The remaining live negative-path matrix (unavailable/conflicting/activation failure) is an acceptance item rather than missing implementation.

### Iteration 5 — Replay child Trace — **SIGNED OFF**

Each Run can create a separately correlated replay child Trace in Vestigare. The Trace Recordings UI presents the child beneath its source Trace, including after the Experiment is unstaged. Child deletion removes Vestigare Trace/messages first and only then Repetere Run metadata.

### Iteration 6 — Terminal results, cleanup and recovery — **SIGNED OFF for implementation**

Matched/divergent terminal results, child Trace finalisation, model/session cleanup, cleanup convergence and recovery representation are implemented. The transient `RECOVERY_REQUIRED` persistence lock is no longer incorrectly exposed by the UI as a real failure while cleanup is merely settling. Persisted `experiment_runs.lifecycle_state` now converges to `COMPLETED` when cleanup completes.

### Iteration 7 — Repetere UI hierarchy and polling — **SIGNED OFF**

The UI now provides source Trace → Run → child Trace hierarchy, model provenance, automatic polling, final authoritative refresh, separate Run result vs Experiment action state, Run Again handling, divergence evidence, cleanup state, consistent Trace inspection, and correct historical hierarchy after Unstage. Stale contradictory banners and the former `Operation failed: replay running` behaviour are resolved.

### Iteration 8 — Fiducia integration — **PARTIALLY SIGNED OFF**

Fiducia can now discover current Repetere staged Experiments using the current Experiment contract. Fiducia 0.7.3 is compatible with current Prepared Replay and Run response shapes; its local test suite was reported at 113 passed with Ruff and mypy clean after remediation.

Actual scheduled execution/terminal-result acceptance and the managed PID/Clear Logs items remain in the outstanding document.

### Iteration 9 — Cross-service/live acceptance — **NOT YET FULLY SIGNED OFF**

Substantial live acceptance has occurred, especially divergent Replay, cleanup, model release, session closure, child Trace persistence/deletion and UI convergence. The final explicit matrix remains outstanding.

## 2. `LUMEN_M0.1_REPLAY_RUNTIME_ARTIFACTS_2026-09-07.md`

**Disposition: CLOSED / SIGNED OFF.**

The three recorded artifacts have been addressed by the subsequent design:

- incorrect running/failure banner — fixed;
- Replay session reuse/context inheritance — fixed through fresh Pontis session ownership and terminal close;
- model activation outside Praebere lifecycle — prevented on the intended Replay path by Pontis/Praebere preparation before Replay execution.

The third item remains useful historical evidence of why the preparation boundary exists; it does not require a separate M0.1 Moderari investigation.

## 3. `LUMEN_REPETERE_CONSOLIDATED_UI_UPDATE_REQUIREMENTS.md`

**Disposition: SIGNED OFF.**

The consolidated UI requirements are implemented to the current agreed behaviour, including:

- Experiment / Run / child Trace hierarchy;
- compact coloured status presentation rather than verbose lifecycle pills;
- separate Experiment action state and Run terminal result;
- immediate active/pending state on Run Again;
- automatic polling and final authoritative refresh;
- Trace Recordings refresh and current model display;
- matched/divergent/failed presentation;
- first-divergence expected/observed evidence;
- recovery/finalising distinction;
- Trace inspector navigation and consistent source/child inspection;
- persistence of historical child hierarchy after Unstage.

Later deletion work extends this document: deleting a child Replay Trace removes its Vestigare evidence first, then its Run metadata and parent Run reference; deleting an Experiment is explicitly destructive and separate from Unstage.

## 4. `LUMEN_REPETERE_MATCHED_AND_DIVERGENT_REPLAY_BEHAVIOUR.md`

**Disposition: SIGNED OFF for the implemented Replay runtime.**

The definitive behaviour is now reflected in the runtime: Repetere compares privately until divergence; the divergent request is represented once in the child evidence; after divergence the live path is handed through Vestigare/Pontis and continues to conversation completion; Pontis/end-turn remains the completion authority; cleanup finalises the child Trace, closes the execution session and releases model ownership.

Divergence is retained as a behavioural result rather than an infrastructure error.

## 5. `LUMEN_REPETERE_REPLAY_STAGE_ONE_SESSION_PREPARATION.md`

**Disposition: SIGNED OFF for implementation.**

The core ownership model is in place:

- Pontis assigns the Replay session identity;
- Pontis owns ACP/session preparation and Praebere lifecycle operations;
- Repetere owns Experiment/Run state and starts Vestigare only after preparation;
- Nuntius carries the control path with the long model-lifecycle timeout;
- terminal cleanup closes the Pontis session and releases model lifecycle state.

The remaining explicit induced-failure cases from the manual acceptance section are carried forward as final acceptance tests.

## 6. `LUMEN_REPETERE_UI_UPDATE_REQUIREMENTS.md`

**Disposition: SUPERSEDED BY THE CONSOLIDATED UI DOCUMENT; IMPLEMENTATION SIGNED OFF.**

Its requirements are represented in the later consolidated UI specification and have been implemented. Retain this document as design history rather than as an open checklist.

## 7. `M0.1_TRACE_MODEL_REPLAY_REQUIREMENT.md`

**Disposition: IMPLEMENTATION SIGNED OFF; FINAL NEGATIVE-PATH ACCEPTANCE REMAINS.**

The core requirement is satisfied: source Trace model provenance exists, Experiment/Run/child relationships exist, each Run receives independent execution identity/evidence, and current Replay uses the recorded model through the Pontis/Praebere lifecycle rather than silently substituting another model.

Fiducia now understands the current Repetere Experiment/Run contract. Scheduled terminal-result validation and explicit missing/unavailable/conflicting-model live tests remain acceptance items.

## 8. `REPETERE_CHILD_OF_STAGED_REPLAY_UI_AND_STATE_CLEANUP.md`

**Disposition: REPLAY/REPETERE PORTION SIGNED OFF; FIDUCIA/SERVIRE ACCEPTANCE REMAINS.**

The Experiment/Run/child-Trace domain model, state separation, Run isolation, Replay prerequisites, failed-cleanup representation, hierarchy, polling, Run Again behaviour and evidence retention are implemented.

### Reconciled Unstage rule

The document's original statement that Unstage always retains the Experiment is superseded by the later agreed evidence rule:

- if `run_ids` is non-empty, Unstage retains the Experiment as `UNSTAGED` and preserves all Run/child evidence;
- if `run_ids` is empty, Unstage deletes the Experiment document because no experiment has actually been performed;
- if the final child Run is later deleted from an already `UNSTAGED` Experiment, the now-empty Experiment document is also deleted.

This is consistent with the evidence-preservation objective while avoiding empty historical Experiment records.

## 9. `REVIEWED_DOCUMENTS_CARRY_FORWARD_NOT_IN_M0.1_ROADMAP.md`

**Disposition: REVIEWED; REMAINS A VALID FUTURE-DEVELOPMENT RECORD.**

These items are deliberately outside M0.1 and therefore do not prevent M0.1 Replay sign-off:

- provider-neutral readiness beyond the current M0.1 provider boundary;
- per-session independent model selection / concurrent multi-model residency;
- Rogare automatic session reconnection and history restoration;
- Pontis shutdown quiescing;
- legacy traversal source-hygiene review;
- providers beyond Ollama through Praebere.

They are reproduced separately in the outstanding document under **Post-M0.1 carry-forward**, so this development-in-progress directory no longer needs to be treated as an undifferentiated open checklist.

## Additional work completed after the supplied documents

The following later work is also signed off because it materially closes or strengthens requirements in these documents:

- safe child Replay deletion across Vestigare and Repetere ownership boundaries;
- explicit destructive **Delete Experiment** separate from **Unstage Experiment**;
- original source Trace retained when deleting an Experiment;
- empty Experiment cleanup on Unstage;
- automatic deletion of an empty `UNSTAGED` Experiment after its final child Run is deleted;
- historical child hierarchy retained in Trace Recordings after Unstage;
- cleanup lifecycle persistence convergence (`CLEANUP_REQUIRED` → `COMPLETED`);
- Fiducia compatibility with current Repetere Prepared Replay and Run contracts.

## Overall sign-off

The **vast majority of the M0.1 Replay implementation is complete**. Repetere's Experiment/Run/child-Trace model, runtime behaviour, cleanup semantics, UI and evidence-management work can be treated as implemented rather than development-in-progress.

The remaining work is predominantly **acceptance and Fiducia/Servire managed-lifecycle validation**, not another Repetere redesign. Those items are isolated in the companion outstanding document.
