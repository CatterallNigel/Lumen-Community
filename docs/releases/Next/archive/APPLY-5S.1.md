# Apply corrected 5S.1 — Moderari local evidence; Vestigare SEP only

Supersedes the original 5S package. Internal Adapter, Filter and SpAd execution evidence is persisted exclusively by Moderari. Vestigare records client-side conversation plus the immutable Session Execution Plan (SEP).

1. Back up your deployment and retain local configuration.
2. Apply `Lumen-Moderari/` to the Moderari checkout. The default local database is `data/moderari-execution-evidence.sqlite3`; provide a writable persistent directory/volume. Keep the existing evidence database: earlier schemas remain readable and obsolete outbox columns are inert. The evidence sender, outbox API, retry setting and background worker have been removed. Restart Moderari to stop any earlier delivery worker.
3. If your Vestigare checkout needs the SEP receipt, apply `Vestigare-SEP-Companion/`. For newer checkouts merge `src/lumen_vestigare/session_plan.py` and the SEP handler in `api/control.py` without overwriting unrelated work. Preserve configuration. The full companion is based on 0.5.7.
4. If the erroneous original 5S companion was applied, remove its `execution_evidence.py` and corresponding tests/handler; use the corrected source here to restore the original `recording.py`. Restart Vestigare. There is no internal-evidence receiver or collection creation in this release. Historical internal-evidence copies/flags are not automatically deleted; back them up and remove them according to your retention decision.
5. Install each project's declared dependencies in its own environment. Verify registered buffered and streaming execution, disabled opaque transport, SEP acknowledgement and local persistence against the running deployment. Internal capture must produce no Vestigare execution-evidence messages. No Nuntius routing changes are needed. Live Windows/provider/Nuntius/Mongo acceptance has not been performed in this package environment.

The First-Ask SEP acknowledgement gate remains. Disabled Adapters continue opaque pass-through regardless of transport mode. Local evidence capture is independent of Vestigare recording state.

See `Lumen-Moderari/MILESTONE-5S-EXECUTION-EVIDENCE.md`, `MILESTONE-5-CLOSEOUT.md`, and `validation/5S1-*` for behavior, results and deployment limits.
