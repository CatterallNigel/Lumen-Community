# Lumen — Future Development Register

## Document Revision History

| Version | Date | Author | Description |
| --- | --- | --- | --- |
| 1.0 | 2026-08-23 | Nigel Catterall | First reviewed release |

**Status:** Living Document  
**Purpose:** Index of accepted future development that is intentionally outside the current release scope

## Purpose

This document is a concise register of future Lumen development already identified during architecture, research and release planning.

It is **not** a roadmap and should not duplicate detailed design documentation.

Each entry should identify the relevant area, point to the document where the requirement or design is described, and provide only enough description to understand why the item exists.

When a future release is planned, this register can be reviewed to decide which items should be promoted into that release's roadmap, requirements and acceptance criteria.

---

## Fiducia — Experiment-Scoped Moderari Execution Policy

**Area:** Fiducia / Moderari / Repetere / Nuntius  
**Document:** `Fiducia/FIDUCIA_EXPERIMENT_SCOPED_MODERARI_EXECUTION_POLICY.md`

Allow Fiducia to establish and positively acknowledge the required Moderari execution policy once for an Experiment, rather than requiring every individual Replay to re-establish the identical policy. The policy must be explicitly Experiment-scoped and have a safe release/expiry lifecycle.

## Runtime Authorization — Release and Update Availability

**Area:** Servire / Illuminates.One Authorization / Distribution  
**Document:** M0.1 runtime authorization architecture (future extension)

Allow the periodic runtime-authorization/heartbeat exchange to report that a newer Lumen release is available for the authorised installation. The future mechanism should distinguish an update that is merely available from one that is recommended or required, and may also indicate when updated licence terms apply.

The exact update-notification protocol, version policy, acceptance flow, upgrade mechanics, and enforcement behaviour are intentionally not defined yet.

## Moderari — Custom System Prompt Ownership and Scope

**Area:** Moderari / Saved System Prompts / Multi-User Configuration  
**Document:** M0.1 Development Requirements — Moderari Saved System Prompts / future extension

Extend saved Custom system prompts beyond the simple M0.1 ownership model to define user-private, organisation-shared, installation-wide and potentially organisation-mandated prompts. Future design must define ownership, visibility and permissions for creating, updating, deleting, sharing, mandating and overriding saved prompts without changing the M0.1 principle that the applied prompt is an explicit Moderari runtime condition.

## Repetere / Moderari — System-Prompt Substitution Experiments

**Area:** Repetere / Moderari / Vestigare / Fiducia / Aestimare  
**Document:** Future experiment-variant design

Allow a researcher to derive an experimental execution from an existing Trace while deliberately replacing the Trace's recorded effective system prompt with a different explicitly Applied Custom system prompt. The source Trace must remain immutable; the substitution is a declared experimental override, and the resulting Trace must record the actual substituted prompt used. This would allow comparison of behavioural deviation caused by changing the system-prompt condition while retaining the remaining recorded execution conditions as far as practicable.

## Repetere / Fiducia — Controlled Experimental Variants

**Area:** Repetere / Fiducia / Vestigare / Aestimare / Moderari / Praebere  
**Document:** Future controlled-experiment architecture

Extend Replay beyond exact reproduction so that a source Trace can become the baseline for controlled experimental variants in which one or more execution conditions are deliberately changed and explicitly recorded. Potential variables include system prompt, model/provider, tool availability or semantics, context treatment and bounded computational resources. Ordinary Replay must remain semantically distinct: **Replay reproduces; a Variant Experiment deliberately changes declared conditions.** This should be revisited as Lumen develops from replay and divergence observation toward a broader controlled experimental framework for behavioural assessment.

## Multi-Session — Independent Execution Conditions

**Area:** Moderari / Praebere / Session Management / Servire / Nuntius  
**Document:** M0.1 Development Requirements — Multi-Session Isolation Validation / future extension

Extend concurrent-session support beyond the M0.1 shared-execution-condition model so that individual active sessions may independently select and retain their own provider, model and Moderari system-prompt condition without affecting other sessions.

Future development must define how these execution conditions are scoped, bound to session identity, changed safely during session lifecycle, and released when a session ends. It should also define the required configuration ownership, locking or isolation semantics across Moderari, Praebere and the wider Lumen control path.

The M0.1 rule remains deliberately simpler: concurrent sessions are investigated for transaction and state isolation, while the first active session establishes the provider, model and Moderari system-prompt behaviour shared by all concurrent sessions.

