---
type: module
title: "ProductionProfileCutItemsHandler — per-machine cut items"
status: done
module: "iCENTER/Production"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Production\\ProductionProfileCutItemsHandler.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `ProductionProfileCutItemsHandler.vb` — per-machine cut items

## Purpose

Writes the **profile cut-items table** for a specific (machine, iCenter-operation, IPPart) tuple — flattens the multi-level BOM of the IPPart's model, filters to parts the production order actually uses, fills `BIdentNo` from EluCad, and persists via `ICenterLib.Production.ProductionProfileCutItemsHandler.Save()`.

Bridges three subsystems:
- **iCenter operations** (the integer `iCenterOperationId`) → **MachGrpCodes** (the routing codes).
- **ICenterObject.GetMBomMultilevel** → flattened BOM with `Modelname`, `Art.nr`, dimensions, angles.
- **Elumatec EluCadApp.GetProfileInfoByPartCode** → per-part `BIdentNo`.

## Public surface

```vb
Public Sub New(machineId As Integer,
               iCenterOperationId As Integer,
               IPPartId As Long)

Public Shared Function OperationIsImplemented(iCenterOperationId As Integer) As Boolean
Public Shared Function GetImplementedOperations() As Integer()

Public Sub Write()                              ' the main entry point
```

The constructor throws `NotImplementedException` if `iCenterOperationId` isn't in the hard-coded dictionary.

## The mapping

```vb
Private Shared Function GetDictionary() As Dictionary(Of Integer, String())
    Dim dict As New Dictionary(Of Integer, String())
    dict.Add(1,  {"A01", "A07"})
    dict.Add(9,  {"S01"})
    dict.Add(31, {"A07", "A01"})
    Return dict
End Function
```

Only three iCenter operations are implemented: **1** (A01+A07), **9** (S01), **31** (A07+A01 — same set as 1 but reversed order). The reversed-order entries suggest the mapping is not just a set — order matters when downstream code iterates MachGrpCodes.

Documented as the [[../business-rules/icenter-operation-machgrp-mapping|iCenter operation → MachGrpCodes mapping]] business rule.

## Behavior in plain language

`Write()`:
1. Call `GetData()` → DataTable of multi-level BOM rows filtered to production-order parts, with `BIdentNo` per part.
2. **Always clear** the existing `ICenterLib.Production.ProductionProfileCutItemsHandler.Clear(MachineId)` — even if the new data is empty. Q-100.
3. If new data has rows, create and `Save()` a new ICenterLib handler.

`GetData()`:
1. `IPPart.GetModelname()` → fail with `Nothing` if empty.
2. `ICenterObject.GetIcenterObject(Modelname)` → fail with `Nothing` if missing.
3. `OrderType = IPPart.GetOrderType()`.
4. `ICO.GetMBomMultilevel(MachGrpCodes, OrderType, True)` — multi-level BOM filtered by *this machine's* MachGrpCodes.
5. Add `BIdentNo` column.
6. `GetFilteredProductionOrderParts(DT, IPPart)` — inner join against `IPOrder2.GetIPorderpartsFromXml(True)` by Modelname.
7. `SetBIdentNo(filteredDT)` — for each unique `Art.nr`, lookup `BIdentNo` from EluCad and apply to all matching rows.

## Surprises

1. **`ClearMachineId` runs unconditionally** even if `GetData()` returns no rows or `Nothing`. A failed model lookup wipes the machine's previously-saved cut items. Q-100.
2. **The mapping table is a `Private Shared Function`**, not a config — adding a fourth operation requires a code change + deploy.
3. **`MachineId = Math.Max(iPPartId, 0)`** in the constructor (line 13). Looks like a copy-paste bug — `iPPartId` is being clamped, not `machineId`. `MachineId` itself isn't normalised. `#needs-review` — Q-101.
4. **Filter uses `RowFilter = "Modelname IN ('a','b',...)"`** built via string concatenation (lines 99–113). If a Modelname contains an apostrophe, the filter breaks. Likely safe since Modelnames are CAD identifiers, but still.
5. **`oEluCad = New Elumatec.EluCadApp`** constructed twice (`Write` line 56 + `SetBIdentNo` line 119). Each construction re-initialises the profile DB schema (same pattern flagged elsewhere).
6. **`OrderType` is consumed by `GetMBomMultilevel`** — different order types produce different BOM views. Phase-3 follow-up on the enum.

## Business rules surfaced here

- [[../business-rules/icenter-operation-machgrp-mapping|`iCenterOperationId → MachGrpCodes` mapping]]: hard-coded `1→A01,A07`, `9→S01`, `31→A07,A01`. The order in entries 1 and 31 differs even though the set is the same — Q-085 asks whether order is load-bearing.

## External systems touched

- [[../external-systems/icenter-db|iCenter DB]] via `ICenterObject`, `ICenterLib.ICenter.IPPart`, `ClsIPorder`.
- [[../external-systems/elumatec-sbz140|Elumatec]] via `EluCadApp.GetProfileInfoByPartCode`.
- iCenter library `ICenterLib.Production.ProductionProfileCutItemsHandler.{Clear, Create, Save}`.

## Open questions

- **Q-085** (from MOC): confirm 1/9/31 are the only iCenter operation IDs; expand with SME-friendly names.
- **Q-100 (new):** `Clear(MachineId)` runs unconditionally. A model-lookup failure wipes the machine's previous cut items. Intentional? `#safety-relevant`
- **Q-101 (new):** `MachineId = Math.Max(iPPartId, 0)` looks like a copy-paste error in the constructor. `MachineId` itself isn't normalised — should it be `Math.Max(machineId, 0)`? `#safety-relevant`

Logged in [[../needs-review/_index]].

## Coverage

`_coverage.md`: `Production\ProductionProfileCutItemsHandler.vb` → `done`.
