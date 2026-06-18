---
type: module
title: "ICenterLib/Production ProfileCutItem family + CEChecklist (the persistence + regulatory pair)"
status: done
module: "ICenterLib/Production"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\Production\\ProductionProfileCutItem.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\Production\\ProductionProfileCutItemHandler.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\Production\\ProductionProfileCutItemsHandler.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\Production\\CEChecklist.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# ICenterLib/Production — ProfileCutItem family + CEChecklist

> Two related subsystems documented in one note. **ProfileCutItem family** = ICenterLib-side persistence layer for the Elumatec NC cut data. **CEChecklist** = CE (Conformité Européenne) regulatory checklist persisted in JIBA's XmlData store. Both `#safety-relevant`.
>
> **Note**: The iCENTER-side caller `ProductionProfileCutItemsHandler.vb` is documented separately at [[production-profile-cut-items]] (different file in `iCENTER\Production\`).

## Part 1: ICenterLib `ProductionProfileCutItem` family

### Three-class layered design

```
ProductionProfileCutItem   ' single cut DTO (length + 4 angles + identifiers)
    ↓ wrapped by
ProductionProfileCutItemHandler   ' single-row INSERT
    ↓ orchestrated by
ProductionProfileCutItemsHandler  ' bulk Save + Create factories + Clear-by-machine
```

The iCENTER-side `ProductionProfileCutItemsHandler` (at `iCENTER\Production\`) is the **caller** that prepares the DataTable and invokes `ICenterLib.Production.ProductionProfileCutItemsHandler.{Clear, Create, Save}`. This note documents the **ICenterLib side**.

### `ProductionProfileCutItem.vb` — the cut DTO

Inherits `BaseProductionItem`. Adds 6 Elumatec-specific ReadOnly properties:

```vb
Public ReadOnly Property BIdentNo  As String     ' bar-identification number from EluCad
Public ReadOnly Property CLength   As Double     ' cut length (mm)
Public ReadOnly Property CAngleLH  As Double     ' left horizontal mitre angle
Public ReadOnly Property CAngleLV  As Double     ' left vertical mitre angle
Public ReadOnly Property CAngleRV  As Double     ' right vertical mitre angle
Public ReadOnly Property CAngleRH  As Double     ' right horizontal mitre angle
```

The four CAngle* fields match the **`Cut.CAngle*` fields** in [[elumatec-elucadfile|EluCadFile parser]] — read from the `.ecw` file's `:CUT` block, defining per-corner mitre angles.

### `ProductionProfileCutItemHandler.vb` — single-row INSERT

14-column parameterised INSERT into `T_ProductionProfileCutItem`. **No `IF NOT EXISTS` guard** — calling Save twice creates duplicates. Coordinator's Clear+Create+Save protocol prevents this; a direct caller of Save can corrupt the table. Q-374.

Silent exception swallow.

### `ProductionProfileCutItemsHandler.vb` — bulk orchestrator

```vb
Public Sub Save()
    For Each PPCI In _ProductionProfileCutItems
        New ProductionProfileCutItemHandler(PPCI).Save()   ' N round-trips
    Next

Public Shared Function Create(DT, IPPartId, MachineId)
    ' Inject IPPartId + MachineId + QtyDone=0 into every row; delegate.

Public Shared Function Create(DT)
    For Each DR In DT.Rows
        Dim Part As New ISAH.Part(DR.Item("Art.nr").ToString())
        Dim PartDescription = Part.GetDescription()   ' default "-"
        ' build ProductionProfileCutItem from DR columns
    Next

Public Shared Sub Clear(MachineId)
    DELETE FROM T_ProductionProfileCutItem WHERE MachineId=@MachineId
```

**Typical flow**: `Clear(MachineId)` → `Create(DT, IPPartId, MachineId)` → `Save()`. N separate INSERTs (Q-375 — bulk-insert opportunity).

**Magic Dutch column literal `"Art.nr"`** required by the DataTable contract — Q-376.

**`QtyDone = 0` always** on create with TODO comment "berekenen wat het echt moet zijn" (calculate what it really should be). Progress increment happens elsewhere. Q-377.

## Part 2: `CEChecklist.vb` — CE-marking regulatory checklist

**CE = Conformité Européenne** — EU regulatory mark. JAZO's products under EU directives must be CE-marked; the checklist captures regulatory data backing the **Declaration of Performance**. `#safety-relevant` — failed CE compliance can ground product shipments.

### Public surface

```vb
Public Class CEChecklist
    Public ReadOnly DossierDetail As ISAH.DossierDetail
    Public ReadOnly DesignCode As String
    Public ReadOnly EmpId As String
    Private Property XmlData As JIBA.XmlData = Nothing
    Private Property _ChecklistChecklistValidated As Boolean = False

    Public Sub New(DossierDetail, DesignCode, EmpId)
    Public Sub ShowGui()                                ' Web checklist form, then validates
    Public Sub CreateXmlData(DTParameter As DataTable)  ' Persists XML to JIBA T_XmlData
    Public ReadOnly Property IsValidated As Boolean
    Public ReadOnly Property SubReference As String     ' = DetailCode & "-" & DetailSubCode
    Public Function GetXmlDataId() As Integer
    Public Shared Function GetCheckLists(DossierDetail) As DataTable
End Class
```

### Workflow

1. **`CreateXmlData(DTParameter)`** — builds base XML from `AppSettings.GetStringSetting("CEChecklistControlDefinitionPath")` template, injects `DesignCode` into all `<Control ControlName='DESIGNCODE'>` defaults, evaluates `<Attribute UseFormula='1'>` filters via `DataView.RowFilter` against DTParameter, persists via `JIBA.XmlData.CreateNew(9, XmlDoc, 1, DossierCode, SubReference, 1, Now, EmpId)`. **Magic literals: `9` = XmlDataGroupId for CE-Checklist, `1` = SecurityGroupId**. Q-379.
2. **`ShowGui()`** — opens `UserControls.FrmWebView` pointed at `AppSettings.GetStringSetting("CEChecklistURL") & "?XmlDataId=" & XmlDataId`. After user closes the form, `XmlData.Reload()` pulls back the modified XML, then `ValidateChecklist()` runs.
3. **`ValidateChecklist()`** — walks `Member[@ControlTypeId='3']/Attributes/Attribute[@Name='Required'][@Value='True']`. For each: skip if `Enabled = False`, fail if `Value = False`. **Exit on first failure** (Q-382). **`ControlTypeId = 3` = checkbox presumably** (Q-380).

### Storage

- CE-Checklist XML lives in **JIBA's `T_XmlData`** (not ISAH, not iCenter).
- Key: `(ReferenceId=DossierCode, SubReferenceId=DetailCode-DetailSubCode)`.
- `GetCheckLists(DossierDetail)` returns all checklists ever created for that dossier-detail line, joined with `T_Employee` for the creator's full name.

### Path dependency

- `CEChecklistControlDefinitionPath` — file-system path to template XML. Q-381.
- `CEChecklistURL` — web checklist-editor URL (likely a JIBA portal page).
- Commented-out fallback `Z:\iCenter\Resources\DocumentTemplates\CE-ChecklistControlDefinition.xml` (line 40) preserves the previous local-path style.

## Business rules surfaced

- **Per-machine cut-item lifecycle**: Clear → Create → Save. Each machine has its own scratch-pad.
- **Cut DTO carries 4 angles** (LH/LV/RV/RH) per cut — JAZO/Elumatec convention.
- **`"Art.nr"` Dutch column literal** is the implicit DataTable-contract.
- **`QtyDone = 0` always on create** — increment happens elsewhere.
- **CE-Checklist requires ControlTypeId=3 Required+Enabled+Value=True for every checkbox to pass**.
- **CE-Checklist XmlDataGroupId = 9** in JIBA's catalog.
- **CE-Checklist storage in JIBA** (not ISAH/iCenter).

## Open questions

(Q-374..Q-383 detailed in needs-review/_index.)

## Related

- [[../mocs/icenterlib-production|Production folder MOC]] — parent.
- [[production-profile-cut-items|iCENTER-side ProductionProfileCutItemsHandler]] — the caller in `iCENTER\Production\` that prepares the DataTable.
- [[../mocs/elumatec-ncpipeline|Elumatec NC pipeline]] — upstream producer of cut data.
- [[elumatec-elucadfile|`EluCadFile` parser]] — defines the CAngle* convention.
- [[icenterlib-icenter-leaves|`DeclarationOfPerformance`]] — CE companion document.
- [[icenterlib-jiba-employee-asset|`JIBA.Employee`]] — joined for creator name in `GetCheckLists`.

## Coverage

- `ICenterLib\Production\ProductionProfileCutItem.vb` → `done` (deep-read)
- `ICenterLib\Production\ProductionProfileCutItemHandler.vb` → `done` (deep-read)
- `ICenterLib\Production\ProductionProfileCutItemsHandler.vb` → `done` (deep-read)
- `ICenterLib\Production\CEChecklist.vb` → `done` (deep-read)
