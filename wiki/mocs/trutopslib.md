---
type: moc
title: "TruTopsLib — Trumpf TruTops file parser (VB + C# mirrored)"
status: draft
module: "TruTopsLib"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\TruTopsLib\\"
tags: [moc, external-system, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# TruTopsLib

## What this hub covers

`C:\DevOps\iCenter\iCenter\TruTopsLib\` — JAZO's **parser library for Trumpf sheet-metal CAD/CAM file formats**. Two file families:

1. **`.GEO` / "FlatGeometry"** — Trumpf's flat-pattern geometry export.
2. **TopsFile** — the legacy TruTops `.geo`/`.dwg`-like file with `#~33` block markers (German tags like `BIEG_END` = bend end, `KONT_END` = contour end).
3. **PMI (Product Manufacturing Information)** — countersink and threaded-hole annotations attached to the geometry.

Plus a `LayerConverter` for DXF colour/layer translation — `#safety-relevant`.

## Structural quirk: every file mirrored as `.vb` + `.cs`

> JAZO maintains **two parallel implementations** of the same parser — one in VB.NET, one in C#. Almost every file has both extensions:
>
> ```
> TopsFile\Contour.vb           7.7 KB
> TopsFile\Contour.cs           7.3 KB
> TopsFile\SubContour\Arc.vb    7.7 KB
> TopsFile\SubContour\Arc.cs    7.7 KB
> ...
> ```
>
> **65 total files = ~33 source files duplicated across two languages** (plus a few VB-only PMI* and one .cs-only Resources.Designer.cs).
>
> Q-308 — why both languages? Likely so VB callers (iCENTER) and C# callers (a Trumpf-side SDK?) can use the same parser. Maintenance hazard: bug fixes must be applied to both. **Auto-generated cross-port?** Or hand-maintained?

## Files (by sub-folder)

### `GeoInterpreter\`
- `FlatGeometry` (entity), `FlatGeometryReader` (parser), `FlatGeometryDxfExport` (DXF export), `FlatGeometryGeoExport` (GEO export), `IFlatGeometryExport` (interface).

### `TopsFile\`
- Root: `BendLine`, `Body`, `Bounds`, `Contour`, `DataTypeHandler`, `Parameters`, `Point`, `PointCollection`, `Properties`, `TextCollection`, `TTInfo`.
- `SubContour\` (sub-folder): `Arc`, `Circle`, `Fillet`, `Line`, `Text`, `SubContour` (abstract base).

### `PMI\` (VB-only)
- `IPmiLabel` (interface), `PMILabel` (base entity), `PMILabelCollection`, `PmiLabelCountersink`, `PmiLabelThreadNote`.

### Root files
- `FileReaderHelper`, `LayerConverter`, `MigrationFix`, `PMILabel.cs`, `PMILabelCollection.cs`, `Resources.Designer.cs` (generated).

## Behaviour highlights

### `FlatGeometryReader` — Trumpf `.GEO` parser

State-machine line-by-line parser. Recognises **German block markers**:

| Marker | Meaning |
|--------|---------|
| `#~33` ... `#~KONT_END` | Contour block (KONT = Kontur) |
| `#~331` | SubContour block start |
| `#~37` ... `#~BIEG_END` | Bend-line block (BIEG = Biegen, German for bend) |
| `|~` | Points sub-block delimiter |
| `##~~` | Points block end |

Two-pass: `ReadFromArray` extracts properties + body + bend lines; if `IncludeContours=True`, `ReadContoursFromArray` extracts the contour geometry.

Depends on `WW.Cad.Model` library (third-party DXF SDK).

### `LayerConverter.ConvertLayers(inFile)` — DXF layer/colour rewriter `#safety-relevant`

Reads a DXF, applies JAZO's colour/layer rewrite rules, writes back. The rules:

| Source colour | Action |
|---------------|--------|
| 9, 8, 10 | → layer `GREEN`; toggle line-type DASHED↔CONTINUOUS |
| 7 (white) AcDbText | **delete entity** ("text replaced with bend info") |
| 5, 15 (blue, pink) | delete entity |
| 1, 2 (red, yellow) | → layer `0` |
| Anything else | delete entity |

**Author's comment at line 53**: `'', 7 7 is levensgevaarlijk omdat zetlijnen soms ook wit zijn` — Dutch for **"color 7 is *life-threatening* because bend lines are sometimes also white"**. The author **literally flags color-7 handling as life-threatening** — likely because deleting a bend line by accident would produce a wrong cut/bend. `#safety-relevant`. Documented as [[../business-rules/trutopslib-color-7-hazard]].

Uses `MigrationFix.TEMPLATEDXF` as the source of the canonical layer set; clones layers in if missing.

### `PMILabel` — Countersink + threaded-hole annotations

```vb
Public Class PMILabel
    Public MyTexts As New TextCollection
    Public Contours As New List(Of Contour)
    Public References As New List(Of SubContour)
    Public Const Layer As String = "5"
    Public LabelType As PmiLabelType = PmiLabelType.Undefined

    Public Enum PmiLabelType
        Undefined = 0
        Countersink = 1
        ThreadedHole = 2
    End Enum
End Class
```

- **Layer "5"** is the convention for PMI annotations.
- Two PMI subclasses: `PmiLabelCountersink`, `PmiLabelThreadNote`. PMI = **Product Manufacturing Information** in the CAD sense — fabrication metadata that must travel with the geometry. `#safety-relevant`.
- `GetMatchingSubContour` does coincident-circle lookups (`Arc.HasCoincidentCircle`) to attach PMI to the right hole.

### `Contour` — TopsFile contour entity

Uses three string constants for parsing:
- `COUNTOURKEYSTART = "#~33"` (note typo "COUNTOUR")
- `CONTOURKEYEND = "#~KONT_END"`
- `SUBCONTOURBLOCKKEYSTART = "#~331"`

Has 9 unnamed `UnknownPropertyN` fields — author hasn't reverse-engineered what they mean yet. Q-309.

## Business rules surfaced here

- [[../business-rules/trutopslib-color-7-hazard|DXF colour 7 is life-threatening to handle]] — author's own warning in LayerConverter.
- **Layer "5" is the PMI annotation layer** — JAZO convention.
- **German block markers** — Trumpf format spec carries through (`#~KONT_END`, `#~BIEG_END`).
- **`PmiLabelType` two types** — Countersink (1) and ThreadedHole (2) are the only annotated holes; other PMI ignored.

## Notable findings

1. **Dual-language mirror** is a maintenance multiplier. Q-308.
2. **`Contour.UnknownProperty1..9`** — 9 reverse-engineered-but-not-named fields. Q-309.
3. **`MigrationFix.TEMPLATEDXF`** — points to a fixed DXF template used to seed correct layers. Q-310 — what path?
4. **`LayerConverter`** has the only `#safety-relevant` author-flagged hazard comment in the entire codebase. Treat with extra care.
5. **C# files import VB-namespace constructs** (likely) — Phase-3 follow-up.

## Open questions

- **Q-308 (new):** Why is TruTopsLib maintained in both VB and C#? Sync mechanism (manual? generated?)? Both consumed by which projects?
- **Q-309 (new):** `Contour.UnknownProperty1..9` — document with SME help.
- **Q-310 (new):** `MigrationFix.TEMPLATEDXF` constant — path on disk?
- **Q-311 (new):** Confirm SME description of every colour-rule in `LayerConverter.ConvertLayers`. The "life-threatening" comment about colour 7 is the strongest warning sign in the codebase.

Logged in [[../needs-review/_index]].

## Coverage

**All 57 source files marked `done` overview-level via this MOC.** Generated `Resources.Designer.cs` marked `generated`. Designer files (6) marked `generated`.

Recommend Phase-4 deep-read of:
- `LayerConverter.vb` (or .cs) — full colour-rule audit.
- `FlatGeometryReader.vb` — the canonical Trumpf-file ingestor.
- `PMILabel.vb` family — annotation flow.
