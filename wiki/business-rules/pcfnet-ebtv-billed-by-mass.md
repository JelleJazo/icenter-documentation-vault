---
type: business-rule
title: "EBTV galvanizing is billed by MASS (kg) — all other surface treatments by area (m²)"
status: needs-review
module: "ICenterLib/PCFNet"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\PCFNet\\GenericPart.vb"
last-reviewed: ""
tags: [business-rule, needs-review, safety-relevant]
created: 2026-06-18
updated: 2026-06-18
---

# EBTV galvanizing is billed by **MASS** — all other surface treatments by area

## Rule

In the surface-treatment expansion logic, when an operation row exists with `MachGrpCode = 'EBTV'`, the **EBTV external-operation Part's `Qty` field is set to the part's `Mass` value** (kg), **not** the `SurfTreatSquareMeasure` (m²) that all other surface treatments use.

This reflects the **commercial reality** that:
- **Powder coating, paint, lacquer** — billed per **square meter** of coated surface (paint covers area).
- **Galvanizing (EBTV)** — billed per **kilogram** of dipped mass (zinc bath cost scales with mass dipped, plus the part's surface area mostly).

The two billing units are not interchangeable, and the code branches on **MachGrpCode = "EBTV"** to switch from one to the other.

## Where it lives

- File: `C:\Users\jelle-r\source\repos\JIBA\iCenter And Tools\ICenterLib\ICenterLib\PCFNet\GenericPart.vb`
- Symbol: `GenericPart.AddSurfaceTreatment(ds, SystemPartCode, ColorPartCode)`
- Lines 1139-1188 (the EBTV special-handling block).

## The code (minimal quote)

```vb
Try
    Dim EbtvExtOperPartCode As String = AppSettings.GetStringSetting("ExtOperationPartCodeEBTV")
    Dim MyRows As DataRow() = ds.Tables("Part").Select("SubPartCode='" & EbtvExtOperPartCode & "'")
    Dim MyRow As DataRow = If(MyRows.Length > 0, MyRows(0), Nothing)

    If ds.Tables("Operation").Select("MachGrpCode='EBTV'").Length > 0 Then
        Dim drs As DataRow() = ds.Tables("OperationTotal").Select("MachGrpCode='EBTV'")
        If drs.Length > 0 AndAlso Not IsDBNull(drs(0).Item("Mass")) Then
            If MyRow Is Nothing Then
                ' Create new row with Qty = Mass
                Dim Part As New ISAH.Part(EbtvExtOperPartCode)
                ' ... build new row ...
                MyRow.Item("Qty") = drs(0).Item("Mass")     ' ⚠ kg, not m²
            Else
                MyRow.Item("Qty") = drs(0).Item("Mass")
            End If
        Else
            If MyRow IsNot Nothing Then MyRow.Item("Qty") = 0
        End If
    Else
        If MyRow IsNot Nothing Then MyRow.Item("Qty") = 0
    End If
Catch ex As Exception
    Log.NewEntry("EBTV error: " & ex.ToString, MsgBoxStyle.Critical)
End Try
```

For comparison, the regular surface-treatment loop (lines 1057-1102) uses:

```vb
nr.Item("Qty") = (drSurfTreatmentPart.Item("Qty") / drSurfTreatmentPart.Item("CalcQty")) * SurfTreatSquareMeasure
```

where `SurfTreatSquareMeasure` is the **coatable surface area in m²**, computed via `ISAH.BillOfMat.GetSurfTreatmentSquareMeasure`.

## Why this is a business rule

This is a **billing-unit branching rule**. Mixing it up has direct financial impact:

- **Underbilling**: if EBTV were billed by area instead of mass, JAZO would charge zinc-bath time as if it were a paint job — likely undercharging by 5-10× for a heavy steel part.
- **Overbilling**: if powder-coat were billed by mass, customers would pay paint as if it were zinc — likely overcharging.
- **Disputes**: galvanizing partners (EBTV vendor) invoice JAZO in kg; if iCenter's quote uses area, the cost forecast diverges from the actual invoice.

The hardcoded `'EBTV'` MachGrp string is the **branch decision** for the entire flow. Adding a new external-galvanize partner (e.g., a backup vendor) requires either re-using the EBTV MachGrp code (which mis-attributes audit trail) or extending this branch with the new MachGrp's billing rule.

## Triggers / when it fires

- Every `GenericPart.AddSurfaceTreatment(...)` call that ends up with an `EBTV` operation in the dataset.

## Effects

- **EBTV Part `Qty` = part mass (kg)**.
- **Other surface-treatment Part `Qty` = `(partQty / CalcQty) * area_m²`**.
- If no EBTV operation in the dataset, any existing EBTV-PartCode row's `Qty` is **zeroed** (Q-370).
- If `Mass` is DBNull on the OperationTotal row, EBTV Qty is zeroed.

## Edge cases

- **`AppSettings.GetStringSetting("ExtOperationPartCodeEBTV")` is null/empty** → `SubPartCode='@'` filter returns nothing → MyRow stays Nothing → no EBTV row created. **Silent miss**.
- **Mass column is DBNull** → Qty=0. Means "no zinc bath happened" downstream, but in fact the operation row exists. May indicate upstream mass-calc failure rather than zero mass.
- **Multiple EBTV operations** → first row's Mass wins (`drs(0).Item("Mass")`). Multiple EBTV ops in one BOM would lose data. Q-371.
- **Exception path** → only logs `"EBTV error: ..."`; the surrounding non-EBTV surface-treatment part of `AddSurfaceTreatment` proceeds. Partial dataset returned to caller without indication of EBTV miss.

## Safety classification

- [ ] Touches physical process — indirectly (drives quote, which drives sale, which drives production).
- [x] Drives cost / pricing — yes, **directly**.
- [x] Reversible if wrong? — yes (re-quote), but customer-quoted prices may not be retractable.
- [x] Blocks production if it fails? — no, but produces wrong cost forecasts.
- `#safety-relevant` because **misbilling EBTV is a material commercial risk** — 5-10× mis-charge per part on a heavy assembly.

## SME questions

- **Q-370 (new):** When no EBTV operation exists in the dataset, any existing EBTV-PartCode row's Qty is zeroed. Should this row also be removed entirely?
- **Q-371 (new):** Multiple EBTV operations → only first row's Mass is used. Aggregation needed?
- **Q-372 (new):** Document the `AppSettings.GetStringSetting("ExtOperationPartCodeEBTV")` value. What PartCode does it resolve to in production?
- **Q-373 (new):** Are there other external operations with mass-based billing? Should the rule generalise to a per-MachGrp billing-unit table?

Logged in [[../needs-review/_index]].

## Related

- [[../modules/pcfnet-generic-part|`GenericPart.AddSurfaceTreatment`]] — defining method.
- [[icenter-coating-executor-enum|`Coating.Executor.Galvanize = 5`]] — the routing decision.
- [[icenter-coating-dept-codes|`EBTV` dept code]] — sibling rule.
- [[../mocs/icenterlib-pcfnet|PCFNet MOC]].
