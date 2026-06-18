---
type: moc
title: "Office → shop-floor handoff — Map of Content"
status: draft
tags: [moc, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Office → shop-floor handoff

## What this hub covers

The four iCENTER sub-folders that span the path from a *sales order* in ISAH to *parts machined on the shop floor*: `Sales/`, `Engineering/`, `WorkPreparation/`, `Production/`. Each owns a slice of the handoff:

```
ISAH order
   │
   ▼
[Sales]              FrmCustomerTeam — assign customer→sales-team (codes 031/032/033)
   │
   ▼
[Engineering]        FrmOrdersAsBuilt — assign engineering owner
                     FrmDrwCheck      — drawing check / as-built / generic-check
                     FrmDesignCodeTool — request new drawing numbers (WebView2 → tekeningnummers.jazo.com)
                     frmEngGeneratedProductOverview — Excel-driven product catalogue
                     frmGenericStatus / frmSelectAnnotGenericStatus — generic-PDF status tree
                     ModelCopies/CopyLocalizer family — find copies of a part on the file server
   │
   ▼
[WorkPreparation]    IPBatchCollector              — SQL pull of T_ProdBillOfOper from ISAH
                     OperationSubstitutionHandler  — swap one machine-group op for another
                     OutsourceOperationsHandler    — package STEP+PDF, create PurOrd, drop in IsahDoc folder
                     FrmOutsourceOperations        — UI for the above
   │
   ▼
[Production]         ProductionProfileCutItemsHandler — write cut-items per machine (operation IDs 1/9/31)
                     ProfileMilling/ImportHandler     — pull BOM, validate profiles, push to UniLink CSV
                     ProfileMilling/PMMExportHandler  — export 3D STEP into MANUF folder, update iCenter XML
                     ProfileMilling/ExternalReferenceHelper — pull STEP files from EXTERNALREFERENCES XML
                     ProfileMilling/ImportData + ICAMImport (interface)
```

## Scope

20 `.vb` source files across the four folders. All small (median ~5 KB; largest is `FrmDrwCheck.vb` at 31 KB). Cross-cuts heavily into [[../external-systems/isah|ISAH]] (`oISAH`, `T_*` and `JZ_*` tables, `T_ProdBillOfOper`, etc.), [[../external-systems/icenter-db|iCenter DB]] (`oICENTER`), the UniLink companion (for CSV CAM imports), and the [[../external-systems/elumatec-sbz140|Elumatec profile-mill subsystem]] (every BOM walk fills `BIdentNo` via `EluCadApp.GetProfileInfoByPartCode`).

## Files (all documented in this batch unless noted)

### Sales/
| File | KB | Note |
|------|---:|------|
| [[../modules/sales-customer-team|`FrmCustomerTeam.vb`]] | 20 | Customer-to-team assignment (`031/032/033`) |

### Engineering/
| File                              |                                               KB | Note |                                                                                                                                  |
| --------------------------------- | -----------------------------------------------: | ---- | -------------------------------------------------------------------------------------------------------------------------------- |
| [[../modules/engineering-overview |                               `FrmDrwCheck.vb`]] | 31   | Drawing check / as-built / generic-check (three ShowModes)                                                                       |
| [[../modules/engineering-overview |                          `FrmOrdersAsBuilt.vb`]] | 13   | Order-engineer assignment (uses `PlanDosDetailDeptCodes` setting)                                                                |
| [[../modules/engineering-overview |            `frmEngGeneratedProductOverview.vb`]] | 8    | Reads Excel at `GeneratedProductFilepath` (= `\\jazo.local\dfs\DATA\Engineering\3D Pui Generator\Overzicht gemaakte puien.xlsm`) |
| [[../modules/engineering-overview |               `frmSelectAnnotGenericStatus.vb`]] | 4    | annotation generic-status selector                                                                                               |
| [[../modules/engineering-overview |                          `frmGenericStatus.vb`]] | 3    | Generic-PDF status tree                                                                                                          |
| [[../modules/engineering-overview |                         `FrmDesignCodeTool.vb`]] | 3    | WebView2 to `https://tekeningnummers.jazo.com` — assigns new drawing numbers                                                     |
| [[../modules/engineering-overview |                 `ModelCopies\CopyLocalizer.vb`]] | 2    | abstract base for "find copies of a part" jobs                                                                                   |
| [[../modules/engineering-overview |          `ModelCopies\GenericSmtPartFinder.vb`]] | 4    | finds `gn*.xml` SmtThickness>0 parts                                                                                             |
| [[../modules/engineering-overview | `ModelCopies\UitsparingVoorplaatMeerpslAlu.vb`]] | 3    | finds parts matching `AD*-P*-D*-01-90[16]` of generics `GN10188-P1-D1-01-901/906`. Registered-out (line 30 of CopyLocalizer)     |

### WorkPreparation/
| File                                         |                                  KB | Note |                                                |
| -------------------------------------------- | ----------------------------------: | ---- | ---------------------------------------------- |
| [[../modules/workprep-outsource-operations   |   `OutsourceOperationsHandler.vb`]] | 25   | The big one — outsource-to-vendor pipeline     |
| [[../modules/workprep-outsource-operations   |       `FrmOutsourceOperations.vb`]] | 20   | UI for the above                               |
| [[../modules/workprep-operation-substitution | `OperationSubstitutionHandler.vb`]] | 5    | Swap one operation for another, migrate timing |
| [[../modules/workprep-ipbatch-collector      |             `IPBatchCollector.vb`]] | 4    | SQL query against ISAH `T_ProdBillOfOper`      |

### Production/
| File                                           |                                            KB | Note |                                                                  |
| ---------------------------------------------- | --------------------------------------------: | ---- | ---------------------------------------------------------------- |
| [[../modules/production-profile-cut-items      |       `ProductionProfileCutItemsHandler.vb`]] | 5    | Maps iCenter operation IDs (1/9/31) → MachGrpCodes               |
| [[../modules/production-profile-milling-import |           `ProfileMilling\ImportHandler.vb`]] | 8    | BOM → validate → UniLink CSV                                     |
| [[../modules/production-profile-milling-import |        `ProfileMilling\PMMExportHandler.vb`]] | 4    | Move STEP into MANUF folder, update iCenter XML                  |
| [[../modules/production-profile-milling-import | `ProfileMilling\ExternalReferenceHelper.vb`]] | 3    | Parse `EXTERNALREFERENCES` nodes                                 |
| [[../modules/production-profile-milling-import |              `ProfileMilling\ImportData.vb`]] | 3    | DataTable shape (23 columns including geometry)                  |
| [[../modules/production-profile-milling-import |              `ProfileMilling\ICAMImport.vb`]] | <1   | one-method interface — implemented by `UniLink.CSVImportHandler` |

## Business rules surfaced in this batch

- [[../business-rules/sales-team-codes]] — the hard-coded `"031" / "032" / "033"` team codes in `FrmCustomerTeam.New`.
- [[../business-rules/outsource-ext-oper-part-code]] — `extOperPartCode = "UITBESTEDING01"` hard-coded in `OutsourceOperationsHandler.ProcessDataByVendIdProfileId`.
- [[../business-rules/icenter-operation-machgrp-mapping]] — `iCenterOperationId → MachGrpCodes` mapping (1→{A01,A07}, 9→{S01}, 31→{A07,A01}).

## Cross-references back to existing wiki

- **ISAH** is the spine: every `Sales/`, `Engineering/`, `WorkPreparation/`, `Production/` class talks to it via `oISAH` (global) or `ICenterLib.ISAH.*`. See [[../architecture/external-surface]].
- The **Elumatec** subsystem ([[elumatec]]) is reached from this workflow at three points: (1) `ProductionProfileCutItemsHandler.Write` calls `EluCadApp.GetProfileInfoByPartCode`; (2) `ImportHandler.GetBom` does the same to fill `BIdentNo` + `Series`; (3) `ImportHandler.ValidateProfileSeriesStatus` checks UniLink's profile catalogue for DXF presence.
- The **CadBatchserver** (not yet documented) and the **JobSendEmail / JobCreatePurOrdDocs** classes there will likely consume `OutsourceOperationsHandler` output.

## Open questions

- **Q-082** — sales-team codes 031/032/033 hard-coded. SME-confirm and document the team semantics.
- **Q-083** — `UITBESTEDING01` ext-operation part code hard-coded. Is there ever a need for a different code?
- **Q-084** — only `ProfileId = 1` is implemented for outsourcing (everything else throws `NotImplementedException`). What does ProfileId 1 mean, and what other values exist?
- **Q-085** — `iCenterOperationId → MachGrpCodes` table has only 3 entries (1, 9, 31). Confirm with SME these are the only ones. The class throws `NotImplementedException` otherwise.
- **Q-086** — `OutsourceOperationsHandler.WriteExchangeFile` produces a CSV with 8 columns hard-coded; no schema versioning. Vendors that change format → silent breakage.
- **Q-087** — `WaitForPurDocFolder` polls every 500ms for 20s before throwing. What happens if ISAH is busy creating the folder during peak hours?
- **Q-088** — `BatchServerMode` is *temporarily flipped* to `True` during `GenericSmtPartFinder.search` (and `UitsparingVoorplaatMeerpslAlu.search`) — why? Q-005 connection: `BatchServerMode` was already flagged as something `FrmMain` flips. This usage hints at a side-effect-suppression pattern.
- **Q-089** — `UitsparingVoorplaatMeerpslAlu` is registered-out in `CopyLocalizer.GetAvailableSearches` with comment _"Orders die niet in iCenter worden hiermee overgeslagen, dus niet toepassen"_ — confirm it's truly dead code.
- **Q-090** — `FrmDesignCodeTool` controls the external `tekeningnummers.jazo.com` website via WebView2 + JavaScript injection (gets/sets `CphMainContent_TxtSearchValue`, clicks `ctl00$CphMainContent$BtnSearch`). Brittle to any UI change on that site.

All logged in [[../needs-review/_index]].

## Related

- [[../mocs/elumatec|Elumatec subsystem MOC]] — destination of profile data
- [[../architecture/external-surface]] — ISAH, JIBA, UniLink, Active Directory all reached from here
- [[../architecture/global-state]] — `oISAH`, `oICENTER`, `oJIBA`, `BatchServerMode`, `MyMainForm` globals used throughout
