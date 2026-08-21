# Lumen Superseded Document Register

**Status:** Canonical  
**Purpose:** Record superseded relationships and resolved overlap decisions  
**Last reviewed:** 2026-08-21

---

## How to Use This Register

A document belongs here when an overlap or superseding relationship has been deliberately reviewed.

Deletion is appropriate only when a document is genuinely an earlier redundant version and contributes no independent historical provenance. Historical documents are retained when they record a meaningful state in Lumen's evolution.

---

## Resolved Relationships

| Document / set | Resolution | Notes |
|---|---|---|
| `EVOLUTION_OF_LUMEN_THINKING-2026-08-03.md` | **Deleted** | Earlier form of the enhanced document; no independent provenance required. |
| `EVOLUTION_OF_LUMEN_THINKING_ENHANCED-2026-08-03.md` | **Historical** | Preserves the state of Lumen thinking on 2026-08-03. |
| `LUMEN_PROJECT_STATUS_AND_ROADMAP_2026-08-10.md` | **Deleted** | Earlier progression of the updated roadmap. |
| `LUMEN_PROJECT_STATUS_AND_ROADMAP_2026-08-10_UPDATED.md` | **Current** | Retained as the presently useful dated roadmap/status record. |
| `VISION-UI.md` | **Historical** | Records an earlier UI vision; the operational architecture subsequently evolved into Servire/Rogare. |
| `VISION-UI-v2.md` | **Historical** | Records the later form of the earlier UI vision. |
| `LUMEN-CONSOLE.md` | **Historical** | Records the earlier unified-console architectural direction. |
| `LUMEN-CONSOLE-v2.md` | **Historical** | Records the evolved unified-console direction; current responsibilities are now separated across Servire/Rogare and the wider service family. |
| Original `ARCHITECTURE.md` | **Historical** | Moved to `docs/architecture/core/history/ARCHITECTURE-2026-08-HISTORICAL.md`. |
| `ARCHITECTURE-APPENDUM.md` | **Historical** | Extension of the original architecture; moved under `docs/architecture/core/history/`. |
| `ARCHITECTURE-APPENDUM-2.md` | **Historical** | Further extension of the original architecture; moved under `docs/architecture/core/history/`. |
| `docs/services/rogare/version-history/LUMEN_OFFLINE_CLEAR_LOGS_STANDARD.md` | **Redundant duplicate** | Canonical standard remains `docs/standards/LUMEN_OFFLINE_CLEAR_LOGS_STANDARD.md`. |

---

## New Canonical Architecture

[`../architecture/core/ARCHITECTURE.md`](../architecture/core/ARCHITECTURE.md) is now the canonical architectural description of Lumen.

It is intentionally distinct from:

- [`../architecture/ecosystem/LUMEN_ECOSYSTEM_CURRENT_ARCHITECTURE.md`](../architecture/ecosystem/LUMEN_ECOSYSTEM_CURRENT_ARCHITECTURE.md) — current topology, service relationships and flows.
- [`../architecture/ecosystem/LUMEN_SERVICE_MATRIX.md`](../architecture/ecosystem/LUMEN_SERVICE_MATRIX.md) — concise service responsibility, dependency, port and maturity index.

The historical architecture sequence is retained as provenance rather than treated as current authority.

---

## Register Maintenance Rule

Whenever future overlap is resolved:

1. Decide whether the older material is redundant, Historical or genuinely Superseded.
2. Preserve historical material when it records an independently useful state of Lumen.
3. Add explicit status to retained important documents.
4. Add the decision here.
5. Keep one discoverable canonical source for each significant current topic.
