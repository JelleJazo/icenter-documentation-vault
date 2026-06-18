---
type: meta
title: "Hot Cache"
updated: 2026-06-18T00:00:00
---

# Recent Context

## Last Updated
2026-06-18. Phase 3b-1: Office → shop-floor handoff batch landed (Sales / Engineering / WorkPreparation / Production).

## Key Recent Facts
- Scope: three projects (iCENTER 1237 + TruTopsLib 65 + ICenterLib 723 = **2025 files**).
- Coverage now: **56 done, 7 needs-review, 1103 todo, 445 config, 414 generated**.
- 106 open SME questions (Q-001, Q-006, Q-031, Q-034 resolved). **42 are `#safety-relevant`**.
- 10 business-rule notes (9 safety-relevant).
- Elumatec subsystem stopped at 31/157 done per user direction; now pivoted to office-to-shop-floor.
- Coverage in this batch:
  - **Sales**: 1/1 done.
  - **Engineering**: 8 done + 1 needs-review out of 9 .vb non-Designer files.
  - **WorkPreparation**: 4/4 done.
  - **Production** (root + ProfileMilling): 6/6 done.

## Office → shop-floor summary (this batch)
- **Sub-MOC** at [[mocs/office-to-shopfloor]] catalogues the full handoff: ISAH order → Sales team → Engineering owner → WorkPreparation (outsource / op-substitution) → Production (per-machine cut-items + UniLink CSV import).
- **Sales/FrmCustomerTeam** assigns customers to teams `031/032/033` (hard-coded) via ISAH `T_Customer`.
- **WorkPreparation/IPBatchCollector** is a single SQL query against ISAH `T_ProductionHeader + JZ_ProdRefNr + T_ProdBillOfOper`. Only surfaces rows with a `JZ_ProdRefNr` entry (Q-099).
- **WorkPreparation/OutsourceOperationsHandler** is the big one (~540 lines): packages STEP+PDF docs per vendor, creates ISAH PurDoc via `PurOrd.CreatePurOrdByExtOperParts("UITBESTEDING01", ...)`, drops zip into `IsahDoc\Purchase\<PurOrdNr>\01\`, marks ShopDoc as started + IPparts as completed. Hard-coded: `UITBESTEDING01`, `AutoSendEmail=False`, `MICROSOFTPRINTTOPDF` sticker printer, only `ProfileId=1` implemented.
- **WorkPreparation/OperationSubstitutionHandler** swaps `<From>` for `<To>` machine-group across DataTables. **Likely bug found** (Q-095, safety-relevant): `SubstituteInSurfTreatmentPart` iterates `dtSurfTreatmentOper` and vice versa — swapped sources.
- **Production/ProductionProfileCutItemsHandler** implements only 3 iCenter operations (1→A01,A07; 9→S01; 31→A07,A01). Operations 1 and 31 have **same set, reversed order** (Q-106). Constructor `MachineId = Math.Max(iPPartId, 0)` looks like a copy-paste error (Q-101).
- **Production/ProfileMilling** (5 files) is the **UniLink CSV import pipeline** — iCenter → CAM counterpart to the Elumatec output. Validates every profile has `Series` filled AND a matching DXF in UniLink, else throws. `PMMExportHandler` wipes all `<PMMEXPORT3D>` XML attributes when updating (Q-103).
- **Engineering**: 9 files. Notable: `FrmDrwCheck` embeds PDF-XChange ActiveX with the license key from `app.config`; `FrmDesignCodeTool` is a WebView2 wrapper around `tekeningnummers.jazo.com` with JavaScript injection (brittle, Q-090); `ModelCopies/CopyLocalizer` has 1 active sub-class and 1 dead (`UitsparingVoorplaatMeerpslAlu`, Q-089).

## Recent Changes
- Created [[mocs/office-to-shopfloor]] sub-MOC.
- Created 7 module notes: [[modules/sales-customer-team]], [[modules/workprep-outsource-operations]], [[modules/workprep-operation-substitution]], [[modules/workprep-ipbatch-collector]], [[modules/production-profile-cut-items]], [[modules/production-profile-milling-import]], [[modules/engineering-overview]].
- Created 3 business-rule notes: [[business-rules/sales-team-codes]], [[business-rules/outsource-ext-oper-part-code]], [[business-rules/icenter-operation-machgrp-mapping]].
- Opened Q-082..Q-106 (25 new questions, 11 `#safety-relevant`).
- Updated [[_coverage]] (+20 done in 4 folders); rollup totals.
- Updated [[mocs/_index]], [[business-rules/_index]], [[needs-review/_index]].

## Notable findings (probable bugs)
- **Q-095** — `OperationSubstitutionHandler`: `*Part` iterates `dtSurfTreatmentOper` and `*Oper` iterates `dtSurfTreatmentPart`. Either harmless (both tables share the column) or a real bug (each updates the wrong table). `#safety-relevant`
- **Q-101** — `ProductionProfileCutItemsHandler.New`: `MachineId = Math.Max(iPPartId, 0)` — looks like `iPPartId` was meant to be `machineId`. `#safety-relevant`
- **Q-100** — `ProductionProfileCutItemsHandler.Write`: `Clear(MachineId)` runs unconditionally; a model-lookup failure wipes the machine's previous cut-items. `#safety-relevant`

## Active Threads
- Recommended next batches:
  1. **CadBatchserver** (27 files) — the headless mode + 16 `Job*.vb` classes (JobAutoManufacturing, JobCreateProdOrd, JobPublishCreo, JobSendEmail, JobRebootMonitor, etc.). Each `Job*` is a candidate business rule. This is the scheduled-work side of iCenter.
  2. **Classes/** (111 files) — core domain classes. Needs a sub-MOC due to size. Subfolders: Coating/, Connectivity/, PreSelectMachGrpCodes/, Production/, StickersAndLabels/, Toolbox/.
  3. **Forms/** (127 .vb non-Designer) — the dialog gallery, including the ShopProcess sub-folder.
  4. **SmtManufacturing** (63 files) — sheet metal subsystem.
  5. **Companion projects** — ICenterLib (723 files) and TruTopsLib (65 files) are in scope but completely untouched. ISAH/JIBA/CAD/SmtProduction plumbing all lives there.

## Notes from working tree
- Three Obsidian auto-stubs at wiki root (`jiba-portal.md`, `kardex.md`, `trutops-oseon.md`) and `.obsidian/` autoupdates remain unstaged.
- `architecture/external-surface.md` was reformatted by the user / a linter (table markdown changed) — leaving as-is per intent.
