# Temporary N9 / Praebere Reviewed-Documents Closeout Requirements

**Status:** Closed — reviewed document chain reconciled; retain only as closeout evidence  
**Date:** 2026-09-05  
**Purpose:** Single closeout checklist for the N9/Praebere/Nuntius documents reviewed on 2026-09-04.  
**Disposition:** Archive after the canonical roadmap and carried-forward work records are committed.

---

## 1. Scope

This document consolidates only the remaining work needed to close the reviewed
N9/Praebere/Nuntius document chain. It does **not** replace the main
`LUMEN_EXTERNAL_RESEARCH_DISTRIBUTION_M0.1_ROADMAP`.

Reviewed sources:

- `N9.1_PRAEBERE_CURRENT_STATE_AUDIT.md`
- `NUNTIUS_CONTROL_PLANE_MIGRATION_PROJECT.md`
- `M0.1_ROADMAP_ADDENDUM_N9_PLUS_PRAEBERE_REQUIREMENTS.md`
- `PONTIS_SESSION_LIVENESS_AND_PRAEBERE_MODEL_LOCK_RELEASE.md`
- `PRAEBERE_N9_6_RUNTIME_READINESS_RECONCILIATION_MINI_ROADMAP.md`
- `PRAEBERE_M0.1_SERVICE_UI_REQUIREMENTS.md`

The following are deliberately carried back to the main M0.1 roadmap and are **not**
N9 closeout blockers:

- Repetere/Fiducia Experiment work;
- nested-Trace / Replay-fidelity experiment work beyond the evidence required to
  prove N9 provider/model selection;
- broader Experiment/failed-run presentation work;
- additional Rogare UI improvements not required to prove the N9 lifecycle.

The full **N9.6.4 Provider-Neutral Readiness Contract** is also explicitly
**nice-to-have/post-M0.1** and is not a release blocker.

---

# 2. Current Reconciled Position

| Area | Current position |
|---|---|
| N9.1 Praebere audit | Complete; historical baseline, no further implementation work |
| N9.2 Native control contract | Complete |
| N9.3 Provider discovery | Complete |
| N9.4 Model discovery | Complete for M0.1 |
| N9.5 Model selection / locking | Complete and live-validated |
| N9.6.1 MongoDB persistence | Complete |
| N9.6.2 Pontis reconciliation | Complete and live-validated |
| N9.6.3 External Ollama / demand residency | Complete and live-validated |
| N9.6.4 Provider-neutral readiness | Deferred to Future Development; not an M0.1 blocker |
| N9.7 Compact Praebere operational UI | Next required M0.1 work |
| N9.7 execution-evidence acceptance | Mostly proven; final UI-associated evidence pass remains |
| N9.8 Associated service validation | Partial; complete after N9.7 UI integration |
| N9.9 Repetere/Fiducia follow-on | Carried to main M0.1 roadmap |
| N9.10 Quality gates | Final clean pass required |
| Nuntius N8 diagnostics/UI | Still marked open in its project document; verify current implementation and close remaining gaps |
| Nuntius N10 compatibility/regression | Still open in its project document |

---

# 3. Completed N9.6.3 Bug-Fix Work

Complete these before declaring the runtime lifecycle finished:

- [x] Correct the known N9.6.3 defects exposed by the 2026-09-04 live lifecycle pass.
- [x] Resolve the Rogare-visible `500` failure path seen during provider/model lifecycle testing.
- [x] Resolve authoritative provider/capability presentation remaining `Not ready` after
  otherwise successful model execution, where still reproducible.
- [x] Confirm Rogare/Pontis displays the authoritative selected/reserved model rather
  than stale configured/default model text.
- [x] Re-run the exact failing paths after each fix and confirm they do not introduce
  model-selection, session-release or residency regressions.

These symptoms should be treated as acceptance defects until the final lifecycle pass
shows they are either fixed or proven to be non-defects with the documents corrected
accordingly.

---

# 4. Completed N9.6.2 Reconciliation Evidence

The final live acceptance pass proved:

- [x] Praebere restart reloads persisted runtime state.
- [x] Persisted active execution registrations enter fail-safe reconciliation rather
  than silently unlocking.
- [x] Praebere obtains authoritative active execution state from Pontis.
- [x] Stale Praebere registrations are removed only after Pontis confirms they are no
  longer active.
- [x] Pontis-active registrations missing from Praebere state are restored.
- [x] Selection changes remain blocked while reconciliation is pending.
- [x] Failed reconciliation remains explicit and retryable.
- [x] Deferred session releases are processed automatically after Praebere returns.
- [x] Reconciliation reaches a terminal complete state without needing an unrelated
  subsequent client request.
- [x] A managed full-stack stop intentionally ends sessions and persists clean runtime
  state.
- [x] A Praebere-only restart does not erase still-valid active-session state.

---

# 5. Completed N9.6.3 External-Ollama / Residency Validation

The superseding runtime rule is:

> **Ollama is external infrastructure. Lumen checks its configured endpoint but never
> starts or stops the Ollama process. Praebere may own only the model residency that
> Praebere itself caused.**

Live validation confirmed:

- [x] Praebere verifies the configured Ollama endpoint during lifecycle startup.
- [x] Ollama unavailable causes the applicable startup to fail explicitly.
- [x] Servire rolls back only services started by that failed stack-start attempt.
- [x] Lumen does not attempt to start Ollama when the endpoint is unavailable.
- [x] Normal Lumen/Praebere stop does not stop the external Ollama process.
- [x] Selecting a model records authoritative model intent without loading it.
- [x] First ask checks model residency.
- [x] First ask loads the selected model only if it is not already resident.
- [x] Praebere records residency ownership only when it issued the load.
- [x] A model resident before Lumen execution remains externally/pre-existing resident.
- [x] Concurrent sessions using the same reserved model share residency safely.
- [x] Ending one of several execution sessions does not unload the shared model.
- [x] Ending the final execution session unloads the model only when Praebere caused
  residency.
- [x] A failed owned-model unload is reported as degraded/partial completion and does
  not cause Ollama process termination.
- [x] Stack shutdown leaves no active Pontis/Praebere execution registrations.

---

# 6. N9.7 Compact Praebere Operational UI — Required M0.1 Work

Implement the compact operator surface without waiting for N9.6.4.

## 6.1 Servire Integration

- [x] Add Praebere as its own Servire service tab in the established ordering.
- [x] Embed the Praebere-owned UI.
- [x] Provide **Open directly** where the existing service-owned UI pattern supports it.
- [x] Do not duplicate Servire's normal Start/Stop/Restart controls in the Praebere UI.
- [x] Keep successful UI polling/refresh noise out of the normal Servire Operational Log.

## 6.2 Minimum Overview

Show authoritative:

- [x] Ollama endpoint/provider availability.
- [x] Provider lifecycle boundary as **External infrastructure**.
- [x] Selected runtime-global model.
- [x] Selection state: available / reserved / locked / reconciliation pending.
- [x] Reservation count and reserving Pontis session identities.
- [x] Active execution-session count.
- [x] Preferred model where useful.
- [x] Model residency state.
- [x] Model residency ownership where known:
  `Praebere-loaded`, `Pre-existing/external`, or `Not resident`.
- [x] Persistence/reconciliation failure or stale-state indication.

## 6.3 Model Table

Show at least:

- [x] provider-reported model identity;
- [x] available;
- [x] resident;
- [x] preferred;
- [x] selected;
- [x] select/current/disabled action state.

Do **not** make the full provider-neutral `warmable/ready/degraded` taxonomy mandatory.
If N9.6.4 is implemented later, extend the table without redefining the concrete states
above.

## 6.4 Active Executions

- [x] Show Pontis `session_id`.
- [x] Show known origin/type.
- [x] Show effective runtime-global model.
- [x] Show activation time where available.
- [x] Show active/reconciliation state.
- [x] Distinguish an open `not active` Pontis session from an active model-execution
  session; `not active` must not be presented as meaning closed or model-available.
- [x] Explain reservation separately from the active-execution lock.
- [x] Explain why selection is locked.
- [x] Do not make Praebere the normal session-termination authority.
- [x] Link or direct the operator to Pontis for ordinary session cleanup where useful.

## 6.5 Bounded Actions

Implement and validate:

- [x] Refresh Models.
- [x] Select Model.
- [x] Reconcile with Pontis.
- [x] Safe Reset Orphaned Execution State with confirmation and proper guards.

Any forced recovery action remains separately authorised and conspicuous.

## 6.6 UI Acceptance

- [x] Embedded and direct-open surfaces show the same authoritative state.
- [x] Model changes made through Rogare or external `\obt` become visible without a
  manual page reload.
- [x] Selection actions are disabled while execution is locked.
- [x] Ending the final active execution makes selection available without manual repair.
- [x] Praebere restart visibly enters/exits reconciliation correctly.
- [x] Pontis unavailable during recovery remains visibly fail-safe.
- [x] Refresh Models does not silently change selection.
- [x] Safe reset is rejected while live sessions exist.
- [x] Failed actions produce concise operator-facing errors.
- [x] UI/control traffic stays outside model context and Vestigare conversational Trace.

---

# 7. N9.7 Execution-Evidence Acceptance

These are acceptance checks for the N9.7 UI milestone, not a separate numbered
development milestone. Already-proven model selection/execution behaviour should
receive one final evidence pass:

- [x] Confirm the authoritative selected model is the model actually used.
- [x] Confirm effective provider/model identity appears in the required execution evidence.
- [x] Confirm Nuntius/`\obt` control traffic remains outside model context.
- [x] Confirm Nuntius/`\obt` control traffic remains outside Vestigare conversational Trace.
- [x] Confirm system-prompt provenance remains unchanged.
- [x] Confirm concurrent sessions use the same runtime-global reserved model while
  retaining their independent session-scoped Moderari prompt policies.
- [x] Confirm selecting a different model after all reservations/sessions are released
  produces visibly different provider/model evidence.

The deeper Repetere/Fiducia Experiment and nested-Trace work remains on the main M0.1
roadmap and is not required here.

---

# 8. N9.8 Remaining Associated-Service Validation

The core Rogare discovery/selection, Moderari authoritative model use, and external
client query/selection path are already treated as proven.

Still close explicitly:

- [x] Rogare reconnect/restart state recovery.
- [x] Nuntius diagnostic correlation for a representative Praebere request.
- [x] Servire response-target/listener behaviour for Praebere operations.
- [x] Praebere UI integration and validation from Section 6.
- [x] Final external-client query/selection/end-session smoke pass.
- [x] Final Rogare query/selection/end-session smoke pass.

Additional Rogare UI improvements should be recorded against the main M0.1 roadmap
rather than expanding this N9 closeout.

---

# 9. Nuntius Project Document — Remaining Closeout

The reviewed Nuntius migration project still records two open phases beyond the
Praebere native-adoption proof.

## 9.1 N8 — Diagnostics and UI

Verify current implementation first; then complete/check off anything genuinely
remaining:

- [x] request lifecycle records;
- [x] origin/session/request correlation;
- [x] resolved owner;
- [x] terminal result;
- [x] elapsed time;
- [x] timeout/unconfirmed outcome;
- [x] late response;
- [x] routing/configuration errors;
- [x] health;
- [x] Servire connectivity;
- [x] active route count;
- [x] in-flight count;
- [x] bounded recent history;
- [x] Servire-exposed Nuntius diagnostics view.

Do not reimplement items already present in the current Nuntius service merely because
the older project document still has unchecked boxes; reconcile the document against
the running implementation first.

## 9.2 N10 — Compatibility Removal and Regression Validation

After N9 is closed and migrated paths are proven:

- [x] identify obsolete legacy consume-or-forward traversal code;
- [x] retain defensive model-context filtering in Repetere/Moderari;
- [x] remove obsolete command traversal paths;
- [x] run cross-service regressions;
- [x] verify `\obt rogare start` remains Pontis-local;
- [x] verify no accidental command loops;
- [x] verify no control command enters model context;
- [x] verify session/request correlation;
- [x] verify success is never inferred from silence;
- [x] verify ordinary ask/answer paths remain unchanged.

The Nuntius project can then mark its completion-definition item for legacy traversal
as satisfied.

---

# 10. N9.10 / Final Quality Gates

For every service modified during N9 closeout:

- [x] `pytest`
- [x] required coverage threshold
- [x] `ruff check .`
- [x] `mypy src tests`
- [x] service startup/lifecycle smoke test
- [x] live cross-service acceptance test

At minimum expect this to cover whichever of the following are modified during closeout:

- Praebere;
- Pontis;
- Rogare;
- Servire;
- Nuntius;
- Moderari, if touched by a corrective change.

No document should be marked technically complete while a modified-service quality
gate is known to be failing.

---

# 11. Documentation Closeout

After the code/acceptance work above is complete:

- [x] Update the canonical N9/Praebere architecture documentation to describe the native
  Nuntius control path.
- [x] Ensure all documents use the superseding external-Ollama boundary consistently.
- [x] Remove/rewrite any remaining requirement that implies Praebere starts/stops Ollama.
- [x] Ensure provider **process** ownership is not confused with Praebere-owned **model
  residency**.
- [x] Mark N9.6.4 consistently as nice-to-have/post-M0.1.
- [x] Reconcile the final Praebere UI acceptance state.
- [x] Reconcile the Nuntius migration project status and completion definition.
- [x] Update the main M0.1 roadmap with N9 completion and carry-over work.
- [x] Update the main M0.1 requirements/limitations document where the reconciled runtime
  boundary changes or clarifies an existing requirement.
- [x] Preserve canonical filenames and use revision-history entries rather than
  `_UPDATED`/`_RECONCILED` suffixes in repository copies.
- [x] Remove this temporary consolidation document once its remaining items have been
  transferred to the canonical roadmap/checklists.

---

# 12. Explicitly Deferred / Not N9 Blockers

These items must remain visible, but should not prevent N9 closure:

- [x] **N9.6.4 Provider-Neutral Readiness Contract** — post-M0.1.
- [x] Repetere/Fiducia Experiment work — main M0.1 roadmap.
- [x] Nested-Trace / Replay experiment work — main M0.1 roadmap.
- [x] Broader Experiment/failed-run UI work — main M0.1 roadmap.
- [x] Additional Rogare UI polish beyond the N9 acceptance surface — main M0.1 roadmap.
- [x] Pontis shutdown quiescing/admission guard for late external requests during stack
  teardown — Future Development.
- [x] Future provider adapters beyond M0.1 Ollama — future roadmap.

---

# 13. N9 Close Decision

N9 may be declared complete when all of the following are true:

- [x] known N9.6.3 bugs are fixed;
- [x] N9.6.2/N9.6.3 final lifecycle validation passes;
- [x] the compact Praebere operational UI is implemented and accepted;
- [x] N9.7 execution/provenance acceptance evidence is captured;
- [x] remaining N9.8 cross-service acceptance checks pass;
- [x] N9.10 quality gates are clean;
- [x] the Nuntius/Praebere canonical documents are reconciled to the final runtime
  boundary; and
- [x] deferred Repetere/Fiducia/Rogare work is clearly present in the main M0.1 roadmap
  rather than left ambiguously inside N9.

At that point development can move cleanly back to the main External Research
Distribution M0.1 roadmap and the distribution/security phases, including Phase 11
Runtime Authorization and Distribution Security.

---

# 14. Final Closeout — 2026-09-06

The close decision is satisfied for the bounded N9/Nuntius document chain:

- [x] N8 diagnostics and Servire UI were reconciled against the running service,
  including a live bounded-history check using a temporary history limit of three.
- [x] N9.1–N9.5 and N9.6.1–N9.6.3 are complete.
- [x] The compact Praebere-owned UI and Servire integration are implemented and
  accepted.
- [x] Selected/reserved/locked model state, demand residency, activation failure,
  session release, persistence and Pontis reconciliation are live-validated.
- [x] Rogare reconnect recovery is accepted for M0.1 through manual re-entry of the
  Pontis session ID. Context is retained; Rogare's prior local conversation display
  is not restored.
- [x] Nuntius request correlation, response targeting/listeners, health, routes,
  in-flight state, errors, timing and bounded history are validated.
- [x] External-client and Rogare query/selection/ask/end-session smoke paths pass.
- [x] N10 live compatibility checks pass: no loops, no control leakage, correlation
  remains intact, silence is not success and ordinary ask/answer behaviour is unchanged.
- [x] External `\obt rogare start` is rejected locally by Pontis with HTTP 422 and
  does not enter Nuntius; this internal command is not part of the M0.1 researcher
  interface.
- [x] System-prompt provenance remains intact: the incoming Pi prompt is retained as
  superseded provenance and exactly one Lumen-generated effective prompt is replay input.
- [x] The main M0.1 roadmap now carries the remaining Vestigare/Repetere/Fiducia
  trace-model fidelity work.
- [x] N9.6.4 and other future-development items remain explicitly deferred.

Unchecked historical boxes in preceding sections are superseded by this final
reconciliation. They do not reopen N8 or N9. Genuine remaining work has been moved to
the main roadmap or to the separate reviewed-documents carry-forward record.

**Final decision:** N9 and the reviewed Nuntius/Praebere document chain are closed.
Development returns to the canonical
`LUMEN_EXTERNAL_RESEARCH_DISTRIBUTION_M0.1_ROADMAP`.
