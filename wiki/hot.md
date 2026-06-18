---
type: meta
title: "Hot Cache"
updated: 2026-06-18T00:00:00
---

# Recent Context

## Last Updated
2026-06-18. Phase 3c-3: ISAH dossier hierarchy landed.

## Key Recent Facts
- Scope: three projects (iCENTER 1237 + TruTopsLib 65 + ICenterLib 723 = **2025 files**).
- Coverage now: **73 done, 9 needs-review, 1084 todo, 445 config, 414 generated**.
- 157 open SME questions (Q-001, Q-006, Q-031, Q-034 resolved). **56 are `#safety-relevant`**.
- 17 business-rule notes (10 safety-relevant).
- ICenterLib coverage: 17/723 done (3 root files + 8 ISAH foundation entities + 6 dossier entities).

## ISAH dossier hierarchy summary (this batch)
- **6 files documented in [[modules/isah-dossier]]** as a single grouped note: DossierMain, DossierDetail, DossierDetailExtra, DossierDetailExtraDto, DossierDocFolder, Helpers/DossierDetailExtraHelper.
- **Hierarchy**: `T_DossierMain` (header: OrdNr/QuotNr/CustId/DelDate/OrdType) → `T_DossierDetail` (per-line, keyed by `(DossierCode, DetailCode, DetailSubCode)`) → `ST_0099_DossierDetailExtra` (per-line extra-info view; the `ST_` prefix is unknown).
- **`DossierType { Unknown, Quote, Order }`** — distinguishes a quote from an order via OrdNr/QuotNr columns.
- **`DossierMain.GetCompany()`** returns a [[modules/isah-identity|`Company`]] (JAZO or FlowGrill) via `MultiFinance.GetAdminCodeByDossierCode`.
- **`DossierDetail.GenProductionHeader()`** creates a new ProductionHeader from the detail line via SP `IP_gen_ProdHeadForDosDet` — with 14 hard-coded-zero parameters (the SP supports custom routing/status flags iCenter doesn't use).
- **`DossierDetailExtraHelper.GetUrl(useTestDb)`** builds an encrypted URL to a JAZO ISAH web app for editing dossier-extra info. **Encryption password is base64-encoded in `app.config` `DossierDetailExtraPassword`** (Q-154, `#safety-relevant` — key + ciphertext in same file).
- **`DossierDocFolder.GetIsahDocSalesFolder(key)`** resolves `<IsahDocRoot>Verkoop\<Order|Offerte>\<year>\<key>` for sales-document folder lookup.
- **`PlanIntMonPartCode = "PLAN INT MONT"`** and **`PlanTekWvBPartCode = "PLAN TEK WVB"`** exposed as constants. `GetMontDetailCode` filters on the hard-coded set `{'MONTAGE TP', '090', 'PLAN EXT MONT', 'CSA00011'}`.
- **`GetYearFromQuotOrdNr`** has a Y2100 bug: `"20" + value.Substring(0, 2)` (Q-148, low priority).
- **`PartCode NOT LIKE 'CA-%' AND NOT LIKE 'CH-%'`** filter in `GetDesignCodes` (Q-149 — unknown prefix meanings).

## Recent Changes
- Created [[modules/isah-dossier]] grouped module note (6 files).
- Created [[business-rules/isah-dossier-mount-partcodes]] business-rule note.
- Opened Q-148..Q-157 (10 new questions, 1 `#safety-relevant`: Q-154 — DossierDetailExtraPassword in app.config).
- Updated [[_coverage]] (+6 done in ICenterLib/ISAH); rollup totals.
- Updated [[business-rules/_index]], [[needs-review/_index]], [[mocs/_index]].

## Active Threads
- Recommended next ISAH batches:
  1. **ISAH production hierarchy**: ProductionHeader (45 KB), PBOO (31 KB), PBOM (11 KB), PBOS (11 KB), BillOfOper (9 KB), BillOfMat (26 KB). This closes the `DossierDetail.GenProductionHeader → ProductionHeader → PBOO → ShopDoc` chain.
  2. **ISAH Part + dispatch**: Part (49 KB — the biggest non-TimeRegistration ISAH file), PartDispatch (8 KB), PartDispatchCollectorDataService (8 KB), PartVendor, PartSelection.
  3. **ISAH TimeRegistration deep-dive** (72 KB) — likely its own batch.
  4. **ISAH Icenter2Isah** (38 KB) — sync layer.
- After ISAH saturates, move to ICenterLib/iCenter (29 files: ProductionMachines 41 KB, IPPart 19 KB, IPBatch 13 KB, IPOrder 7 KB, Client).

## Notes from working tree
- Three Obsidian auto-stubs at wiki root (`jiba-portal.md`, `kardex.md`, `trutops-oseon.md`) and `.obsidian/` autoupdates remain unstaged.
- `architecture/external-surface.md` user-reformatted (table layout changed) — kept as-is.
