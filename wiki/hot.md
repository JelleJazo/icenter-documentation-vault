---
type: meta
title: "Hot Cache"
updated: 2026-06-18T00:00:00
---

# Recent Context

## Last Updated
2026-06-18. Phase 3c-4: ISAH production hierarchy landed.

## Key Recent Facts
- Scope: three projects (iCENTER 1237 + TruTopsLib 65 + ICenterLib 723 = **2025 files**).
- Coverage now: **79 done, 9 needs-review, 1078 todo, 445 config, 414 generated**.
- 167 open SME questions (Q-001, Q-006, Q-031, Q-034 resolved). **57 are `#safety-relevant`**.
- 18 business-rule notes (10 safety-relevant).
- ICenterLib coverage: 23/723 done (3 root + 10 ISAH foundation + 6 dossier + 6 production hierarchy).
- ISAH coverage: 22/65 done (one-third of the ISAH wrapper now documented).

## ISAH production hierarchy summary (this batch)
- **6 files in [[modules/isah-production-hierarchy]]** as a single grouped note: ProductionHeader (817 lines), PBOO (585), PBOM (200), PBOS (132), BillOfOper (192), BillOfMat (483).
- **Hierarchy**: ProductionHeader (T_ProductionHeader, `PD\d{8}` id) → PBOO (per-operation lines, line-numbers go 5/10/15…) + PBOM (per-material) + PBOS (per-surcharge). BillOfOper + BillOfMat are *template-side*, per-Part, separate from the per-job PBOO/PBOM.
- **Closes the chain**: dossier-detail → `GenProductionHeader` → ProductionHeader → PBOO → ShopDoc. State writes for "started/finished" land on T_ProdBillOfOper (confirms Q-136).
- **`PD\d{8}` ProductionHeader format** documented as [[business-rules/isah-prod-header-format]] — 2-char `PD` prefix + 8 digits.
- **ShopDoc status code `"20"` = "started"** hardcoded in `ProductionHeader.SetOperStarted` (Q-160).
- **`PBOS.IP_Ins_ProdBOS` uses inline T-SQL string-concat** (~60 lines of declared T-SQL variables before calling the SP) instead of a parametrised SP call — looks like a transliteration of an ISAH SP-reference snippet (Q-158).
- **BillOfMat.ApplySurfTreatment** shares an undocumented DataTable schema contract with [[modules/workprep-operation-substitution|`OperationSubstitutionHandler`]] — both mutate `DtObjects`, `dtSurfTreatmentPart`, `dtSurfTreatmentOper`, `dtOperTotal` with `MachSetupTime_<code>`, `MonoMachCycleTime_<code>` etc. columns (Q-163).
- **PBOO and PBOM both have giant commented-out 30-parameter insert functions** — documenting the SP signatures but unused, because iCenter relies on ISAH's `IP_gen_ProdHeadForDosDet` to populate them as a side effect.

## Recent Changes
- Created [[modules/isah-production-hierarchy]] grouped module note (6 files).
- Created [[business-rules/isah-prod-header-format]] business-rule note.
- Opened Q-158..Q-167 (10 new questions, 0 `#safety-relevant`).
- Updated [[_coverage]] (+6 done in ICenterLib/ISAH); rollup totals.
- Updated [[business-rules/_index]], [[needs-review/_index]], [[mocs/_index]].

## Active Threads
- Recommended next ISAH batches:
  1. **ISAH Part + dispatch** (49 + 8 + 8 + 1 + 3 = ~70 KB): Part.vb (49 KB — the biggest non-TimeRegistration ISAH file), PartDispatch, PartDispatchCollectorDataService, PartVendor, PartSelection. Plus `DataServices/PartDataService.vb` (4 KB) and `DataServices/PartDispatchCollectorDataService.vb` (7 KB).
  2. **ISAH TimeRegistration deep-dive** (72 KB) — biggest single ISAH file. Likely needs its own batch. Adjacent: `TimeRegCollector.vb` (7 KB), `TimeRegistration/` folder (2 files).
  3. **ISAH Icenter2Isah** (38 KB) — the sync layer between iCenter2 (out-of-scope sibling) and ISAH. Cross-cutting findings expected.
  4. **ISAH small leaves**: Design (7 KB), CallRegistration (5 KB), FrmJConfigParamDesignCode (8 KB) + JConfigParam (8 KB), Database (3 KB), DateDimension (3 KB), MultiFinance (2 KB), WorkView (2 KB), Helpers/* (~5 files).

## Notes from working tree
- Three Obsidian auto-stubs at wiki root (`jiba-portal.md`, `kardex.md`, `trutops-oseon.md`) and `.obsidian/` autoupdates remain unstaged.
- `architecture/external-surface.md` user-reformatted (table layout changed) — kept as-is.
