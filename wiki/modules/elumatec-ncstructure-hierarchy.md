---
type: module
title: "Elumatec in-memory NC hierarchy — Job/Bar/Cut/Plane"
status: done
module: "iCENTER/Elumatec/NcStructure"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\NcStructure\\Job.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\NcStructure\\Bar.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\NcStructure\\Cut.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\NcStructure\\Plane.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\NcStructure\\PlaneCollection.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Elumatec NC structure — `Job` → `Bar` → `Cut` → `Work` + `Plane`

## Purpose

The **in-memory data model** for a single Elumatec NC submission. Five classes that compose a tree from the workshop order down to individual machining features.

```
Job          (a customer/work order — one .ecw file)
 └── Bar*    (one per profile PartCode in the job)
      ├── PlaneCollection      (the bar's user-defined planes; standard sides 1-6 are implicit)
      └── Cut*  (one per piece cut from the bar)
           ├── Works: List<Work>      (features — see Works MOC)
           ├── _Planes: PlaneCollection
           └── RuntimeManipulations
```

Workflow:
1. Built either by [[elumatec-elucadfile|`EluCadFile.ReadFromArray`]] (parsing `.ecw`) or assembled programmatically by the rest of the Elumatec pipeline.
2. Mutated in place by [[elumatec-works-replacement-base|`WorksReplacement.ApplyTo`]] (replacements rewrite `Cut.Works`).
3. Serialised back out via `GetEcwText()` (ECW format) or `GetNcwText()` (NCW format).

## `Job` — top-level work order

**Source:** `Elumatec\NcStructure\Job.vb` (14 KB, ~380 lines)

### Public surface

| Property / method | Type / signature | Meaning |
|-------------------|------------------|---------|
| `Name` | String | job name |
| `cncdriver` | String = `"1.1elu"` | driver marker (Q-052) |
| `order` | String | order number |
| `info` | String | free-text annotation |
| `Var0`..`Var9` | Double × 10 | macro-variable slots, defaulted to 0 |
| `JActive` | Integer = 1 | 0 = suppressed |
| `JNo` | Integer | job number |
| `iCenterVersion` | String | iCenter version that produced the job |
| `AppVersionId` | Integer = 1 | EluCad app version |
| `NcVersion` | Integer = 0 | NC version |
| `ProfMillJob` | `ProfMillJob` | link back to the ProfMillJob runtime representation |
| `Bars` | `Dictionary(String → Bar)` keyed by `PartCode` | the bars in this job |
| `AddCut(PartCode, Cut, Optional myBarNode)` | Sub | adds a cut; creates a Bar if PartCode is new; resolves `BIdentNo` from `oELUCAD.GetProfileInfoByPartCode(PartCode, BIdentNo)`; reads dimensions |
| `GetBars()`, `GetCuts()` | Functions | flatten the dictionary / iterate all cuts |
| `MaxCutNo`, `MaxWorkNo`, `MaxGroupNo` | properties | numbering for adding new features |
| `GetXmlNode(oXMLDoc)`, `GetEcwText()`, `GetNcwText()` | serialisation | three output formats (XML / ECW text / NCW text) |
| `PropertiesTable` | DataTable | UI-bound property bag |
| `ContainsOnlySawCuts()` | Function | branch helper |
| `GetGroupByName(Name | Names)` | Function | feature lookup by Work name across all bars |
| `SetEdittedManually`, `SetCutsOutdated`, `SetCutsNcVersionOutdated` | Subs | cascade flag state to all cuts |
| `GetOutdatedManufacturingCuts`, `GetOutdatedCuts`, `GetNcVersionOutdatedCuts` | Functions | filter helpers |
| `ProcessRuntimeManipulation(InitialStatus)` | Sub | run all queued runtime manipulations on every Work |
| `ContainsRuntimeManipulations`, `ContainsOnlySuppressedByConverter(IgnoreTriggerSawCut)` | properties | state queries |
| `GetRuntimeManipulations()` | Function | aggregate from all bars |
| `GetCutByCNo(CNo)` | Function | lookup |
| `ExportProfilesToDxf(folder)` | Sub | export all bars' profiles as DXF |

### Surprises

- **`oELUCAD = New EluCadApp` is a private field** (line 32), constructed eagerly on every `Job`. Means each Job carries its own `EluCadApp` instance — re-reads the profile DB if asked. Wasteful but isolates state.
- The 10 `VarN` slots are hard-coded — adding an 11th means changing `Job` *plus* every macro file. They're written out unconditionally in `GetEcwText` (lines 184–193). The `GetNcwText` path comments them out (`'CB 2021-11-30: obsolete`) — meaning the NCW pipeline stopped using them.
- `Const HeaderText As String = ":JOB"` is the text-serialisation marker (matches the ECW parser).
- `AddCut` re-throws with a profile-id-prefixed message: `String.Format("{0}: {1}", PartCode_or_BIdentNo, ex.ToString)` — useful for log forensics.
- `EluCadApp.UnknownBIdentNo = "100999"` triggers a `Dim pp As Integer = 9` no-op (line 81) — looks like a forgotten debug breakpoint. `#needs-review`

## `Bar` — one physical profile bar

**Source:** `Elumatec\NcStructure\Bar.vb` (20 KB, ~470 lines). Namespace: `Elumatec.NCStructure` (note the `NCStructure` casing — differs from `Cut`/`Job`/`Plane` which live in plain `Elumatec`).

### Public surface

| Property / method | Type / signature | Meaning |
|-------------------|------------------|---------|
| `BIdentNo` | String | profile ID (e.g. `"100142"`) |
| `BWidth`, `BHeight`, `BLength` | Double | dimensions (mm) from the `.epd` profile DB |
| `BFileDB` | String | profile-DB path or filename |
| `PartCode` | String | ISAH part code, e.g. `JAZO`-internal SKU |
| `BActive` | Integer = 1 | 0 = suppressed |
| `BNo` | Integer | sequential bar number within the Job |
| `Job` | `Job` | back-reference |
| `Planes` | `PlaneCollection` | custom planes (Free / user-defined) |
| `Cuts` | `Private List(Of Cut)` | the cuts on this bar |
| `AddCut(Cut)` | Sub | adds a cut **and moves the cut's Planes onto the bar** (lines 60–62). Cut keeps `Planes = Nothing` afterwards. |
| `GetCuts()`, `MaxCutNo`, `MaxWorkNo`, `MaxGroupNo` | reduction helpers | |
| `GetXmlNode(oXMLDoc)`, `GetEcwText()`, `GetNcwText(NcwExportProfile, oMachine, ExportFixtureDxf)` | serialisation | |
| `PropertiesTable` | DataTable | UI-bound; includes ISAH `GetPartInfo(PartCode, Description)` |
| `ReadDimensions(oEluCad)` | Sub | lookup `BWidth`/`BHeight` from EluCad |
| `AutoAddIaNrEnabled` | Boolean | reads `oICENTER.GetEluProfilesForAutoAddIaNr(False)` and checks if this profile has the flag — selects whether to append an IA number to the NC program |
| `ContainsOnlySawCuts()` | Function | branch helper |
| `GetNextPlaneNo()` | Function | get next free plane number across all cuts |
| `CalcPartCode()` | Sub | reverse-lookup PartCode from BIdentNo via `EluCadApp.GetProfileInfoByBIdentNo` |
| `GetGroupByName`, `GetCutByCNo`, `ExportProfilesToDxf(folder)` | utility | |
| same flag cascade as Job | | |

### Key behaviours

- **`AddCut` mutates the input Cut** by stealing its `Planes` collection. Cut goes from owning planes to owning `Nothing` for planes. This means *the Bar owns the canonical plane definitions*, not the Cut. Cuts read planes via `Bar.Planes`. `#needs-review` — confirm this is the documented invariant.
- **`GetNcwText` calls Profile/Offset lookups** including reading the offset DB via `OffsetFile.Read(OffsetDbFilepath)` and a `FixtureCollection.ReadAll(FixtureDbFilepath)` per call (lines 158–168). I/O per bar serialisation; possibly slow for large jobs.
- **`BSuppl = "JAZO"`** hard-coded when `NcwExportProfile.IncludeBSuppl = True` (line 148). Embeds the company name as the bar's supplier marker.
- **`ContourData` from the profile** (line 181) is written out as-is when `IncludePolylines = True`. The commented-out block (183–198) shows an earlier per-vertex serialisation; replaced by passing the contour data verbatim.
- **`TreeNodeText` calls `oISAH.GetPartInfo(PartCode, Description)`** — pulls the ISAH description for display. Means rendering the tree triggers an ISAH lookup per bar.

## `Cut` — one piece cut from a bar

**Source:** `Elumatec\NcStructure\Cut.vb` (40 KB, ~820 lines)

### Public surface (high-level)

~30 properties describing the geometry and state of one cut piece:

- **Identity**: `CNo`, `CPartNo`, `CStation`, `CComNo`, `CDescription`
- **Geometry**: `CLength`, four corner angles `CAngle{L,R}{H,V}`, `CCorL/R`, `CChopAngleLeft/Right`, `CChopLeft/Right`
- **Transforms**: `CCopies`, `CRotation`, `CSawRotation`, `CMirror`, `COffsetX/Y/Z`
- **Status**: `CActive` (1 default), `EdittedManually`, `OutDated`, `NcVersionOutDated`, `Status` (StatusCode enum: OK / MissingProeManufData), `ExtraLength` (default 0), `BIdentNoRecognition`, `ContainsFullLengthOper`, `PublishProfileTimeStamp`, `DatabaseProfileTimeStamp`
- **Owned children**: `Works As List(Of Works.Work)`, `_Planes As PlaneCollection`, `RuntimeManipulations`, `SectionCutOffBox`
- **Back-references**: `Bar`, `Machine`, `Job` (read-only, via `Bar.Job`)

### Notable methods (~40 publics)

- `CmpCActive()` — recomputes whether the cut should be active.
- `UpdatePlanesFromWorks()` — derive plane definitions from the features.
- `GetEcwText()`, `GetNcwText()`, `GetXmlNode(...)` — three serialisations.
- `GetWorksAt(WSide)` / `GetWorksAt(WSide, MyType)` / `GetWorksOfType(MyType)` — feature filters used heavily by replacements (see [[elumatec-replacement-flowdrill|Flowdrill]]).
- `GetWorks(Optional IncludeGroupMembers)` — flatten groups into individual works.
- `GetWorksAsDatatable()` — DataTable view of features for UI binding.
- `RepairOriginRemoval(oBar)` — repair origin handling on the bar.
- `MinimizeMillTools(oTools)` — tool-minimisation pass.
- `ContainsInactiveSawCutsOnly()` — branch helper.
- `Clone() As Object Implements ICloneable.Clone` — used by replacements when they need an "original" copy.
- `GetMaxWNo()`, `MaxPlaneNo()`, `MaxGroupNo` — numbering.
- `ContainsOnlySawCuts()`, `GetProfMillIsRequired()` — branch helpers.
- `GetGroupByName(Name | Names)` — group lookup.
- `GetDxfAsStream(EluProfile, Side)` — render the cut as DXF.
- `GetWRotations()` — list of rotation values used.
- `SetGroupNames(ByRef MyCounter)` — re-number groups.
- `ProcessRuntimeManipulation(InitialStatus)` — apply manipulations.
- `WorkUpdatedHandler(RuntimeManipulationInstruction)` — event hook.
- `GetWork(WNo)`, `GetRuntimeManipulations()` — lookups.

### Surprises

- `StatusCode.MissingProeManufData = 1` — references *Pro*/E (Pro/Engineer, the predecessor to PTC Creo) manufacturing data. Suggests the recogniser still distinguishes Pro/E-originated parts. `#needs-review` (likely Q-052 cousin).
- `_Planes As PlaneCollection` is *private* with a `Public Property Planes` wrapper (line 253) — accessor likely has custom logic (transfer to/from `Bar`).
- `Cut.Job` is read-only via `Bar.Job`; if `Bar Is Nothing` returns `Nothing`. Cuts created in isolation have no Job.
- `Implements ICloneable` — the depth of `Cut.Clone()` is unspecified from the public surface. Phase-3 follow-up if Works/Replacements relies on a particular semantic.

## `Plane` + `PlaneCollection` — bar-face geometry

**Source:** `Elumatec\NcStructure\Plane.vb` (4 KB) + `PlaneCollection.vb` (2 KB)

### `Plane`

Eight properties describing one custom plane:

- `WPNo` (Integer), `WPRot`, `WPName` (default `"P<WPNo+1>"`), `WPAngleX`, `WPAngleZ`, `WPTransX`, `WPTransY`, `WPTransZ`
- `HeaderText = ":PLANE"` for text serialisation
- `GetEcwText()` / `getXmlNode(oXMLDoc)` — serialisation
- `SetWPNo(Value)` — sets `WPNo` *and* derives `WPName = "P" & (WPNo + 1)`
- `GetFromXmlNode(node, VarH, VarZ, VarY)` — parse from XML, evaluating `WPTransX/Y/Z` as expressions via `Works.Work.GetCalcContext()` + `GetCalculatedValue(...)`. **So plane translations can be formulas referencing the Job's Var0..Var9 + extra VarH/Z/Y inputs.**

### `PlaneCollection : Inherits CollectionBase`

| Method | Behaviour |
|--------|-----------|
| `Add(Plane)` | auto-assigns `Plane.SetWPNo(List.Count)` then appends |
| `Remove(index)` | silent no-op if out of range (an old commented `MessageBox` shows where the UI used to show an error) |
| `Item(index) As Plane` | get-only indexer |
| `getXmlNode(oXMLDoc)` | serialise the whole collection as `<PLANES>` |
| `GetEcwText()` | concat all plane ECW texts |
| `Contains(Plane)` | passthrough |
| `GetPlaneByWSide(WSide) As Plane` | **`Index = WSide - 7`** — returns `Nothing` for WSide < 7 (i.e. all six standard sides) |

The `WSide - 7` mapping (Q-051) means **`GetPlaneByWSide(1)` (Top) returns `Nothing`** — the collection is meant for *custom* planes only (Free=7 plus user-added 8+). Code calling this expecting to get the Top plane silently gets nothing.

## Domain concepts

- **Job** — single submission to an Elumatec machine.
- **Bar** — one physical extrusion / profile; many bars per Job.
- **Cut** — one piece cut from a bar; many cuts per Bar (`CCopies` for duplicates).
- **Plane** — a custom coordinate system on the bar (in addition to the 6 standard sides).
- **PartCode** — JAZO/ISAH internal part identifier (string).
- **BIdentNo** — Elumatec profile ID (3-6 digit string, e.g. `"100142"`).
- **CNo / BNo / JNo / WNo / WPNo** — sequence numbers within their container.

All [[../domain-concepts/_index|domain-concept]] candidates; SME-friendly names per `BIdentNo` should land during Phase 4.

## Open questions

- **Q-051** (from MOC): `PlaneCollection.GetPlaneByWSide(WSide → WSide - 7)` — confirm intended for custom planes only.
- **Q-055 (new):** `Bar.AddCut` moves the Cut's Planes onto the Bar — confirm invariant that Cut.Planes is always `Nothing` after being added. `#needs-review`
- **Q-056 (new):** `Cut.StatusCode.MissingProeManufData` references Pro/E. Still relevant after the Creo migration? `#needs-review`
- **Q-057 (new):** `Plane.GetFromXmlNode` evaluates `WPTransX/Y/Z` as expressions through `Works.Work.GetCalcContext()`. Document the expression grammar (variables, operators). `#safety-relevant`
- **Q-058 (new):** `Job` carries a private `oELUCAD = New EluCadApp` field; constructed on every Job. Confirm safe to share / pool. `#needs-review`
- **Q-059 (new):** `Bar.GetNcwText` performs I/O (`OffsetFile.Read`, `FixtureCollection.ReadAll`) per bar. Cache these for large multi-bar jobs? `#needs-review`

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/elumatec-ncpipeline|NC pipeline MOC]]
- [[elumatec-elucadfile|EluCadFile]] — parses ECW text into this hierarchy.
- [[elumatec-nc-program-family|NcProgram family]] — emits final NC files from this hierarchy.
- [[../mocs/elumatec-works|Works MOC]] — `Cut.Works` is the working set.

## Coverage

`_coverage.md`:
- `Elumatec\NcStructure\Job.vb` → `done`
- `Elumatec\NcStructure\Bar.vb` → `done`
- `Elumatec\NcStructure\Cut.vb` → `done`
- `Elumatec\NcStructure\Plane.vb` → `done`
- `Elumatec\NcStructure\PlaneCollection.vb` → `done`
