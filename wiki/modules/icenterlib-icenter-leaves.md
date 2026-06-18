---
type: module
title: "iCenter folder leaves — Part / XmlFile / Servicedesk / TimeRegistration / etc."
status: done
module: "ICenterLib/iCenter"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\Part.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\XmlFile.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\Servicedesk.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\TimeRegistration.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\DossierContactFavorite.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\DataServices\\CoatingLayerThicknessDataService.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\ExternalReference.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\ExternalReferences.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\SurfaceTreatmentDefinition.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# iCenter folder leaves

> Grouped note covering 9 small files: `Part`, `XmlFile`, `Servicedesk`, `TimeRegistration`, `DossierContactFavorite`, `CoatingLayerThicknessDataService`, `ExternalReference[s]`, `SurfaceTreatmentDefinition`.

## `Part.vb` — SMT material catalog + cycle-time formulas

180 lines, four shared functions:

### `GetSmtDeburrCalcCycleTime(SmtOutlineX, SmtOutlineY, SmtThickness, SmtMaterialId, SmtFlowGrillQuality) → Double` `#safety-relevant`

Formula:
```
CalcCycleTime = (SmtOutlineX / 1000 * SmtOutlineY / 1000) / DeburrSpeed
If SmtThickness >= 4 AndAlso SmtMaterialId.StartsWith("AL") Then CalcCycleTime *= 2
If SmtFlowGrillQuality Then CalcCycleTime *= 2
```

Where `DeburrSpeed = AppSettings.SmtDeburrSpeed`.

**Three rules baked in**:
1. Base: area-divided-by-DeburrSpeed.
2. **Aluminium ≥ 4mm = 2× time** ("Niet netjes, eigenlijk oplossen door de T_SmtMaterials tabel aan te vullen" — author admits it's a hack; real fix would be DB-driven material-thickness coefficients).
3. **FlowGrill quality = 2× time** ("FlowGrill altijd 2x afbramen of op halve snelheid" — FlowGrill orders always deburr twice or at half speed).

Documented as [[../business-rules/icenter-smt-deburr-cycle-time]].

### `GetSmtMaterialProperty(Material, SmtThickness, ReturnField)` / `GetSmtMaterialBySmtMaterialId(SmtMaterialId, ReturnField) → String`

Material catalog lookups on `T_SmtMaterials`. Two-key access patterns: (Material + Thickness) or (TruTopsMatId). `ReturnField` is a fixed enum-like string ("PartCode" / "TruTopsMatId" / "MATERIAL" / "PARTCODE" — case sensitivity varies between the two methods — Q-241). Returns `"-"` sentinel on no-data.

### `GetMaterials() → DataTable`

`SELECT Material, SmtThickness, PartCode, TruTopsMatId, NonMachinedInd, CONCAT(Material,' ',SmtThickness,'mm') AS DisplayName FROM T_SmtMaterials` — used by UI dropdowns.

### `GetSmtCalcCycleTime(MaterialId, LayerName, ClosedContours, OpenContours, PathLength) → Double` `#safety-relevant`

Calls SP `JIP_GetSmtCycleTime` with five parameters. Pre-validates contour counts:

```vb
If ClosedContours > Int16.MaxValue Then Throw New Exception(...)
ElseIf ClosedContours < 0 Then ClosedContours = 0
' same for OpenContours
Dim sOpenContours As Int16 = sOpenContours = CType(OpenContours, Int16)  ' BUG: see Q-221
```

**Line 147 typo**: `Dim sOpenContours As Int16 = sOpenContours = CType(OpenContours, Int16)`. The middle `=` is parsed as a comparison `(sOpenContours = CType(OpenContours, Int16))` returning Boolean — since `sOpenContours` is uninitialised (defaults to 0), the comparison `0 = OpenContours` returns True only if OpenContours == 0. The result (Boolean) is then assigned to `sOpenContours` as 0 or -1 (False/True converted). **`sOpenContours` is always 0 or -1, never the actual OpenContours value**. Q-221 **`#safety-relevant`**.

The SP gets called with the bugged `sOpenContours` — likely produces wrong cycle times.

## `XmlFile.vb` — per-Modelname document-pointer XML

158 lines. Manages **`<Modelname>.xml` files** stored under `AppSettings.XML_ROOT_SERVER\<subdir>\<Modelname>.xml`. The XML has a `<DOCUMENTS>` section listing all CAD/PDF/image/etc. files associated with this Modelname.

### `Shared GetXMLFilepath(Modelname) → String`

Returns the per-Modelname XML path. Modelname is upper-cased; sub-directory is computed by `Common.GetSubDir(XML_ROOT_SERVER, Modelname)`. Q-242 — what is `GetSubDir`'s sharding rule?

### `AddDocument(DocumentPath) → Boolean`

Appends a `<DOCUMENT>` node under `<DOCUMENTS>` with `<TYPE>`, `<DESCRIPTION>`, `<PATH>` children. The `<PATH>` is **UNC-normalised** via `MySystem.Network.GetUNCfromMapping` (so drive-mapped paths become `\\server\share\...`). De-duplicates: skips if the doc path already exists (case-insensitive).

Adds an XML **comment** with `Last modified by: {USERNAME}@{Computername} on {timestamp}` — provenance trail.

### `Shared GetDocumentInfo(file, returnValue As String) → String`

The **file-extension → document-type classifier**:

| Extension | Type | Description |
|-----------|------|-------------|
| URL starting `HTTP` | `HTML` | filename |
| `.jpg`/`.bmp`/`.png` ending `_main.jpg` | `MAINIMAGE` | "Afbeelding" |
| `.jpg`/`.bmp`/`.png` else | `PICTURE` | "FOTO" |
| `.pdf` ending `DeclarationOfPerformance.PdfSuffix` | `PDF` | "CE" |
| `.pdf` else | `PDF` | "TEKENING" |
| `.dxf` | `DXF` | "DXF" |
| `.ecw`/`.ncw` | `OTHER` | "EluCad" |
| `.pvz` | `PVZ` | "ProductView" |
| `.ifc` | `IFC` | "IFC" |
| `.dwg` | `DWG` | "DWG" |
| `.obj` | `OBJ` | "OBJ" |
| `.vb` | `VB` | "PCFNet Source" |
| `.stp`/`.step` | `STEP` | "STEP" |
| `.rld` | `LASERWORKS` | `LaserWork.AppWrapper.AppLabel` (dynamic!) |
| else | `OTHER` | extension uppercased |

Documented as domain-concept (Q-223).

## `Servicedesk.vb` — Zammad integration URL builder

144 lines. Builds URLs into JAZO's **Zammad** ticketing system:

- `GetNewTicketUrl(EmpId)` — appends `&empid=...&name=...&computername=...` to `AppSettings.GetStringSetting("ServicedeskFeedbackUrl")`.
- `GetOverviewUrl()` — `BaseUrl + "#ticket/view/my_subscribed_tickets"`.

Display helpers exposing **icons + labels** for menu integration: `MenuItemText` ("Servicedesk"), `MenuItemImage` (`My.Resources.Zammad_icon_24`), `NewTicketText` ("Nieuw ticket"), `OverviewText` ("Overzicht"), and image resources.

### Email-fallback rule

```vb
If EmpId = Common.GuestEmpId OrElse Employee.IsMachineEmpId
   OrElse LMA Is Nothing OrElse LMA.Trim = "" Then
    Email = "pvs@jazo.com"
Else
    Email = LMA
End If
```

**Hardcoded fallback `pvs@jazo.com`** for guest/machine/unmailed employees. Documented as [[../business-rules/icenter-servicedesk-fallback-email]]. Q-243 — what does pvs@jazo.com correspond to (probably ICT support / "Per Vorm-Schade"-like distro)?

### `CreateNewEmailMessage(ProgId, EmpId)` — entirely commented out

Method body is `Try ... Catch` block of comments. Comment: `"Niet in gebruik om alles via één spoor te laten verlopen"` — kept as historical artifact. **`#dead-code`** in spirit.

## `TimeRegistration.vb` — iCenter-side time-reg display

127 lines. Two functions for the time-reg display UI:

### `GetFilteredRecords(DictFilter)`

Builds a `T_TimeRegistration ⨝ T_ProdMachines ⨝ T_IPpartLines` query with WHERE LIKE filters from a `Dictionary(Of String, String)`. **String-concat WHERE clause construction** with no parameterisation:

```vb
Filter &= ColumnName & " LIKE '" & DictFilter.Values(i) & "'"
```

**SQL injection vector** if the dictionary values come from an untrusted source. Q-244 `#safety-relevant`. Comment on line 48 ("CB 2023-02-08: added handling of column MachineId") suggests recent maintenance — probably internal-only UI use, but still risky.

Fallback when filter is empty: `IPRL.IPpartId=0` — returns no rows.

### `GetRecord(TimeRegLineNr)`

Same query shape but parameterised by `@TimeRegLineNr`. Single-row lookup.

## `DossierContactFavorite.vb` — per-dossier favourite contacts

168 lines, 5 shared methods on `T_DossierContactFav`:
- `GetDossierContactFavorites(dossierCode)` — list.
- `InsertDossierContactFavoriteManual(...)` — typed insert with full contact details (7 columns); `IF NOT EXISTS` guard.
- `InsertDossierContactFavoriteCustRelation(DossierCode, CustId, CustRelCode)` — minimal insert pointing at a CustomerRelation.
- `InsertDossierContactFavoriteInternal(DossierCode, EmpId)` — internal-employee bookmark.
- `DeleteDossierFavorite(FavoriteId)`.

Three insert paths reflect three contact origins: manual (typed in), customer-relation (linked), internal-employee (linked).

## `CoatingLayerThicknessDataService.vb` — coating-thickness query

50 lines, two methods:

- `GetLatestEntryDate(ProdHeaderDossierCode)` — `DT.AsEnumerable().Max(Function(r) r.Field(Of Date)("CreatedOn"))`. Returns the most-recent CreatedOn from `T_CoatingLayerThickness`. Used by [[isah-sub-services|`UpdateProdLeadTimeHandler` for MachGrp M38]] as the "previous-operation end-time" surrogate.
- `GetByProdHeaderDossierCode(ProdHeaderDossierCode)` — full DataTable of `(CreatedOn, CreatedBy, LayerThickness, ProdHeaderDossierCode, Info)`.

The **coating-thickness measurements** drive M38 (powder-coat?) lead-time-actuals. Each measurement is presumably entered by an operator on the coating line.

## `ExternalReference.vb` / `ExternalReferences.vb` — Creo external-ref bag

Two-file pair representing **Creo CAD external references** (copy-geom links between parts/assemblies):

```vb
Public Class ExternalReference
    Public Property Origin As String                              ' the source filename
    Public Property ExtRefType As ExternalReferenceType
    Public Property OrigType As OriginType

    Enum ExternalReferenceType : CopyGeom = 1                     End Enum
    Enum OriginType            : Part = 1, Assembly = 2           End Enum

    Public Function GetImageKey() As String                       ' "PART_LINK" / "ASSY_LINK"
End Class
```

`ExternalReferences` (collection) inherits `CollectionBase`. Standard `Add (no-dup by Origin)`, `AddRange`, `Find` (case-insensitive), `Contains`, `Last`, indexer by index or key, `GetDataTable()`.

Two enum values only — `CopyGeom` and (Part / Assembly). Used by CAD-side code to track which parts have copy-geom dependencies on other parts/assemblies.

## `SurfaceTreatmentDefinition.vb` — DTO (10 lines)

Plain DTO:
```vb
Public Class SurfaceTreatmentDefinition
    Public Property SurfTreatSystemPartCode As String
    Public Property SurfTreatColorPartCode As String
    Public Property DTSealant As DataTable
End Class
```

The three-key surface-treatment definition: paint system + color + sealant lookup. Consumers presumably build this from `T_SurfTreatSystem` etc. ([[isah-part-and-dispatch|Part.SurfTreatSystem*PartCode]] columns mentioned).

## Surprises

1. **`Part.GetSmtCalcCycleTime` line 147 typo** — almost certainly a production bug. Q-221 `#safety-relevant`.
2. **`Part.GetSmtMaterialProperty` vs `GetSmtMaterialBySmtMaterialId` case-mismatch** — one uses `Select Case ReturnField` (case-sensitive), the other `ReturnField.ToUpper`. Q-241.
3. **`Part.GetSmtDeburrCalcCycleTime` author admits hack** for AL≥4mm — should be DB-driven. Comment-as-bug-report.
4. **`XmlFile.GetDocumentInfo`** has a `Return "PDF"` fallthrough at the end (after the `Select Case sReturnValue`) — if `sReturnValue` is none of "TYPE"/"DESCRIPTION"/"PATH", default to "PDF". Surprising default. Q-245.
5. **`TimeRegistration.GetFilteredRecords` SQL-LIKE-injection** vector. Q-244 `#safety-relevant`.
6. **`Servicedesk.CreateNewEmailMessage` is entirely commented-out body** — `#dead-code`.
7. **`ExternalReferences.GetDataTable`** sorts by column "Name" but the table only has Origin/OrigType/ExtRefType. **Crashes at runtime** — `DataView.Sort = "Name ASC"` with no Name column throws. Q-246.

## Open questions

- **Q-221 (existing):** `Part.GetSmtCalcCycleTime` typo. `#safety-relevant`
- **Q-241 (new):** Case-mismatch in Part material-lookup ReturnField handling.
- **Q-242 (new):** `Common.GetSubDir(root, Modelname)` — what is the sharding rule?
- **Q-243 (new):** `pvs@jazo.com` — what distribution list / inbox?
- **Q-244 (new):** `TimeRegistration.GetFilteredRecords` interpolates WHERE values. SQL-LIKE injection risk if dictionary values are user-supplied. `#safety-relevant`
- **Q-245 (new):** `XmlFile.GetDocumentInfo` default-returns `"PDF"` when sReturnValue is unrecognised. Surprising.
- **Q-246 (new):** `ExternalReferences.GetDataTable` sorts by missing "Name" column. Runtime crash.

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/icenterlib-icenter]] — parent.
- [[icenterlib-sub-services|`UpdateProdLeadTimeHandler`]] — consumer of `CoatingLayerThicknessDataService`.
- [[../external-systems/zammad|Zammad]] — Servicedesk target.

## Coverage

- `iCenter\Part.vb` → `done`
- `iCenter\XmlFile.vb` → `done`
- `iCenter\Servicedesk.vb` → `done`
- `iCenter\TimeRegistration.vb` → `done`
- `iCenter\DossierContactFavorite.vb` → `done`
- `iCenter\DataServices\CoatingLayerThicknessDataService.vb` → `done`
- `iCenter\ExternalReference.vb` → `done`
- `iCenter\ExternalReferences.vb` → `done`
- `iCenter\SurfaceTreatmentDefinition.vb` → `done`
