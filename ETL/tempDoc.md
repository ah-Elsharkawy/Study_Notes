## 1) File / SP map (one-line responsibilities)

- [Upload_OMAP_ETL_Mapping_Service.py](vscode-file://vscode-app/d:/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) — GP tool to extract Excel sheets into a temp geodatabase, then upload to target gdb / file tables.
- [SP_ETL_Mapping_Tables_Preparation.sql](vscode-file://vscode-app/d:/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) — converts uploaded raw sheets into canonical `fieldsMapping` and `valuesMapping` tables.
- [SP_ETL_FastChange_Duqm.sql](vscode-file://vscode-app/d:/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) — Fast-change pipeline for Duqm zone (job calls this SP).
- [SP_ETL_FastChange_Plots_Other_Than_Duqm.sql](vscode-file://vscode-app/d:/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) — Fast-change pipeline for other zones.
- [SP_ETL_Plot_Pre_Sync.sql](vscode-file://vscode-app/d:/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) — Prepares destination plot tables (adds RowHash/meta fields, creates archives, statistics, views).
- [SP_ETL_FC_Sync_Entity.sql](vscode-file://vscode-app/d:/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) — Sync engine used by fast-change to sync one entity (calls core sync logic).
- [SP_ETL_CppLotWGS84_Cpplotpoint_Sync_Delete.sql](vscode-file://vscode-app/d:/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) — Sync deletes on destination geometry/point replicas.
- [projectionScript.py](vscode-file://vscode-app/d:/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) / [SP_ETL_Plot_Projection.sql](vscode-file://vscode-app/d:/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) — Projection pipeline (project cpplot39/40 → WGS84; Python used to assist).
- [SP_ETL_Plot_Post_Sync.sql](vscode-file://vscode-app/d:/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) — Finalize sync to `cpplotPoint`, recalc statistics.
- [SP_ETL_SC_Prepare_Job.sql](vscode-file://vscode-app/d:/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) — Prepares views and trims mapping values before slow-change run.
- [SP_ETL_SC_Run_Job.sql](vscode-file://vscode-app/d:/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) — Loop over mapping combinations and call sync for each (wrapper for slow-change).
- [TRGR_ETL_JOBS_After_Insert.sql](vscode-file://vscode-app/d:/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) — Trigger that inserts into ETL_JOBS then starts job `ETL_SC_RUN_NotExecuted_Job`.
- [ETL_SC_Run_NotExecuted_Job.sql](vscode-file://vscode-app/d:/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) — SQL Agent job to execute `SP_ETL_SC_Run_Job` decoupled from trigger transaction.
- [SP_ETL_SC_Sync_Entity.sql](vscode-file://vscode-app/d:/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) — Main slow-change sync procedure with validations, dynamic SQL, rowhash compare, logging.
- `fieldsMapping` / `valuesMapping` — Canonical mapping tables used by the ETL logic.
- [fn_ExtractDigitsAsFloat.sql](vscode-file://vscode-app/d:/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) — helper to parse numbers from strings in mappings.
- `fn_Get_*` — mapping helper functions (value/field lookups, codes).
- `ETL_Jobs`, `ETL_Logs` — control and logging tables.

---

## 2) Polished developer-focused documentation draft

You can place this as `Docs/ETL_DeveloperGuide.md` or [README.md](vscode-file://vscode-app/d:/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) in the repo.

## ETL Developer Guide — OMAP_ETL (draft)

### Purpose

This ETL synchronizes data from 6 source systems (separate schemas/DBs) into a single unified schema in the `OMAP` database. It supports near-real-time "FastChange" updates (every ~10s) and ad-hoc or batched "SlowChange" runs initiated by the ETL UI or scheduled job.

### High-level architecture

- Mapping: data-team provides an Excel workbook (4 sheets). The upload tool extracts those sheets and the `SP_ETL_Mapping_Tables_Preparation` stored procedure converts them into canonical `fieldsMapping` and `valuesMapping` tables in `OMAP_ETL`.
- FastChange: SQL Agent jobs run stored procedures every 10s to capture changes from source and push them to `cpplot39/cpplot40/cpplotWGS84`.
- SlowChange: Trigger or scheduled job inserts into `ETL_Jobs`; a separate SQL Agent job picks up the job and runs `SP_ETL_SC_Run_Job` which loops over requested mapping combos and calls `SP_ETL_SC_Sync_Entity`.

(Diagram suggestion: Mapping flow → fieldsMapping/valuesMapping → FastChange/SlowChange → cpplot tables → projection → app)

### Mapping overview

- Excel sheets:
    - Object Classes (subtypes)
    - Fields (schema mapping)
    - Domains
    - Domains | Subtypes | Values (value mappings)
- Canonical runtime tables:
    - `fieldsMapping` — mapping at field/entity level used by sync procedures.
    - `valuesMapping` — domain and value mappings.
- Example (conceptual):
    - fieldsMapping row: { sourceDB: "Sohar", sourceEntity: "plots", sourceField: "area", destEntity: "cpplot", destField: "AREA_M2", mappingFunction: "fn_ExtractDigitsAsFloat" }

### FastChange (near real-time)

Purpose: capture and apply source edits quickly.

- Jobs:
    - `FastChange_Plot_Duqm_Job` → calls `SP_ETL_FastChange_Duqm`.
    - `FastChange_Plots_Other_Than_Duqm_Job` → calls `SP_ETL_FastChange_Plots_Other_Than_Duqm`.
- Why two jobs: ensure each zonal pipeline completes within 10s; Duqm ~5s worst-case, others ~8s worst-case.
- Typical flow (per zone):
    1. Verify primary replica (AG) before running.
    2. `SP_ETL_Plot_Pre_Sync` — prepare destination fields & archives.
    3. `SP_ETL_FC_Sync_Entity` — perform sync (select, compute RowHash, compare, delete/insert).
    4. `SP_ETL_CppLotWGS84_Cpplotpoint_Sync_Delete` — handle deletions.
    5. `SP_ETL_Plot_Projection` → [projectionScript.py](vscode-file://vscode-app/d:/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) — project to WGS84 and load `cpplotWGS84`.
    6. `SP_ETL_Plot_Post_Sync` — load into `cpplotPoint` & update statistics.

FastChange contract (example)

- Input: zone param (Duqm or other), internal mapping tables preloaded.
- Output: updated `cpplot` tables and logs in `ETL_Logs`.
- Failure mode: job logs to `ETL_Logs`; if dynamic SQL fails, fast-change should handle/exit gracefully.

### SlowChange (bulk / on-demand)

Purpose: full/batched synchronization for entities; triggered via UI or scheduled job.

- Initiation: insert to `OMAP_ETL.dbo.ETL_Jobs` with optional filter params (DestEntity, SourceDB, SourceEntityGroup).
- Trigger `TRGR_ETL_JOBS_After_Insert` starts job `ETL_SC_RUN_NotExecuted_Job` → runs `SP_ETL_SC_Run_Job`.
- `SP_ETL_SC_Run_Job`:
    - Calls `SP_ETL_SC_Prepare_Job` (creates versioned views, trims mapping table values).
    - Builds distinct set of mapping combinations and loops; calls `SP_ETL_SC_Sync_Entity` per combination.
- `SP_ETL_SC_Sync_Entity`:
    - Validation phase (mapping existence, duplicates, fields exist in dest, one whereCondition, primary table exists, etc.)
    - Destination preparation (add RowHash, source_globalID, source_objectid, LastSyncAt, sourceEntityGroup).
    - Build dynamic SQL to fetch source data into global temp table `##source_table`.
    - Compute RowHash deterministically.
    - Compare RowHash and synchronize (delete changed rows, insert new/updated).
    - Log outcome to `ETL_Logs`.

SlowChange contract (summary)

- Inputs: @SourceDB, @SourceEntityGroup, @DestEntity, @JobID (see `SP_ETL_SC_Run_Job` for loop handling).
- Outputs: ETL_Logs entry, destination table changes.
- Failure handling: SPs have TRY/CATCH, rollback on error inside `SP_ETL_SC_Sync_Entity`; `SP_ETL_SC_Run_Job` captures errors and logs them without aborting whole run.

### Validation rules

- Mapping must exist for the SourceDB/EntityGroup/DestEntity.
- No duplicate destField mappings from multiple source fields (1.3).
- Destination table must contain all mapped destFields (1.5).
- Exactly one or zero whereCondition per sourceEntityGroup.
- Must have one primary table for sourceEntityGroup.
- Refer to `Mapping_Rules.pdf` for full rules and examples.

### Important implementation notes & gotchas

- `RowHash` must be computed with a deterministic concatenation order — otherwise comparisons break.
- Global temp table `##source_table` is used because dynamic SQL is executed; it must be created with the correct schema to survive dynamic context.
- `fn_ExtractDigitsAsFloat` is used to extract numeric parts of strings — watch edge cases and locale issues.
- The slow-change stored procedure runs dynamic SQL inside a transaction — errors will rollback the transaction inside the SP. That’s why the trigger starts a SQL Agent job to decouple it.
- Projection uses Python ([projectionScript.py](vscode-file://vscode-app/d:/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html)) — make sure the server's Python environment and permissions exist for the job.
- Performance: some SPs can be long-running — consider job timeouts, progress logging, and breaking very large jobs into chunks.

### How to run / test (runbook)

- Upload mapping:
    1. Place mapping Excel in upload location.
    2. Run [Upload_OMAP_ETL_Mapping_Service.py](vscode-file://vscode-app/d:/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) (geoprocessing tool) or use UI “Upload mapping” tab.
    3. Confirm `fieldsMapping` and `valuesMapping` updated.
    4. Run `EXEC OMAP_ETL.dbo.SP_ETL_Mapping_Tables_Preparation;` to finalize mapping.
- Run a SlowChange job manually:
    1. Insert into `OMAP_ETL.dbo.ETL_Jobs` with desired filters.
    2. Start the job `ETL_SC_RUN_NotExecuted_Job` manually from SQL Agent if needed.
- Run a FastChange job manually:
    1. Run the respective stored procedure: `EXEC OMAP_ETL.dbo.SP_ETL_FastChange_Duqm` or `SP_ETL_FastChange_Plots_Other_Than_Duqm` on the primary node.

### Troubleshooting checklist

- If a slow-change job fails:
    - Check `ETL_Logs` for the JobID.
    - Inspect error messages printed by SP — often due to missing mapping fields or destination schema drift.
    - Validate that `fieldsMapping.destField` exists in destination table columns.
- If projection fails:
    - Verify Python is installed on server and accessible by SQL Agent.
    - Check [projectionScript.py](vscode-file://vscode-app/d:/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) logs.
- If RowHash mismatches:
    - Ensure mapping order and concatenation haven't changed.
    - Verify data types and trimming logic.

### Tests & validation to add (recommended)

- Unit test harness for mapping-to-SQL generation (small TSQL tests that assert expected SELECT creation from a mock fieldsMapping).
- Integration test: sample mapping file + small source dataset → run slow-change with JobID against a test database, assert expected rows.
- Health check job: simple query to check mappings are consistent (no duplicates, missing dest fields).

### Next steps / recommended repo improvements (priority)

1. Add a single `ETL_DeveloperGuide.md` with the structure above and the file/SP map. (low-risk)
2. Add a small ER diagram (fieldsMapping, valuesMapping, ETL_Jobs, ETL_Logs). (visual)
3. Add a sequence diagram showing SlowChange flow (insert → trigger → job → SP → sync). (visual)
4. Add SP signatures and single-line contracts to the doc (inputs/outputs/errors). (developer ergonomics)
5. Add sample mapping Excel and a minimal integration test runner (SQL script that loads sample data and runs one sync). (higher effort)
6. Add query snippets for common debugging (check missing mapping rows, view current RowHash mismatches). (low-risk)
7. Add linter and SQL formatting rules to repo (optional).

---

## 3) Concrete improvements I can make now (choose any)

- Produce the polished `ETL_DeveloperGuide.md` file and add to [Docs](vscode-file://vscode-app/d:/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) in repo.
- Create the file/SP map table inside that doc.
- Create a small "Runbook" snippet of T-SQL queries to inspect and debug `ETL_Jobs` and `ETL_Logs`.
- Create a simple SQL test script that inserts a sample ETL_Jobs row and shows expected logs (non-destructive) — or an example mapping row.