---
type: module
title: "iCenter batch hierarchy — IPorder/IPbatch/IPpacket/IPpart + BillOfOper"
status: done
module: "ICenterLib/iCenter"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\IPOrder.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\IPBatch.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\IPPacket.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\IPPart.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\BillOfOper.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# iCenter batch hierarchy — `IPorder` → `IPbatch` → `IPpacket` → `IPpart`

## The 4-level model

iCenter's **production-batch decomposition** has four levels. Each row at level *N* points at one row at level *N-1*:

```
T_IPorderLines      IoId         (1 per production-order)
└── T_IPbatchLines  IbId         (1 per MachGrp under the order)
    └── T_IPpacketLines IpId     (1 per Modelname under the batch)
        └── T_IPpartLines  IPpartId  (1 per physical part / final-product instance)
```

This is the **shop-floor decomposition** of a production order — distinct from the **ISAH-side BOM** (`T_ProdBillOfMat`) and **PBOO** ([[isah-production-hierarchy]]).

Each level has its own wrapper class with `New(Id As Long)`, `GetMetaData()`, parent/child traversal, and per-table mutations.

## `IPOrder.vb` — top level

205 lines.

```vb
Public Class IPOrder
    Public Property Id As Long          ' = IoId

    Public Sub New(Id As Long)
    Public Function GetBomFilteredByMachGrpCode(MachGrpCodes As String) As DataTable    ' XML-shredding query
    Public Function DeleteSmtProdOrd(oseonAppContext) As Boolean                        ' delete from Oseon for {P44,P46}
    Public Function GetIPBatch(MachGrpCode As String) As IPBatch
    Public Function GetIPBatches(MachGrpCodes As String()) As List(Of IPBatch)
    Public Function GetIPbatchId(MachGrpCode As String) As Long
    Public Function GetOrderType() As Enums.Application.OrderType
    Public Function GetMetaData() As DataTable
    Public Shared Function GetOrdNrByIPorder(IPorderId) As String
End Class
```

### `GetBomFilteredByMachGrpCode(MachGrpCodes)`

The interesting one. Takes a **comma-separated list** of MachGrpCodes. Builds an XML `<i>code1</i><i>code2</i>...` token list via `REPLACE`, then uses XQuery to extract `(DesignCode, MachGrpCode)` pairs from `T_IPorderLines.XmlParams`:

```sql
SELECT o.x.value('(@Modelname)[1]','varchar(35)') AS DesignCode,
       p.x.value('(@Name)[1]','varchar(35)')      AS MachGrpCode
FROM dbo.T_IPorderLines IPL
  CROSS APPLY XmlParams.nodes('(/ORDER/PARTS/PART)') AS o(x)
  CROSS APPLY o.x.nodes('OPERATIONS/OPER') AS p(x)
WHERE IoId = @IoId AND p.x.value('(@Name)[1]','varchar(35)') IN (
    SELECT RTRIM(LTRIM(i.value('.','nvarchar(8)'))) AS MachGrpCode
    FROM @MachGrpCodeXML.nodes('for $i in /i where $i != "" return $i') AS T(i))
```

So `T_IPorderLines.XmlParams` carries the **ORDER/PARTS/PART → OPERATIONS/OPER** tree — the master BOM in XML form. Q-224 — what writes this XML? (Probably `Icenter2Isah` or a sibling sync.)

The commented-out alternative single-MachGrpCode version is preserved as comments — useful to know the simpler shape.

### `DeleteSmtProdOrd(oseonAppContext)`

Specifically iterates **`{"P44", "P46"}`** MachGrp codes (the SMT-production groups) and calls `IPBatch.DeleteSmtProdOrd` on each. Collects exceptions via `MySystem.ExceptionList`, throws aggregate if any failed. Returns `True` if any single batch was deleted.

**`{P44, P46}` is hardcoded** — adding a third SMT MachGrp requires source edit. Q-228.

## `IPBatch.vb` — per-MachGrp batch

342 lines.

```vb
Public Class IPBatch
    Public ReadOnly Property Id As Long          ' = IbId
    Public Property MachGrpCode As String        ' set by IPOrder.GetIPBatch{es}

    Public Function GetIPParts(IgnoreCompleted)                                       As DataTable
    Public Function GetFirstIPpartId(Modelname)                                       As Long
    Public Function GetPartsAreCompleted()                                            As Boolean
    Public Function GetGroupedIPParts()                                               As DataTable   ' SP SIP_Sel_GroupedIPparts
    Public Function GetIncompletedPartsCount()                                        As Integer
    Public Function GetCompletedPartsCount()                                          As Integer
    Public Function GetShopDocCode()                                                  As String
    Public Function GetShopDoc()                                                      As ISAH.ShopDoc
    Public Function GetProdHeaderDossierCode()                                        As String
    Public Function GetOrdRef()                                                       As String
    Public Function GetIoId()                                                         As Long
    Public Function GetMetaData()                                                     As DataTable
    Public Function DeleteSmtProdOrd(oseonAppContext)                                 As Boolean   ' refuses if started

    Public Shared Function GetIPpartsByIPbatch(IPbatchId, IgnoreCompleted)            As DataTable
    Public Shared Function GetShopDocCodeByIPbatchNr(IPbatchId)                       As String
    Public Shared Function GetProdHeaderDossierCodeByBatchId(IPbatchId)               As String
    Public Shared Function GetOrdRefByIPbatchId(IPbatchId)                            As String
    Public Shared Function CreateByProdHeaderDossierCodeAndMachGrpCode(prodHeader, mg) As IPBatch
    Public Shared Function GetIPbatchIdByProdHeaderDossierCodeAndMachGrpCode(prodHeader, mg) As Long
End Class
```

### `GetPartsAreCompleted()`
**Important defensive comment**: `'Do not use IgnoreCompleted = True. If there is a database problem the batch might be considered completed while it's not.` — so the function loads ALL parts (`IgnoreCompleted=False`) and only returns True if every row has `Completed=1`. If the DB query throws and returns empty, an empty `DT.Rows.Count = DV.Count = 0` would return True — **bug**: empty result counts as "all complete". Q-229 `#safety-relevant`.

### `DeleteSmtProdOrd(oseonAppContext) As Boolean`

Refuses to delete if `GetCompletedPartsCount() > 0`. Throws Dutch exception: `"{MachGrpCode}: Opdrachten uit Boost verwijderen is niet mogelijk. Productie is reeds gestart."` (Boost = JAZO's name for the Oseon-based SMT production system). Otherwise calls `SmtProduction.DataServices.ProductionOrderDataService.DeleteICenterBatch(Id)`. Documented as [[../business-rules/icenter-boost-delete-blocked]].

### `GetGroupedIPParts()`
SP `SIP_Sel_GroupedIPparts` — likely groups parts by Modelname returning Modelname + Qty per group. Used by `IPPart.GetQtyInBatch`.

## `IPPacket.vb` — per-Modelname grouping

82 lines, three methods:

```vb
Public Class IPPacket
    Public ReadOnly Property Id As Long          ' = IpId

    Public Function UpdateCompletedByModelname(Modelname As String, Completed As Boolean) As DataTable
        ' UPDATE T_IPpartLines SET Completed=@Completed WHERE IpId=@IpId AND Modelname=@Modelname

    Public Function GetIPpartsByIPpacket() As DataTable
    Public Function GetMetaData() As DataTable     ' projects IpId/IbId/IpDisabled/IpDone/NextOper
End Class
```

The simplest of the four. `UpdateCompletedByModelname` is the **per-Modelname mark-as-done** — sets `Completed` flag on all IPpartLines in this packet matching the model. **Returns a DataTable** but the SQL is an UPDATE — `SqlDataAdapter.Fill` on a non-SELECT yields an empty DataTable. So the return value is always empty — Q-230, dead return value.

## `IPPart.vb` — the leaf level

492 lines, 20+ methods. The most-touched class in this folder.

```vb
Public Class IPPart
    Public ReadOnly Property Id As Long          ' = IPpartId

    Public Sub New(Id As Long)
    Public Sub New(IPpartId As String)           ' parses IPpartId.Replace(Common.IPPARTPREFIX, "")

    Public Function PartDispatchExists(MachGrpCode)                                As Boolean
    Public Function GetCompleted()                                                 As Boolean   ' from T_IPpartLines.Completed
    Public Function GetPartDispatch(MachGrpCode)                                   As DataTable ' SELECT FROM T_IPpartDispatch
    Public Sub InsertPartDispatch(MachGrpCode)                                                ' INSERT INTO T_IPpartDispatch
    Public Function GetProdHeaderDossierCode()                                     As String   ' 4-level join up
    Public Function GetModelname()                                                 As String
    Public Function GetIPpartInfo(ReturnField As String)                           As Object   ' dynamic-column lookup on T_IPpartLines
    Public Shared Function GetIPpartInfo(IPpartId, ReturnField)                    As Object
    Public Sub SetPartIdentifierInfo(ByRef Location, ByRef ShopDocCode, ByRef NextOper)        ' 3-out via ByRef
    Public Function GetIPorderByIPpart()                                           As Long     ' resolves IoId
    Public Function GetIPbatchId()                                                 As Long     ' resolves IbId
    Public Function GetOrderType()                                                 As Enums.Application.OrderType
    Public Function GetIPOrder()                                                   As IPOrder
    Public Function GetIPBatch()                                                   As IPBatch
    Public Function GetQtyInBatch()                                                As Double   ' from SIP_Sel_GroupedIPparts
    Public Sub UpdateCompleted(Completed As Boolean)
    Public Sub UpdateCompletedPartsInBatch(IPBatch As IPBatch, Completed As Boolean)
    Private Sub UpdateCompletedByBatch(IPBatchId, Modelname, Completed)                       ' the actual UPDATE
    Public Function GetIPOrderPartOpers()                                          As BillOfOper
    Public Function SIP_Get_PartOpers(Modelname As String, MachGrpCode As String)  As DataTable
    Public Function GetFirstMachGrpCodeOfSubPart(Modelname)                        As String
    Public Sub EvalVcNc(MachGrpCode, NcCycleTime, Qty, TraceKey)                              ' XML report → Log.NewEntry
End Class
```

### `Sub New(IPpartId As String)` — prefix parsing

```vb
Id = Long.Parse(IPpartId.Replace(Common.IPPARTPREFIX, ""))
```

Confirms there is a **`Common.IPPARTPREFIX`** string that gets prepended/stripped on serialised IPpartIds. Q-222 — what's the prefix?

### `SetPartIdentifierInfo(ByRef Location, ByRef ShopDocCode, ByRef NextOper)`

Three-output via ByRef. Joins `T_IPbatchLines + T_IPpacketLines + T_IPpartLines` filtered by `IbDisabled=0 AND IpDisabled=0 AND IpartDisabled=0`. Used by callers that need all three identifiers in one query (avoids N+1).

### `EvalVcNc(MachGrpCode, NcCycleTime, Qty, TraceKey)`

The **NC-vs-Vc (Velocity-Cycle) cycle-time comparator**. Compares actual NC cycle time vs the planned cycle time from BOO, emits a `<ncvc>` XML report to `Log.NewEntry(SB.ToString, MsgBoxStyle.Information)`:

```vb
Dim t As Decimal = PBOO.GetCycleTime(MachGrpCode)        ' planned per-MachGrp from BOO
Dim CalcQty As Decimal = GetIPpartInfo("Qty")
Dim CycleTimePerPiece As Decimal = t
If CalcQty > 0 Then
    CycleTimePerPiece = CycleTimePerPiece / CalcQty
End If

Dim Diff As Decimal = If(CycleTimePerPiece = 0, 0, NcCycleTime / CycleTimePerPiece)
' XML emit with <value>{Diff:N4}</value>
```

**Three bugs visible**:
- The XML body has `<cycletimeperpiece>{TraceKey}</cycletimeperpiece>` and `<nccycletime>{TraceKey}</nccycletime>` — both wrongly using `TraceKey` as the value (should be `CycleTimePerPiece` and `NcCycleTime`). Q-231 `#safety-relevant`.
- Same `TraceKey` is used 3 times in the format calls (line 481-482), only the first usage (`<tracekey>`) is correct.
- `Diff = NcCycleTime / CycleTimePerPiece` — divisor is per-piece-planned, so `Diff` is a **ratio** (e.g., 1.05 = NC took 5% longer than planned). The XML field is labelled `<value>` — non-obvious unit.

### `UpdateCompletedByBatch`

Bulk-update via SQL `IN (subquery)`:
```sql
UPDATE T_IPpartLines SET Completed = @Completed
WHERE IPpartId IN (
    SELECT IPRL.IPpartId FROM T_IPpartLines IPRL
    INNER JOIN T_IPpacketLines IPL ON IPL.IpId = IPRL.IpId
    WHERE IPL.IbId = @IbId AND IPRL.Modelname = @Modelname)
```

Marks all parts of a given Modelname within a batch as complete/incomplete simultaneously. **Internal `Catch ex As Exception` is empty** (line 393) — silent failure. Caller can't tell if the update worked.

### `InsertPartDispatch`

**Re-throws** wrapped in a generic `Throw New Exception(ex.ToString, ex)`. Unusual — most other methods swallow.

## `BillOfOper.vb` — in-memory bill-of-operations

58 lines. Thin DataTable wrapper with three columns: `LineNr` + `OrderType` (composite PK), `MachGrpCode`, `MachSetupTime` (Decimal), `MachCycleTime` (Decimal).

```vb
Public Class BillOfOper
    Public Shared Function CreateFromDataTable(Value As DataTable) As BillOfOper
        ' requires MachGrpCode + MachSetupTime + MachCycleTime columns
        ' throws if missing, copies rows via AddOperation

    Public Function GetCycleTime(MachGrpCode As String) As Decimal
        ' _DT.Compute("SUM(MachCycleTime)", "MachGrpCode='" & MachGrpCode & "'")

    Public Sub AddOperation(MachGrpCode, MachSetupTime, MachCycleTime)
        ' OrderType always 0 — comment: "OrderType always 0. Handling code must filter the ordertypes"
End Class
```

`OrderType = 0` is **hardcoded** with comment `'CB 2022-09-29: OrderType always 0. Handling code must filter the ordertypes` — the field is composite-PK but only one value is ever used. Q-232 — vestigial? Remove from PK?

**`_DT.Compute("...", "MachGrpCode='" & ... & "'")`** — single-quoted MachGrpCode interpolated into the filter. If a MachGrpCode could contain `'`, this would break / inject. MachGrpCodes are short alphanumeric codes (`P44`, `M38`, `TR05`) — probably safe in practice. Q-233.

## Surprises

1. **`IPOrder.GetBomFilteredByMachGrpCode`** uses `T_IPorderLines.XmlParams` XML-shredding (Q-224). The XML schema isn't documented elsewhere.
2. **`{P44, P46}`** hardcoded in `IPOrder.DeleteSmtProdOrd` — the SMT MachGrps. Q-228.
3. **`IPBatch.GetPartsAreCompleted()` returns `True` on empty DataTable** — DB error → batch shows complete. Q-229 `#safety-relevant`.
4. **`IPPart.EvalVcNc`** emits XML with `TraceKey` substituted in 3 fields where 2 should be `CycleTimePerPiece` / `NcCycleTime`. Q-231 `#safety-relevant`.
5. **`IPPacket.UpdateCompletedByModelname`** returns a DataTable that's always empty (UPDATE statement run through SqlDataAdapter.Fill). Q-230.
6. **`Common.IPPARTPREFIX`** mentioned but not defined here. Q-222.
7. **Inconsistent error handling**: `IPPart.InsertPartDispatch` rethrows, `IPPart.UpdateCompletedByBatch` swallows silently. Same class.
8. **`BillOfOper` composite PK includes OrderType but OrderType is always 0** — vestigial design. Q-232.

## Business rules surfaced

- [[../business-rules/icenter-boost-delete-blocked|Boost delete blocked once parts have started]] — `IPBatch.DeleteSmtProdOrd`.
- **`{P44, P46}` are JAZO's SMT-production MachGrps** — hardcoded in `IPOrder.DeleteSmtProdOrd`.

## Open questions

- **Q-222 (existing):** Document `Common.IPPARTPREFIX`.
- **Q-224 (existing):** Document `T_IPorderLines.XmlParams` schema.
- **Q-228 (new):** `{P44, P46}` hardcoded SMT MachGrp set. New SMT MachGrp = source edit.
- **Q-229 (new):** `IPBatch.GetPartsAreCompleted` returns True on empty DataTable. DB error → "all done". `#safety-relevant`
- **Q-230 (new):** `IPPacket.UpdateCompletedByModelname` returns always-empty DataTable. Dead return value.
- **Q-231 (new):** `IPPart.EvalVcNc` XML emit uses `TraceKey` in 3 fields where 2 should be `CycleTimePerPiece` / `NcCycleTime`. `#safety-relevant`
- **Q-232 (new):** `BillOfOper.OrderType` always 0. Composite PK but unused dimension.
- **Q-233 (new):** `BillOfOper.GetCycleTime` interpolates MachGrpCode into DataTable filter — quoted. Probably safe given alphanumeric MachGrpCodes.

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/icenterlib-icenter]] — parent.
- [[isah-production-hierarchy|`PBOO` / `BillOfOper`]] — ISAH-side BOO; this folder's `BillOfOper` is the iCenter shop-floor copy.
- [[isah-part-and-dispatch|`Part` + dispatch in ISAH]] — `T_IPpartDispatch` (iCenter) parallels the part-dispatch flow.
- [[../external-systems/oseon|Oseon / Boost]] — `DeleteSmtProdOrd` deletes from Oseon.

## Coverage

- `iCenter\IPOrder.vb` → `done`
- `iCenter\IPBatch.vb` → `done`
- `iCenter\IPPacket.vb` → `done`
- `iCenter\IPPart.vb` → `done`
- `iCenter\BillOfOper.vb` → `done`
