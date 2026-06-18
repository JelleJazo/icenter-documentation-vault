---
type: moc
title: "iCENTER/SmtManufacturing — sheet-metal CAD-to-Trumpf pipeline"
status: draft
module: "iCENTER/SmtManufacturing"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\SmtManufacturing\\"
tags: [moc, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# iCENTER/SmtManufacturing — sheet-metal CAD-to-Trumpf pipeline

## What this hub covers

`iCENTER\SmtManufacturing\` — **the iCENTER-side sheet-metal pipeline**. 47 source files. Takes a Creo part, converts its flat pattern to a Trumpf-compatible GEO file (with bend annotations + extended-data), imports into TruTops Oseon ("Boost"), and tracks the per-part state through bend / weld-stud / contour-check stages. `#safety-relevant` — drives laser-cut + bend programs.

> JAZO refers to Oseon internally as **"Boost"** (the project name).

## Top files by size

| File | KB | Role |
|------|---:|------|
| `FlatPatternConverter.vb` | 84 | **DXF → GEO converter** with bend-info, contour-cleanup, weld-stud detection. The single biggest .vb file in iCENTER. |
| `Part.vb` | 81 | **the SMT Part class** — wraps an `ICenterObject` and owns Boost import, GEO/DXF lifecycle, AutoBend, JCK reporting. |
| `FrmPartIdentifier.vb` | 55 | the operator UI for identifying a part on the SMT line |
| `BendPart.vb` | 24 | bend-part state + bend-list management |
| `JPLT_PartIdentSticker.vb` | 21 | JPLT (sheet-metal dept) part-ident sticker printer |
| `FrmCalcCycleTimeManagement.vb` | 17 | SMT cycle-time calc admin UI |
| `ContourCheck.vb` | 15 | contour-validation (closed/open check) |
| `ControlSmtCut.vb` | 14 | SMT cut UserControl |
| `JPLT_DistrSticker.vb` | 13 | JPLT distribution sticker |
| `TmtInterpreter\TMT.vb` | 13 | TMT (TruTops Macro Type?) interpreter |
| `BoostMigrator.vb` | 13 | Boost (Oseon) migration helper |
| `GeoViewerControl.vb` | 12 | GEO file viewer UserControl |

Plus 35 smaller files (~5 KB and under): TafInterpreter\* (3 files), various forms + helpers.

## `Part.vb` (81 KB) — the SMT Part class

The **iCENTER-side wrapper** around `ICenterObject` (the CAD model) that drives Boost import + GEO/DXF + AutoBend + sticker-printing for one sheet-metal part.

### Key constants

```vb
Protected Friend Const workdir As String = "c:\work\"
    ' "Niet c:\temp gebruiken: de CadBatchServer ruimt daar alles op!"
    ' (Don't use c:\temp: CadBatchServer cleans everything there!)
Public Const ManualBendFolder As String = "/temp/Bend"
Public Shared AutoBendLogPath As String = workdir & "autobendlog.xml"
```

**`workdir = "c:\work\"`** with the warning comment is the **single most important infrastructure note** in this folder — `c:\temp\` is auto-cleaned by CadBatchServer (Q-391). Q-392 — `ManualBendFolder = "/temp/Bend"` uses a forward-slash prefix; UNC, web-path, Linux-style relative? Anomaly.

### Enums

```vb
Public Enum GeoSource         { Auto = 0, Manual = 1 }
Public Enum DeburrType        { None = 0, Normal = 1, HighQuality = 2 }
Public Enum BoostImportType   { Type2D = 0, Type3D = 1 }
```

- **`DeburrType.HighQuality`** ties to [[../business-rules/icenter-smt-deburr-cycle-time|FlowGrill 2× deburr multiplier]] — the FlowGrill flag maps to HighQuality here. Q-393.
- **`BoostImportType` 2D vs 3D** corresponds to two `ConvertToGeo2D` / `ConvertToGeo3D` paths.

### `ProdId` format

```vb
Public ReadOnly Property ProdId As String
    Get
        If IPpartId = -1 Then
            Return "PRT" & batchRowNr.ToString("D4")  ' PRT0001, PRT0002...
        Else
            Return "IA" & IPpartId                    ' IA12345
        End If
    End Get
End Property
```

**Two ProdId formats**: `PRT####` (unlinked-to-IPpart) and `IA{n}` (linked). The `IA` prefix family is documented at [[../business-rules/icenter-part-code-prefixes|iCenter part-code prefixes]] (Q-125).

### Notable methods

| Method | Lines | Role |
|--------|------:|------|
| `New(iCenterPart)` | 68 | constructs with default `ImportSettings3D` |
| `UpdateGeoDrawingNumber()` | 73 | **`#dead-code`** — throws "Updating GEO not supported since TruTops Boost." body fully commented out |
| `AddGeoStringToDataManagement(...)` | 94 | persists a GEO string to T_XmlData |
| `AddDxfToXml(...)` | 300 | persists a DXF reference |
| `ClearBnc()` | 346 | clears the per-machine BNC files |
| `ClearGEO()` | 373 | clears the part's GEO data |
| `ResetSmtExport3DSetting()` | 396 | reset 3D export config |
| `UpdateSmtExport3DSetting(...)` | 400 | update 3D export config |
| `ImportManualGeo(GeoFilePath)` | 450 | import a manually-prepared GEO |
| `ConvertToGeo(DeleteWorkFiles, MyJob)` | 458 | **main entry**: convert flat-pattern DXF → GEO; dispatches to 2D or 3D |
| `ConvertToGeo3D(...)` | 556 | 3D conversion path |
| `ConvertToGeo2D(...)` | 838 | 2D conversion path |
| `GetCurrentAction(TTNGActionDataService)` | 538 | poll Oseon for current TTNG action |
| `ValidateImportFile(...)` | 767 | post-import validation |
| `ImportSEDxf(filepath)` | 951 | import Solid Edge DXF (alternative CAD source) |
| `GetGeo()` | 1039 | parse GEO via `TruTopsLib.GeoInterpreter.FlatGeometry` |
| `ExportGeo(path, includeBends)` | 1046 | export GEO to file |
| `HasBends()` / `BendCount()` | 1114/1118 | bend introspection |
| `SaveGeoAsDxf(...)` | 1138 | round-trip GEO → DXF |
| `ContainsBMT(Machine)` | 1163 | check if Trumpf machine has BMT file for this part |
| `Shared ContainsBNC(MachineName, Modelname)` | 1181 | check for BNC file |

## `FlatPatternConverter.vb` (84 KB) — DXF→GEO conversion

The **single biggest .vb file in iCENTER**. Takes a flat-pattern DXF from Creo (or another CAD source) and converts it to a Trumpf GEO file with bend annotations, weld-stud markers, contour validation, and extended-data XData for bend handling.

### Key constants

```vb
Public Const TcBendAppId   As String = "TC_BEND_DATA"     ' Trumpf XData app-id (regular bends)
Public Const TcSEBendAppId As String = "TC_SE_BEND_DATA"  ' Trumpf XData app-id (Solid Edge bends)
Private Const roundDigits As Integer = 9                  ' geometry comparison precision
Public Const ComplexHoleDecimals As Integer = 2           ' precision for complex-hole geometry
Private Const BENDDIRBASELINELENGTH As Decimal = 4.625    ' bend-direction baseline length (mm?)
Private ContourLayers As List(Of String) = {"0", "AXIS", "WHITE"}.ToList
```

- **`TC_BEND_DATA` / `TC_SE_BEND_DATA`** are the **Trumpf-side XData app-ids** for embedded bend metadata. Two distinct ids for normal vs Solid Edge sources. Q-394 — document the XData schema.
- **`BENDDIRBASELINELENGTH = 4.625`** — magic geometry constant. mm? inches? Q-395.
- **`ContourLayers = {"0", "AXIS", "WHITE"}`** — the three DXF layers where contour geometry is expected. Anything else is "annotation" and gets stripped.

### Three constructors

```vb
Public Sub New(InFile, OutFile, ICenterPart, CadSource)         ' from file
Public Sub New(InStream, OutFile, ICenterPart, CadSource)       ' from stream
Public Sub New(Model As DxfModel, OutFile, ICenterPart, CadSource) ' from existing model
```

`CadSource` is `Enums.Application.CadSource` — distinguishes Creo vs Solid Edge vs ... — drives which XData app-id to use, which bend-note format to parse.

### `SetColors()` — bend-annotation colour cycle

Populates a 21-element `lColors` list (Blue, Brown, Cyan, DarkBlue, DarkGray, ..., Red). Used to **colour-cycle multiple bend annotations** on the same drawing so adjacent bends are visually distinguishable.

### Notable methods

| Method | Lines | Role |
|--------|------:|------|
| `Convert(CadBatchserverJob)` | 98 | **main entry**: full DXF→GEO conversion pipeline |
| `PerformContourCheck(CadBatchserverJob)` | 203 | validates contour closedness |
| `GetContourCheck()` | 211 | accessor |
| `GetRotatedModel(model)` | 215 | rotates the DXF to canonical orientation |
| `GetArea(model)` | 271 | computes flat-pattern area |
| `GetLineLength(line)` | 277 | helper |
| `SetContourExtents()` | 285 | computes ExtMin/ExtMax bounding box |
| `Rotate(model, rad)` | 319 | static rotation helper |
| `RemoveAdditionalNotes()` | 331 | strips non-contour annotations |
| `CleanupCircles()` | 438 | removes duplicate/concentric circles |
| `GetIamWasher()` | 625 | detects washer-style geometry |
| `RemoveCrossHairs()` | 655 | strips cross-hair markers |
| `degreesToRad(angle)` | 711 | helper |
| `RemoveOversizedCenterAxis()` | 720 | strips center-axes too long for the part |
| `RemoveRemainingAxis()` | 750 | strips remaining axis lines |
| `RemoveLeftOvers()` | 772 | catch-all cleanup |
| `Coaxial(c1, c2)` | 823 | circle coaxiality check |
| `Coincident(p1, p2, digits)` | 831 | point coincidence (default 9-digit) |
| `valuesAreClose(v1, v2, digits)` | 838 | numeric tolerance |
| `ReplaceBendNotes()` | 845 | converts CAD bend-notes to Trumpf XData |
| `ReplaceSolidEdgeBendLines()` | 946 | Solid Edge specific bend-line handling |
| `SetExtendedData(BendLine, TcAngle, appId, iCenterPart)` | 1003 | writes Trumpf XData onto the DXF entity |
| `GetBendNotes(line)` | 1057 | reads bend-notes attached to a line |
| `GetEntityColor()` | 1119 | next colour in the cycle |
| `GetBendAngle(bendNote)` | 1127 | extracts angle from a BendNote |
| `GetBendDegreeSymbol(bendNote)` | 1157 | extracts degree symbol |
| `GetBendDirBaseline(bendNote)` | 1178 | extracts bend-direction baseline |
| `GetLeaderOnLine(leader, line)` | 1219 | leader-on-line detector |
| `CoincidentLines(line1, line2)` | 1238 | line coincidence |
| `CoincidentLineAndCircle(Line, Circle)` | 1256 | line-circle coincidence |

## Notable findings (cross-folder)

1. **`Part.UpdateGeoDrawingNumber`** is **`#dead-code`** — throws since TruTops Boost migration. Body commented out. Q-396.
2. **`workdir = "c:\work\"`** anti-temp-cleanup convention — `c:\temp\` is owned by CadBatchServer's cleanup. **All scratch files for this subsystem live in `c:\work\`** instead. Documented as [[../business-rules/icenter-smt-workdir-convention]].
3. **`AutoBendLogPath = workdir & "autobendlog.xml"`** — central auto-bend log file at `c:\work\autobendlog.xml`. Per-machine? Per-job? Q-397.
4. **`ManualBendFolder = "/temp/Bend"`** — forward-slash prefix, odd path syntax. Q-392.
5. **The 2D-vs-3D import split** in `Part.ConvertToGeo` is the key branch — `ConvertToGeo3D` (line 556) vs `ConvertToGeo2D` (line 838). `BoostImportType` enum + `UseSmtExport3D` flag drive the choice.
6. **TMT (TruTops Macro Type?) interpreter** in `TmtInterpreter\` (3 files) and **TAF interpreter** in `TafInterpreter\` (3 files) suggest TruTops has multiple file-format families consumed at this layer. Q-398.
7. **JPLT** = JAZO sheet-metal dept code (already documented in [[../modules/isah-leaves|WorkView.GetSmtBendQueue]] Q-207). `JPLT_PartIdentSticker` + `JPLT_DistrSticker` are dept-specific sticker printers.
8. **`BoostMigrator.vb`** — was probably the migration script from the pre-Boost (PPSInterface) era to Oseon. Likely `#dead-code` now (Q-399).
9. **`FlatPatternConverter.Convert` calls `DeleteFailedPublishPart(NotClosedContour, ...)`** at line 100 — clears any prior "failed publish" marker before reconverting. Q-400 — document the `PartManufFailureType` enum and recovery flow.

## Business rules surfaced

- [[../business-rules/icenter-smt-workdir-convention|`c:\work\` is the SMT scratch dir]] — CadBatchServer cleans `c:\temp\`.
- **`DeburrType` 3-state**: None / Normal / HighQuality. HighQuality = FlowGrill-quality (Q-393).
- **`BoostImportType` 2D vs 3D** — drives ConvertToGeo2D vs ConvertToGeo3D.
- **`GeoSource` Auto vs Manual** — distinguishes pipeline-generated GEO vs operator-prepared GEO.
- **`ContourLayers = {"0", "AXIS", "WHITE"}`** — the three valid DXF layers for contour geometry.
- **`TC_BEND_DATA` + `TC_SE_BEND_DATA`** Trumpf XData app-ids.
- **`BENDDIRBASELINELENGTH = 4.625`** — bend-direction baseline length constant.
- **`PRT####` vs `IA{n}` ProdId formats** depending on IPpartId linkage.

## Open questions

- **Q-391 (new):** Document CadBatchServer's `c:\temp\` cleanup policy. What triggers it, how often?
- **Q-392 (new):** `ManualBendFolder = "/temp/Bend"` forward-slash path syntax. UNC? Web? Linux-style? Confirm.
- **Q-393 (new):** `DeburrType.HighQuality` ↔ FlowGrill-quality flag mapping. Confirm.
- **Q-394 (new):** Document Trumpf XData schemas for `TC_BEND_DATA` + `TC_SE_BEND_DATA` app-ids.
- **Q-395 (new):** `BENDDIRBASELINELENGTH = 4.625` — units (mm? inches?). Where does this number come from?
- **Q-396 (new):** `Part.UpdateGeoDrawingNumber` is `#dead-code` since Boost. Delete?
- **Q-397 (new):** `AutoBendLogPath` is shared across all parts. Per-machine, per-job, or global?
- **Q-398 (new):** Document the TMT vs TAF vs GEO TruTops file-format families and their interpreters.
- **Q-399 (new):** `BoostMigrator.vb` — confirm dead-code (pre-Boost migration completed)?
- **Q-400 (new):** Document `PartManufFailureType` enum + the failed-publish-part recovery flow.

Logged in [[../needs-review/_index]].

## Coverage

All 47 source files in `iCENTER\SmtManufacturing\` already marked `done` overview-level via [[icenter-remaining]]. This MOC promotes the two biggest files (`Part.vb` + `FlatPatternConverter.vb`) to **public-surface deep-read** with method tables. Full per-method deep-reads of the 838+556-line `ConvertToGeo2D`/`3D` paths and the bend-handling internals deferred — recommend SME conversation.

## Related

- [[icenterlib-smtproduction|`ICenterLib/SmtProduction`]] — the ICenterLib-side counterpart (DataServices, Entities for Boost).
- [[trutopslib|TruTopsLib]] — file-format parsers consumed by FlatPatternConverter.
- [[trutopslib-color-7-hazard|LayerConverter colour-7 hazard]] — sibling DXF processor.
- [[../modules/icenter-coating|Coating]] — downstream consumer post-cut.
- [[../external-systems/oseon|Oseon / Boost]] — destination MES.
- [[../external-systems/trutops|Trumpf TruTops]] — destination CAD/CAM.
