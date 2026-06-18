---
type: module
title: "TruTopsLib LayerConverter + MigrationFix — DXF layer/colour normaliser"
status: done
module: "TruTopsLib"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\TruTopsLib\\LayerConverter.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\TruTopsLib\\MigrationFix.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# TruTopsLib `LayerConverter` + `MigrationFix` — DXF layer/colour normaliser

> **The most safety-flagged single file in the codebase** — see [[../business-rules/trutopslib-color-7-hazard|the colour-7 hazard rule]] for the author's own `'levensgevaarlijk'` warning. This deep-read covers the **full colour/layer rewrite pipeline** that prepares a CAD-output DXF for downstream Trumpf consumption.

## `MigrationFix.vb` — the layer template

```vb
Public Class MigrationFix
    Public Shared ReadOnly Property TEMPLATEDXF As String
        Get
            Dim s As String = CAD.Creo.Environment.GetProManufDir
            If Not s.EndsWith("\") Then s &= "\"
            s &= "SheetMetal\template_with_bendinfo.dxf"
            Return s
        End Get
    End Property

    Public Shared Function GetMergedTemplateDxf(Model As DxfModel) As DxfModel
        Try
            Dim TemplateModel = DxfReader.Read(TEMPLATEDXF)
            Dim CloneContext = New CloneContext(TemplateModel, Model, ReferenceResolutionType.CloneMissing)
            Model.Layers.AddCopiesFrom(TemplateModel.Layers, CloneContext)
            Model.LineTypes.AddCopiesFrom(TemplateModel.LineTypes, CloneContext)
            Return TemplateModel
        Catch ex As Exception
            Return Nothing                        ' silent
        End Try
    End Function
End Class
```

### `TEMPLATEDXF` path resolution (resolves Q-310)

The template lives inside **Creo's `pro-manuf` directory**:

```
{CAD.Creo.Environment.GetProManufDir}\SheetMetal\template_with_bendinfo.dxf
```

So the template:
1. Is **bundled with the Creo installation** (or at least lives inside its directory tree).
2. Requires `CAD.Creo.Environment.GetProManufDir` to resolve correctly — if Creo isn't installed / configured, `TEMPLATEDXF` may be malformed.
3. Is named `template_with_bendinfo.dxf` — explicitly carrying the bend-info layer set that `LayerConverter` clones in.

Q-384 — verify `CAD.Creo.Environment.GetProManufDir` behaviour when Creo isn't installed (does it throw, return empty, return a default?).

### `GetMergedTemplateDxf(Model)`

Helper for callers that want to **add the template's layers + line-types to an existing model** without the rest of the conversion. Returns the template model (so the caller can reference it for further cloning). Silent on error.

## `LayerConverter.vb` — the full DXF rewrite

### Public surface

```vb
Public Class LayerConverter
    Public Sub New()
    Public Sub ConvertLayers(inFile As String)
    Public Shared Function GetDxfLayerNameByColorIndex(Index As Integer) As String
    Public Shared Function GetDxfLineType(TypeId As Integer) As String
End Class
```

### `ConvertLayers(inFile)` — top-level entry

```vb
Dim newModel As DxfModel = GetConvertedModel(inFile)
If newModel IsNot Nothing Then
    DxfWriter.Write(inFile, newModel, False)
End If
```

**Overwrites the input file in-place** (`DxfWriter.Write(inFile, ...)`). No backup, no temp-file-and-rename. If anything writes the file during the call, data is lost. Q-385 `#safety-relevant`.

If conversion failed (`Nothing`), the original file is **left untouched** — so a failed conversion is silent-no-op (which is safer than corrupting), but the caller has no signal. Q-386.

### `GetConvertedModel(inFile)` — the heart of it

Reads the input DXF, clones the template's layer set in, applies colour/layer/line-type rules per entity, removes non-template layers, removes entities marked for deletion. Returns the cleaned model.

#### Phase 1 — read + merge templates

```vb
Dim model = DxfReader.Read(inFile)
Dim templateModel = DxfReader.Read(MigrationFix.TEMPLATEDXF)
Dim cloneContext = New CloneContext(templateModel, model, ReferenceResolutionType.CloneMissing)
model.Layers.AddCopiesFrom(templateModel.Layers, cloneContext)
```

Adds **missing layers** from the template into the model. If the input DXF lacks `GREEN`, `0`, etc., they're created here. **Existing layers in the input are kept** (the `CloneMissing` mode).

```vb
If model.Layers.Contains("0") Then
    model.GetLayerWithName("0").Color = WW.Cad.Model.Colors.White
End If
```

**Forces layer "0" to colour White** regardless of incoming colour. **The "0" layer is the AutoCAD default layer** — JAZO's convention is "0 = White". Q-387.

#### Phase 2 — per-entity colour rules

```vb
For Each entity In model.Entities
    Select Case entity.Color.ColorIndex
        Case 9, 8, 10
            entity.Layer = model.Layers("GREEN")
            If entity.LineType.Name = "DASHED" Then
                entity.LineType = model.LineTypes("CONTINUOUS")
            Else
                entity.LineType = model.LineTypes("DASHED")
            End If

        Case 7                                     ' white
            If entity.AcClass = "AcDbText" Then
                lEntitiesToDelete.Add(entity)      ' delete text; non-text falls through
            End If

        Case 5, 15                                  ' blue, pink (Dutch: roze)
            lEntitiesToDelete.Add(entity)

        Case 1, 2                                   ' red, yellow
            entity.Layer = model.Layers("0")
            ' ⚠ comment: ", 7  7 is levensgevaarlijk omdat zetlijnen soms ook wit zijn"

        Case Else
            lEntitiesToDelete.Add(entity)
    End Select
    entity.Color = WW.Cad.Model.Entities.EntityColor.ByLayer
Next
```

The full colour map (consolidated):

| Source colour index | Action | Line-type | Final colour resolution |
|--------------------:|--------|-----------|--------------------------|
| 9, 8, 10 | → layer `GREEN`, **toggle DASHED↔CONTINUOUS** | DASHED → CONTINUOUS; else → DASHED | `ByLayer` → green |
| 7 (white) `AcDbText` | **delete** ("text replaced with bend info") | n/a | n/a |
| 7 (white) non-text | **fall through**: keeps original layer + `ByLayer` | unchanged | `ByLayer` → whatever its original layer's colour is |
| 5 (blue) | **delete** | n/a | n/a |
| 15 (pink) | **delete** | n/a | n/a |
| 1 (red) | → layer `0` | unchanged | `ByLayer` → White (per phase 1) |
| 2 (yellow) | → layer `0` | unchanged | `ByLayer` → White (per phase 1) |
| any other (0, 3, 4, 6, 11..14, 16+) | **delete** | n/a | n/a |

**Critical post-rule line**: `entity.Color = ByLayer`. This **strips the entity's explicit colour override** and forces it to inherit from its layer's colour. Even for the `Case 7` non-text fall-through entities, the explicit colour-7 is replaced with `ByLayer`. So:

- A colour-7 line on layer `0` → after conversion, line keeps layer 0 (with no explicit colour), and the layer-0 colour is White (set in phase 1). Net: line is white.
- A colour-7 line on layer `GREEN` → keeps GREEN layer, ByLayer, GREEN colour. Net: line is green (which is wrong if it was meant to be white).

This is **subtle**: an unhandled colour-7 entity's final colour depends on what layer it was already on. **The author's `'levensgevaarlijk'` warning** captures the gap: bend lines drawn in white on a non-template-layer might end up on the wrong final layer with the wrong final colour.

#### Phase 3 — layer cleanup

```vb
For i = model.Layers.Count - 1 To 0 Step -1
    Dim layer = model.Layers(i)
    If Not templateModel.Layers.Contains(layer.Name) Then
        model.Layers.Remove(layer)
    End If
Next
```

**Removes every layer that isn't in the template** — even if entities still reference it. The DXF library presumably handles orphaned references (entities re-bound to a default layer or throws), but Q-388 — verify behaviour.

#### Phase 4 — entity deletion

```vb
For Each entity In lEntitiesToDelete
    model.Entities.Remove(entity)
Next
```

Collected during Phase 2; removed at the end.

### `GetDxfLayerNameByColorIndex(Index)` — AutoCAD ACI mapping

```vb
Case 1: Return "WHITE"      ' ACI 1 = red in standard AutoCAD; JAZO maps it differently?
Case 2: Return "RED"        ' ACI 2 = yellow in standard AutoCAD
Case 3: Return "YELLOW"     ' ACI 3 = green in standard AutoCAD
Case 4: Return "GREEN"      ' ACI 4 = cyan in standard AutoCAD
Case 5: Return "CYAN"       ' ACI 5 = blue in standard AutoCAD
Case 6: Return "BLUE"       ' ACI 6 = magenta in standard AutoCAD
Case 7: Return "MAGENTA"    ' ACI 7 = white/black in standard AutoCAD
Case Else: Throw NotImplementedException
```

**The mapping is off-by-one vs the AutoCAD standard ACI palette.** AutoCAD ACI palette: 1=red, 2=yellow, 3=green, 4=cyan, 5=blue, 6=magenta, 7=white. JAZO's mapping is shifted: 1=WHITE, 2=RED, etc. So this function returns **what JAZO calls each colour-index slot**, not what AutoCAD calls them. Q-389 — confirm with SME this is the JAZO convention vs a bug.

Throws above index 7 — supports only the basic 7-colour palette.

### `GetDxfLineType(TypeId)` — line-type mapping

```vb
Case 0: Return "CONTINUOUS"
Case 1, 2: Return "HIDDEN"
Case Else: Throw NotImplementedException
```

Three TypeIds → two line-type names. Above 2 throws. Q-390 — what are the values 3+ meant to be?

## Surprises

1. **In-place file overwrite** with no backup/temp/rename (Q-385).
2. **Layer cleanup** removes all non-template layers — entities still referencing them get re-bound silently or throw (Q-388).
3. **`entity.Color = ByLayer`** after rules strips the explicit colour. Combined with the layer-rebinding, an unhandled colour-7 entity may end up with a different visible colour than expected.
4. **MigrationFix.TEMPLATEDXF** depends on Creo's installation directory — Creo absence breaks TruTops processing (Q-384).
5. **JAZO ACI palette mapping** in `GetDxfLayerNameByColorIndex` is shifted vs standard AutoCAD (Q-389).
6. **Silent error handling** throughout — caller sees Nothing on failure, no exception, no log entry (except in `LayerConverter.GetConvertedModel`'s outer catch).
7. **Author hazard comment preserved verbatim** — the most explicit safety flag in the codebase.

## Business rules surfaced

- [[../business-rules/trutopslib-color-7-hazard|DXF colour 7 (white) handling is life-threatening]] — author's own warning.
- **Layer "0" is forced to White** by JAZO convention.
- **`GREEN` layer holds colour 9/8/10 entities** with line-type toggled DASHED↔CONTINUOUS.
- **Colour 5 (blue) and 15 (pink) entities are deleted** (consistent with TrumPF "informational-only" annotations).
- **Colour 1 (red) and 2 (yellow) → layer 0** (default white layer).
- **The DXF template lives in Creo's pro-manuf dir** — `SheetMetal\template_with_bendinfo.dxf`.
- **Layers not in the template are removed** after entity rewrite.

## Open questions

- **Q-384 (new):** Document `CAD.Creo.Environment.GetProManufDir` behaviour when Creo isn't installed.
- **Q-385 (new):** `ConvertLayers` writes in-place with no backup. Concurrent reader/writer hazard.
- **Q-386 (new):** Failed conversion leaves original file untouched + no caller signal. Should LayerConverter return Boolean / log error?
- **Q-387 (new):** "Layer 0 = White" is JAZO convention. Confirm.
- **Q-388 (new):** Phase-3 layer cleanup removes layers; what happens to entities still referencing them?
- **Q-389 (new):** `GetDxfLayerNameByColorIndex` palette is shifted vs standard AutoCAD ACI. Confirm JAZO convention vs bug.
- **Q-390 (new):** `GetDxfLineType` only implements 0/1/2. What values 3+ mean.

Logged in [[../needs-review/_index]].

## Related

- [[../business-rules/trutopslib-color-7-hazard|Colour-7 hazard business rule]] — defining rule.
- [[../mocs/trutopslib|TruTopsLib MOC]] — parent.
- [[../mocs/icenterlib-cad|ICenterLib/CAD MOC]] — Creo environment source.
- **`CAD.Creo.Environment.GetProManufDir`** — Creo path resolver (in ICenterLib/CAD/Creo).

## Coverage

- `TruTopsLib\LayerConverter.vb` → `done` (full deep-read)
- `TruTopsLib\MigrationFix.vb` → `done` (deep-read)
