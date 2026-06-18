---
type: module
title: "iCENTER/Classes/Coating — surface-treatment process subsystem"
status: done
module: "iCENTER/Classes/Coating"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Classes\\Coating\\Coating.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Classes\\Coating\\IControlCoating.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Classes\\Coating\\EbtvSticker.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Classes\\Coating\\CoatingPickLabel_v3.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Classes\\Coating\\FrmCoatingPick.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `iCENTER\Classes\Coating\` — surface-treatment process subsystem

> **The shop-floor "Coating" subsystem.** Powder-coat, paint, galvanize, lacquer flows. Drives the per-batch material picking, label printing, layer-thickness recording, and Kardex collect-then-distribute pipeline that feeds the coating lines. `#safety-relevant` — coating affects part appearance, corrosion resistance, and customer acceptance.

## Files documented in this batch

| File | KB | Role |
|------|---:|------|
| `Coating.vb` | 2 | the entity + `Executor` enum + 2 static helpers |
| `IControlCoating.vb` | 0.3 | interface for the coating-display controls |
| `EbtvSticker.vb` | 5 | **`#dead-code`** — comment says "Vervallen!!!" (Obsolete!!!) |
| `CoatingPickLabel_v3.vb` | 14 | A4 portrait label printer (the v3 supersedes v1/v2) |
| `FrmCoatingPick.vb` | 58 | the central coating-pick UI (top 120 lines deep-read) |

## Other files in the folder (overview-tag via [[../mocs/icenter-remaining]])

- `FrmCoatingPick.vb` + `.Designer.vb` — 58 KB UI (head-only deep-read)
- `CtrlCoatingPickLocation.vb` + `.Designer.vb` — 54 KB UserControl (locations grid)
- `ControlCoatingGrouped.vb` + `.Designer.vb` — 43 KB grouped view
- `ControlCoating.vb` + `.Designer.vb` — 36 KB base view
- `FrmGetCoatingMaterials.vb` + `.Designer.vb` — 23 KB materials picker
- `FrmAddLayerThickness.vb` + `.Designer.vb` — 9 KB measurement input
- `FrmGetCoatingNextJob.vb` + `.Designer.vb` — 7 KB next-job dialog
- `FrmCoatingLayerThickness.vb` + `.Designer.vb` — 5 KB layer-thickness list
- `DataGridViewProgressColumn.vb` — 5 KB progress-bar grid column
- `frmKardexJobIncomplete.vb` + `.Designer.vb` — 1.3 KB incomplete-job alert
- `frmScanNext.vb` + `.Designer.vb` — 1.3 KB scan-next dialog
- `DebugLog.vb` — 1.4 KB debug helper

## `Coating.vb` — the entity

```vb
Public Class Coating
    Public Enum Executor
        None                = 0
        InternalPowderCoat  = 1
        ExternalCoating     = 2
        ExternalOther       = 3
        InternalLacquer     = 4
        Galvanize           = 5
    End Enum

    Public Shared Function GetColorsByShopDocCode(ShopDocCode As String) As String
    Public Shared Function GetSystem(Optional IPbatchId = -1, Optional ShopDocCode = Nothing) As String
End Class
```

### `Executor` enum — the **5 coating-execution paths**

1. **InternalPowderCoat (1)** — JAZO's own powder-coat line.
2. **ExternalCoating (2)** — sent to a coating partner.
3. **ExternalOther (3)** — sent to a different external (probably plating / anodizing).
4. **InternalLacquer (4)** — JAZO's own paint/lacquer line.
5. **Galvanize (5)** — galvanizing (probably EBTV partner — see [[../business-rules/icenter-coating-dept-codes]]).

Documented as [[../business-rules/icenter-coating-executor-enum]].

### `GetColorsByShopDocCode(ShopDocCode)`

Reads `oISAH.GetCoatingPartsByShopDocCode(ShopDocCode)` → DataTable. Concatenates `PartCode` values with spaces. **Comment line 17-18** preserves a Kardex anti-pattern: `"Niet gebruiken, zorgt voor lege waarde bij EmpId in JZ_PartDispatch"` — "don't use this approach, it leaves an empty EmpId in JZ_PartDispatch". Historical workaround.

### `GetSystem(IPbatchId, ShopDocCode)`

Returns `CoatingSystem` value from `oICENTER.GetIPbatchInfo` / `ClsIPbatch.GetIorderProperties`. The **coating system** (e.g., "RAL 9006 powder-coat") is a per-batch attribute carried alongside the IPbatch.

## `IControlCoating` — the polymorphic display interface

```vb
Public Interface IControlCoating
    Property Name As String
    Property Dock As DockStyle
    Sub HighlightChildren(ParentModelname As String)
    Sub SetAdminUsername(Value As String)
    Sub FormatDataGridViews()
    Sub CreateControl()
    Sub Show()
End Interface
```

Two implementations: `ControlCoating` (single view) and `ControlCoatingGrouped` (grouped view). Allows the parent UI to flip between display modes without conditionals.

## `EbtvSticker` — **`#dead-code`**

```vb
''' <summary>
''' Vervallen!!!
''' </summary>
Public Class EbtvSticker
    ...
End Class
```

XML doc-comment header says "**Vervallen!!!**" (Dutch for "Discarded/Obsolete!!!"). The `Print()` method body is **entirely commented out** (lines 33-69). The `pd_PrintPage` callback is still defined but unused. Constructor still runs (it queries `Client.GetPrefLabelPrinter`). Q-340 — confirm `#dead-code`, then remove.

(EBTV = JAZO's external galvanizing partner.)

## `CoatingPickLabel_v3.vb` — the A4 coating-pick label

Renders an A4 portrait label (827×1169 — paper-size units) via `PrintDocument`. Key constants:

- **Paper**: `A4`, 827×1169, margins `(50, 50, 50, 50)` (Altec-printer-specific margins).
- **Fonts**: Arial 14 (default), 10 (small), 24 (medium), 86 (large), 180 (XLarge), IDAutomationHC39M 14 (Code 39 barcode).
- **Coating-only rendering**: `If DeptCode = "JCOA" Then` — color, coating-system description, and remark are only fetched/printed when the order is coating-bound. Other departments get `"-"` placeholders. JCOA = the canonical Coating department code.
- **Print-on-failure**: if `oISAH.GetShopDocDetails(ShopDocCode)` returns empty, the label is **still printed** with explicit warning text:
  > `Project = "** Bewerking niet gevonden! **"` ("Operation not found")
  > `ColorCaption = "Actie: bel Werkvoorbereiding"` ("Action: call Work Preparation")
  > `CoatingSystDescr = "Bewerking bestaat niet meer in ISAH"` ("Operation no longer exists in ISAH")
  > `MachGrpCode = "- Werkkaartnummer"`
  
  **Operator gets a usable-but-error-flagged label** with instructions on what to do. `#safety-relevant` — defensive UX.
- **Fallback to `PdfCreatorPath`** if no preferred label printer is found (`Client.GetPrefLabelPrinter` returns Nothing). Logs `"No preferred printer found for {Computername}"` via `oApplicationLog`. **Then pops MsgBox** `"Let op: printer = {PdfCreatorPath}"`. Q-341 — what does PdfCreatorPath resolve to? PDF generator that the operator must then print manually?
- **`PaperKind.A4` + explicit 827×1169**: redundant — `PaperKind.A4` should set the size, but the constants are also provided. Q-342.
- **`GetRemark(ShopDocCode)`** — pulls a special SubPart with `SubPartCode = 'OPMERKING'` (Dutch "Remark"); concatenates all `SubPartDesc` values with `; ` separator. Filter `Counter=0 AND MachGrpCode='{MachGrpCode}'`. Hard-coded magic SubPartCode. Q-343.

## `FrmCoatingPick.vb` (head-only) — central coating-pick UI

Key constants visible in the head:

```vb
Private DestinationMachGrpCodes As String() = {"F32", "F33"}
Private WareHouseNames As String() = {"KD05A", "KD10A"}
Private Const DefaultDestinationMachGrpCode As String = "F32"
Private Const CollectEBTVDeptCode As String = "EBTV"   ' "Was JSTL"
Private Const BtnCollectEbtvText1 As String = " Klaarzetten voor Thermisch Verzinken"
Private Const BtnCollectEbtvText2 As String = " Afronden"
```

### `WareHouseNames = {"KD05A", "KD10A"}`

**The coating-pick UI sees `KD05A` not `KD06A`**. The Kardex MOC documents `KD06A` (small-parts) — Q-344 documents the **possible mismatch** between Coating and Kardex warehouse-code conventions. Either:
- Coating uses `KD05A` (its own pick warehouse) and Kardex uses `KD06A` (different shuttle).
- Or one is stale and they should match.

### Per-dept MachGrp resolution

`FrmCoatingPick.GetPickMachGrpCode(DeptCode)`:
- **`JALU`** → `A16` (Aluminium pick)
- **`JSTL`** / **`EBTV`** → `S16` (Steel/EBTV pick)
- else → `Nothing`

`GetSourceMachGrpCode(DeptCode)`:
- **`JALU`** → `A80` (source)
- **`JSTL`** / **`EBTV`** → `S80` (source)

`GetCurrentMachGrpCode(DeptCode, DestinationMachGrpCode)`:
- **`JALU`** → `A16`
- **`JSTL`** / **`EBTV`** with `DestinationMachGrpCode = EBTV` (or empty) → `S16`
- **`JSTL`** / **`EBTV`** otherwise → `S17`

**The `A`-prefix (aluminium) vs `S`-prefix (steel/EBTV)** is the convention; numeric suffix (16/17/80) is operation stage. Q-345.

### Polling timers

- `TimerRegenerate` (view-only mode): full regenerate poll.
- `TimerRefresh`: configured via `AppSettings.GetIntegerSetting("FrmCoatingPickRefreshInterval")` seconds.
- `TimerFetching`: starts on non-view-only mode.

`AppSettings.GetIntegerSetting("FrmCoatingPickRefreshInterval")` returning <=0 disables refresh. Magic-default behaviour — Q-346.

## Business rules surfaced

- [[../business-rules/icenter-coating-executor-enum|`Coating.Executor` 5-state enum]] — InternalPowderCoat / ExternalCoating / ExternalOther / InternalLacquer / Galvanize.
- [[../business-rules/icenter-coating-dept-codes|`JCOA` / `JALU` / `JSTL` / `EBTV` dept codes]] in coating flows.
- **`KD05A` + `KD10A`** are the coating-side Kardex warehouses (Q-344).
- **`F32` + `F33`** are the post-coating destination MachGrps; `F32` is default.
- **`A16` / `A80` / `S16` / `S17` / `S80`** MachGrp convention (A=aluminium, S=steel).
- **`'OPMERKING'` SubPartCode** = the "remark" subpart in the BOM, surfaced on the label.

## Open questions

- **Q-338 (new):** Document `Coating.Executor` enum semantics — which downstream code paths react to each value?
- **Q-339 (new):** `KD05A` (Coating) vs `KD06A` (Kardex MOC) — are these distinct warehouses or stale-code drift?
- **Q-340 (new):** `EbtvSticker` is `'Vervallen!!!'` — confirm dead-code; remove.
- **Q-341 (new):** `PdfCreatorPath` fallback in `CoatingPickLabel_v3` — what does it resolve to? Manual print step?
- **Q-342 (new):** `CoatingPickLabel_v3` sets both `PaperKind.A4` and explicit 827×1169 dimensions. Redundant.
- **Q-343 (new):** `'OPMERKING'` SubPartCode magic literal in `GetRemark`. Document SubPart-codes convention.
- **Q-344 (new):** `WareHouseNames = {"KD05A", "KD10A"}` vs `KD06A` — clarify Coating-vs-Kardex warehouse split. `#safety-relevant`
- **Q-345 (new):** Document MachGrp `A`/`S` prefix convention (aluminium / steel) + suffix-stage numbers.
- **Q-346 (new):** `FrmCoatingPickRefreshInterval` ≤ 0 disables refresh — magic default.
- **Q-347 (new):** Color/CoatingSystem/Remark only rendered for `DeptCode='JCOA'`. What is the label format for non-JCOA coating-bound parts?

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/icenter-remaining]] — parent MOC.
- [[icenter-kardex|`KardexProcessor`]] — Kardex pipeline.
- [[../business-rules/icenter-kardex-warehouse-codes|KD%-warehouse-code business rule]] — overlapping (Q-344).
- [[icenterlib-icenter-leaves|`SurfaceTreatmentDefinition`]] — DTO used by `Product.GetDefaultSurfaceTreatmentDefinition`.
- [[isah-part-and-dispatch|`PartDispatch` `M38` MachGrp]] — coating-related dispatch flow.

## Coverage

- `Classes\Coating\Coating.vb` → `done` (deep-read)
- `Classes\Coating\IControlCoating.vb` → `done` (deep-read)
- `Classes\Coating\EbtvSticker.vb` → `done` (deep-read; `#dead-code` flagged)
- `Classes\Coating\CoatingPickLabel_v3.vb` → `done` (deep-read)
- `Classes\Coating\FrmCoatingPick.vb` → `done` (head deep-read; bulk via MOC)
- Other 11 Coating files → `done` via [[../mocs/icenter-remaining]]
