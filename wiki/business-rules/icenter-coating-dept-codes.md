---
type: business-rule
title: "Coating dept codes (JCOA / JALU / JSTL / EBTV)"
status: needs-review
module: "iCENTER/Classes/Coating"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Classes\\Coating\\FrmCoatingPick.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Classes\\Coating\\CoatingPickLabel_v3.vb"
last-reviewed: ""
tags: [business-rule, needs-review, safety-relevant]
created: 2026-06-18
updated: 2026-06-18
---

# Coating-related dept codes — `JCOA` / `JALU` / `JSTL` / `EBTV`

## Rule

Four department codes appear repeatedly across the Coating subsystem and drive per-department UI variants, label rendering, and MachGrp resolution:

| Code | Meaning (inferred) | Effect in code |
|------|--------------------|----------------|
| **`JCOA`** | **J**AZO **Coa**ting (internal coating line) | `CoatingPickLabel_v3` renders color + system + remark only if `DeptCode = "JCOA"`; everything else gets `"-"` placeholders. |
| **`JALU`** | **J**AZO **Alu**minium | `FrmCoatingPick.GetPickMachGrpCode("JALU")` → `A16`; `GetSourceMachGrpCode("JALU")` → `A80`. Drives the aluminium-specific MachGrp set. |
| **`JSTL`** | **J**AZO **St**ee**l** | `FrmCoatingPick.GetPickMachGrpCode("JSTL")` → `S16`; `GetSourceMachGrpCode("JSTL")` → `S80`. Drives the steel-specific MachGrp set. |
| **`EBTV`** | **E**lektrolytisch **B**ad **T**hermisch **V**erzinken (external galvanizing partner) | Routed through `S16/S80` same as `JSTL` — but the `CollectEBTVDeptCode = "EBTV"` constant marks the **collect-and-ship** flow. |

The previous-state comment in `FrmCoatingPick.vb` (line 27) preserves history: `Private Const CollectEBTVDeptCode As String = "EBTV" ''"JSTL"` — was JSTL, **changed to EBTV** to separate the collect-for-EBTV flow from the regular JSTL flow.

## Where it lives

- File: `C:\DevOps\iCenter\iCenter\iCENTER\Classes\Coating\FrmCoatingPick.vb`
  - Constants: lines 9-27 (`DestinationMachGrpCodes`, `WareHouseNames`, `CollectEBTVDeptCode`)
  - DeptCode→MachGrpCode resolvers: lines 35-77 (`GetPickMachGrpCode`, `GetSourceMachGrpCode`, `GetCurrentMachGrpCode`)
- File: `C:\DevOps\iCenter\iCenter\iCENTER\Classes\Coating\CoatingPickLabel_v3.vb`
  - JCOA branch: line 150 (`If DeptCode = "JCOA" Then`)

## The code

```vb
' From FrmCoatingPick.vb
Public Shared Function GetPickMachGrpCode(ByVal deptCode As String) As String
    Select Case deptCode
        Case "JALU":           Return "A16"
        Case "JSTL", "EBTV":   Return "S16"
        Case Else:             Return Nothing
    End Select
End Function

Public Shared Function GetCurrentMachGrpCode(deptCode, DestinationMachGrpCode) As String
    Select Case deptCode
        Case "JALU":          Return "A16"
        Case "JSTL", "EBTV":
            If DestinationMachGrpCode = "EBTV" Or DestinationMachGrpCode = "" Then
                Return "S16"
            Else
                Return "S17"           ' post-pick stage
            End If
    End Select
End Function

' From CoatingPickLabel_v3.vb
If DeptCode = "JCOA" Then
    ColorCaption       = Coating.GetColorsByShopDocCode(ShopDocCode)
    CoatingSystem      = Coating.GetSystem(IPbatchId)
    CoatingSystDescr   = oISAH.GetPartInfo(CoatingSystem, "Description")
    Remark             = GetRemark(ShopDocCode)
Else
    CoatingSystDescr   = "-"
End If
```

## Why this is a business rule

These four codes are **the canonical dispatch points** of the coating subsystem:

- **`JCOA`** is JAZO's powder-coat / lacquer line — fully internal, has all metadata (color, system, layer-thickness records).
- **`JALU`** and **`JSTL`** are the **material-split** dept codes that pre-process material (aluminium vs steel) before coating. They share UI structure but use different MachGrp prefixes.
- **`EBTV`** is the **external thermal-zinc partner** — the part leaves JAZO, comes back coated. The "Klaarzetten voor Thermisch Verzinken" / "Afronden" button-text reflects the two-step "prepare for EBTV / close once returned" lifecycle.

The reason `JSTL` was renamed to `EBTV` for the collect-flow constant (per the inline `''"JSTL"` historical note) is that **steel jobs heading to galvanizing are operationally distinct from regular steel work** — they need to be grouped, packed for shipment, and tracked separately.

## Triggers / when it fires

- Every time an operator opens `FrmCoatingPick`, `CoatingPickLabel_v3` prints, or a downstream coating-related query checks DeptCode.

## Effects

- **JCOA** → full coating metadata on labels and screens.
- **JALU** → A-prefix MachGrps (A16 pick / A80 source).
- **JSTL** → S-prefix MachGrps (S16 pick / S80 source).
- **EBTV** → same as JSTL by default, but the EBTV-collect flow uses the `EBTV` const to drive different button text and grouping.

## Edge cases

- **A new material department** (e.g., `JSTS` for stainless) would need source edits in three places (`GetPickMachGrpCode`, `GetSourceMachGrpCode`, `GetCurrentMachGrpCode`) plus possibly `CollectXxxDeptCode` analogues.
- A part-master row with `DeptCode = ""` falls through to `Case Else → Return Nothing` — the UI then likely shows no MachGrp filter at all. Behavior undefined for empty dept code.
- **`JCOA` is treated entirely separately** from `JALU` / `JSTL` / `EBTV` — the rendering branch + the MachGrp branch are independent. A part with `DeptCode = "JCOA"` but no matching JCOA MachGrp will render coating metadata but fail to filter by MachGrp.

## Safety classification

- [x] Touches physical process — yes (routes material to coating lines / external).
- [x] Drives cost / pricing — yes (internal vs external billing).
- [x] Reversible if wrong? — yes (rebook), but **wrong-routed material may already have been physically processed**.
- [ ] Blocks production if it fails? — yes if `DeptCode` is unrecognised.

## SME questions

- **Q-351 (new):** Document the JCOA / JALU / JSTL / EBTV expansion meanings — are these the only material-flow dept codes for coating?
- **Q-352 (new):** What is the lifecycle of an EBTV-bound job — prepare → ship → return → re-receive? Where is the return tracked?
- **Q-353 (new):** Stainless steel — does it have its own dept code, or share JSTL?

Logged in [[../needs-review/_index]].

## Related

- [[../modules/icenter-coating|`iCENTER\Classes\Coating\`]] — defining module.
- [[icenter-coating-executor-enum|`Coating.Executor` enum]] — the parallel "executor" classification.
- [[icenter-jalu-jala-dept-merge|`JALU/JALA` dept merge in identification UI]] — same JALU but different concern.
- [[../modules/isah-leaves|`MultiFinance.GetAdminCodeByDossierCode`]] — dept→entity mapping at the ISAH level.
