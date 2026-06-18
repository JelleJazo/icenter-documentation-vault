---
type: module
title: "PCFNet.GenericPart — the configurator base class (82 KB)"
status: done
module: "ICenterLib/PCFNet"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\PCFNet\\GenericPart.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `PCFNet.GenericPart.vb` — the configurator base class

> **The single biggest .vb file in the codebase** at 82 KB / 1742+ lines. `MustInherit` base for every PCFNet **product family** (StlDoor001, AluDoor001, AluBasicWallLouver001, etc.). Owns the **Regenerate** lifecycle, the BillOfMaterial tree, the BillOfOperation, surface-treatment expansion, fastener expansion, EBTV-galvanize special-handling, and the runtime DLL loader that pulls per-family code from disk on demand.

## Class shape

```vb
Public MustInherit Class GenericPart
    Protected MustOverride Sub Compute()                          ' per-family: compute dimensions/materials
    Protected Friend MustOverride Sub RunProProgram()             ' per-family: emit Pro/Program
    Public MustOverride ReadOnly Property GenericModelname As String

    ' State
    Public Property Name              As String                   ' family-instance identity
    Public Property RuntimeModelname  As String
    Public Property BillOfMaterial    As New Dictionary(Of GenericPart, Double)   ' children
    Public Property BillOfOperation   As New BOO
    Public Property Features          As New Features
    Public Overridable Property Length As Double                  ' meters
    Public Overridable Property Width  As Double
    Public Overridable Property Height As Double
    Public Property SquareMeasure     As Double                   ' m²
    Public Property Mass              As Double                   ' kg
    Public Property InputParameters   As New List(Of String)
    Public Property Materials         As Materials = Nothing
    Public Property NameMapping       As NameMapping = Nothing
    Public Property NameMappingRule   As NameMappingRule = Nothing
    Public Property SecondaryNameMappingRule As NameMappingRule = Nothing
    Public Property SvgGraphics       As DataTable = Nothing
    Protected Friend ExecutePart      As GenericPart
    Protected Friend SmtCalculator    As SmtCalculator = Nothing
    Protected Friend Property LoadedFromFile As Boolean = False

    Public Shared ReadOnly Property DEBUGMODE As Boolean = False  ' hardcoded; toggle requires recompile
    Public Const EnableLogging As Boolean = False
End Class
```

## Lifecycle: `Regenerate(DesignConfiguration)`

Top-level entry called by callers (e.g., `CadBatchserver` workers):

```
1. Resolve design parameters from DesignConfiguration (recursive lookup by RuntimeModelname).
2. ApplyConfigParameters(Dict)                                — apply per-parameter setter.
3. Compute()                                                  — abstract: per-family geometry/material.
4. CalculateSolidProperties()                                 — fills SquareMeasure + Mass.
5. ComputeOperations()                                        — overridable: per-family ops.
6. Execute()                                                  — overridable: per-family side-effects.
7. WriteBomToXml("Execute")                                   — debug dump (gated by EnableLogging).
8. UpdateBOM()                                                — rebuild BOM after computation.
9. WriteBomToXml("UpdateBOM")
10. For Each child In BillOfMaterial: child.Regenerate(DesignConfiguration)  — recurse.
```

The top-level `Compute` is **abstract**; each family supplies its own. Per-child `Regenerate` runs the same lifecycle, so a configured assembly with 50 child parts triggers 50× the Compute/Execute pairs.

`UpdateBOM()` is at line 422 — it traverses children and reconciles parent-quantities. Q-361 — deep behaviour requires SME.

## DLL-from-disk loading: `CreateByFilepath`

```vb
Public Shared Function CreateByFilepath(Classname, Modelname, MyAppDomain) As GenericPart
    Dim Filepath As String = CPart.GetDllFilePath(CPart.VBCodeFolder, Classname)
    Dim AssemblyBytes As Byte() = File.ReadAllBytes(Filepath)
    Dim oAssembly As Assembly = MyAppDomain.Load(AssemblyBytes)
    Dim oType As Type = oAssembly.GetType(Classname, False)
    If oType Is Nothing Then Return Nothing
    Dim obj As Object = Activator.CreateInstance(oType, {Modelname})
    Dim GP As GenericPart = CType(obj, GenericPart)
    GP.LoadedFromFile = True
    Return GP
End Function
```

**Per-family product code is a separate DLL on disk** at `CPart.VBCodeFolder`. iCenter loads it via `File.ReadAllBytes → AppDomain.Load(bytes)` (in-memory load — does NOT lock the DLL, allows replacement without app restart). Classname is the type name within the loaded assembly.

**All errors caught silently** (`Catch ex As Exception : Return Nothing`). A failed DLL load returns Nothing — caller can't tell why.

Q-357 — document the family-DLL deployment pipeline and the `CPart.VBCodeFolder` path.

## `AddBOMMember(Classname, Modelname)`

```vb
Public Function AddBOMMember(Classname, Modelname) As GenericPart
    Dim AppDomain As AppDomain = AppDomain.CurrentDomain
    Dim GP As GenericPart = CreateByFilepath(Classname, Modelname, AppDomain)
    If GP IsNot Nothing Then
        GP.NameMapping = NameMapping
        AddToBillOfMaterial(GP, 0)               ' Qty=0 initially; later UpdateBOM fills it
    End If
    Return GP
End Function
```

`AddToBillOfMaterial(GP, Qty)` just appends to the `Dictionary(Of GenericPart, Double)`. The comment confirms a 2021-08-23 refactor (`'2021-08-23 CB: replaced by AddToBillOfMaterial`).

## `AddSurfaceTreatment(ds, SystemPartCode, ColorPartCode)` `#safety-relevant`

Adds surface-treatment **Parts** + **Operations** to a `PcfNetDataSet`. Three phases:

### Phase 1 — Phantom marker row

When both PartCodes are non-empty, prepend a synthetic `~PHANTOM` row to `Part` representing the **chosen surface-treatment system**:

```vb
PhantomDR.Item("SubPartCode") = "~PHANTOM"
PhantomDR.Item("Description") = SystemPartCode    ' will be overwritten when written to ISAH
PhantomDR.Item("CADReference") = SystemPartCode
PhantomDR.Item("PartType") = Koopdeel             ' purchase part
PhantomDR.Item("OrdCode") = CalcInkopierenArtikelNietVerwijderen
PhantomDR.Item("OriginType") = CalculatieInkopierenPhantom
```

The `~PHANTOM` prefix is **JAZO's marker for synthetic BOM rows that should not be deleted by recalculation but also not really sourced** (`OrdCode = CalcInkopierenArtikelNietVerwijderen` = "Calc-purchase-don't-remove"). Q-360.

### Phase 2 — Surface-treatment parts

Calls `ISAH.BillOfMat.CreateSurfTreatment(SystemPartCode, ColorPartCode, ds.Tables("OperationTotal"), dtSurfTreatmentPart, dtSurfTreatmentOper)` — the ISAH side returns the parts + ops needed to coat the geometry.

For each part row:
- `Qty = (drSurfTreatmentPart.Qty / CalcQty) * SurfTreatSquareMeasure` — scales the per-unit material consumption by the actual coatable surface area.
- Phantom's Qty is set to `SurfTreatSquareMeasure` after the loop.

### Phase 3 — Surface-treatment operations

For each operation row:
- Lookup-or-insert by `MachGrpCode`.
- `MachCycleTime += drOper.MachCycleTime / CalcQty * SurfTreatSquareMeasure` — additive, area-scaled.
- `OccupationSetupTime = MachSetupTime` (same value duplicated into Occupation field).

### Phase 4 — Special EBTV (external galvanize) handling `#safety-relevant`

If `Operation` table has an `MachGrpCode = 'EBTV'` row:

```vb
Dim EbtvExtOperPartCode As String = AppSettings.GetStringSetting("ExtOperationPartCodeEBTV")
' If there's a 'Mass' value on the EBTV OperationTotal row:
MyRow.Item("Qty") = drs(0).Item("Mass")          ' EBTV billed by MASS (kg), not area!
```

So **EBTV galvanizing is billed by part mass** (zinc-bath cost scales with mass), whereas powder-coat is billed by surface area. The two SurfTreat paths differ at this exact line. Documented as [[../business-rules/pcfnet-ebtv-billed-by-mass]].

`AppSettings.GetStringSetting("ExtOperationPartCodeEBTV")` resolves the EBTV "external operation" PartCode. Q-362 — document this setting.

Logs `"EBTV error: "` on exception — preserves error context but doesn't fail the caller (the rest of the dataset is built).

## `ApplyFasteners(PNDS, InstalledBy)` `#safety-relevant`

Adds fastener parts + ops to the dataset. Triggered by the **magic literal `'AJPKL7000'`**:

```vb
Dim dv As New DataView(PNDS.PartTable) With {
    .RowFilter = "SubPartCode='AJPKL7000' AND Qty>0"
}
```

`AJPKL7000` is JAZO's marker SubPartCode meaning "**this row requires fastener calculation**". For each match:

1. Resolve fastening type (`PNDS.GetFasteningType(LineNr)`).
2. `FastenerCalculator.GetFastenerPartCode(SubPartCode, FasteningType, InstalledBy)` → the actual fastener PartCode (e.g., a specific bolt size).
3. `GetFastenerQty(SubPartCode, FasteningType.LageKant, Length*1000)` → quantity needed (`LageKant` = Dutch "low edge" — a specific fastening style). Q-363.
4. Multiply by parent Qty.
5. **`nr.Item("Qty") = Math.Ceiling(nr.Item("Qty"))`** — fasteners **always round up**. Safety-conservative.
6. Insert/merge into Part table.
7. For each fastener oper (drilling, etc.), insert/merge into Operation table by MachGrpCode (cycle + setup times accumulate).

`InstalledBy` is `Enums.Application.InstallBy` — drives which fastener variant gets picked (e.g., installed by JAZO vs by customer).

## Material property accessors

```vb
Public Property MATERIAL_PARAM(Propertyname)              As Object   ' default material
Public Property MATERIAL_PARAM(Propertyname, MaterialName) As Object   ' per-material override
```

Fallback to `Material.GetDefaultPropertyValue(Propertyname)` if `Materials Is Nothing`. **Per-family code references `MATERIAL_PARAM("density")` etc. uppercase** — Q-364 — what are the standard property names?

## `PartCode` resolution

```vb
Public Overridable ReadOnly Property PartCode As String
    Get
        Dim SmartPartCode = GetSmartPartCode()
        If SmartPartCode Is Nothing Then
            If PropertyExists(Me, "ARTIKELNUMMER") Then
                Return GetPropertyValue("ARTIKELNUMMER")
            Else
                Return "-"
            End If
        Else
            Return SmartPartCode
        End If
    End Get
End Property

Public Function GetSmartPartCode() As String
    If PropertyExists(Me, "SMT_THICKNESS") Then
        Return ICenter.Part.GetSmtMaterialProperty(
            MATERIAL_PARAM("CONDITION"),
            Math.Round(GetPropertyValue("SMT_THICKNESS"), 2),
            "PartCode")
    Else
        Return Nothing
    End If
End Function
```

**3-level fallback**: SmartPartCode (SMT material lookup) → `ARTIKELNUMMER` property → `"-"`. The Dutch `ARTIKELNUMMER` literal is the JAZO part-master article-number field.

If `SMT_THICKNESS` exists and `MATERIAL_PARAM("CONDITION")` resolves, look up the matching SMT row's PartCode from `T_SmtMaterials` (via `ICenter.Part.GetSmtMaterialProperty`). Q-365 — document `CONDITION` material-property semantics.

## `REL_MODEL_NAME` — the GN→PC translation

```vb
If Name.StartsWith("GN") AndAlso Name.Length > 2 Then
    Return "PC" & Name.Substring(2)
Else
    Return Name
End If
```

**`GN`** = Generic (the abstract template). **`PC`** = PCFNet (the concrete instance). Naming convention: `GN10195` is the generic, `PC10195` is the same product as a PCFNet-loaded class.

Author's commented-out alternative version used `RuntimeModelname` instead of `Name` — "**Dit resulteert in een stack overflow exception**" (Dutch: results in stack overflow). The author hit a recursion bug in the runtime-name path and worked around it by using the static Name. Q-358 — investigate the recursion.

## Reflection-driven parameter assignment

```vb
Public Sub SetParam(Name, Value)
    Try
        Dim sender As Object = Me
        If PropertyExists(sender, Name) Then
            CallByName(sender, Name, CallType.Set, Value)
        End If
    Catch ex As Exception
        ''Trying to set value to a non-existing parameter
    End Try
End Sub
```

Silent assignment. A typo in parameter name → silently ignored. **No logging**. Q-366.

## Debug dump

```vb
Private Sub DumpParameters(StepName As String)
    If DEBUGMODE Then
        Dim Folder As String = "c:\temp\PCFNet dump"
        If Not Directory.Exists(Folder) Then Directory.CreateDirectory(Folder)
        Dim Filepath = Folder & "\field_" & Name & " " & StepName & " " &
                       DateTime.Now.ToString("mm-ss-fff") & ".txt"
        ...
```

**Hardcoded `c:\temp\PCFNet dump\`** path. Matches the Q-268 (`ProductDb`) and Q-359 patterns. `DEBUGMODE` is False by default; flipping requires recompile.

## Other significant methods (overview)

| Method | Lines | Role |
|--------|-----:|------|
| `CalculateSolidProperties()` | 473 | computes mass + square-measure from geometry |
| `GetMyCalculation()` | 501 | returns the configured `PcfNetDataSet` |
| `GetCalculation(Qty, ...)` | 622 | composed calculation for a quoted quantity |
| `WriteBomToXml(StepName)` | 681 | debug XML dump (gated by `EnableLogging`) |
| `AddBOM(ByRef oXmlNode, Qty)` | 699 | recursive XML serializer of the BOM tree |
| `GetImage(...)` | 744 | renders product preview image (System.Drawing) |
| `ApplySealant(ds, dtSealant)` | 961 | adds sealant parts to BOM |
| `AddCalculationEntireTree(...)` | 1191 | full-tree calculation aggregator |
| `LoadDefaultParameterValues(Filepath)` | 1557 | restores parameter values from file |
| `GetCreoInputParameters()` | 1633 | builds the Creo `.par` input |
| `ExportCreoInputFile(Filepath)` | 1645 | writes the Creo input file |
| `ApplyConfigParameters(Dict)` | 1664 | applies per-design-config parameters in bulk |
| `GetFasteningType()` | 1721 | returns the FasteningType enum value |
| `CreateSvgGraphicDataTable()` | 1742 | empty SvgGraphic DataTable factory |
| `GetPatternQtyFromPath(...)` | 1472 | resolves a Creo pattern count from a component path |
| `ListFeaturesByType(FeatType)` | 1516 | filters Features by type |

## Business rules surfaced

- [[../business-rules/pcfnet-ebtv-billed-by-mass|EBTV galvanizing is billed by MASS (kg), not surface area (m²)]] — special branch in `AddSurfaceTreatment`.
- **`~PHANTOM` SubPartCode** = JAZO marker for synthetic BOM rows representing chosen-system-only (carries `OrdCode = CalcInkopierenArtikelNietVerwijderen` and `OriginType = CalculatieInkopierenPhantom`).
- **`AJPKL7000` SubPartCode** = fastener-calculation trigger. Q-356.
- **`GN` → `PC` Modelname prefix swap** in `REL_MODEL_NAME` (Generic vs PCFNet).
- **`ARTIKELNUMMER`** = standard JAZO article-number property name (Dutch).
- **Fasteners always round up** (`Math.Ceiling`) — safety-conservative.
- **Surface-treatment cycle-time is area-scaled** (`MachCycleTime / CalcQty * SurfTreatSquareMeasure`).
- **Per-family code is a separate DLL on disk** loaded via in-memory `AppDomain.Load(bytes)` — no app restart needed for family changes.

## Notable findings

1. **Two parallel "BillOf" naming families**: `BillOfMaterial` (Dictionary in this class) vs `BOO` (PCFNet's BOO type) for operations.
2. **All commented-out code preserved** — `'CB 2019-04-25 added separate step`, `'CB 2019-05-24: undo move dd 2019-04-23`, etc. The class has a long author-comment audit trail (CB = author initials). Q-367 — these comments are valuable history; preserve when refactoring.
3. **GetDebugNames hard-codes specific Generic Modelnames** (line 293) — `GN10195-P1-D1-R11-R1`, `GN10213-P1-R1`, `GN10209`. Test data baked in. Q-368.
4. **`AppDomain.CurrentDomain` is used directly** (not the passed-in `MyAppDomain` parameter) at line 1330. The parameter is ignored. Q-369.
5. **`GP.LoadedFromFile = True`** is the only consumer of `LoadedFromFile`. Used downstream to distinguish "loaded from disk" vs "instantiated in-process".

## Open questions

- **Q-354 (new):** Document `~PHANTOM` SubPartCode pattern + `OrdCode = CalcInkopierenArtikelNietVerwijderen` semantics.
- **Q-355 (new):** EBTV galvanize is billed by mass, all other surface treatments by area. Document the per-method billing-unit table. `#safety-relevant`
- **Q-356 (new):** `AJPKL7000` SubPartCode magic literal — what kind of fastener config does it represent?
- **Q-357 (new):** Document `CPart.VBCodeFolder` path + family-DLL deployment pipeline. How is a new family rolled out?
- **Q-358 (new):** `REL_MODEL_NAME` stack-overflow alternative — root cause of the recursion?
- **Q-359 (new):** Hardcoded `c:\temp\PCFNet dump\` debug path (matches Q-268).
- **Q-360 (new):** `~PHANTOM` SubPartCode pattern — document the canonical phantom-row creation pattern.
- **Q-361 (new):** `UpdateBOM()` behaviour deep-read needed.
- **Q-362 (new):** `AppSettings.GetStringSetting("ExtOperationPartCodeEBTV")` — document.
- **Q-363 (new):** `FasteningType.LageKant` — document the fastening-type enum values.
- **Q-364 (new):** Standard `MATERIAL_PARAM` property names (density, CONDITION, etc.).
- **Q-365 (new):** `SMT_THICKNESS` + `MATERIAL_PARAM("CONDITION")` semantics for SmartPartCode lookup.
- **Q-366 (new):** `SetParam` silently swallows non-existing-property assignments. Logging? Throw?
- **Q-367 (new):** Long author-comment audit trail (CB = ...). Preserve when refactoring.
- **Q-368 (new):** `GetDebugNames` hardcodes test-data Modelnames. Move to test fixtures.
- **Q-369 (new):** `CreateByFilepath` ignores the passed-in `MyAppDomain` and uses `AppDomain.CurrentDomain` directly. Bug or intentional?

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/icenterlib-pcfnet|PCFNet MOC]] — parent.
- [[icenterlib-productdb-product|`ProductDb.Product`]] — surface-treatment defaults source (via `GetDefaultSurfaceTreatmentDefinition`).
- [[isah-production-hierarchy|`ISAH.BillOfMat.CreateSurfTreatment`]] — peer that builds the surface-treatment SP-result.
- [[icenter-coating-executor-enum|`Coating.Executor`]] — the routing decision that ultimately drives which surface-treatment goes where.
- [[icenterlib-icenter-leaves|`SurfaceTreatmentDefinition`]] — the DTO carrying SystemPartCode + ColorPartCode + DTSealant.
- **Per-family inheritors**: `PCFNet.StlDoor001` (37 KB), `AluDoor001` (18 KB), `AluBasicWallLouver001` (17 KB).

## Coverage

`PCFNet\GenericPart.vb` → `done` (deep-read of public surface + key methods AddSurfaceTreatment, ApplyFasteners, CreateByFilepath, AddBOMMember, Regenerate; deferred internals: UpdateBOM, CalculateSolidProperties, GetCalculation, AddCalculationEntireTree, GetImage, ApplyConfigParameters).
