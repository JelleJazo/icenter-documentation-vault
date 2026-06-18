---
type: meta
title: "Hot Cache"
updated: 2026-06-18T00:00:00
---

# Recent Context

## Last Updated
2026-06-18. Phase 3c-2: ICenterLib/ISAH foundation entities landed.

## Key Recent Facts
- Scope: three projects (iCENTER 1237 + TruTopsLib 65 + ICenterLib 723 = **2025 files**).
- Coverage now: **67 done, 9 needs-review, 1090 todo, 445 config, 414 generated**.
- 147 open SME questions (Q-001, Q-006, Q-031, Q-034 resolved). **55 are `#safety-relevant`**.
- 16 business-rule notes (10 safety-relevant).
- ICenterLib coverage: 11/723 done — root files + 8 foundation ISAH entities.

## ISAH foundation summary (this batch)
- **[[mocs/icenterlib-isah]]** — sub-MOC catalogues all 65 ISAH files with size + role + recommended doc order (DossierMain/Detail next, then ProductionHeader/PBOO/PBOM/PBOS, then BillOfMat/BillOfOper/Part, then TimeRegistration, then Icenter2Isah).
- **[[modules/isah-lookups]]** — `Selection` (universal `T_Selection` code-with-description accessor via `IP_sel_SelectionRecord`) and `Setting` (single-method stub for `IP_get_Setting`).
- **[[modules/isah-identity]]** — Employee + User + Customer + Vendor + Company. `Employee` 17 KB, instance-based, two parallel SQL paths (`UseIsahNoDtrTimeReg` toggle — DTR-status path is dead code, Q-132). `User` is purely static, exposes 3 bidirectional accessors (EmpId↔UserCode↔WindowsLogin) + `GetJobDescription` reading `T_MemoDetail` filtered on `MemoTypeCode='JEM10' AND LangCode='NL'` (Q-134, hard-coded Dutch-only). `Company` hard-codes JAZO/`041` and FlowGrill/`042` (no SQL — pure model).
- **[[modules/isah-shop-and-pur-doc]]** — `ShopDoc` (T_ShopDoc, 8 KB) + `PurDoc` (T_PurchaseDocument, 2 KB). **Critical finding**: ShopDoc state actually lives on `T_ProdBillOfOper`, not `T_ShopDoc` — `SetShopDocStartedInd` delegates to `PBOO.SetOperStartInd`, and `GetShopDocFinInd` joins to PBOO (Q-136). `SetShopDocStartedInd(started)` ignores its argument — only ever marks started (Q-142).
- **[[modules/isah-machgrp]]** — `MachGrp` (T_MachGrp). Obsolete-detection via `Description LIKE '~%'` (Q-135). `IsTrackOperation` regex `TR\d\d` ([[business-rules/isah-track-operation-pattern]]). Regex recompiled per-access (Q-144).
- **2 new business-rule notes**: [[business-rules/isah-company-codes]] (JAZO `041` vs FlowGrill `042`), [[business-rules/isah-track-operation-pattern]] (`TR\d\d`).

## Recent Changes
- Created [[mocs/icenterlib-isah]] sub-MOC for ISAH (65 files catalogued).
- Created 4 module notes covering 10 ISAH entities (Selection, Setting, Employee, User, Company, ShopDoc, PurDoc, MachGrp) — Customer + Vendor flagged `needs-review` for follow-up.
- Created 2 new business-rule notes.
- Opened Q-132..Q-147 (16 new questions, 1 `#safety-relevant`: Q-133 — `Employee.GetIsObsolete` fail-closed).
- Updated [[_coverage]] (+8 done, +2 needs-review in ICenterLib).

## Active Threads
- Recommended next batches:
  1. **ISAH dossier hierarchy**: DossierMain (19 KB), DossierDetail (28 KB), DossierDetailExtra (2 KB) + Helpers/DossierDetailExtraHelper.
  2. **ISAH production hierarchy**: ProductionHeader (45 KB), PBOO (31 KB), PBOM (11 KB), PBOS (11 KB), BillOfOper (9 KB), BillOfMat (26 KB).
  3. **ISAH Part + dispatch**: Part (49 KB), PartDispatch (8 KB), PartDispatchCollectorDataService (8 KB), PartVendor, PartSelection.
  4. **ISAH TimeRegistration deep-dive** (72 KB) — biggest single ISAH file. Likely will need its own batch.
  5. **ISAH Icenter2Isah** (38 KB) — the sync layer to iCenter2.
- After ISAH saturates, move to ICenterLib/iCenter (29 files: ProductionMachines, IPPart, IPBatch, IPOrder).

## Notes from working tree
- Three Obsidian auto-stubs at wiki root (`jiba-portal.md`, `kardex.md`, `trutops-oseon.md`) and `.obsidian/` autoupdates remain unstaged.
- `architecture/external-surface.md` user-reformatted (table layout changed) — kept as-is.
