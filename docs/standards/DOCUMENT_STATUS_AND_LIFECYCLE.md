# Lumen Documentation Status and Lifecycle Convention

**Status:** Canonical  
**Version:** 1.0  
**Last reviewed:** 2026-08-21

---

## Purpose

Lumen preserves engineering history, but historical documents must not compete with current architecture for authority.

This convention defines a small set of document-status labels so that readers can distinguish current truth, supporting material, proposals, research, evidence and history without relying on filenames or dates alone.

---

## Required Status Labels

### Canonical

The authoritative current document for a significant engineering topic.

Rules:

- There should normally be only one canonical document for a defined topic.
- Architecture or responsibility changes are applied here first.
- Other current documents should link to or summarise the canonical document rather than redefine it.

Suggested header:

```text
**Status:** Canonical
```

### Current

Accurate supporting documentation for the current implementation, but not the single authority for the wider topic.

Examples include runtime guides, focused design details and current operational instructions.

Suggested header:

```text
**Status:** Current
```

### Historical

Accurately describes an earlier state of Lumen and is retained for provenance.

Historical documents must not be rewritten simply to match current architecture.

Suggested header:

```text
**Status:** Historical
```

Where useful:

```text
**Current reference:** <link to current/canonical document>
```

### Superseded

A document whose purpose has been replaced by a newer authoritative document.

Superseded does **not** mean incorrect for its time and does not imply deletion.

Suggested header:

```text
**Status:** Superseded
**Superseded by:** <relative link>
```

Every superseded document should also be recorded in the repository superseded-document register.

### Proposal

A proposed architecture, feature or change that has not been adopted as current implementation.

Suggested header:

```text
**Status:** Proposal
```

If adopted, it should either become Canonical/Current or remain as Historical with a link to the resulting document.

### Research

Exploration, hypotheses, experimental methodology or conceptual investigation. Research may influence architecture but does not itself establish implemented behaviour.

Suggested header:

```text
**Status:** Research
```

### Evidence

Captured engineering evidence such as test results, logs, screenshots or acceptance artefacts.

Evidence supports claims but is not itself the architectural authority.

Suggested header or directory classification:

```text
**Status:** Evidence
```

### Publication

Outward-facing explanatory material. Publications may distil canonical engineering sources but should not become the engineering source of truth.

Suggested header:

```text
**Status:** Publication
```

---

## Optional Maturity Qualifier

Document status and service maturity are different concerns.

For example:

```text
**Status:** Canonical
**Capability maturity:** Active development
```

This permits Aestimare to have a canonical architecture document while clearly stating that the implementation is still under development.

---

## Status Precedence

When documents disagree, use the following reading order:

```text
Canonical
  > Current
  > Proposal / Research
  > Historical / Superseded
```

Evidence does not override architecture by itself; it may demonstrate that canonical documentation needs to be updated.

Publications are explanatory outputs and do not override engineering documentation.

---

## Lifecycle

A common document lifecycle is:

```text
Research / Proposal
        |
        v
Current supporting design
        |
        v
Canonical architecture
        |
        +--------------------+
        |                    |
        v                    v
Historical              Superseded
```

Not every document follows every stage. Evidence and publications have separate purposes.

---

## When Architecture Changes

When implementation changes the current architecture:

1. Update the canonical document.
2. Update any current index/matrix affected by the change.
3. Decide whether an older document remains useful as Historical or should be marked Superseded.
4. Add any newly superseded relationship to `SUPERSEDED_DOCUMENT_REGISTER.md`.
5. Do not rewrite milestone/release evidence to make it look current.

---

## Filename Rules

Dates and version numbers remain useful, but they are not substitutes for explicit status.

A reader should be able to open a document and determine its authority without inferring from:

- the newest date;
- the highest version number;
- words such as `FINAL`, `UPDATED` or `NEW`;
- its current folder location alone.

---

## Repository Classification

The repository's folder structure expresses **purpose**, while this convention expresses **authority/state**.

Examples:

- `docs/services/moderari/LUMEN_MODERARI_ARCHITECTURE.md` may be **Canonical**.
- `docs/services/moderari/version-history/RELEASE-NOTES-v3.2.4.md` is **Historical**.
- `docs/research/...` is normally **Research**.
- `docs/evidence/...` is **Evidence**.
- `docs/publications/...` is **Publication**.

Folder placement and document status should reinforce each other, but status remains explicit for important engineering documents.

---

## Minimal Rule

> **One topic should have one discoverable canonical source, while Lumen's history remains preserved and clearly labelled.**
