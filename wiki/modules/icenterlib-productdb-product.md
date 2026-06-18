---
type: module
title: "ProductDb.Product — the central product entity"
status: done
module: "ICenterLib/ProductDb"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ProductDb\\Product.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `ProductDb.Product` — the central product entity

> 915 lines. The wrapper around `T_Product` and ~10 related tables. **Spans two databases** (ProductDb + ISAH) and the JIBA-portal automation-job table.

## Public surface

```vb
Public Class Product
    Public ReadOnly Property ProductId As Integer
    Public ControlDefinitionBuilder As ControlDefinitionLib.ControlDefinitionBuilder_v2

    Enum TypeOfEnvironment        { PROD=1, TEST=2 }
    Enum StatusId                 { Progress=1, Ready=2, Released=99, Obsolete=999 }
    Enum DisplayMode              { BasicReadOnly=0, AdminMode=99 }
    Enum GetProductSelectionType  { Source=0, CheckedIn=1 }

    Public Sub New(ProductId As Integer)
    Public Shared Function CreateByPartCode(PartCode As String)              As Product

    ' Read
    Public ReadOnly Property IsConfigurable                                  As Boolean
    Public ReadOnly Property Exists                                          As Boolean
    Public Function GetField(Field As String)                                As Object        ' ⚠ column injection
    Public Function GetPartCode()                                            As String
    Public Function GetDesignCode()                                          As String         ' bridges to ISAH.Part
    Public Function GetStatusId()                                            As StatusId
    Public Function GetStatusDescription()                                   As String
    Public Function GetControlDefinitionId()                                 As Integer
    Public Function GetExcelTemplateId()                                     As Integer
    Public Function GetProductTypeId()                                       As Integer        ' magic ControlId=82
    Public Function GetTranslatedName(LangCode)                              As String
    Public Function GetWebsiteUrl(LangCode)                                  As String
    Public Function GetIsUploadAllowed(Environment)                          As Boolean
    Public Function GetIsUploadAllowed(Environment, StatusId)                As Boolean
    Public Function ValidateTemplate()                                       As Boolean
    Public Function GetControlDefinitionCombinedXml(LangCode)                As XmlDocument
    Public Function GetControlDefinitionCombinedXml(Source, LangCode)        As XmlDocument
    Public Function GetProductDefinition(LangCode, SelectionType)            As DataSet
    Public Function SIP_Get_ProductUrl(LangCode)                             As DataSet
    Public Function GetDefaultSurfaceTreatmentDefinition(SelectionType)      As ICenter.SurfaceTreatmentDefinition

    ' Write
    Public Sub CreateCheckinJob(Environment, EmpId, UploadToWeb)             ' submits JIBA automation job
    Public Sub CopyDefinition(ToProductId, Description, Employee)            ' SP SIP_Cpy_ProductDef
    Public Function CopyToNew(Name, CopyDocs, EmpId, StatusId)               As Product   ' SP SIP_Cpy_Product + per-row File.Copy
    Public Sub UpdatePartCode(Value)
    Public Sub UpdateExcelTemplateId(Value)
    Public Sub UpdateControlDefId(Value)
    Public Sub UpdateControlMapId(Value, ControlDefId)

    ' UI navigation
    Public Sub OpenFilterUrl()      ' Chrome-only, JIBA-portal URL builder
    Public Sub OpenSettingsUrl()    ' same

    ' Static
    Public Shared Function GetProductsByMaxStatus(MaxStatus)                 As DataTable
    Public Shared Function GetTreeNodes(DisplayMode, UserLangCode, ShowPrivate) As Windows.Forms.TreeNode()
End Class
```

## Behavior — key flows

### Status lifecycle

```
Progress (1)  →  Ready (2)  →  Released (99)  →  Obsolete (999)
```

The **gap from 2 to 99** is intentional — Released sits at 99, Obsolete at 999, leaving room for intermediate statuses. Magic-number sentinels. Documented as [[../business-rules/productdb-status-id]].

### Upload gate (`GetIsUploadAllowed`)

```vb
Select Case StatusId
    Case StatusId.Released:                                       Result = True       ' both envs
    Case StatusId.Ready:    If Environment = TEST Then Result = True                  ' test only
End Select
```

- **Released** → upload to PROD or TEST.
- **Ready** → upload to TEST only.
- **Progress / Obsolete** → no upload at all.

Documented as [[../business-rules/productdb-upload-status-gate]].

### `CreateCheckinJob(Environment, EmpId, UploadToWeb)` — submits a JIBA automation job

Builds an XML payload:

```xml
<JIBATask>
  <ProductId>{ProductId}</ProductId>
  <Environment>{PROD|TEST}</Environment>
  <EmpId>{EmpId}</EmpId>
  <UploadToWeb>{0|1}</UploadToWeb>
</JIBATask>
```

Then calls `SIP_Ins_AutomJob_v2` on the **JIBA database** (not ProductDb) with hardcodes:
- `@AutomationId = 44` — JIBA-side "product checkin" automation type.
- `@JobTypeId = 2` — "scheduled job".
- `@EmpId = "0798"` — **system-publisher account EmpId**. The `EmpId` from the iCenter user is in the XML payload; the SP parameter is always "0798".

The "0798" is documented as [[../business-rules/productdb-system-publisher-empid]] `#safety-relevant`.

### `GetField(Field) → Object` — the column-injection vector

```vb
Dim sSQL As String = "SELECT " & Field & " FROM T_Product WHERE ProductID = @ProductId"
```

Field is interpolated raw into the SQL. Used internally by `GetPartCode`, `GetStatusId`, `IsConfigurable`, `Exists`, etc. — all callers pass fixed string literals. **No external caller may supply a Field value** without injection risk. Q-269 `#safety-relevant`.

### `GetDefaultSurfaceTreatmentDefinition(SelectionType) → SurfaceTreatmentDefinition`

Pulls `GetProductDefinition(DefaultLangCode, SelectionType)` then reads **`DS.Tables(24)`** for the control-default values. ControlIds:
- **81** → `ColorCode` (with `AttrName='DefaultValue'`, falling back to `DefaultTestValue` if absent).
- **168** → `CoatingWarrantyId` (same fallback).

Then calls `CommonDb.SetSurfTreatPartCodes(PartCode, ColorCode, "", ByRef SurfTreatSystemPartCode, ByRef ColorPartCode, "")` to resolve PartCodes. Returns a [[icenterlib-icenter-leaves|`SurfaceTreatmentDefinition`]] with three populated fields.

**Magic table index `24`** is the canonical fragility — any change to the SP's output shape rotates this index. Q-272 `#safety-relevant`.

### `GetProductTypeId() → Integer`

Reads `ControlGroup/Controls/Control[@ControlId='82']/@DefaultValue` from the control-definition XML. Magic ControlId=82.

Author comment: `''Temporarily using this method. Use GetParamsFromProductDef again when new XmlDefinition is live` — there's a commented-out alternative implementation below. Tech-debt marker.

### `CopyToNew(Name, CopyDocs, EmpId, StatusId)`

1. Calls SP `SIP_Cpy_Product` with `@NewProductId` output param.
2. SP returns a DataTable of `(Source, Destination)` file pairs to copy.
3. Creates the destination directory if missing.
4. For each row, `If Not File.Exists(Destination) Then File.Copy(Source, Destination)`.
5. Returns `New Product(NewId)` or `Nothing` on failure.

**Non-transactional**: if file 5 of 20 fails, the first 4 are copied and committed. The new ProductId may already be created in the DB. Q-273.

### `OpenFilterUrl()` / `OpenSettingsUrl()`

Read `JIBA.AppParameter("JibaUrl").GetValue` for the base URL, then call `Common.OpenInChrome(Url, False, True)` with:
- Filters: `{BaseUrl}js/ProductDb/Filters.html?ProductId={ProductId}`
- Settings: `{BaseUrl}ContentPages/ProductDb/Productlauncher.aspx?ProductId={ProductId}`

**Chrome-only**. Q-271.

## Surprises

1. **Spans 3 databases**: ProductDb (most queries), ISAH (via `GetDesignCode`), JIBA (`SIP_Ins_AutomJob_v2`).
2. **Magic numbers everywhere**: ControlIds 81/82/168, table index 24, status values 1/2/99/999, AutomationId 44, JobTypeId 2, EmpId "0798". The folder smells like a config table waiting to be created.
3. **`Product` doesn't cache anything** — calling `GetPartCode` then `IsConfigurable` then `GetStatusId` runs three separate ProductDb queries.
4. **Silent error handling** in every method — `Catch ex As Exception ... Return Nothing/0/Empty` family. Same as ProductionMachines.
5. **`StatusId = 999` Obsolete sentinel** — UI must check for both Released=99 and Obsolete=999 in filters; easy to miss.
6. **`DisplayMode.AdminMode = 99`** — coincidentally same value as Released status. Different semantic.
7. **`CreateCheckinJob` builds XML by string concat** — `EmpId` or `Description` containing `<`/`&`/`"` would corrupt the XML. Q-274.

## Open questions

- **Q-267..Q-272 (existing):** ControlId magic numbers, c:\temp\dump path, GetField injection, EmpId 0798, OpenInChrome dependency, DS.Tables(24).
- **Q-273 (new):** `Product.CopyToNew` non-transactional partial-copy hazard.
- **Q-274 (new):** `Product.CreateCheckinJob` string-concat XML — XSS / XML-injection in EmpId / Description.

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/icenterlib-productdb]] — parent.
- [[isah-part-and-dispatch|`ISAH.Part`]] — `Product.GetDesignCode` bridges here.
- [[icenterlib-jiba-employee-asset|`JIBA.AppParameter`]] — used for portal URL base.
- [[../business-rules/productdb-status-id|Product.StatusId lifecycle]].
- [[../business-rules/productdb-upload-status-gate|Upload-to-environment gate]].
- [[../business-rules/productdb-system-publisher-empid|System EmpId "0798"]].

## Coverage

`ProductDb\Product.vb` → `done`
