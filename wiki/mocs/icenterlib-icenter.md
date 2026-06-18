---
type: moc
title: "ICenterLib/iCenter — Map of Content"
status: draft
tags: [moc, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# ICenterLib/iCenter

## What this hub covers

`ICenterLib\iCenter\` — **the iCenter-database wrappers**. 29 files mapping the iCenter SQL schema (`T_ProdMachines`, `T_IPorderLines`, `T_IPbatchLines`, `T_IPpacketLines`, `T_IPpartLines`, `T_TimeRegistration`, `T_SmtMaterials`, `T_CoatingLayerThickness`, `T_DossierContactFav`, `T_WebClockAssistant`, `T_DgxStickers`, etc.) — opposite of [[icenterlib-isah|`ICenterLib/ISAH`]] which wraps the ISAH ERP database. Both folders share the same namespace strategy: typed entity classes opening their own `Connections.ConnectICenter()` / `.ConnectIsah()` connections.

> iCenter has two SQL Servers it touches: **`ICENTER`** (production-execution / shop-floor state) and **`ISAH`** (ERP master). This folder is the iCenter side.

## Scope

29 `.vb` files; ~130 KB of source. Top by size:

| File | KB | Role |
|------|---:|------|
| `ProductionMachines.vb` | 41 | per-machine state + worktime + sticker queue |
| `IPPart.vb` | 19 | the leaf-level production-batch part (final-product instance) |
| `IPBatch.vb` | 13 | per-MachGrp batch under an IPorder |
| `FrmIdentification.vb` | 8.3 | shop-floor employee-picker form |
| `DossierContactFavorite.vb` | 7.9 | dossier-level contact-shortcut bookmarks |
| `Part.vb` | 7.6 | SMT-material lookups + deburr/cycle-time formulas |
| `IPOrder.vb` | 7.5 | top-level production-batch container |
| `XmlFile.vb` | 6.6 | per-Modelname XML doc-pointer file editor |
| `Client.vb` | 6.0 | iCenter-client (Windows machine) → ProdMachine mapping |
| `Servicedesk.vb` | 5.3 | Zammad servicedesk URL/form builder |
| `TimeRegistration.vb` | 5.0 | iCenter-side time-reg display query |
| `WebClockAssistant\WebClockAssistantRepository.vb` | 4.1 | T_WebClockAssistant CRUD |
| `ExternalReferences.vb` | 3.7 | Creo external-ref bag |
| `CtrlImageIdentification.vb` | 2.8 | employee-button control with photo |
| `IPPacket.vb` | 2.7 | per-Modelname grouping under IPbatch |
| `WebClockAssistant\WebClockAssistantGenericHandler.vb` | 2.6 | base for both flavours |
| `WebClockAssistant\WebClockAssistantUserSelectionHandler.vb` | 2.5 | UI-driven assistant pick |
| `BillOfOper.vb` | 2.4 | in-memory bill-of-operations DataTable wrapper |
| `WebClockAssistant\WebClockAssistantAnonymousHandler.vb` | 2.2 | auto-pick virtual assistant |
| `DataServices\CoatingLayerThicknessDataService.vb` | 1.7 | T_CoatingLayerThickness query |
| `CtrlRadButtonIdent.vb` | 1.4 | radio-button for department codes |
| `ExternalReference.vb` | 1.3 | Creo external-ref entity |
| `ProductionMachineMultiPurpose.vb` | 1.2 | MP-machine listing |
| `SurfaceTreatmentDefinition.vb` | 0.3 | DTO |

Plus 4 `.designer.vb` files (generated; mark `generated`).

## Documented in this batch

| File(s) | Module note |
|---------|-------------|
| `ProductionMachines.vb`, `Client.vb`, `ProductionMachineMultiPurpose.vb` | [[../modules/icenterlib-icenter-production-machines]] |
| `IPOrder.vb`, `IPBatch.vb`, `IPPacket.vb`, `IPPart.vb`, `BillOfOper.vb` | [[../modules/icenterlib-icenter-batch-hierarchy]] |
| `FrmIdentification.vb`, `CtrlImageIdentification.vb`, `CtrlRadButtonIdent.vb`, all `WebClockAssistant\*` | [[../modules/icenterlib-icenter-identification]] |
| `Part.vb`, `XmlFile.vb`, `Servicedesk.vb`, `TimeRegistration.vb`, `DossierContactFavorite.vb`, `CoatingLayerThicknessDataService.vb`, `ExternalReference[s].vb`, `SurfaceTreatmentDefinition.vb` | [[../modules/icenterlib-icenter-leaves]] |

## iCenter schema fragments documented

| Table | Used by | Notes |
|-------|---------|-------|
| `T_ProdMachines` | ProductionMachines, Client | physical machine registry |
| `T_ProdMachinesMP` | ProductionMachineMultiPurpose, FrmIdentification | multi-purpose machines (operator-shared workplaces) |
| `T_ProdMachineWorkTime` | ProductionMachines | per-machine accumulated work time |
| `T_ProdMachineWorkTimeRegistration` | ProductionMachines | per-part time-reg lines |
| `T_IPorderLines` | IPOrder | top-level production batch (1 row per IoId) |
| `T_IPbatchLines` | IPBatch, IPOrder | per-MachGrp batches under an IPorder |
| `T_IPpacketLines` | IPPacket, IPBatch | per-Modelname grouping under IPbatch |
| `T_IPpartLines` | IPPart, IPBatch | leaf-level part rows (one per physical part) |
| `T_IPpartDispatch` | IPPart | tracks "this part has been dispatched to that MachGrp" |
| `T_IPorderRef` | IPBatch.GetOrdRef | references like RefNr/FinInd per ProdHeaderDossierCode |
| `T_TimeRegistration` | TimeRegistration, ProductionMachines | per-employee per-part time slips |
| `T_DgxStickers` | ProductionMachines | sticker-print queue |
| `T_CoatingLayerThickness` | CoatingLayerThicknessDataService | empirical coating thickness measurements |
| `T_SmtMaterials` | Part | SMT material catalog + Material/TruTopsMatId mapping |
| `T_Operations` | ProductionMachineMultiPurpose | iCenter operation definitions (OperId → DeptCode) |
| `T_DossierContactFav` | DossierContactFavorite | per-dossier favourite contacts |
| `T_WebClockAssistant` | WebClockAssistantRepository | per-employee "virtual assistant" mapping |

## Business rules surfaced in this batch

- [[../business-rules/icenter-dynamic-workplace-prefix|`WP`-prefix dynamic workplaces]] — `Client.DynamicWorkplacePrefix = "WP"`, machine ID derived from numeric suffix.
- [[../business-rules/icenter-boost-delete-blocked|Boost delete blocked once parts have started]] — `IPBatch.DeleteSmtProdOrd` refuses if any parts have `Completed=1` flag set.
- [[../business-rules/icenter-smt-deburr-cycle-time|SMT deburr cycle-time formula]] — area × DeburrSpeed × material/quality multipliers in `Part.GetSmtDeburrCalcCycleTime`. `#safety-relevant`
- [[../business-rules/icenter-jalu-jala-dept-merge|JALU+JALA dept merge in identification UI]] — `FrmIdentification` collapses JALA into JALU; JALA on its own is skipped.
- [[../business-rules/icenter-servicedesk-fallback-email|Servicedesk fallback to `pvs@jazo.com`]] — when employee has no `LocalEmailAddress`.

## Notable findings

1. **Massive use of `MsgBoxStyle.Critical` in `Catch ex As Exception` → `Log.NewEntry(ex.ToString, MsgBoxStyle.Critical)`** — the dominant error pattern across ProductionMachines. `Log.NewEntry` must be the central log+notify entry point (Q-219, find consumers / behaviour).
2. **`FrmIdentification.RadioButton_CheckedChanged` constructs a `CtrlImageIdentification` per employee per dept selection** — for a depth-heavy dept it can show 50+ photo buttons. Performance unknown.
3. **WebClockAssistant** is a 2-flavour pattern: `AnonymousHandler` (auto-pick from pool of "virtual" assistants) vs `UserSelectionHandler` (operator picks a real coworker). The anonymous pool is what gets used when a real coworker isn't available.
4. **`IPOrder.GetBomFilteredByMachGrpCode` uses XML-shredding** — the `T_IPorderLines.XmlParams` column holds an ORDER/PARTS/PART/OPERATIONS/OPER tree. XQuery `CROSS APPLY` extracts (DesignCode, MachGrpCode) pairs. Performance + XML-injection risk if anyone manages to seed `MachGrpCodes` from outside (probably internal-only).
5. **`Client.SmartUpdateBySession` only triggers `FrmProdMachineSelector` if `Common.GetIsAdsServer(_Computername)`** — ADS-server presence drives whether the machine picker shows. ADS = Active Directory Services? Or "Application-Dependent Service"? Q-220.
6. **`Part.GetSmtCalcCycleTime` has assignment-typo `sOpenContours = sOpenContours = CType(...)`** (line 147). Evaluates `(sOpenContours = CType(...))` as a Boolean comparison, then assigns False to `sOpenContours` via `=`. **Almost certainly a production bug** — Q-221 `#safety-relevant`.
7. **`ProductionMachines` is unusually written**: every method opens its own connection, swallows exceptions, returns Nothing/empty. Same problem family as Q-095 / Q-183 (silent exception swallow → can't distinguish "no data" from "DB error").
8. **`IPpart.New(IPpartId As String)`** parses `IPpartId.Replace(Common.IPPARTPREFIX, "")` to recover the Long. Confirms a project-wide prefix string for encoded IDs. Q-222 — what's the prefix?
9. **`XmlFile.GetDocumentInfo`** is the **file-extension → document-type rule**: `.jpg`/`.bmp`/`.png` → PICTURE (with `_main.jpg` suffix → MAINIMAGE), `.pdf` → PDF (with `DeclarationOfPerformance.PdfSuffix` → CE), `.ecw`/`.ncw` → "EluCad", `.pvz` → "ProductView", `.rld` → LaserWorks app label, etc. Single source of truth for the iCenter document-classifier. Q-223 — document the full map in a domain-concept note.

## Open questions

- **Q-219 (new):** Document `Log.NewEntry(message, MsgBoxStyle)` — is it just file/EventLog logging or does it really pop a MsgBox? In a CadBatchserver context, popping MsgBoxes would block.
- **Q-220 (new):** `Common.GetIsAdsServer(computername)` — what is ADS? When does it return true?
- **Q-221 (new):** `Part.GetSmtCalcCycleTime` line 147 has typo `sOpenContours = sOpenContours = CType(...)` — almost certainly assigns `False` not the converted value. `#safety-relevant` — cycle-time used in production scheduling.
- **Q-222 (new):** `Common.IPPARTPREFIX` — what's the string? (Likely "IP-" or similar.) Where is it formatted/parsed?
- **Q-223 (new):** Document the file-extension → document-type map from `XmlFile.GetDocumentInfo`.
- **Q-224 (new):** `IPOrder.GetBomFilteredByMachGrpCode` — confirm `T_IPorderLines.XmlParams` schema (ORDER/PARTS/PART → OPERATIONS/OPER). Generated by what?

Logged in [[../needs-review/_index]].

## Related

- [[icenterlib|ICenterLib top-level MOC]] — parent.
- [[icenterlib-isah|ICenterLib/ISAH]] — sibling MOC; many entities here link to ISAH counterparts via `MachGrpCode`, `EmpId`, `ProdHeaderDossierCode`.
- [[office-to-shopfloor|Office → shop-floor handoff]] — shop-floor consumers of ProductionMachines and IPbatch.
