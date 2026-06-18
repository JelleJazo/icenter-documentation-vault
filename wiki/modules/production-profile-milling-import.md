---
type: module
title: "Production/ProfileMilling — UniLink CSV import pipeline"
status: done
module: "iCENTER/Production/ProfileMilling"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Production\\ProfileMilling\\ImportHandler.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Production\\ProfileMilling\\PMMExportHandler.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Production\\ProfileMilling\\ExternalReferenceHelper.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Production\\ProfileMilling\\ImportData.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Production\\ProfileMilling\\ICAMImport.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `Production\ProfileMilling\` — UniLink CSV import pipeline

## Purpose

The **iCenter → UniLink CAM** data export. Five files that together:

1. Pull a multi-level BOM for a `(MachGrpCode, IPPart, ICenterObject)` tuple.
2. Enrich each row with `BIdentNo`, `Series` (Elumatec) and a `ProdTraceId` (if the profile is flagged for auto-add IA-number).
3. Resolve the STEP file path per row (via [[#pmmexporthandler-stepfile-discovery|`PMMExportHandler`]]).
4. Validate that every profile has a `Series` set and that UniLink has the corresponding DXF.
5. Hand off as a 23-column `ImportData` DataTable to a `UniLink.CSVImportHandler` (the `ICAMImport` interface implementation).

The pipeline is the **counterpart to the [[../mocs/elumatec-ncpipeline|Elumatec NC pipeline]]**: where Elumatec drives the SBZ140/141 machines from iCenter, this pipeline pushes BOM data into UniLink's CSV-based CAM workflow.

## Files

| File | Role |
|------|------|
| `ImportHandler.vb` | orchestrator — `Export()` runs the whole pipeline |
| `ImportData.vb` | DTO — 23-column DataTable with primary key `Pos` |
| `ICAMImport.vb` | one-method interface (`Sub Import(Data As ImportData)`) — implemented by `UniLink.CSVImportHandler` |
| `PMMExportHandler.vb` | resolves STEP filepath via iCenter XML `<MANUFACTURING type='NCW'><PMMEXPORT3D name="..."/>`; moves source STEP into the MANUF folder and updates the XML element |
| `ExternalReferenceHelper.vb` | parses `EXTERNALREFERENCES/EXTERNALREFERENCE` XML nodes; returns Dict of `{Origin → STEP ICenterDoc}` for refs of a given `ExternalReferenceType` |

## Public surface (handler)

```vb
Public Class ImportHandler
    Public Sub New(machGrpCode As String, iPPartId As Long, icenterobject As ICenterObject)
    Public Sub Export()                                    ' entry point
End Class
```

Constructor `iPPartId` clamps to 0 minimum (`Math.Max(iPPartId, 0)`). `machGrpCode` and `icenterobject` are stored as-is.

## `Export()` flow

```
GetData()
    GetBom()                  → ICO.GetMBomMultilevel({MachGrpCode}, OrderType, True)
        for each row: EluCadApp.GetProfileInfoByPartCode → BIdentNo, BSeries
    ValidateSeries(DT)
        ValidateProfileSeriesDefinition(DT)
            throws ProfileSeriesException if any row Series IS NULL
        ValidateProfileSeriesStatus(DT)
            for each (BIdentNo, Series): UniLink.Entities.Profile.DxfExists
            throws ProfileSeriesException listing missing profiles
    AddColumns(DT)            → ProdTraceId, StepFilepath, StepFileExists
    SetConverterData(DT)
        for each row:
            ICO = ICenterObject.GetIcenterObject(Modelname)
            StepFilepath = PMMExportHandler.GetExport3DFilepath(Modelname)
            StepFileExists = File.Exists(...)
            If oICENTER.GetEluGenAutoAddIaNr(ICO.GenName) Then
                ProdTraceId = NumberLib.GetNumberFromIaNr(IPPartId)
            End If
    ConvertDataTable(DT) → ImportData

CAMImport = New UniLink.CSVImportHandler()
CAMImport.Import(Data)
```

## `ImportData` schema (23 columns)

```
Pos               Int32   (primary key)
Qty               Decimal
Modelname         String
Rev               String
Description       String
PartCode          String
Serie             String
BIdentNo          String
Material          String
Length            Double
CLength           Double
Width             Double
CAngleLH          Double
CAngleLV          Double
CAngleRV          Double
CAngleRH          Double
DgxOrient         Int32
CutOnDg           Boolean
VertCutAllowed    Boolean
ProdTraceId       String
StepFilepath      String
StepFileExists    Boolean
```

Note: `Serie` is the column name in `ImportData` (Dutch), while the source BOM uses `Series` (English). The `ConvertDataTable` step maps `Series → Serie`.

## `PMMExportHandler` — STEP-file discovery {#pmmexporthandler-stepfile-discovery}

`PMMExportHandler` does two things:

- **Reads**: `GetExport3DFilepath(Modelname)` resolves the STEP filepath from the iCenter XML by reading `<MANUFACTURING type='NCW'><PMMEXPORT3D name="..."/>` (lines 89–102). Returns `<MANUF folder>\<name>`.
- **Writes**: `ProcessFile()` is the inverse — moves a freshly-generated STEP from `SourceFilepath` to the MANUF folder, then updates the iCenter XML's `<PMMEXPORT3D>` element with the new filename (lines 17–76). `Attributes.RemoveAll()` then `SetAttribute("name", TargetFilename)` — replaces any prior attributes.

The handler is **constructed differently depending on direction**: callers reading just use the `Shared GetExport3DFilepath`; callers writing instantiate with `(Modelname, SourceFilepath, ElementName)` and call `ProcessFile`.

## `ExternalReferenceHelper.GetExternalReferenceSTEPFiles`

Walks the iCenter XML's `EXTERNALREFERENCES/EXTERNALREFERENCE` children. Each EXTERNALREFERENCE has `<ORIGIN>`, `<ORIGTYPE>`, `<EXTREFTYPE>` sub-elements. For refs matching the requested type, looks up the referenced model's `GetFirstStpDoc()` and returns a `Dictionary(Of String, ICenterDoc)` keyed by Origin.

Used by code that needs the full set of supporting STEP files for a model (e.g. assembly drawings whose components live in separate models).

## Business rules surfaced here

- **Every profile *must* have a `Series`** filled in the EluCad profile DB or the import throws (line 113–121). Operator-friendly error: _"Bij deze profielen onbreekt de parameter 'Serie' in de EluCad profieldatabank."_
- **Every (BIdentNo, Series) must have a matching DXF in UniLink** or the import throws (line 124–146). Operator-friendly error: _"Deze profielen ontbreken in UniLink."_
- **`AutoAddIaNr` is per-profile-generic, not per-job.** `oICENTER.GetEluGenAutoAddIaNr(GenName)` decides whether a `ProdTraceId` gets injected — and the injected value is `NumberLib.GetNumberFromIaNr(IPPartId)`. Q-102.
- The STEP filepath comes from the iCenter XML, not a file-system scan. If the XML's `<PMMEXPORT3D name>` is stale, `StepFileExists = False` and downstream code must handle it.

## Surprises

1. **The two error messages above are Dutch** — production-floor language. Translating them changes documented behaviour.
2. **`PMMExportHandler.ProcessFile`** does `Attributes.RemoveAll()` then `SetAttribute("name", TargetFilename)`. Any *other* attributes on `<PMMEXPORT3D>` are wiped. Q-103.
3. **`ImportHandler.GetBom` builds two EluCad instances** (one in `GetBom`, one in `SetConverterData` via `ICenterObject`). Same pattern as elsewhere.
4. **`ConvertDataTable` does 23 column-by-column copies** — verbose. Refactor into an automapper if more columns ever land.
5. **`ICAMImport` is an interface with one method.** Currently has exactly one implementor (`UniLink.CSVImportHandler`). Pluggability point if iCenter ever exports to a non-UniLink CAM.
6. **`ImportData.PrimaryKey = Pos`** — if two BOM rows have the same `Pos`, the second `Add` throws. Whether multi-level BOM is guaranteed unique-per-Pos isn't checked here.

## Open questions

- **Q-102 (new):** the `AutoAddIaNr` per-generic flag — document the SME workflow for turning this on/off per profile. `#needs-review`
- **Q-103 (new):** `PMMExportHandler.ProcessFile` wipes all `<PMMEXPORT3D>` attributes before setting `name`. Should any other attributes be preserved? `#needs-review`

Logged in [[../needs-review/_index]].

## External systems touched

- [[../external-systems/icenter-db|iCenter DB]] via `oICENTER.GetEluGenAutoAddIaNr`, `ICenterObject.*`.
- [[../external-systems/elumatec-sbz140|Elumatec / EluCad]] via `EluCadApp.GetProfileInfoByPartCode`.
- **UniLink** — `UniLink.CSVImportHandler` (implements `ICAMImport`), `UniLink.Entities.Profile`, `UniLink.Exceptions.ProfileSeriesException`. UniLink is a sibling project at `iCENTER\UniLink\` (30 files; not yet documented).

## Coverage

`_coverage.md`:
- `Production\ProfileMilling\ImportHandler.vb` → `done`
- `Production\ProfileMilling\PMMExportHandler.vb` → `done`
- `Production\ProfileMilling\ExternalReferenceHelper.vb` → `done`
- `Production\ProfileMilling\ImportData.vb` → `done`
- `Production\ProfileMilling\ICAMImport.vb` → `done`
