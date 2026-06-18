---
type: business-rule
title: "SMT deburr cycle-time formula (area × speed × material multipliers)"
status: needs-review
module: "ICenterLib/iCenter"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\Part.vb"
last-reviewed: ""
tags: [business-rule, needs-review, safety-relevant]
created: 2026-06-18
updated: 2026-06-18
---

# SMT deburr cycle-time formula

## Rule

The cycle-time (in the same time-unit as `AppSettings.SmtDeburrSpeed`) for the SMT-deburring station is computed from sheet area, base speed, and two multipliers:

```
base   = (OutlineX/1000) × (OutlineY/1000) / DeburrSpeed       (area in m² / speed)
if Thickness ≥ 4 mm  AND  Material code starts with "AL"  →  base *= 2
if FlowGrillQuality flag set                              →  base *= 2
return base
```

So a 4mm aluminium FlowGrill sheet is **4× the baseline** cycle time.

## Where it lives

- File: `ICenterLib\iCenter\Part.vb` lines 7-25
- Symbol: `Shared Function Part.GetSmtDeburrCalcCycleTime(SmtOutlineX, SmtOutlineY, SmtThickness, SmtMaterialId, SmtFlowGrillQuality) As Double`

## The code (verbatim — the formula is the rule)

```vb
Dim CalcCycleTime As Double = 0
Dim DeburrSpeed As Double = AppSettings.SmtDeburrSpeed
CalcCycleTime = (SmtOutlineX / 1000 * SmtOutlineY / 1000) / DeburrSpeed

If SmtThickness >= 4 AndAlso SmtMaterialId.ToUpper.StartsWith("AL") Then
    ''Niet netjes, eigenlijk oplossen door de T_SmtMaterials tabel aan te vullen
    ''Maar dat betekent weer een extra query, tenzij ik alles ombouw.
    CalcCycleTime = CalcCycleTime * 2
End If

If SmtFlowGrillQuality Then
    CalcCycleTime = CalcCycleTime * 2 ''FlowGrill altijd 2x afbramen of op halve snelheid
End If

Return CalcCycleTime
```

## Why this is a business rule

Three rules baked into one function:

1. **Base formula**: `area ÷ speed`. Units depend on `AppSettings.SmtDeburrSpeed` (probably m²/h or m²/min). If the speed setting has the wrong units, every downstream cycle estimate is wrong.

2. **Thick aluminium = 2×**. Aluminium ≥4mm produces more burrs / requires extra passes. Author's comment admits this is a hack — the right fix is per-material-thickness coefficients in `T_SmtMaterials`. **Hardcoded magic threshold `4` mm**.

3. **FlowGrill quality = 2×**. FlowGrill is JAZO's subsidiary / brand serving the **fine-mesh / decorative-grille market**. FlowGrill orders demand higher surface quality so deburring runs at half speed (= 2× cycle time).

All three rules drive **production planning** and **price calculation** for SMT-deburred parts.

## Triggers / when it fires

- Anywhere `Part.GetSmtDeburrCalcCycleTime(...)` is called. Phase-3 follow-up to find consumers (likely SMT cycle-time aggregator in `SmtManufacturing` or PCFNet).

## Effects

- Returned cycle-time feeds:
  - **Scheduling** (lead-time at the deburr station).
  - **Costing** (machine-hour × deburr-time = cost component).
  - Potentially the planned-vs-actual comparator (`IPPart.EvalVcNc`).

## Edge cases / known exceptions

- `SmtMaterialId.ToUpper.StartsWith("AL")` — matches "AL", "ALU", "AL5754", "AL-anything", but also `"ALPACA"` etc. if there's such a material. Q-252.
- The 4mm threshold is in millimetres; outline is in mm but converted to m for the area calculation. Don't conflate.
- Aluminium <4mm doesn't get the 2× multiplier — but the FlowGrill multiplier still applies if set.
- Both multipliers can stack (aluminium FlowGrill ≥4mm = 4×).
- `DeburrSpeed = 0` → division by zero. Q-253 — guard?
- `SmtMaterialId Is Nothing` → NullReferenceException on `.ToUpper`. Q-254.

## Safety classification

- [ ] Touches physical process — no (just computes a time estimate).
- [x] Drives cost / pricing — yes.
- [x] Reversible if wrong? — yes (re-run estimate); but quoted prices to customers are not reversible.
- [ ] Blocks production if it fails? — no.
- `#safety-relevant` because the formula is the single source of truth for SMT deburr cycle time — a refactor that miscomputes affects every quote.

## SME questions

- **Q-252 (new):** `StartsWith("AL")` could match non-aluminium materials (e.g., `ALPACA`, `ALLOY-X`). Tighten to exact-code list?
- **Q-253 (new):** `DeburrSpeed = 0` → div-by-zero. Guard or fail-fast?
- **Q-254 (new):** `SmtMaterialId Is Nothing` → NRE. Should this default to non-AL or throw?
- **Q-255 (new):** Author's comment proposes adding per-material-thickness coefficients to `T_SmtMaterials`. Should this be ticketed?
- **Q-256 (new):** What is `AppSettings.SmtDeburrSpeed`'s unit? m²/h or m²/min? Where is it set?

Logged in [[../needs-review/_index]].

## Related

- [[../modules/icenterlib-icenter-leaves|`Part`]] — defining module.
- [[../mocs/icenterlib-icenter]] — parent MOC.
- [[icenter-flowgrill-quality|FlowGrill quality flag]] (TODO) — the related multiplier-trigger flag.
