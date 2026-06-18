---
type: module
title: "Icenter2Isah — iCenter → ISAH XML sync layer"
status: done
module: "ICenterLib/ISAH"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\Icenter2Isah.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `Icenter2Isah.vb` — iCenter → ISAH XML sync layer

> **Filename is misleading.** Despite the `Icenter2Isah` name, this class is **not** related to the out-of-scope iCenter2 sibling project. It is the **iCenter → ISAH** sync direction: read iCenter parts and operations via a stored procedure, convert to JIBA.NET-flavoured XML, then push back into ISAH via per-domain insert wrappers (`IP_ins_PurchaseDocument`, `IP_ins_PurDocPartLine`, `IP_Ins_DossierMain`).

## Purpose

The one-file sync layer that **transfers iCenter-side production data into ISAH**. Used when iCenter has computed parts + operations that need to materialise as ISAH master records (purchase documents, dossier-main, etc.) so the rest of ISAH's workflow can see them.

760 lines. Sits in the `Public` (root) namespace, not `ISAH` — one of the few ICenterLib files outside its expected namespace.

## Public surface

```vb
Public Class Icenter2Isah
    Enum TypeOfAutomationJobType { Waiting=1, Processing=2, Completed=3, Failed=4 }
    Enum TypeOfField             { Varchar=0, Numeric=1 }
    Public Enum InsertOption     { PartsOpers=0, Opers=1 }

    Public Const DefaultSurChargeCode As String = "EX10"
    Private Const DefaultSIP_Get_PartsAndOpersSelectionType As Integer = 6

    Public Function SIP_Get_PartsAndOpers(IoId, ReferenceId, ProdHeaderDossierCode, ProdPlanEndDate,
                                          ReRoute, ReNumber, MergeOpers, ChangePlanData, PlanLeadTime,
                                          AutoJobPrepFiatInd, CreateProdOrdNr, ProdStatusCode, GenerateShopDoc,
                                          RunPurAdvice, DossierCode, DossierStatusCode, DetailCode,
                                          DetailSubCode, SurChargeCode)                            As DataSet
    Public Shared Function SIP_Get_PartsAndOpers(IoId As Long)                                      As DataSet

    Public Function GenerateXML(DS As DataSet, SaveXml As Boolean)                                  As XmlDocument
    Public Function GetAutomationJobCompleted(JobId As Integer)                                     As Boolean

    Public Function IP_ins_PurchaseDocument(...)                                                    As Integer
    Public Function IP_ins_PurDocPartLine(...)                                                      As Integer
    Public Sub      IP_Ins_DossierMain(ByRef new_DossierCode As String, ...)
End Class
```

## Behavior

### 1. Pull from iCenter DB — `SIP_Get_PartsAndOpers`

The 19-parameter instance overload calls SP `SIP_Get_PartsAndOpers` on `Connections.ConnectICenter` (note: **iCenter DB, not ISAH**). Returns a `DataSet` with three tables (General, Parts, Operations).

The shared 1-parameter overload is the simple lookup-by-`IoId` form. Both hardcode `@SelectionType = 6` (the `DefaultSIP_Get_PartsAndOpersSelectionType` private const). Q-197 — what does SelectionType 6 mean?

`DefaultSurChargeCode = "EX10"` is a `Public Const` — the JAZO default surcharge code when none is provided by the caller. Documented as [[../business-rules/isah-default-surcharge-code]].

### 2. Convert to JIBA.NET XML — `GenerateXML`

~170-line builder that walks the three DataTables and emits a single `<JIBA-ConfigPart>` XML document with:

- `<General>` block carrying employee info and per-column entries.
- `<ConfigPart>` block enumerating parts.
- Operations attached per-part.

Hard-codes:
- `Encoding = Unicode` (UTF-16).
- `XmlSettings.Indent = True`.
- Inserts `<!-- JIBA.NET ConfigPart Created on … by … -->` comment.
- `WriteAttributeString("AutoGenerate", False)` for the General element.

`FormatItemValue(value, dataType)` (private helper, not shown above) is the per-cell formatter that handles `Varchar` vs `Numeric` field types.

Also calls `AddPartInfoFromIsah(DTParts)` and `AddOperationInfoFromIsah(DTOperations)` (private subs) to enrich the iCenter data with ISAH-side master-data lookups before XML generation.

### 3. Push into ISAH — three SP wrappers

| Method | SP | Role |
|--------|-----|------|
| `IP_ins_PurchaseDocument` | `IP_ins_PurchaseDocument` | create a new ISAH purchase document |
| `IP_ins_PurDocPartLine` | `IP_ins_PurDocPartLine` | append a part-line to an existing purchase document |
| `IP_Ins_DossierMain(ByRef new_DossierCode, ...)` | `IP_Ins_DossierMain` | create a new ISAH dossier-main; returns new DossierCode via output parameter |

Each wraps the SP with ~30-40 parameters, similar to other ISAH insert wrappers documented in [[isah-production-hierarchy]] and [[isah-part-and-dispatch]].

### 4. Automation-job polling — `GetAutomationJobCompleted`

The `TypeOfAutomationJobType` enum (Waiting=1, Processing=2, Completed=3, Failed=4) is the 4-state machine for **JAZO's iCenter↔ISAH automation jobs**. `GetAutomationJobCompleted(JobId)` polls a JZ-prefixed table to check whether the job's state has reached `Completed`. Used by callers running the sync as a background task. Q-198 — document the table.

## Surprises

1. **Filename misleads.** `Icenter2Isah` (`2` = "to") means "iCenter to ISAH", not "iCenter2 (sibling) ↔ ISAH". The out-of-scope iCenter2 has no class here. Q-199 — rename to `IcenterToIsah` to avoid confusion.
2. **No namespace.** Sits at the root of `ICenterLib` rather than inside `ICenterLib.ISAH` — accessed as `Icenter2Isah.X` from callers, not `ISAH.Icenter2Isah.X`. Outlier in the ISAH folder.
3. **Mixes iCenter DB and ISAH DB.** `SIP_Get_PartsAndOpers` reads from iCenter DB (`ConnectICenter`); `IP_ins_*` writes to ISAH (`ConnectIsah`). The class is the explicit bridge.
4. **`SaveXml` parameter on `GenerateXML` is unused in the visible 200 lines.** Likely controls whether to write the XML to disk for debugging. Phase-3 follow-up.
5. **All public methods swallow exceptions** (single outer `Try…Catch` per method, returns `Nothing` / empty `DataSet` on failure). Consumers can't tell "no rows" from "ISAH down".
6. **`AddPartInfoFromIsah` and `AddOperationInfoFromIsah`** are private enrichment passes — they run ISAH SP calls per row of the iCenter-side DataTables to fill in ISAH master-data columns. For a large parts list, this can N+1 the ISAH database. Q-200.
7. **Hardcoded XML structure**. Adding a new column to iCenter's `SIP_Get_PartsAndOpers` output requires either updating `GenerateXML` to emit it or accepting that it gets serialised generically by the `WriteElementString` per-column loop.

## Business rules surfaced here

- [[../business-rules/isah-default-surcharge-code|`EX10` default surcharge code]] — `DefaultSurChargeCode` const. Used by `SIP_Get_PartsAndOpers` callers that don't supply their own.
- **`SelectionType = 6`** for `SIP_Get_PartsAndOpers` is hardcoded; the SP supports other selection types but iCenter only uses 6. Q-197.
- **`TypeOfAutomationJobType` 4-state**: Waiting → Processing → Completed/Failed. Standard async-job lifecycle.
- **JIBA.NET XML schema** for ConfigPart — anyone consuming the XML on the other side must speak this schema.

## Open questions

- **Q-197 (new):** Document `SIP_Get_PartsAndOpers` `SelectionType = 6` semantics. What do values 1-5 mean?
- **Q-198 (new):** Document the `TypeOfAutomationJobType` table (presumably `JZ_AutomationJob` or similar) — what callers create rows? What populates the Status column?
- **Q-199 (new):** Rename `Icenter2Isah` → `IcenterToIsah` (or move into `ISAH` namespace as `Sync`) to eliminate the iCenter2-sibling confusion.
- **Q-200 (new):** `AddPartInfoFromIsah` / `AddOperationInfoFromIsah` enrich row-by-row — N+1 ISAH calls per parts list. Refactor to batch lookups?

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/icenterlib-isah]] — parent MOC.
- [[isah-dossier|`DossierMain`]] — `IP_Ins_DossierMain` creates rows there.
- [[isah-part-and-dispatch|`PurDoc` / `PurDocPartLine`]] — `IP_ins_PurchaseDocument` / `IP_ins_PurDocPartLine` create rows.
- [[icenterlib-connections|`Connections.ConnectICenter` + `ConnectIsah`]] — this class uses both.
- [[../external-systems/isah|ISAH]] — destination.

## Coverage

`_coverage.md`: `ICenterLib\ISAH\Icenter2Isah.vb` → `done` (overview; 760 lines, 8 publics, per-method enumeration deferred).
