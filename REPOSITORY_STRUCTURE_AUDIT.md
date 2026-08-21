# Lumen Community Repository — Structure Audit

## Scope

This audit reorganises folders only. Existing document contents are unchanged. New README files were added solely for navigation.

## Main structural problems found

1. **Root overload** — roadmaps, publications, history, architecture and brand material were mixed at repository root.
2. **Service documentation split in two places** — current service documents lived under `docs/lumen-ecosystem/`, while milestone/release history lived separately under `docs/lumen-family-version-notes/`.
3. **Research mixed with architecture and vision** in the top-level `docs/` directory.
4. **Test evidence mixed with architectural documentation** under `docs/lumen-ecosystem/test-results/`.
5. **Historical and current roadmaps mixed together**, making the active development direction harder to identify.
6. **Rendered digest assets mixed with source documentation**.
7. **Sparse/empty version-note folders** existed for Aestimare and Fiducia.
8. **One exact duplicate document** exists: `LUMEN_OFFLINE_CLEAR_LOGS_STANDARD.md` appears both as an ecosystem standard and under Rogare version history.
9. Several likely superseded pairs remain and should be reviewed before deletion or archival:
   - `EVOLUTION_OF_LIMEN_THINKING-2026-08-03.md` vs `EVOLUTION_OF_LIMEN_THINKING_ENHANCED-2026-08-03.md`
   - `LUMEN_PROJECT_STATUS_AND_ROADMAP_2026-08-10.md` vs `..._UPDATED.md`
   - `VISION-UI.md` vs `VISION-UI-v2.md`
   - `LUMEN-CONSOLE.md` vs `LUMEN-CONSOLE-v2.md`
   - `ARCHITECTURE-APPENDUM-2.md` and `LUMEN-CONSOLE.md` contain substantial overlapping material and deserve manual review.

## Organising principle used

```text
README.MD
docs/
  README.md
  architecture/
    core/
    ecosystem/
    extensions/
  services/
    README.md
    aestimare/
    audire/
    fiducia/
    moderari/
    pontis/
    praebere/
    repetere/
    rogare/
    servire/
    vestigare/
  research/
  roadmaps/
    archive/
    component-plans/
  engineering/
    ENGINEERING_DIARY.md
    history/
    digests/
  standards/
  vision/
  evidence/
    screenshots/
    test-results/
  publications/
  brand/
  site/
  archive/
examples/
```

## Why service release history was moved beside each service

The previous split between `lumen-ecosystem/<service>` and `lumen-family-version-notes/Lumen-<Service>` made a reader know two separate repository conventions before finding all material about one component.

The proposed structure makes the service the primary navigation unit:

```text
docs/services/pontis/
  current architecture and design
  README.md
  version-history/
```

This preserves history without placing historical milestone material in the main reading path.

## Missing or useful documentation

These are recommendations, not evidence that the engineering itself is missing.

### High priority

- **Repository documentation index (`docs/README.md`)** — added in this organised copy.
- **Service-level README indexes** — added in this organised copy.
- **Current Lumen ecosystem architecture / topology document** — **added in this organised copy** as `docs/architecture/ecosystem/LUMEN_ECOSYSTEM_CURRENT_ARCHITECTURE.md`.
- **Canonical service matrix** — **added in this organised copy** as `docs/architecture/ecosystem/LUMEN_SERVICE_MATRIX.md`.
- **Documentation lifecycle/status convention** — **added in this organised copy** as `docs/standards/DOCUMENT_STATUS_AND_LIFECYCLE.md`.
- **Superseded-document register** — **added in this organised copy** as `docs/standards/SUPERSEDED_DOCUMENT_REGISTER.md`; unresolved overlap remains explicitly queued for manual classification.

### Medium priority

- **Integration contracts overview** — especially as Moderari/Pontis/Praebere increasingly support external capability providers. The new third-party architecture gives the principle; a future contract index can list actual implemented interfaces.
- **Evidence/provenance guide** — explain what belongs under `evidence/`, what should be retained, and how test logs/screenshots correlate to milestones or Diary entries.
- **Glossary / naming index** — particularly useful because engineering names, Latin service names, historical English names, and concepts such as Trace/Replay/Assess coexist.
- **Current status page** — a short, frequently updated page pointing at the current roadmap and stating what is implemented, active, deferred and research-only.
- **Contribution/development entry point** — once external contributions become appropriate, a `CONTRIBUTING.md` and developer setup guide will be needed.

## Document-status / overlap pass — completed 2026-08-21

The five overlap areas identified by this audit have now been manually reviewed.

Resolved outcomes:

- the unenhanced `EVOLUTION_OF_LUMEN_THINKING-2026-08-03.md` was deleted as a redundant earlier version;
- the enhanced Evolution document is retained as **Historical**;
- the earlier 2026-08-10 roadmap was deleted; the updated roadmap is retained as **Current**;
- both UI Vision documents are retained as **Historical**;
- both Lumen Console documents are retained as **Historical**;
- the original Architecture plus Appendices 1 and 2 are retained under `docs/architecture/core/history/` as **Historical**;
- a new canonical `docs/architecture/core/ARCHITECTURE.md` now describes the solidified current Lumen architecture.

The superseded-document register records these decisions.

## Recommendation

Adopt the proposed folder structure first, then do a second pass concerned only with **document status and overlap**. Do not combine structural reorganisation with content consolidation. That keeps the Git change easy to understand and protects the historical/provenance value of the repository.
