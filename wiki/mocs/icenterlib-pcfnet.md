---
type: moc
title: "ICenterLib/PCFNet — sheet-metal calculator + control-definition library"
status: draft
module: "ICenterLib/PCFNet"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\PCFNet\\"
tags: [moc, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# ICenterLib/PCFNet

## What this hub covers

`ICenterLib\PCFNet\` — JAZO's **product-configuration / sheet-metal-calculator** library. 39 source files (~464 KB) + 3 generated. **The configurator's brain**: takes user inputs (via Controls), maps them to PCF (Product Configuration Form) parameters, calculates BOM + BOO, generates parts. The PCFNet name is JAZO-internal.

> **The largest single .vb file in the codebase** lives here: `GenericPart.vb` at 82 KB. **The configurator's "do everything" class.**

## Sub-folder breakdown

```
PCFNet\
├── (root)                       ' GenericPart, CPart, Calculation, ControlDefinition, etc.
├── ParametersMapping\           ' per-product-family parameter-mapping classes (StlDoor001, AluDoor001, etc.)
├── ControlMapping\              ' control → parameter mapping
└── ControlMappingDesigner\      ' designer UI for control mappings
```

## Top files by size

| File | KB | Role |
|------|---:|------|
| `GenericPart.vb` | **82** | the central configurator class — generates configured parts |
| `ParametersMapping\StlDoor001.vb` | 37 | steel-door product family parameter mapping |
| `UCInputControls.vb` | 31 | input-controls UserControl |
| `ExcelObject.vb` | 30 | Excel calculation object |
| `UCOption.vb` | 26 | option UserControl |
| `ControlDefinition.vb` | 25 | control-definition (XML-driven UI) |
| `CPart.vb` | 24 | configurator-part entity |
| `ControlMappingDesigner\UCControlMappingDesigner.vb` | 24 | designer UI for control mappings |
| `Calculation.vb` | 23 | calculation engine |
| `PcfNetDataSet.vb` | 20 | typed dataset |
| `ParametersMapping\AluDoor001.vb` | 18 | aluminium-door product family |
| `ParametersMapping\AluBasicWallLouver001.vb` | 17 | aluminium basic-wall-louver family |
| `PropertyDefinition.vb` | 15 | property-definition entity |
| `ControlMapping\ControlMap.vb` | 10 | control-map entity |
| `BOO.vb` | 9 | PCFNet-side bill-of-operations |

## Notable findings (at a glance)

1. **`GenericPart.vb` at 82 KB is the single biggest .vb file across all three projects.** It's almost certainly the home of a large family of business rules: how a "generic part" gets configured into a real part with material, dimensions, coating, etc. **Phase-4 priority**.
2. **`ParametersMapping\` sub-folder** = **one class per product family**: StlDoor001 (steel door), AluDoor001 (aluminium door), AluBasicWallLouver001 (basic-wall-louver). These map configurator inputs → PCF parameters for each family. The naming convention `<Material><Type>001` suggests sequential generations (001 = first revision).
3. **`ExcelObject.vb` (30 KB)** — Excel-formula-based calculation. The configurator's quote-engine likely runs cell-based formulas inside an Excel object.
4. **`Calculation.vb` (23 KB)** — the calculation engine. Probably the orchestrator that drives the ExcelObject.
5. **`ControlDefinition.vb` (25 KB)** — reads control-definition XML (mentioned in [[icenterlib-productdb-product|`Product.GetControlDefinitionCombinedXml`]]).
6. **`PCFNet.BOO.vb`** vs **`ICenter.BillOfOper.vb`** vs **`ISAH.BillOfOper.vb`** — **three BOO classes** in three folders. Each represents BOO at a different layer. Q-304.
7. **`PcfNetDataSet.vb`** — a typed DataSet definition (likely with adapter). Pre-EF data-binding style.

## Open questions

- **Q-304 (new):** Three BOO classes (`PCFNet.BOO`, `ICenter.BillOfOper`, `ISAH.BillOfOper`). Document the responsibility split.
- **Q-305 (new):** `ParametersMapping\` per-family classes — is the `001` suffix versioning? What's the upgrade path when a family revises?
- **Q-306 (new):** `GenericPart.vb` deep-read needed. 82 KB of business logic, possibly the **single biggest business-rule lode**.
- **Q-307 (new):** `ExcelObject.vb` — does it open real Excel files at calc time? Performance / dependency concern.

Logged in [[../needs-review/_index]].

## Coverage decision

**All 39 source files marked `done` overview-level via this MOC.** Top 5 by size + `GenericPart` flagged for **Phase 4 deep-read** as a priority.

Generated `*.Designer.vb` marked `generated`.
