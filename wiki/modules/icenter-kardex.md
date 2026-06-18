---
type: module
title: "iCENTER/Kardex — Kardex Shuttle storage integration"
status: done
module: "iCENTER/Kardex"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Kardex\\KardexProcessor.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Kardex\\FrmKardexInterface.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `iCENTER\Kardex\` — Kardex Shuttle storage integration

> **The bridge between iCenter/ISAH part-dispatch and the Kardex Shuttle vertical-lift storage system.** Physical material moves into and out of Kardex bins are driven by the XML files this module writes (one per warehouse) and consumed by the Kardex's own PowerPick controller via XSLT-transformed orders. `#safety-relevant` — wrong XML → wrong material moved → wrong part assembled.

## Files

- **`KardexProcessor.vb`** (~500 lines) — XML-generator + Kardex DB queries.
- **`FrmKardexInterface.vb`** (~67 lines) — thin WebView shell around `KardexInterfaceUrl` (`My.Settings.Properties("KardexInterfaceUrl").DefaultValue`).

## Public surface (`KardexProcessor`)

```vb
Namespace Kardex
    Public Class KardexProcessor
        ReadOnly PickOrderStyleSheet = "\\JAZO.LOCAL\DFS\JIBA.NET\XSLT\PartDispatch\Isah2KardexOrder.xslt"

        Enum TypeOfDirection
            Put = 1
            Pick = 2
        End Enum

        Enum TypeOfDispatch
            Planned    = 1   ' Productieorder
            Unplanned  = 2   ' Productieorder ongepland
            ReturnJob  = 3
            BackFlush  = 4
            ManualPut  = 5   ' new type created on 9 july 2014
            ProdOrder  = 6   ' new type created on 22 april 2015
        End Enum

        Public  Sub      Generate(DispatchWarehouseCode, Filter, EmpId, ShopDocCode,
                                  DirectionType, DispatchType, FileName)
        Public  Function GetPickOrReturnData(DispatchWarehouseCode, EmpId, ShopDocCode,
                                             Filter, Direction, FromJobStatusId,
                                             TillJobStatusId) As DataTable
        Public  Function GetShopDocCodeByPKey(PKey As Long) As String
        Public  Sub      UpdateShopCart(Pkey, VcAlbh As Decimal)
        Public  Function GetMaterialsByWarehouse(WarehouseName, MaterialName,
                                                 SelectionType) As DataTable
        Public  Function GetHistoryByMaterial(MaterialName) As DataTable
        Public  Function TransFormXml(XmlDoc, XsltTransFormFile) As XmlDocument
        Private Sub      SaveXml(XmlDocument, FileName, DispatchWarehouseCode)
        Private Sub      UpdateJobStatusId(Pkey, JobStatusId)
        Private Function FormatItemValue(Value, Type) As String
    End Class
End Namespace
```

## Behavior — the `Generate(...)` pipeline

This is the **central data path**. Called when the operator confirms a Pick or Put on the Kardex web UI:

1. **Fetch dispatch rows** via SP `JIP_Kdx_Icenter(@DispatchWarehouseCode, @EmpId, @ShopDocCode, @Direction, @FromJobStatusId, @TillJobStatusId)`. For Pick → `JobStatusId 1..5`; for Put → `JobStatusId 5..5`. Q-319 — document the JobStatusId state machine.
2. **Force Dutch culture** via `Thread.CurrentThread.CurrentCulture = "nl-NL"` — guarantees decimal separator + date formatting consistent with Kardex's PowerPick. `#safety-relevant` — a culture-mismatch could swap `1.5` for `15` in numeric fields.
3. **Resolve user full name** via `ICenterLib.JIBA.Employee.GetEmployeeInfoByEmpId(EmpId, FullName)`. Falls back to bare EmpId if JIBA lookup fails (silent catch).
4. **Build `<JibaDataXchange><General><FileName/><DateTimeGenerated/></General><Orders>` XML** — Unicode (UTF-16) encoded, indented.
5. **Per order** (`ProdHeaderOrdNr` grouping):
   - Build `<Order>` with `<DirectionType Name="Put|Pick">{1|2}</DirectionType>`.
   - Apply per-dispatch-type **description rules** (Dutch user-facing text built into the data):
     | DispatchType | DmDescription pattern |
     |--------------|-----------------------|
     | Planned | `{DmDescription} {OrdNr} {DetailCode}-{DetailSubCode} {UserFullName}` (truncated to 80 chars) |
     | Unplanned | `"Ongeplande uitgifte van {UserFullName}"` |
     | ReturnJob | `"Retour opdracht van {UserFullName}"` |
     | ManualPut | `"Handmatige inruim opdracht van {UserFullName}"` |
     | ProdOrder | `"Prod. order inruim opdracht van {UserFullName}"` |
   - Apply per-dispatch-type **ProdHeaderOrdNr suffix rules** (mutating the order-number tag):
     | DispatchType | ProdHeaderOrdNr suffix |
     |--------------|------------------------|
     | Unplanned + IsBackFlush=1 | prefix `BF-` |
     | Unplanned + KD10A | suffix `-10` |
     | Unplanned + other | unchanged |
     | ProdOrder | suffix `-V` |
     | ReturnJob | suffix `-R` |
     | Else KD06A | suffix `-06` |
     | Else KD10A | suffix `-10` |
     | Else other | unchanged |
   - For each **Detail** row:
     - **PKey** stripped of `.000` suffix.
     - **PartCode**: if `PurOnOrdPartCode` is non-empty, write that as both `PartCode` and `KardexPartCode` instead of the regular PartCode. **Effect**: a purchase-order-style PartCode override (`Pur` = purchase) takes precedence. Q-320.
     - **DispatchType = 3 (ReturnJob)** is hardcoded as the wire value for any return-job case, regardless of input.
     - **DispatchWarehouseCode** can be **mutated mid-loop** to `PurWarehouseCode` if `DispatchType = ManualPut`, or to `BfWarehouseCode` if `IsBackFlush = True`. `IsBackFlush` is read but **never assigned True in the visible code** — Q-321, where does it get flipped?
     - **VcALBH (weight)** rules for `KD06A` (the small-parts shuttle):
       - **`Put` direction**: write hard-coded `"0.001"` ("Set the weight to 1 gram, the product will be weighted in PowerPick afterwards"). The Kardex PowerPick weighs it physically and corrects.
       - **`Pick` direction**: passes the VcALBH through as-is (the commented-out `WeightConvertedValue * 1000` is the dead "convert to grams" path).
     - **ProductionDate**: only written for `Put` direction.
   - After detail loop, calls `UpdateJobStatusId(PKey, 2)` — **marks each dispatched detail as "in flight to Kardex"** (JobStatusId=2). `#safety-relevant` — if the XML write fails after status flip, the row is stuck at 2 with no actual Kardex job to clear it.
6. **Transform XML** via `TransFormXml(XmlDoc, PickOrderStyleSheet)` — applies the XSLT `\\JAZO.LOCAL\DFS\JIBA.NET\XSLT\PartDispatch\Isah2KardexOrder.xslt` to produce the final Kardex-format file.
7. **Save** via `SaveXml(XmlDoc, FileName, DispatchWarehouseCode)` to `{Isah2KardexExportPath}\{DispatchWarehouseCode}\{FileName}`. Path comes from `My.Settings.Properties("Isah2KardexExportPath").DefaultValue` — **`.DefaultValue` anti-pattern (Q-002 family)** — Q-322.

## Surprises

1. **`PickOrderStyleSheet` UNC path is a hard-coded constant**: `\\JAZO.LOCAL\DFS\JIBA.NET\XSLT\PartDispatch\Isah2KardexOrder.xslt`. **Single point of failure** — if the file moves or the JAZO.LOCAL DFS is renamed, every Kardex put/pick fails silently (the XSLT load throws, but the outer Try/Catch returns Nothing). `#safety-relevant`. Q-323.
2. **`Generate` writes `Thread.CurrentThread.CurrentCulture = "nl-NL"`** but **never restores** the prior culture. Side-effect leaks into whatever runs next on the same thread (e.g., on the UI thread it sticks). Q-324.
3. **The `If ProdHeaderOrdNr <> OrderRow("ProdHeaderOrdNr").ToString.Trim Then` guard at line 105 is the order-grouping gate.** But the orders DataView is `DtOrders = Dv.ToTable(False, OrderColumns)` — **NOT distincted**. So if the SP returns multiple rows per ProdHeaderOrdNr, only the first triggers the inner block — subsequent rows get skipped. Q-325 — confirm SP guarantees one row per ProdHeaderOrdNr.
4. **`KD06A` (small-parts shuttle)** and **`KD10A` (large-parts shuttle?)** are the two known warehouses; the suffix-mutation logic treats them differently. **`KD%`-prefix convention** matches the [[isah-part-and-dispatch|Part dispatch warehouse-code convention]] (Q-175). Documented as [[../business-rules/icenter-kardex-warehouse-codes]].
5. **`WeightConversionFactor = 1000`** declared but **not used in the active path** (only the commented-out `WeightConvertedValue * WeightConversionFactor` line). Dead variable.
6. **`SqlConn.Close()`** is called after `SqlCmd.Connection.Close()` in many places — double-close. Harmless but suggests author wasn't sure which controlled the underlying connection.
7. **`GetMessages` / `AutoCleanDb` are in a sibling file** (`ClsMarkToolDb.vb`) — no Kardex-side equivalent. Kardex history queries (`GetHistoryByMaterial`) hit Kardex's own DB directly via `Connections.ConnectKardex`.
8. **`FrmKardexInterface`** is just a wrapper: opens `KardexInterfaceUrl?EmpId=...&PartDispatchType=1` in a WebView. The combo box flips PartDispatchType {1=Planned, 2=Unplanned, 3=Retour}. **All Kardex business logic lives in `KardexProcessor`**; the form is presentation only.
9. **`KardexInterfaceUrl` uses `My.Settings.Properties("...").DefaultValue`** — Q-322 same as Q-002.

## Business rules surfaced

- [[../business-rules/icenter-kardex-warehouse-codes|`KD06A` / `KD10A` Kardex warehouse codes]] — small-parts vs large-parts shuttles, with per-warehouse ProdHeaderOrdNr suffix mutations.
- **`KD06A` Put weight defaults to `0.001` kg (1 gram)** — the Kardex PowerPick reweighs physically after placement.
- **JobStatusId 2 = "in flight to Kardex"** — set by `Generate` immediately before the XML file is written.
- **DispatchType 6 (ProdOrder) and 5 (ManualPut)** were **added on specific dates** (`9 july 2014` and `22 april 2015`) per inline comments — preserved as audit-trail evidence.
- **Dutch-only descriptions**: hardcoded `"Ongeplande uitgifte van ..."`, `"Retour opdracht van ..."`, etc.

## Open questions

- **Q-319 (new):** Document the `JobStatusId` state machine (1..5 + 2 written by `Generate`).
- **Q-320 (new):** `PurOnOrdPartCode` override — when does it apply? Purchase-on-order semantics.
- **Q-321 (new):** `IsBackFlush` flag — read multiple times, never assigned True. Where is it flipped?
- **Q-322 (new):** `My.Settings.Properties("...").DefaultValue` for `Isah2KardexExportPath` and `KardexInterfaceUrl` — same anti-pattern as Q-002. Settings panel changes ignored.
- **Q-323 (new):** Hard-coded UNC `\\JAZO.LOCAL\DFS\JIBA.NET\XSLT\PartDispatch\Isah2KardexOrder.xslt` — if the file moves, every Kardex flow fails. `#safety-relevant`
- **Q-324 (new):** `Generate` mutates `Thread.CurrentThread.CurrentCulture` without restoration. Leaks to subsequent thread work.
- **Q-325 (new):** `ProdHeaderOrdNr` grouping assumes the SP returns at most one row per order — confirm.
- **Q-326 (new):** **No transaction across** `UpdateJobStatusId(PKey, 2)` + `SaveXml` — if save fails after status flip, dispatch row is orphaned at status 2. **`#safety-relevant`** — operator can't re-dispatch without manual DB intervention.

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/icenter-remaining]] — parent MOC.
- [[isah-part-and-dispatch|`ISAH.PartDispatch`]] — produces the `JZ_PartDispatch` rows that `KardexProcessor` consumes.
- [[icenterlib-jiba-employee-asset|`JIBA.Employee.GetEmployeeInfoByEmpId`]] — full-name lookup.
- [[../external-systems/kardex|Kardex Shuttle]] — destination system.

## Coverage

- `iCENTER\Kardex\KardexProcessor.vb` → `done`
- `iCENTER\Kardex\FrmKardexInterface.vb` → `done`
- `iCENTER\Kardex\FrmKardexInterface.designer.vb` → `generated`
