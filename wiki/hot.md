---
type: meta
title: "Hot Cache"
updated: 2026-06-18T00:00:00
---

# Recent Context

## Last Updated
2026-06-18. Phase 3c-5: ISAH Part + dispatch landed.

## Key Recent Facts
- Scope: three projects (iCENTER 1237 + TruTopsLib 65 + ICenterLib 723 = **2025 files**).
- Coverage now: **86 done, 9 needs-review, 1071 todo, 445 config, 414 generated**.
- 178 open SME questions (Q-001, Q-006, Q-031, Q-034 resolved). **58 are `#safety-relevant`**.
- 20 business-rule notes (11 safety-relevant).
- ICenterLib coverage: 30/723 done. ISAH coverage: **29/65 (45%)**.

## ISAH Part + dispatch summary (this batch)
- **7 files in [[modules/isah-part-and-dispatch]]** as a single grouped note: Part (49 KB), PartDispatch (8 KB), PartDispatchCollectorDataService (8 KB root + 7 KB DataServices), PartDataService (4 KB), PartSelection (3 KB), PartVendor (1 KB).
- **Critical finding Q-168**: `PartDispatchCollectorDataService` exists in **two files** with the *same class name* in different namespaces (`ISAH` and `ISAH.DataServices`). Both share the query but have **divergent `CalculateJobDone`** — root version adds a `PickedQty < InvtQty AND JobDoneInd = False` cross-check and uses *its* count, DataServices version just uses `MaxJobDoneInd`. **Same input → different output** depending on which version the caller picked up. Q-172, `#safety-relevant`.
- **Q-169**: `@MachGrpCode` parameter is bound on the dispatch-collector query but **never referenced** in the SQL — hardcoded `'M38'` is the real filter.
- **Q-170**: `Part.GetPartField(name)` is the standard read pattern — one round-trip per field. A form displaying 10 Part fields runs 10 queries. No caching.
- **[[business-rules/isah-partdispatch-collector-filters]]** (`#safety-relevant`) — five hardcoded filters: `MachGrpCode='M38'`, `WarehouseCode LIKE 'KD%'`, `DispatchWarehouseCode <> 'ALG'`, `ProdStatusCode='40'`, `FinishedInd=0`. JAZO's dispatch-collector is single-MachGrp by design.
- **[[business-rules/isah-partdispatch-process-status]]** — 3-state Dutch-named enum `NietVerwerken / KlaarVoorVerwerken / Verwerkt`. No transition validation.
- **Coating sentinels**: `PartCode LIKE 'VPR-%'` = paint primer; `VPR-INVULLEN` = manual-entry placeholder; `SelectionCode '018'` = "use dark primer"; `'024'`/`'025'` = paint categories.

## Recent Changes
- Created [[modules/isah-part-and-dispatch]] grouped module note (7 files).
- Created 2 new business-rule notes (1 safety-relevant).
- Opened Q-168..Q-178 (11 new questions, 1 `#safety-relevant`: Q-172).
- Updated [[_coverage]] (+7 done in ICenterLib/ISAH); rollup totals.

## Active Threads
- Recommended next ISAH batches:
  1. **TimeRegistration** (72 KB) — biggest single ISAH file. Likely its own batch. Adjacent: `TimeRegCollector.vb`, `TimeRegistration/` folder.
  2. **Icenter2Isah** (38 KB) — sync layer between iCenter2 (out-of-scope sibling) and ISAH.
  3. **ISAH small leaves** — Design, CallRegistration, FrmJConfigParamDesignCode + JConfigParam, CustomerSelection, CustomerRelation, Database, DateDimension, MultiFinance, WorkView, IsahFieldML, DeliveryLine, PurchaseDocumentPartLine, Language, DataServices/* (remaining 6), Helpers/* (5), ViewModels/* (2).
- After ISAH saturates, move to ICenterLib/iCenter (29 files).

## Notes from working tree
- Three Obsidian auto-stubs at wiki root (`jiba-portal.md`, `kardex.md`, `trutops-oseon.md`) and `.obsidian/` autoupdates remain unstaged.
- `architecture/external-surface.md` user-reformatted (table layout changed) — kept as-is.
