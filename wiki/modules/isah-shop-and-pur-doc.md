---
type: module
title: "ISAH ShopDoc + PurDoc"
status: done
module: "ICenterLib/ISAH"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\ShopDoc.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\PurDoc.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# ISAH `ShopDoc` and `PurDoc` — workflow documents

## Purpose

Two thin wrappers around ISAH's per-document tables:

- **`ShopDoc`** — `T_ShopDoc` — the per-operation document that lives on the shop floor. One ShopDoc per `(ProdHeaderDossierCode, ProdBOOLineNr)` pair. Marks "this operation is started / finished".
- **`PurDoc`** — `T_PurchaseDocument` — the purchase document (one per ISAH purchase order).

Both are the back-end side of workflows already documented in [[../modules/workprep-outsource-operations|OutsourceOperationsHandler]] — that handler creates a PurDoc via `PurOrd.CreatePurOrdByExtOperParts` and marks ShopDocs as started via `ShopDoc.SetShopDocStartedInd`.

## `ShopDoc`

**Source**: `ISAH\ShopDoc.vb` (8 KB)

```vb
Public Class ShopDoc
    Public ReadOnly Property ShopDocCode As String

    Public Sub New(ShopDocCode As String)

    Public Shared Function GetShopDocCode(ProdHeaderDossierCode, ProdBOOLineNr) As String

    Public Function GetProdHeaderDossierCode() As String
    Public Function GetShopDocFinInd()         As Boolean
    Public Function GetMachGrpCode()           As String
    Public Function GetOrderType()             As Enums.Application.OrderType

    Public Sub SetShopDocStartedInd(StartedInd As Boolean)   ' delegates to PBOO.SetOperStartInd
    Public Sub SetShopDocFinInd(FinishedInd As Boolean)      ' runs SP JIP_Upd_MyShopDoc
End Class
```

### Behaviour highlights

**Lookup**: `Shared GetShopDocCode(ProdHeaderDossierCode, ProdBOOLineNr)` is the canonical lookup — given the ProductionHeader's dossier code + the bill-of-operations line number, returns the matching ShopDocCode (the primary identifier in this class).

**State**: three read-only flags via inline SQL:
- `GetShopDocFinInd()` — joins `T_ShopDoc ⨝ T_ProdBillOfOper` to read `PBOO.FinishedInd`. **The finished flag lives on PBOO, not on T_ShopDoc.**
- `GetMachGrpCode()` — same join, reads `PBOO.MachGrpCode`.
- `GetOrderType()` — joins `T_ProductionHeader ⨝ T_ShopDoc`, reads `DossierCode`. Returns `SalesOrder` if non-empty, `ProductionOrder` otherwise.

**Mutation**:
- `SetShopDocStartedInd(started)` — **does NOT touch T_ShopDoc**. Delegates to `PBOO.SetOperStartInd(ShopDocCode, True)` (note: the `started` argument is ignored — always passes `True`). Q-142.
- `SetShopDocFinInd(finished)` — runs SP `JIP_Upd_MyShopDoc` with `@ShopDocCode`, `@FinishedInd` (0/1), and (if finishing) `@FinishedDate` (today as `yyyyMMdd`). Idempotent — early-exits if the current state already matches. Writes a `Log.NewEntry` with the action.

### Surprises

1. **The "finished" flag actually lives on `T_ProdBillOfOper`**, not `T_ShopDoc`. `GetShopDocFinInd` joins to PBOO; `SetShopDocStartedInd` writes to PBOO. The class name suggests T_ShopDoc-only, but the data model isn't 1:1. Q-136.
2. **`SetShopDocStartedInd(started)` ignores its argument** — always passes `True` to `PBOO.SetOperStartInd`. Means: you can mark something started but you cannot unmark it via this API. The argument is presumably for future symmetry. Q-142.
3. **`SetShopDocFinInd`** rethrows the SP's exception (after closing the connection) — proper cleanup. Only method in this batch with rigorous exception propagation.
4. **`GetOrderType()` defaults to `SalesOrder` on any failure** — fail-open to the more permissive code path. Q-143.
5. **No `Update` method for the MachGrpCode** — the class exposes only reads on the PBOO-joined columns and writes for the two state flags. Other PBOO mutation lives in `PBOO.vb` (not yet read).

## `PurDoc`

**Source**: `ISAH\PurDoc.vb` (2 KB) — the smallest deep-read in this batch.

```vb
Public Class PurDoc
    Public ReadOnly Property PurDocCode As String

    Public Sub New(PurDocCode As String)

    Public Function GetPurOrdNr()          As String   ' looks up via IP_sel_PurchaseDocumentRecord
    Public Function GetVendId()            As String   ' same
    Public Function GetRecord()            As DataTable

    Public Shared Function CreateByPurOrdNr(Value As String) As PurDoc
End Class
```

### Behaviour highlights

- **Primary key**: `PurDocCode` (numeric internal id, stored as String).
- **Display id**: `PurOrdNr` (the operator-visible purchase-order number).
- **`GetRecord()`** runs SP `IP_sel_PurchaseDocumentRecord` via `DataHandler.GenericQuery` (the modern pattern) — returns the full row.
- **`GetPurOrdNr()` / `GetVendId()`** are thin accessors over `GetRecord()`.
- **`CreateByPurOrdNr(value)`** is the lookup-by-display-id factory — `SELECT PurDocCode FROM T_PurchaseDocument WHERE PurOrdNr = @PurOrdNr`. Returns `Nothing` on no match.

### Surprises

1. **Smallest ISAH class with the most modern style** — `DataHandler.GenericQuery`, `IsNothing` checks, no nested try/catches.
2. **No `Update` method.** Mutations on T_PurchaseDocument must go through `PurOrd.CreatePurOrdByExtOperParts` (the method used by `OutsourceOperationsHandler`). The `PurDoc` class is **read-only**.
3. **Trim on `VendId` but not on `PurOrdNr`** (lines 16 vs 24). Inconsistent.

## Cross-class wiring

The two classes are connected via the outsourcing pipeline:

```
OutsourceOperationsHandler.ProcessDataByVendIdProfileId
    └── PurOrd.CreatePurOrdByExtOperParts(...) → PurDocCode
        └── new ISAH.PurDoc(PurDocCode).GetPurOrdNr() → PurOrdNr (operator display)
    └── for each batch:
        ShopDoc = IPBatch.GetShopDoc
        ShopDoc.SetShopDocStartedInd(True)     ' delegates to PBOO
        IPpart.UpdateCompleted(True)
        IPpart.UpdateCompletedPartsInBatch(IPBatch, True)
        ' SetShopDocFinInd(True) is commented-out — Q-094
```

The commented-out `SetShopDocFinInd(True)` call in [[../modules/workprep-outsource-operations]] is the only place that would mark a ShopDoc as *finished* after outsourcing. Currently dormant.

## Business rules surfaced here

- A ShopDoc's `(ProdHeaderDossierCode, ProdBOOLineNr)` pair is the natural key; `ShopDocCode` is the surrogate.
- ShopDoc state actually lives on `T_ProdBillOfOper`, not `T_ShopDoc`. (Q-136)
- `GetOrderType()` returns `SalesOrder` if the production-header has a non-empty DossierCode, else `ProductionOrder`. Drives whether an order goes through the sales-order workflow.

## Open questions

- **Q-136** (from MOC): document that "started" and "finished" flags live on PBOO, not T_ShopDoc.
- **Q-142 (new):** `ShopDoc.SetShopDocStartedInd(started)` ignores its argument — only ever marks started. Should it support unmarking?
- **Q-143 (new):** `ShopDoc.GetOrderType()` defaults to `SalesOrder` on error. Fail-open — confirm intentional.
- **Q-094** (from prior batch): `SetShopDocFinInd(True)` in OutsourceOperationsHandler is commented out — is the ShopDoc ever marked finished after outsourcing?

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/icenterlib-isah]] — parent MOC.
- [[../modules/workprep-outsource-operations]] — biggest visible consumer.
- [[isah-machgrp]] — MachGrpCode (returned by `GetMachGrpCode`).
- [[isah-identity|`Employee`]] — used in time-registration that joins ShopDocs.

## Coverage

`_coverage.md`:
- `ICenterLib\ISAH\ShopDoc.vb` → `done`
- `ICenterLib\ISAH\PurDoc.vb` → `done`
