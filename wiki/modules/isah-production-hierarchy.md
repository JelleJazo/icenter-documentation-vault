---
type: module
title: "ISAH production hierarchy — ProductionHeader / PBOO / PBOM / PBOS / BillOfOper / BillOfMat"
status: done
module: "ICenterLib/ISAH"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\ProductionHeader.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\PBOO.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\PBOM.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\PBOS.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\BillOfOper.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\BillOfMat.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# ISAH production hierarchy

## Purpose

Six classes that together model the **ISAH production-side workflow** — the *result* of [[isah-dossier|`DossierDetail.GenProductionHeader`]]:

```
ProductionHeader      T_ProductionHeader     (PD<8 digits>)
   ├── PBOO           T_ProdBillOfOper       — per-operation lines under this header
   │     └── ShopDoc  T_ShopDoc              — per-PBOO-line shop-floor document
   ├── PBOM           T_ProdBillOfMat        — per-material lines
   └── PBOS           T_ProdBillOfSurcharge  — per-surcharge lines

BillOfOper            T_BillOfOper           — *template* operations (per Part, not per ProductionHeader)
BillOfMat             T_BillOfMat            — *template* materials (per Part)
```

The **template** classes (`BillOfOper` / `BillOfMat`) live one level up from the per-ProductionHeader classes — they describe what a Part *would* look like, while PBOO/PBOM are *the per-job instance* materialised onto a specific ProductionHeader.

Closes the chain from [[isah-dossier|dossier]] → production-header → PBOO → [[isah-shop-and-pur-doc|ShopDoc]].

## `ProductionHeader` (817 lines)

```vb
Public Class ProductionHeader
    Public ReadOnly Property ProdHeaderDossierCode As String
    Public ReadOnly PBOS As PBOS                                ' eagerly constructed

    Public Shared Function IsProdHeaderDossierCode(value)        As Boolean
    Public Sub New(ProdHeaderDossierCode)

    Public Sub SetStatus(NewStatus)                              ' SP IP_upd_ProdHeader, output @LastUpdatedOn
    Public Sub SetOperStarted(MachGrpCode, value)                ' finds PBOO line + ShopDoc + delegates
    Public Function GetLastUpdatedOn()                           As String
    Public Function GetRecord()                                  As DataTable
    ' ... ~700 more lines of read accessors + insert helpers (deferred)
End Class
```

Key facts:

- **Identifier shape**: `IsProdHeaderDossierCode(value)` validates `value` against the regex **`PD\d{8}`** — 2-char `PD` prefix + 8 digits. A 10-character production-header dossier code.
- **`PBOS` is eagerly constructed** in the constructor — opening a ProductionHeader also opens the surcharge accessor. Cheap (no SQL), but creates the asymmetry: PBOO and PBOM must be opened on demand by callers.
- **`SetStatus(NewStatus)`** calls SP `IP_upd_ProdHeader` with `@old_LastUpdatedOn` as the optimistic-concurrency token. The SP returns a new `@LastUpdatedOn` value via an **output parameter** — but the wrapper *discards it after assigning to a local* (line 47). Q-159: if a caller re-calls `SetStatus`, the cached `LastUpdatedOn` is stale.
- **`SetOperStarted(MachGrpCode, value)`** is a 3-step orchestrator:
  1. `PBOO.GetFirstBooLineNr(ProdHeaderDossierCode, MachGrpCode)` — find the first PBOO line for that machine group.
  2. `ShopDoc.GetShopDocCode(ProdHeaderDossierCode, ProdBOOLineNr)` — translate to ShopDocCode.
  3. If found: `PBOO.SetOperStartInd(ShopDocCode, value)` AND (only when `value = True`) `PBOO.SetShopDocStatusCode(ShopDocCode, "20")`. Status `"20"` = "started" by convention. Q-160.
- **`IsahUserCode = "ISAH"`** hardcoded again (consistent drift from `Common.APPLISAHUSERCODE = "ICENTER"`; same as DossierDetail).

## `PBOO` — Production Bill of Operations (585 lines)

```vb
Public Class PBOO
    Public ReadOnly Property ProdHeaderDossierCode As String
    Public Const LineNrStepSize As Integer = 5                   ' lines go 5, 10, 15, ...

    Public Sub New(ProdHeaderDossierCode)

    ' lookups
    Public Function GetFirstExistingMachGrpCode(MachGrpCode, PartCode)        As String
    Public Function GetFirstStartDateByMachGrpCode(MachGrpCode)               As Date?
    Public Function GetFirstEndDateByMachGrpCode(MachGrpCode)                 As Date?
    Public Shared Function GetFirstBooLineNr(ProdHeaderDossierCode, MachGrpCode) As Integer

    ' state mutators
    Public Shared Sub SetOperStartInd(ShopDocCode, ProdStartedInd)            ' writes T_ProdBillOfOper.StartedInd
    Public Shared Sub SetShopDocStatusCode(ShopDocCode, NewStatusCode)        ' "20" = started (per convention)

    ' ... ~500 more lines (deferred — Phase-3 follow-up)
End Class
```

Key facts:

- **`LineNrStepSize = 5`** — line numbers within a ProductionHeader's PBOO go 5, 10, 15, …. This convention leaves room to insert new operations between existing ones without renumbering. Same pattern in `PBOS` (and in [[#bilLofOper|`BillOfOper.InsertData`]] which accepts `LineNrStepSize` as a parameter). Q-161 — document the LineNrStepSize convention.
- **`GetFirstExistingMachGrpCode(MachGrpCode, PartCode)`** is a sequence-aware lookup: finds the first PBOO line whose MachGrp *exists at-or-after* the requested MachGrp in the Part's `T_BillOfOper`. Used when iCenter needs to know which PBOO line to attach a new operation to.
- **State writes are `Shared`** — `SetOperStartInd` and `SetShopDocStatusCode` are static. PBOO instances don't carry state, but the static-vs-instance asymmetry is yet another readability hazard.
- **`SetShopDocStatusCode(_, "20")` is hard-coded "20"** in `ProductionHeader.SetOperStarted` (line 67). Status code 20 = "started" by convention; the next codes presumably represent finished / failed / etc. — Phase-4 candidate.

## `PBOM` — Production Bill of Materials (200 lines)

```vb
Public Class PBOM
    Public ReadOnly Property ProdHeaderDossierCode As String
    Public Sub New(ProdHeaderDossierCode)

    Public Function GetPartCode(ProdBOMLineNr)                        As String      ' SubPartCode
    Public Function GetFirstProdBOMLineNrFromInvt(CADReference)       As Integer     ' CADReference + InvtQty > 0
    Public Function GetLastUpdatedOn(ProdBOMLineNr)                   As Date
    Public Function IP_sel_ProdBillOfMatRS(ProdBOMLineNr)             As DataTable
    Public Function IP_sel_ProdBillOfMatRS()                          As DataTable   ' all lines
    Public Function GetLastLineNr()                                   As Integer
End Class
```

Key facts:

- Wraps `T_ProdBillOfMat`.
- `GetFirstProdBOMLineNrFromInvt(CADReference)` was annotated `'2023-01-03 CB: added condition AND InvtQty > 0` — meaning before that date, the query could match a stub line with no inventory. Worth flagging when reading historical iCenter data: behaviour changed Jan 2023.
- A **giant 30-parameter `IP_Ins_ProdBOM` insert function is commented out** at the bottom of the file (lines 147–232). Dead code, but documents what an insert *would* look like if iCenter ever needed to programmatically create PBOM lines.

## `PBOS` — Production Bill of Surcharges (132 lines)

```vb
Public Class PBOS
    Public ReadOnly Property ProdHeaderDossierCode As String
    Public Const LineNrStepSize As Integer = 5

    Public Sub New(ProdHeaderDossierCode)

    Public Function GetLastLineNr()      As Integer
    Public Function GetNextLineNr()      As Integer   ' GetLastLineNr() + LineNrStepSize
    Public Function GetRecordCount()     As Integer
    Public Function GetRecords()         As DataTable
    Public Sub IP_Ins_ProdBOS(SurchargeCode, Optional LineNr = 5)
End Class
```

Key facts:

- Wraps `T_ProdBillOfSurcharge`.
- **`IP_Ins_ProdBOS`** is the standout: instead of calling SP `IP_ins_ProdBOS` with parameters, it builds an **inline `DECLARE @x ... ; SELECT @x = ...; EXECUTE IP_ins_ProdBOS @x, ...`** T-SQL batch as a giant string-concatenated `SqlCommand` (~60 lines, lines 75–149). The local variables fetch surcharge fields from `T_Surcharge` server-side, then pass them to the SP. Reads exactly like an ISAH-supplied reference snippet that someone transliterated verbatim instead of calling the SP directly with parameters from the client. Q-158 — refactor candidate.
- **String concatenation of `ProdHeaderDossierCode` and `SurchargeCode` into the T-SQL** — both are internal identifiers, low injection risk, but the pattern is the same as [[icenterlib-common|`Common.GetSqlInString`]] flagged earlier (Q-114).
- **`LogProgramCode = 0`** hardcoded in this insert — no per-call-site identification. Q-151 connection.

## `BillOfOper` — template operations per Part (192 lines)

```vb
Public Class BillOfOper
    Private Const LogProgramCode As String = "0"
    Public ReadOnly Part As Part

    Public Sub New(Part As Part)

    Public Shared Function GetLastLineNr(PartCode)                          As Integer
    Public Sub DeleteExistingCalculation(BOOPartDescription, IsahUserCode)
    Public Sub InsertData(DT, IsahUserCode, LineNrStepSize, LineNrSeed, ICenterPartCalcPartCode)
    Public Function IP_sel_BOORecordSet(IsahUserCode)                       As DataTable
    Public Sub IP_del_BOO(old_PartCode, old_BOOLineNr, old_LastUpdatedOn, IsahUserCode)
End Class
```

Key facts:

- Templates the operations attached to a Part (via `T_BillOfOper`). Each row carries `MachGrpCode`, `Qty`, `MachCycleTime`, `MachSetupTime`, `OccupationCycleTime`, `OccupationSetupTime`, `LineNr`, `BOOPartDescription`.
- **`InsertData(DT, ..., LineNrStepSize, LineNrSeed, ICenterPartCalcPartCode)`** bulk-inserts BOO rows from a DataTable. `LineNrSeed = Max(LastLineNr, LineNrSeed)` then increment by `LineNrStepSize` per row. Lets the caller decide the starting line number (e.g. `BillOfOper.GetLastLineNr(partCode) + 100` to leave a gap).
- **`DeleteExistingCalculation(BOOPartDescription, IsahUserCode)`** filters by `BOOPartDescription` (which the caller has set to `ICenterPartCalcPartCode`) and deletes each match. Used when iCenter is overwriting a previous calculation pass for a part. Q-162 — document the BOOPartDescription convention.
- **`IP_ins_BOO`** has **18 commented-out parameters** (lines 102–119) plus 12 active ones. Active list: `PartCode, MachGrpCode, BOOPartDescription, Qty, MachCycleTime, MachSetupTime, OccupationCycleTime, OccupationSetupTime, LineNr, LogProgramCode, IsahUserCode`. The commented-out 18 (ScriptListCode, StandCapacity, MachSetoffTime, LeadTime, Info, ProcessValue, PhantomBOMLineNr, EmpId, MachCode, PlanGrpCode, ActiveFromDate, ActiveToDate, …) are SP parameters iCenter doesn't currently pass.

## `BillOfMat` — template materials per Part (483 lines)

```vb
Public Class BillOfMat
    Private Const LogProgramCode As String = "0"
    Public ReadOnly Part As Part

    Public Sub New(Part As Part)

    Public Shared Function GetLastLineNr(PartCode) As Integer
    Public Shared Sub ApplySurfTreatment(ByRef DtObjects, dtSurfTreatmentPart, dtSurfTreatmentOper, dtOperTotal)
    ' ... ~400 more lines (Phase-3 follow-up — InsertData, IP_ins_BOM, IP_sel_BOM etc.)
End Class
```

Key facts:

- The **`ApplySurfTreatment` static** is the headline behaviour: it takes a "production-objects" DataTable (`DtObjects`) and merges three input tables (`dtSurfTreatmentPart`, `dtSurfTreatmentOper`, `dtOperTotal`) into it. For each surface-treatment-part row, it:
  - Finds or creates a `DtObjects` row with `Modelname = SubstPartCode`, `PartType = Koopdeel` ("purchase part").
  - Computes `Qty = (drPart.Qty / drPart.CalcQty) * SurfTreatSquareMeasure`.
  - Sets the MachGrp-named boolean column to `True`.
- Then it iterates `dtSurfTreatmentOper` to attach operation times to a "selected system" DataTable row of `PartType = Functiestuklijst` ("function-piece list"). Computes cycle times by dividing the per-operation `MachCycleTime` by `60 * 60` (seconds → hours, decimal-rounded to 3 places) and scaling by the square-measure.
- Cross-cuts directly with [[workprep-operation-substitution|`OperationSubstitutionHandler`]] which mutates the *same* `DtObjects` / `dtSurfTreatmentPart` / `dtSurfTreatmentOper` / `dtOperTotal` tables. The two classes share an undocumented schema contract on `DtObjects` (columns like `MachSetupTime_<code>`, `MonoMachCycleTime_<code>`, `TotalMachTime_<code>`, `MachCycleTime_<code>`, plus per-MachGrp boolean flags). Q-163 — document the `DtObjects` schema.
- **`PartType.Koopdeel` / `Functiestuklijst`** — Dutch domain terms (`Koopdeel` = "purchase part" = bought-in component; `Functiestuklijst` = "function-piece list" = phantom assembly grouping operations). Phase-4 domain-concept candidates.

## Cross-class wiring

```
[isah-dossier]
DossierDetail.GenProductionHeader
    → SP IP_gen_ProdHeadForDosDet (14 zero-params)
    → returns new ProdHeaderDossierCode (matches PD\d{8})

ProductionHeader.SetOperStarted(MachGrpCode, True)
    → PBOO.GetFirstBooLineNr(...) → ProdBOOLineNr
    → ShopDoc.GetShopDocCode(ProdHeaderDossierCode, ProdBOOLineNr) → ShopDocCode
    → PBOO.SetOperStartInd(ShopDocCode, True)
    → PBOO.SetShopDocStatusCode(ShopDocCode, "20")

[isah-shop-and-pur-doc]
ShopDoc.SetShopDocStartedInd(True)
    → PBOO.SetOperStartInd(ShopDocCode, True)   ' same path

ShopDoc.GetShopDocFinInd()
    → joins T_ShopDoc ⨝ T_ProdBillOfOper        ' status flag lives on PBOO

[BillOfMat surface-treatment]
BillOfMat.ApplySurfTreatment(DtObjects, dtSurfTreatmentPart, dtSurfTreatmentOper, dtOperTotal)
    ↕ shares schema with
[workprep-operation-substitution]
OperationSubstitutionHandler.Execute(ProdObjects)
    → mutates DtObjects + dtSurfTreatmentPart + dtSurfTreatmentOper + dtOperTotal
    (likely-bug Q-095 swapped iteration sources)
```

## Business rules surfaced here

- [[../business-rules/isah-prod-header-format|`PD\d{8}` ProductionHeader dossier-code format]] — every ProductionHeader identifier is exactly 10 chars: `PD` + 8 digits. Validated by `ProductionHeader.IsProdHeaderDossierCode`.
- **`LineNrStepSize = 5`** — PBOO, PBOS, and the BillOfOper.InsertData default use line-number steps of 5. Convention leaves gaps for inserts. Q-161.
- **ShopDoc status code `"20"` = "started"** — hard-coded in `ProductionHeader.SetOperStarted`. Q-160.
- **`BOOPartDescription`** acts as a *correlation key* for "this Part's operations belong to a specific iCenter calculation" — `InsertData` writes it as the per-row `ICenterPartCalcPartCode`, and `DeleteExistingCalculation` later uses it to find rows to delete. Q-162.

## Surprises

1. **PBOS uses string-concatenated T-SQL** instead of parametrised SP call (Q-158). Looks like a transliteration of an ISAH SP-reference snippet rather than idiomatic .NET data access.
2. **PBOO and PBOM both have giant commented-out insert functions** (~30 parameters each) — documenting the SP signature but not currently used. iCenter doesn't programmatically create PBOO or PBOM rows; it relies on ISAH's `IP_gen_ProdHeadForDosDet` to populate them as a side effect of generating the ProductionHeader.
3. **`ProductionHeader.SetStatus` discards the SP's `@LastUpdatedOn` output param** after assigning to a local. Subsequent `SetStatus` calls re-read via `GetLastUpdatedOn()` — extra round-trip per write.
4. **ShopDoc-status-code `"20"` is hardcoded** (`ProductionHeader.SetOperStarted`). Other codes (presumably "30" finished, "40" cancelled, etc.) not visible in this batch. Q-160.
5. **`BillOfMat.ApplySurfTreatment` and `OperationSubstitutionHandler` share a schema contract** but neither documents it. Adding a column to `DtObjects` means updating both.
6. **`MachCycleTime / (60 * 60)` to convert sec → hours, then `Decimal.Round(..., 3)`** — `BillOfMat` assumes the source `MachCycleTime` is in seconds. PartType-, source-, vs unit-conversion conventions aren't documented in this file. Q-164.
7. **All six classes hardcode `IsahUserCode = "ISAH"`** — drift from `Common.APPLISAHUSERCODE = "ICENTER"`. ISAH's audit log will show "ISAH" did all writes from iCenter. Q-165.

## Open questions

- **Q-158 (new):** PBOS.IP_Ins_ProdBOS uses inline T-SQL string concat instead of a parametrised SP call. Refactor?
- **Q-159 (new):** ProductionHeader.SetStatus discards the SP's `@LastUpdatedOn` output param. Cache it on the instance for subsequent calls?
- **Q-160 (new):** ShopDoc status code `"20"` = "started" hardcoded. Document the full status-code enum.
- **Q-161 (new):** Document the `LineNrStepSize = 5` convention (PBOO, PBOS) and the historical reason for the gap.
- **Q-162 (new):** Document `BOOPartDescription` / `ICenterPartCalcPartCode` correlation key — what naming conventions are used?
- **Q-163 (new):** Document the `DtObjects` schema shared between `BillOfMat.ApplySurfTreatment` and `OperationSubstitutionHandler.Execute`.
- **Q-164 (new):** Document unit conventions: `MachCycleTime` in seconds, `MonoMachCycleTime_<code>` in hours after `/3600` rounding.
- **Q-165 (new):** All ISAH writes use `IsahUserCode = "ISAH"`, not `APPLISAHUSERCODE = "ICENTER"`. Audit-trail correctness?

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/icenterlib-isah]] — parent MOC.
- [[isah-dossier|DossierDetail.GenProductionHeader]] — the caller that creates a ProductionHeader.
- [[isah-shop-and-pur-doc|ShopDoc]] — PBOO state writes are delegated through here too.
- [[isah-machgrp|MachGrp]] — `PBOO.GetFirstExistingMachGrpCode` joins through `T_MachGrp` indirectly via BillOfOper.
- [[../modules/workprep-operation-substitution]] — shares the surface-treatment schema with `BillOfMat.ApplySurfTreatment`.

## Coverage

`_coverage.md`:
- `ICenterLib\ISAH\ProductionHeader.vb` → `done` (overview; full ~700 lines deferred to follow-up)
- `ICenterLib\ISAH\PBOO.vb` → `done` (overview; full ~500 lines deferred)
- `ICenterLib\ISAH\PBOM.vb` → `done`
- `ICenterLib\ISAH\PBOS.vb` → `done`
- `ICenterLib\ISAH\BillOfOper.vb` → `done`
- `ICenterLib\ISAH\BillOfMat.vb` → `done` (overview; `ApplySurfTreatment` + headers documented, full file deferred)
