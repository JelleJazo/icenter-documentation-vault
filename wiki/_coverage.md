---
type: meta
title: "File Coverage Tracker"
status: active
created: 2026-06-18
updated: 2026-06-18
tags: [meta, coverage]
---

# File Coverage Tracker

**Source of truth for "done".** A module is *done* only when every file under it has a non-`todo` status here.

## Status values

| Status | Meaning |
|--------|---------|
| `todo` | Not yet documented |
| `done` | Documented in a [[modules/_index\|module]] note with frontmatter + source-paths |
| `config` | Configuration file: no behavior; pointer in module note is enough |
| `generated` | Generated code: do not document; record generator |
| `dead-code` | Verified unused: flagged with #dead-code, SME confirmation pending |
| `needs-review` | Documented but intent unclear: open question in [[needs-review/_index]] |

## How to populate

The codebase at `C:\DevOps\iCenter\iCenter\iCENTER` is **not git-tracked**. Use PowerShell to enumerate:

```powershell
Get-ChildItem 'C:\DevOps\iCenter\iCenter\iCENTER' -Recurse -File `
  | Where-Object { $_.FullName -notmatch '\\(bin|obj|\.vs|packages|My Project|Web References)\\' } `
  | Select-Object @{n='Rel';e={$_.FullName.Substring('C:\DevOps\iCenter\iCenter\iCENTER\'.Length)}}, Extension `
  | Sort-Object Rel
```

Then convert each line into a row in the appropriate sub-folder table below. **Do not** populate from memory.

---

## Inventory

Scope was widened on 2026-06-18 to three projects per [[../CLAUDE]]:

| Project | Root | Files | Notes |
|---------|------|------:|-------|
| **iCENTER** | `C:\DevOps\iCenter\iCenter\iCENTER\` | 1237 | WinForms shell + integrations |
| **TruTopsLib** | `C:\DevOps\iCenter\iCenter\TruTopsLib\` | 65 | Trumpf TruTops file parsing; mixed `.vb`/`.cs` |
| **ICenterLib** | `C:\Users\jelle-r\source\repos\JIBA\iCenter And Tools\ICenterLib\ICenterLib\` | 723 | Shared library: ISAH/JIBA/CAD/SmtProduction/PCFNet plumbing |
| **TOTAL** | | **2025** | |

Tables below are grouped per project. Phase 1 status heuristic: `generated` for `*.Designer.vb|cs` + `*.resx`; `config` for assets, project metadata, signing keys, solution files; `todo` for everything else.

---

# Project: iCENTER

### (root)

**Total: 11** &nbsp; | &nbsp; .vb: 3 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `app.config` | done | covered in [[architecture/build-and-deploy]] + [[architecture/external-surface]] | 2026-06-18 |
| `ApplicationEvents.vb` | done | empty `My.MyApplication` partial; covered in [[architecture/entry-points]] | 2026-06-18 |
| `FrmMain.Designer.vb` | generated | VS Forms designer partial | â€” |
| `FrmMain.resx` | generated | resource bundle (designer-managed) | â€” |
| `FrmMain.vb` | needs-review | 15 216-line god-form; Phase 3 will split into multiple notes. Briefly characterized in [[architecture/entry-points]] + [[architecture/global-state]]. | â€” |
| `iCenter.vbproj` | done | covered in [[architecture/build-and-deploy]] + [[architecture/project-references]] | 2026-06-18 |
| `iCenter.vbproj.user` | config | per-user VS metadata | â€” |
| `iCenter.vbproj.vspscc` | config | source-control plugin metadata | â€” |
| `JAZO Comodo Code Signing Certificate.pfx` | config | signing key (Authenticode; expired 2023-07 per inline comment in vbproj) | â€” |
| `JAZO Zevenaar bv.snk` | config | strong-name signing key | â€” |
| `packages.config` | done | covered in [[architecture/build-and-deploy]] | 2026-06-18 |

### Batchserver

**Total: 4** &nbsp; | &nbsp; .vb: 3 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Batchserver\BatchserverToolkit.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Batchserver\FrmBatchServer.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Batchserver\FrmBatchServer.resx` | generated | resource bundle (designer-managed) | â€” |
| `Batchserver\FrmBatchServer.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |

### CAD

**Total: 3** &nbsp; | &nbsp; .vb: 2 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `CAD\PLM\FrmPDMLinkOrdTypeContext.Designer.vb` | generated | VS Forms designer partial | â€” |
| `CAD\PLM\FrmPDMLinkOrdTypeContext.resx` | generated | resource bundle (designer-managed) | â€” |
| `CAD\PLM\FrmPDMLinkOrdTypeContext.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |

### CadBatchserver

**Total: 29** &nbsp; | &nbsp; .vb: 27 | .resx: 2 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `CadBatchserver\BatchServerWatch.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CadBatchserver\CadBatchserverTools.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CadBatchserver\FormRunJob.Designer.vb` | generated | VS Forms designer partial | â€” |
| `CadBatchserver\FormRunJob.resx` | generated | resource bundle (designer-managed) | â€” |
| `CadBatchserver\FormRunJob.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CadBatchserver\FrmCadBatchServer.Designer.vb` | generated | VS Forms designer partial | â€” |
| `CadBatchserver\FrmCadBatchServer.resx` | generated | resource bundle (designer-managed) | â€” |
| `CadBatchserver\FrmCadBatchServer.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CadBatchserver\Job.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CadBatchserver\JobArchive.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CadBatchserver\JobAutoManufacturing.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CadBatchserver\JobCreateProdOrd.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CadBatchserver\JobCreatePurOrdDocs.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CadBatchserver\JobEngOrdFinNotification.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CadBatchserver\JobGenerateAndReleaseModel.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CadBatchserver\JobGenericModelBackup.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CadBatchserver\JobGeo2Dxf.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CadBatchserver\JobModelGeneratorCreo.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CadBatchserver\JobPartDispatch.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CadBatchserver\JobProductionRegistration.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CadBatchserver\JobPublishCreo.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CadBatchserver\JobRebootMonitor.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CadBatchserver\JobSendEmail.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CadBatchserver\JobSmtOperSync.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CadBatchserver\JobSmtPartDispatch.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CadBatchserver\Modelgenerator\ModelgeneratorTask.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CadBatchserver\Publisher\Trailfile.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CadBatchserver\PublishWatchDirProcessor.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CadBatchserver\ServerReboot.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |

### CAM

**Total: 4** &nbsp; | &nbsp; .vb: 4 | .resx: 0 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `CAM\ManufMachine.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CAM\ManufPart.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CAM\ManufPartItem.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `CAM\ManufPartMember.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |

### Classes

**Total: 111** &nbsp; | &nbsp; .vb: 101 | .resx: 10 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Classes\ApplicationLog.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\AutoDocAttach.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\AxControl.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\BriefcaseCollection.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\BriefcaseItem.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\ClsEmailEBTV.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\ClsPdfCommentLines.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\ClsProdObjects.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\ClsRecOrdNr.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\ClsXMLfile.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Coating\Coating.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Coating\CoatingPickLabel_v3.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Coating\ControlCoating.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Classes\Coating\ControlCoating.resx` | generated | resource bundle (designer-managed) | â€” |
| `Classes\Coating\ControlCoating.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Coating\ControlCoatingGrouped.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Classes\Coating\ControlCoatingGrouped.resx` | generated | resource bundle (designer-managed) | â€” |
| `Classes\Coating\ControlCoatingGrouped.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Coating\CtrlCoatingPickLocation.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Classes\Coating\CtrlCoatingPickLocation.resx` | generated | resource bundle (designer-managed) | â€” |
| `Classes\Coating\CtrlCoatingPickLocation.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Coating\DataGridViewProgressColumn.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Coating\DebugLog.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Coating\EbtvSticker.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Coating\FrmAddLayerThickness.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Classes\Coating\FrmAddLayerThickness.resx` | generated | resource bundle (designer-managed) | â€” |
| `Classes\Coating\FrmAddLayerThickness.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Coating\FrmCoatingLayerThickness.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Classes\Coating\FrmCoatingLayerThickness.resx` | generated | resource bundle (designer-managed) | â€” |
| `Classes\Coating\FrmCoatingLayerThickness.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Coating\FrmCoatingPick.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Classes\Coating\FrmCoatingPick.resx` | generated | resource bundle (designer-managed) | â€” |
| `Classes\Coating\FrmCoatingPick.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Coating\FrmGetCoatingMaterials.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Classes\Coating\FrmGetCoatingMaterials.resx` | generated | resource bundle (designer-managed) | â€” |
| `Classes\Coating\FrmGetCoatingMaterials.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Coating\FrmGetCoatingNextJob.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Classes\Coating\FrmGetCoatingNextJob.resx` | generated | resource bundle (designer-managed) | â€” |
| `Classes\Coating\FrmGetCoatingNextJob.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Coating\frmKardexJobIncomplete.designer.vb` | generated | VS Forms designer partial | â€” |
| `Classes\Coating\frmKardexJobIncomplete.resx` | generated | resource bundle (designer-managed) | â€” |
| `Classes\Coating\frmKardexJobIncomplete.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Coating\frmScanNext.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Classes\Coating\frmScanNext.resx` | generated | resource bundle (designer-managed) | â€” |
| `Classes\Coating\frmScanNext.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Coating\IControlCoating.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Connectivity\CalcExcelExport.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Connectivity\ClsActiveDirectory.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Connectivity\ClsICenter.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Connectivity\ClsISAH.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Connectivity\ClsJIBA.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Connectivity\DeviceInfo.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Connectivity\LabelWriter.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Connectivity\NetworkPrinter.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\DocElementConverter.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\DossierDetail.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\DossierDocFolder.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\EngineeringOrders.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\ExtOperPart.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\ICenterDoc.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\ICenterObject.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\ICenterPartBasicXML.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\InvtOrd.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\IsahDoc.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\MachGrp.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\MachineFilter.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\ModelTree.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\OrdRefNrStatusUpdate.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\PartCalculation.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\PartOptimisationExcelExport.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\PreSelectMachGrpCodes\PreSelectMachGrpCode.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\PreSelectMachGrpCodes\PreSelectMachGrpCodeOrdType.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\PreSelectMachGrpCodes\PreSelectMachGrpCodeOrdTypeHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\PreSelectMachGrpCodes\PreSelectMachGrpCodes.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\PreSelectMachGrpCodes\PreSelectMachGrpCodesHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\ProcessWatch.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Production\ClsIPbatch.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Production\ClsIPorder.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Production\ClsIPpacket.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Production\ClsIPpart.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Production\Icenter2IsahJob.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Production\LeanJob.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Production\LeanWorkTime.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Production\MachGrpPerformance.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Production\PartProgress.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Production\ProdPlanView.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Production\ProdStatus.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Production\ProductionRegistration.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Production\WorkView.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\PurOrd.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\RecurrenceCheck.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\StandardPartUpdater.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\StickersAndLabels\AltecLabel.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\StickersAndLabels\DymoLabelTest.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\StickersAndLabels\FG_ProductLabel.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\StickersAndLabels\InvtPartSticker.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\StickersAndLabels\KanbanBinLabel.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\StickersAndLabels\LeanBatchSticker.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\StickersAndLabels\LeanSticker_v1.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\StickersAndLabels\LeanStickerCopy.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\StickersAndLabels\OrdRefLabel.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\StickersAndLabels\PartIdent.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\StickersAndLabels\PartIdentProfMill.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Toolbox\ApplicationHelper.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Toolbox\ClipboardHelper.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Toolbox\HelpHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Toolbox\JzWindow.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Toolbox\RtfBuilder.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Toolbox\SessionHelper.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Toolbox\WindowHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Classes\Toolbox\Zip.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |

### Comparers

**Total: 4** &nbsp; | &nbsp; .vb: 4 | .resx: 0 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Comparers\BewerkingsGroepComparer.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Comparers\DefaultNodeSorter.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Comparers\DPrintDocumentComparer.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Comparers\JAZOTreeViewNodeSorter.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |

### Controls

**Total: 114** &nbsp; | &nbsp; .vb: 78 | .resx: 36 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Controls\BomControl.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\BomControl.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\BomControl.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\ButtonExtended.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\ButtonExtended.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CheckBoxExtended.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CompBriefcaseButton.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CompBriefcaseButton.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CompBriefcaseButton.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CreoViewControl.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CreoViewControl.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CreoViewControl.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CtrlCeChecklistViewer.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlCeChecklistViewer.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlCeChecklistViewer.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CtrlDocumentReplace.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlDocumentReplace.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlDocumentReplace.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CtrlDosDesignCodes.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlDosDesignCodes.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlDosDesignCodes.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CtrlDossierDetailProdDosCompare.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlDossierDetailProdDosCompare.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlDossierDetailProdDosCompare.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CtrlDossierDetailProperties.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlDossierDetailProperties.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlDossierDetailProperties.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CtrlDossierDetailText.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlDossierDetailText.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlDossierDetailText.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CtrlEditOperations.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlEditOperations.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlEditOperations.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CtrlExplorer.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlExplorer.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlExplorer.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CtrlFgProductLabel.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlFgProductLabel.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlFgProductLabel.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CtrlIpInfo.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlIpInfo.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlIpInfo.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CtrlIsahContacts.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlIsahContacts.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlIsahContacts.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CtrlModelgenerator.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlModelgenerator.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlModelgenerator.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CtrlModelgenerators.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlModelgenerators.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlModelgenerators.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CtrlOfficeClockFavorites.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlOfficeClockFavorites.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlOfficeClockFavorites.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CtrlOfficeClockStatus.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlOfficeClockStatus.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlOfficeClockStatus.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CtrlPhoneNr.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlPhoneNr.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlPhoneNr.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CtrlProdChecklistViewer.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlProdChecklistViewer.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlProdChecklistViewer.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CtrlProdRoutingDetail.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlProdRoutingDetail.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlProdRoutingDetail.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CtrlProductConfiguration.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlProductConfiguration.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlProductConfiguration.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CtrlSalesFavorites.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlSalesFavorites.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlSalesFavorites.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CtrlTreeNodeDetails.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlTreeNodeDetails.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlTreeNodeDetails.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CtrlTreeNodeProperties.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlTreeNodeProperties.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlTreeNodeProperties.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\CtrlWebClockAssistant.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlWebClockAssistant.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlWebClockAssistant.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\DxfView\DxfViewControl.designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\DxfView\DxfViewControl.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\DxfView\DxfViewControl.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\DxfView\DxfViewDisplayControl.designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\DxfView\DxfViewDisplayControl.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\DxfView\DxfViewDisplayControl.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\DxfView\PolygonWireframeGraphicsFactory.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\ExtendedDateTimePicker.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\ExtendedDateTimePicker.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\Isah\CtrlDosDetailExtra.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\Isah\CtrlDosDetailExtra.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\Isah\CtrlDosDetailExtra.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\Isah\DossierDetailExtraAuthorizationHelper.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\Isah\FrmDosDetailExtra.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\Isah\FrmDosDetailExtra.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\Isah\FrmDosDetailExtra.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\Isah\ICtrlDosDetailExtra.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\ModelOpers\ControlModelOper.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\ModelOpers\ControlModelOper.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\ModelOpers\ControlModelOper.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\ModelOpers\ControlModelOpers.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\ModelOpers\ControlModelOpers.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\ModelOpers\ControlModelOpers.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\MSVistaPBar.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\RichTextBoxEditor.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\RichTextBoxEditor.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\RichTextBoxEditor.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\TabPageExtended.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\TabPageExtended.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\TabPageExtended.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Controls\UCKanbanPart.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\UCKanbanPart.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\UCKanbanPart.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |

### DataMigration

**Total: 50** &nbsp; | &nbsp; .vb: 50 | .resx: 0 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `DataMigration\BasicDataHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\BasicMigrationHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\CacheHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\ContextMenuHelper.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\DataBaseConnection.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\DataBaseConnectionHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\DataHandlerToolbox.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\DossierItemMigrationHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\DossierMigrationHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Entities\AuditableEntity.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Entities\BillOfMaterialItem.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Entities\BillOfOperation.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Entities\BillOfOperationItem.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Entities\Document.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Entities\Dossier.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Entities\DossierItem.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Entities\IEntity.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Entities\Material.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Entities\Operation.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Entities\Part.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Entities\ProductionDossier.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Entities\Project.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Entities\User.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Features\BillOfMaterialItemFactory.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Features\BillOfMaterialItemHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Features\BillOfOperationItemFactory.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Features\BillOfOperationItemHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Features\DocumentFactory.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Features\DocumentHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Features\DossierFactory.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Features\DossierHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Features\DossierItemFactory.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Features\DossierItemHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Features\MaterialFactory.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Features\MaterialHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Features\OperationFactory.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Features\OperationHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Features\PartFactory.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Features\PartHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Features\ProductionDossierFactory.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Features\ProductionDossierHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Features\ProjectFactory.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Features\ProjectHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Features\UserFactory.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\Features\UserHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\IMigrationHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\MockDataHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\PartMigrationHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\ProductionDossierMigrationHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DataMigration\UserMigrationHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |

### DesignComments

**Total: 10** &nbsp; | &nbsp; .vb: 7 | .resx: 3 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `DesignComments\ClsDesignComments.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DesignComments\dgCommentLines.designer.vb` | generated | VS Forms designer partial | â€” |
| `DesignComments\dgCommentLines.resx` | generated | resource bundle (designer-managed) | â€” |
| `DesignComments\dgCommentLines.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DesignComments\frmCommentLine.designer.vb` | generated | VS Forms designer partial | â€” |
| `DesignComments\frmCommentLine.resx` | generated | resource bundle (designer-managed) | â€” |
| `DesignComments\frmCommentLine.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `DesignComments\FrmDesigncomments.designer.vb` | generated | VS Forms designer partial | â€” |
| `DesignComments\FrmDesigncomments.resx` | generated | resource bundle (designer-managed) | â€” |
| `DesignComments\FrmDesigncomments.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |

### Elumatec

**Total: 157** &nbsp; | &nbsp; .vb: 140 | .resx: 17 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Elumatec\AppVersion.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\AufSerializer\EluXmlJob.vb` | done | [[modules/elumatec-nc-program-family]] | 2026-06-18 |
| `Elumatec\AufSerializer\EluXmlJobItem.vb` | done | [[modules/elumatec-nc-program-family]] | 2026-06-18 |
| `Elumatec\AufSerializer\EluXmlJobSubItem.vb` | done | [[modules/elumatec-nc-program-family]] | 2026-06-18 |
| `Elumatec\AufSerializer\EluXmlProgram.vb` | done | [[modules/elumatec-nc-program-family]] | 2026-06-18 |
| `Elumatec\AufSerializer\EluXmlProgramDetail.vb` | done | [[modules/elumatec-nc-program-family]] | 2026-06-18 |
| `Elumatec\AufSerializer\Kontur.vb` | done | [[modules/elumatec-nc-program-family]] | 2026-06-18 |
| `Elumatec\AufSerializer\NcProgram.vb` | done | [[modules/elumatec-nc-program-family]] | 2026-06-18 |
| `Elumatec\AufSerializer\NcProgramAuf.vb` | done | [[modules/elumatec-nc-program-family]] | 2026-06-18 |
| `Elumatec\AufSerializer\NcProgramEluXml.vb` | done | [[modules/elumatec-nc-program-family]] | 2026-06-18 |
| `Elumatec\AufSerializer\Programm.vb` | done | [[modules/elumatec-nc-program-family]] | 2026-06-18 |
| `Elumatec\AufSerializer\ZeileAuftrag.vb` | done | [[modules/elumatec-nc-program-family]] | 2026-06-18 |
| `Elumatec\AufSerializer\ZeileKontur.vb` | done | [[modules/elumatec-nc-program-family]] | 2026-06-18 |
| `Elumatec\AufSerializer\ZeileProgramm.vb` | done | [[modules/elumatec-nc-program-family]] | 2026-06-18 |
| `Elumatec\AufSerializer\ZeileTTab.vb` | done | [[modules/elumatec-nc-program-family]] | 2026-06-18 |
| `Elumatec\AutoProfMillProgApproval.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\ClsComWatcher.vb` | done | [[modules/elumatec-com-watcher]] | 2026-06-18 |
| `Elumatec\ClsDgxShoppingList.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\ClsDgxStickerPrinter.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\ClsEluLanguage.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\ClsSawList.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\ControlProfSaw.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\ControlProfSaw.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\ControlProfSaw.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\CRs232.vb` | done | third-party (Corrado Cavalli Â©2003); [[modules/elumatec-com-watcher]] | 2026-06-18 |
| `Elumatec\CtrlEluOpenGLViewer.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\CtrlEluOpenGLViewer.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\CtrlEluOpenGLViewer.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\CtrlProfMillCam.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\CtrlProfMillCam.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\CtrlProfMillCam.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\CtrlProfMillElu.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\CtrlProfMillElu.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\CtrlProfMillElu.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\CutFactory.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\CycleTime.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Database\Fixture.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Database\FixtureCollection.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Database\Offset.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Database\OffsetCollection.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Database\OffsetFile.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Database\Offsets.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Database\Profile.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Database\ProfileExportHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Database\ProfileMachineSetting.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Database\ToolDbSimplified.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\DXF\EluDxf.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\DXF\EluDxfEntity.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\DXF\EluDxfPolyline.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\DXF\EluDxfVertex.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\EluCadApp.vb` | needs-review | overview only in [[modules/elumatec-cad-app]]; per-cluster sub-notes pending | â€” |
| `Elumatec\EluCadSetting.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\FileFormat.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\FrmComWatcher.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\FrmComWatcher.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\FrmComWatcher.vb` | done | [[modules/elumatec-com-watcher]] | 2026-06-18 |
| `Elumatec\frmDgxStack.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\frmDgxStack.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\frmDgxStack.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\FrmEluMissingProfile.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\FrmEluMissingProfile.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\FrmEluMissingProfile.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\frmExportDgx.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\frmExportDgx.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\frmExportDgx.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\frmManualProfile.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\frmManualProfile.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\frmManualProfile.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\FrmNcxErrors.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\FrmNcxErrors.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\FrmNcxErrors.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\FrmProfileView.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\FrmProfileView.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\FrmProfileView.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\frmSawQtyDone.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\frmSawQtyDone.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\frmSawQtyDone.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\frmSawQtyDoneExt.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\frmSawQtyDoneExt.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\frmSawQtyDoneExt.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\LicenseHelper.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\LicenseManagementCenter.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Machine\Sbz140Alu.vb` | done | [[modules/elumatec-machine-base]] | 2026-06-18 |
| `Elumatec\Machine\Sbz140Rvs.vb` | needs-review | [[modules/elumatec-machine-base]] (values not transcribed; Q-027) | â€” |
| `Elumatec\Machine\Sbz140Stl.vb` | needs-review | [[modules/elumatec-machine-base]] (values not transcribed; Q-027) | â€” |
| `Elumatec\Machine\Sbz141Alu.vb` | needs-review | [[modules/elumatec-machine-base]] (values not transcribed) | â€” |
| `Elumatec\Machine\Sbz14x.vb` | done | [[modules/elumatec-machine-base]] | 2026-06-18 |
| `Elumatec\MacroDatabase.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\NcStructure\Bar.vb` | done | [[modules/elumatec-ncstructure-hierarchy]] | 2026-06-18 |
| `Elumatec\NcStructure\Cut.vb` | done | [[modules/elumatec-ncstructure-hierarchy]] | 2026-06-18 |
| `Elumatec\NcStructure\EluCadFile.vb` | done | [[modules/elumatec-elucadfile]] | 2026-06-18 |
| `Elumatec\NcStructure\Job.vb` | done | [[modules/elumatec-ncstructure-hierarchy]] | 2026-06-18 |
| `Elumatec\NcStructure\Plane.vb` | done | [[modules/elumatec-ncstructure-hierarchy]] | 2026-06-18 |
| `Elumatec\NcStructure\PlaneCollection.vb` | done | [[modules/elumatec-ncstructure-hierarchy]] | 2026-06-18 |
| `Elumatec\NcVersionHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\NcwExportProfile.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\NcwViewer.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\NcxContainer.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\NumberLib.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Optimizer\CutOptimizer.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Optimizer\frmSawJobOptimizer.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\Optimizer\frmSawJobOptimizer.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\Optimizer\frmSawJobOptimizer.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\ProfileMatcher.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\ProfMillConverter.vb` | done | [[modules/elumatec-profmill-converter]] | 2026-06-18 |
| `Elumatec\ProfMillJob.vb` | done | [[modules/elumatec-profmill-job]] | 2026-06-18 |
| `Elumatec\ReferenceDxf.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\SawListReport.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\SectionCutOffBox.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\UCDgxWorksheet.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\UCDgxWorksheet.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\UCDgxWorksheet.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Workpiece.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Circle.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Deburr.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Drill.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\DxfFreeForm.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\FreeForm.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\FreeFormPoint.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\FrmTestDrwProfile.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\Works\FrmTestDrwProfile.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\Works\FrmTestDrwProfile.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Group.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Line.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Macro.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Rectangle.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Replacements\AluGeneral.vb` | needs-review | [[modules/elumatec-replacement-alu-general]] (overview only; per-branch notes pending) | â€” |
| `Elumatec\Works\Replacements\AluHinge.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Replacements\AluHUPO.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Replacements\AluSinglePnotch.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Replacements\AluSRkom.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Replacements\DoorPlankCalibration.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Replacements\DoorPlankCalibrationMessageCutOff.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Replacements\DoublePnotch.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Replacements\ExtraLength.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Replacements\ExtraLengthMacro.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Replacements\ExtraLengthMacroFactory.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Replacements\Flowdrill.vb` | done | [[modules/elumatec-replacement-flowdrill]] | 2026-06-18 |
| `Elumatec\Works\Replacements\IDoorPlankCalibrationMessage.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Replacements\LargeRectangle.vb` | done | [[modules/elumatec-replacement-large-rectangle]] | 2026-06-18 |
| `Elumatec\Works\Replacements\OpdekH.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Replacements\RDHS27Notch.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Replacements\RuntimeManipulation.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Replacements\RuntimeManipulationInstruction.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Replacements\RuntimeManipulations.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Replacements\StlDoublePnotch.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Replacements\StlFlowDrill.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Replacements\StlGeneral.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Replacements\StlHinge.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Replacements\UCDoorPlankCalibration.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\Works\Replacements\UCDoorPlankCalibration.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\Works\Replacements\UCDoorPlankCalibration.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Replacements\WorksReplacement.vb` | done | [[modules/elumatec-works-replacement-base]] | 2026-06-18 |
| `Elumatec\Works\Replacements\WorksTranslation.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Sawcut.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\SlottedHole.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Elumatec\Works\Work.vb` | done | [[modules/elumatec-work-base]] | 2026-06-18 |

### Engineering

**Total: 21** &nbsp; | &nbsp; .vb: 15 | .resx: 6 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Engineering\FrmDesignCodeTool.designer.vb` | generated | VS Forms designer partial | â€” |
| `Engineering\FrmDesignCodeTool.resx` | generated | resource bundle (designer-managed) | â€” |
| `Engineering\FrmDesignCodeTool.vb` | done | [[modules/engineering-overview]] | 2026-06-18 |
| `Engineering\FrmDrwCheck.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Engineering\FrmDrwCheck.resx` | generated | resource bundle (designer-managed) | â€” |
| `Engineering\FrmDrwCheck.vb` | done | [[modules/engineering-overview]] | 2026-06-18 |
| `Engineering\frmEngGeneratedProductOverview.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Engineering\frmEngGeneratedProductOverview.resx` | generated | resource bundle (designer-managed) | â€” |
| `Engineering\frmEngGeneratedProductOverview.vb` | done | [[modules/engineering-overview]] | 2026-06-18 |
| `Engineering\frmGenericStatus.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Engineering\frmGenericStatus.resx` | generated | resource bundle (designer-managed) | â€” |
| `Engineering\frmGenericStatus.vb` | done | [[modules/engineering-overview]] | 2026-06-18 |
| `Engineering\FrmOrdersAsBuilt.designer.vb` | generated | VS Forms designer partial | â€” |
| `Engineering\FrmOrdersAsBuilt.resx` | generated | resource bundle (designer-managed) | â€” |
| `Engineering\FrmOrdersAsBuilt.vb` | done | [[modules/engineering-overview]] | 2026-06-18 |
| `Engineering\frmSelectAnnotGenericStatus.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Engineering\frmSelectAnnotGenericStatus.resx` | generated | resource bundle (designer-managed) | â€” |
| `Engineering\frmSelectAnnotGenericStatus.vb` | needs-review | [[modules/engineering-overview]] (deferred â€” likely selector before frmGenericStatus) | â€” |
| `Engineering\ModelCopies\CopyLocalizer.vb` | done | [[modules/engineering-overview]] | 2026-06-18 |
| `Engineering\ModelCopies\GenericSmtPartFinder.vb` | done | [[modules/engineering-overview]] | 2026-06-18 |
| `Engineering\ModelCopies\UitsparingVoorplaatMeerpslAlu.vb` | done | [[modules/engineering-overview]] | 2026-06-18 |

### Forms

**Total: 188** &nbsp; | &nbsp; .vb: 127 | .resx: 61 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Forms\FrmProdChecklistViewer.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\FrmProdChecklistViewer.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\FrmProdChecklistViewer.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\FrmProdObjects.designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\FrmProdObjects.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\FrmProdObjects.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\FrmProdObjectsSelectCoating.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\FrmProdObjectsSelectCoating.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\FrmProdObjectsSelectCoating.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\ISAH\FrmPartBrowse.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\ISAH\FrmPartBrowse.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\ISAH\FrmPartBrowse.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Management\ClsApplRevisions.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Management\frmAddLeanProdTraject.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmAddLeanProdTraject.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmAddLeanProdTraject.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Management\frmAdminActivePdfMarkups.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmAdminActivePdfMarkups.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmAdminActivePdfMarkups.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Management\FrmApplNewRelease.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\FrmApplNewRelease.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\FrmApplNewRelease.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Management\FrmApplRevisions.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\FrmApplRevisions.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\FrmApplRevisions.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Management\frmCoatManagement.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmCoatManagement.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmCoatManagement.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Management\frmCoatManagementAdd.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmCoatManagementAdd.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmCoatManagementAdd.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Management\frmColors.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmColors.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmColors.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Management\frmConvertDocElement.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmConvertDocElement.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmConvertDocElement.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Management\frmEditCadBatchserverRules.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmEditCadBatchserverRules.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmEditCadBatchserverRules.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Management\frmEditCapacityTickets.designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmEditCapacityTickets.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmEditCapacityTickets.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Management\frmEditCoatingDefPrimer.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmEditCoatingDefPrimer.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmEditCoatingDefPrimer.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Management\frmEditKanbanBin.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmEditKanbanBin.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmEditKanbanBin.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Management\FrmEditLeanMachines.designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\FrmEditLeanMachines.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\FrmEditLeanMachines.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Management\frmEditMachGrps.designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmEditMachGrps.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmEditMachGrps.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Management\FrmEditTable.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\FrmEditTable.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\FrmEditTable.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Management\frmExtractIconFromFile.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmExtractIconFromFile.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmExtractIconFromFile.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Management\frmFeedbackUpdate.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmFeedbackUpdate.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmFeedbackUpdate.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Management\frmGenericsAdmin.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmGenericsAdmin.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmGenericsAdmin.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Management\FrmHelp.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\FrmHelp.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\FrmHelp.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Management\FrmUpdateProdTrackInWorkView.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\FrmUpdateProdTrackInWorkView.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\FrmUpdateProdTrackInWorkView.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\ShopProcess\FrmDgxPickJob.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\ShopProcess\FrmDgxPickJob.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\ShopProcess\FrmDgxPickJob.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\ShopProcess\FrmLeanAdmin.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\ShopProcess\FrmLeanAdmin.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\ShopProcess\FrmLeanAdmin.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\ShopProcess\FrmLeanDashboard.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\ShopProcess\FrmLeanDashboard.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\ShopProcess\FrmLeanDashboard.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\ShopProcess\FrmLeanFlowChart.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\ShopProcess\FrmLeanFlowChart.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\ShopProcess\FrmLeanFlowChart.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\ShopProcess\FrmLeanStatus.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\ShopProcess\FrmLeanStatus.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\ShopProcess\FrmLeanStatus.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\ShopProcess\FrmPartDispatchCollector.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\ShopProcess\FrmPartDispatchCollector.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\ShopProcess\FrmPartDispatchCollector.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\ShopProcess\FrmPartPick.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\ShopProcess\FrmPartPick.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\ShopProcess\FrmPartPick.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\ShopProcess\FrmWorkChange.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\ShopProcess\FrmWorkChange.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\ShopProcess\FrmWorkChange.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\ShopProcess\FrmWorkViewWithStatus.designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\ShopProcess\FrmWorkViewWithStatus.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\ShopProcess\FrmWorkViewWithStatus.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\ShopProcess\FrmWorkViewWithStatusJCOA.designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\ShopProcess\FrmWorkViewWithStatusJCOA.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\ShopProcess\FrmWorkViewWithStatusJCOA.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Template\AboutBox1.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Template\AboutBox1.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Template\AboutBox1.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Template\FrmBulkRename.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Template\FrmBulkRename.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Template\FrmBulkRename.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Template\FrmFadingMsgBox.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Template\FrmFadingMsgBox.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Template\FrmFadingMsgBox.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Template\frmHappyNewYear.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Template\frmHappyNewYear.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Template\frmHappyNewYear.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Template\FrmPrintPdf.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Template\FrmPrintPdf.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Template\FrmPrintPdf.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Template\FrmTextBox.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Template\FrmTextBox.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Template\FrmTextBox.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Template\LoginForm1.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Template\LoginForm1.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Template\LoginForm1.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Template\SelectFromListDialog.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Template\SelectFromListDialog.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Template\SelectFromListDialog.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Test3DSpace_Form1.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Test3DSpace_Form1.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Test3DSpace_Form2.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Test3DSpace_Form2.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Toolbox\FrmAddICenterDoc.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmAddICenterDoc.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmAddICenterDoc.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Toolbox\FrmBatchProdOrd.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmBatchProdOrd.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmBatchProdOrd.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Toolbox\FrmBomMember.designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmBomMember.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmBomMember.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Toolbox\FrmCadLicensesStatus.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmCadLicensesStatus.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmCadLicensesStatus.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Toolbox\FrmComPortScanner.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmComPortScanner.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmComPortScanner.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Toolbox\FrmContact.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmContact.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmContact.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Toolbox\FrmCrystalReport.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmCrystalReport.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmCrystalReport.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Toolbox\FrmDossierFromPurOrd.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmDossierFromPurOrd.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmDossierFromPurOrd.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Toolbox\frmGetValidPartcode.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\frmGetValidPartcode.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\frmGetValidPartcode.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Toolbox\frmJumpToOrdNr.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\frmJumpToOrdNr.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\frmJumpToOrdNr.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Toolbox\FrmLinkIsahToIPO.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmLinkIsahToIPO.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmLinkIsahToIPO.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Toolbox\FrmModelGenerator.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmModelGenerator.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmModelGenerator.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Toolbox\FrmNewDossierPos.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmNewDossierPos.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmNewDossierPos.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Toolbox\FrmNewICenterObject.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmNewICenterObject.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmNewICenterObject.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Toolbox\FrmPartCalculationUpdate.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmPartCalculationUpdate.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmPartCalculationUpdate.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Toolbox\FrmPosGenerator.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmPosGenerator.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmPosGenerator.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Toolbox\FrmPrintCeSticker.designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmPrintCeSticker.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmPrintCeSticker.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Toolbox\frmQuickview.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\frmQuickview.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\frmQuickview.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Forms\Toolbox\frmSelectProject.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\frmSelectProject.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\frmSelectProject.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |

### IcImporter

**Total: 6** &nbsp; | &nbsp; .vb: 5 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `IcImporter\FrmIcImport.Designer.vb` | generated | VS Forms designer partial | â€” |
| `IcImporter\FrmIcImport.resx` | generated | resource bundle (designer-managed) | â€” |
| `IcImporter\FrmIcImport.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `IcImporter\ICenterPart.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `IcImporter\IcImportHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `IcImporter\IcImportStatus.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |

### Kardex

**Total: 4** &nbsp; | &nbsp; .vb: 3 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Kardex\FrmKardexInterface.designer.vb` | generated | VS Forms designer partial | â€” |
| `Kardex\FrmKardexInterface.resx` | generated | resource bundle (designer-managed) | â€” |
| `Kardex\FrmKardexInterface.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Kardex\KardexProcessor.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |

### MarkTool

**Total: 5** &nbsp; | &nbsp; .vb: 4 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `MarkTool\ClsComPort.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `MarkTool\ClsMarkToolDb.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `MarkTool\frmMarkTool.Designer.vb` | generated | VS Forms designer partial | â€” |
| `MarkTool\frmMarkTool.resx` | generated | resource bundle (designer-managed) | â€” |
| `MarkTool\frmMarkTool.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |

### Modules

**Total: 3** &nbsp; | &nbsp; .vb: 3 | .resx: 0 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Modules\Functions.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Modules\MailMessageExt.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `Modules\Main.vb` | done | [[modules/main-module]] | 2026-06-18 |

### PCFNetStudio

**Total: 17** &nbsp; | &nbsp; .vb: 12 | .resx: 5 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `PCFNetStudio\CPartBuilder.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `PCFNetStudio\FrmCodeConverter.Designer.vb` | generated | VS Forms designer partial | â€” |
| `PCFNetStudio\FrmCodeConverter.resx` | generated | resource bundle (designer-managed) | â€” |
| `PCFNetStudio\FrmCodeConverter.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `PCFNetStudio\FrmProductComparer.Designer.vb` | generated | VS Forms designer partial | â€” |
| `PCFNetStudio\FrmProductComparer.resx` | generated | resource bundle (designer-managed) | â€” |
| `PCFNetStudio\FrmProductComparer.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `PCFNetStudio\PartEditor.Designer.vb` | generated | VS Forms designer partial | â€” |
| `PCFNetStudio\PartEditor.resx` | generated | resource bundle (designer-managed) | â€” |
| `PCFNetStudio\PartEditor.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `PCFNetStudio\ProductionSet.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `PCFNetStudio\UCCalculation.Designer.vb` | generated | VS Forms designer partial | â€” |
| `PCFNetStudio\UCCalculation.resx` | generated | resource bundle (designer-managed) | â€” |
| `PCFNetStudio\UCCalculation.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `PCFNetStudio\UCProductComparer.Designer.vb` | generated | VS Forms designer partial | â€” |
| `PCFNetStudio\UCProductComparer.resx` | generated | resource bundle (designer-managed) | â€” |
| `PCFNetStudio\UCProductComparer.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |

### Production

**Total: 6** &nbsp; | &nbsp; .vb: 6 | .resx: 0 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Production\ProductionProfileCutItemsHandler.vb` | done | [[modules/production-profile-cut-items]] | 2026-06-18 |
| `Production\ProfileMilling\ExternalReferenceHelper.vb` | done | [[modules/production-profile-milling-import]] | 2026-06-18 |
| `Production\ProfileMilling\ICAMImport.vb` | done | [[modules/production-profile-milling-import]] | 2026-06-18 |
| `Production\ProfileMilling\ImportData.vb` | done | [[modules/production-profile-milling-import]] | 2026-06-18 |
| `Production\ProfileMilling\ImportHandler.vb` | done | [[modules/production-profile-milling-import]] | 2026-06-18 |
| `Production\ProfileMilling\PMMExportHandler.vb` | done | [[modules/production-profile-milling-import]] | 2026-06-18 |

### Resources

**Total: 354** &nbsp; | &nbsp; .vb: 0 | .resx: 0 | assets: 353

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Resources\1downarrow1-32.png` | config | image/icon asset | â€” |
| `Resources\1leftarrow-32.png` | config | image/icon asset | â€” |
| `Resources\1rightarrow-32.png` | config | image/icon asset | â€” |
| `Resources\1uparrow-32.png` | config | image/icon asset | â€” |
| `Resources\2008-09-23 JAZO witte omranding 3cm.jpg` | config | image/icon asset | â€” |
| `Resources\2113918.png` | config | image/icon asset | â€” |
| `Resources\2dowarrow-32.png` | config | image/icon asset | â€” |
| `Resources\2leftarrow-32.png` | config | image/icon asset | â€” |
| `Resources\2rightarrow-32.png` | config | image/icon asset | â€” |
| `Resources\2uparrow-32.png` | config | image/icon asset | â€” |
| `Resources\5x-icon-sm.png` | config | image/icon asset | â€” |
| `Resources\accept-32.png` | config | image/icon asset | â€” |
| `Resources\Add Button-24.png` | config | image/icon asset | â€” |
| `Resources\Add Button-64.png` | config | image/icon asset | â€” |
| `Resources\Add_green_16.png` | config | image/icon asset | â€” |
| `Resources\Add_green_32.png` | config | image/icon asset | â€” |
| `Resources\Add-32.png` | config | image/icon asset | â€” |
| `Resources\AddButton_32_SE.png` | config | image/icon asset | â€” |
| `Resources\address_book-32.png` | config | image/icon asset | â€” |
| `Resources\Add-to_32.png` | config | image/icon asset | â€” |
| `Resources\Add-to-database-32.png` | config | image/icon asset | â€” |
| `Resources\agt_action_success-32.png` | config | image/icon asset | â€” |
| `Resources\agt_reload-32.png` | config | image/icon asset | â€” |
| `Resources\apply-64.png` | config | image/icon asset | â€” |
| `Resources\ArcEye_48.png` | config | image/icon asset | â€” |
| `Resources\Arrow_white_NE_16.png` | config | image/icon asset | â€” |
| `Resources\ArrowLong_right_64.png` | config | image/icon asset | â€” |
| `Resources\Arrow-sans-down-32.png` | config | image/icon asset | â€” |
| `Resources\Arrow-sans-up-32.png` | config | image/icon asset | â€” |
| `Resources\Assembly_16.png` | config | image/icon asset | â€” |
| `Resources\Assembly_link_16.png` | config | image/icon asset | â€” |
| `Resources\Ball-black-32.png` | config | image/icon asset | â€” |
| `Resources\Ball-green-32.png` | config | image/icon asset | â€” |
| `Resources\Ball-green-64.png` | config | image/icon asset | â€” |
| `Resources\Ball-red-32.png` | config | image/icon asset | â€” |
| `Resources\Ball-red-64.png` | config | image/icon asset | â€” |
| `Resources\Ball-yellow-64.png` | config | image/icon asset | â€” |
| `Resources\barcode_16.jpg` | config | image/icon asset | â€” |
| `Resources\barcode_laser_64.png` | config | image/icon asset | â€” |
| `Resources\Batch_128.png` | config | image/icon asset | â€” |
| `Resources\Batch_32.png` | config | image/icon asset | â€” |
| `Resources\bend_3d_16.png` | config | image/icon asset | â€” |
| `Resources\bend_3d_32.png` | config | image/icon asset | â€” |
| `Resources\bend_3d_64.png` | config | image/icon asset | â€” |
| `Resources\bendDie_16.png` | config | image/icon asset | â€” |
| `Resources\bendPiston_16.png` | config | image/icon asset | â€” |
| `Resources\Billiard-Marker-32.png` | config | image/icon asset | â€” |
| `Resources\Bin-blue-16.png` | config | image/icon asset | â€” |
| `Resources\Bin-blue-32.png` | config | image/icon asset | â€” |
| `Resources\Blue-Dossier-32.png` | config | image/icon asset | â€” |
| `Resources\Blue-Dossier-64.png` | config | image/icon asset | â€” |
| `Resources\Blue-Dossier-New-64.png` | config | image/icon asset | â€” |
| `Resources\bmt_16.png` | config | image/icon asset | â€” |
| `Resources\Box_65_65.png` | config | image/icon asset | â€” |
| `Resources\button_cancel-24.cur` | config | image/icon asset | â€” |
| `Resources\button_cancel-24.png` | config | image/icon asset | â€” |
| `Resources\button_cancel-32.png` | config | image/icon asset | â€” |
| `Resources\ButtonStop.png` | config | image/icon asset | â€” |
| `Resources\Calculator-32.png` | config | image/icon asset | â€” |
| `Resources\Calendar_16x16.png` | config | image/icon asset | â€” |
| `Resources\change_blue_32.png` | config | image/icon asset | â€” |
| `Resources\Chart-Line-32.png` | config | image/icon asset | â€” |
| `Resources\Check-64.png` | config | image/icon asset | â€” |
| `Resources\checkbox_checked.gif` | config | image/icon asset | â€” |
| `Resources\checkbox_checked_32.png` | config | image/icon asset | â€” |
| `Resources\checkbox_checked_png.png` | config | image/icon asset | â€” |
| `Resources\checkbox_unchecked.gif` | config | image/icon asset | â€” |
| `Resources\checkbox_unchecked_png.png` | config | image/icon asset | â€” |
| `Resources\Check-cropped-16.png` | config | image/icon asset | â€” |
| `Resources\Check-cropped-64.png` | config | image/icon asset | â€” |
| `Resources\Cinema4D_32.png` | config | image/icon asset | â€” |
| `Resources\Close_text_cursor.cur` | config | image/icon asset | â€” |
| `Resources\Close_text_cursor.png` | config | image/icon asset | â€” |
| `Resources\Close-48.png` | config | image/icon asset | â€” |
| `Resources\CMD922.PNG` | config | image/icon asset | â€” |
| `Resources\Cog-Edit-32.png` | config | image/icon asset | â€” |
| `Resources\Color-Edit-16.png` | config | image/icon asset | â€” |
| `Resources\Color-Select-32.png` | config | image/icon asset | â€” |
| `Resources\Comment-delete-icon16.png` | config | image/icon asset | â€” |
| `Resources\Comment-delete-icon24.png` | config | image/icon asset | â€” |
| `Resources\comment-edit-icon24.png` | config | image/icon asset | â€” |
| `Resources\compile_32.png` | config | image/icon asset | â€” |
| `Resources\compile_64.png` | config | image/icon asset | â€” |
| `Resources\Compress-16.png` | config | image/icon asset | â€” |
| `Resources\cone_vlc_24.png` | config | image/icon asset | â€” |
| `Resources\Copy_barcode_16.png` | config | image/icon asset | â€” |
| `Resources\creo_logo_32.jpg` | config | image/icon asset | â€” |
| `Resources\Database-Edit-32.png` | config | image/icon asset | â€” |
| `Resources\Date-From-32.png` | config | image/icon asset | â€” |
| `Resources\Delete_16.png` | config | image/icon asset | â€” |
| `Resources\Delete_32.png` | config | image/icon asset | â€” |
| `Resources\delete-32.png` | config | image/icon asset | â€” |
| `Resources\Delete-from_32.png` | config | image/icon asset | â€” |
| `Resources\dgw_32.png` | config | image/icon asset | â€” |
| `Resources\Document Edit_32.png` | config | image/icon asset | â€” |
| `Resources\Document-32.png` | config | image/icon asset | â€” |
| `Resources\Document-Import-32.png` | config | image/icon asset | â€” |
| `Resources\dwg_16.png` | config | image/icon asset | â€” |
| `Resources\Dxf_34.png` | config | image/icon asset | â€” |
| `Resources\edit_green_16.png` | config | image/icon asset | â€” |
| `Resources\edit_green_16NW.png` | config | image/icon asset | â€” |
| `Resources\edit_green_32.png` | config | image/icon asset | â€” |
| `Resources\Edit-16.png` | config | image/icon asset | â€” |
| `Resources\Edit-16-nw.png` | config | image/icon asset | â€” |
| `Resources\Edit-32.png` | config | image/icon asset | â€” |
| `Resources\Edit-64.png` | config | image/icon asset | â€” |
| `Resources\Edit-No-32.png` | config | image/icon asset | â€” |
| `Resources\Edit-Yes-32.png` | config | image/icon asset | â€” |
| `Resources\elfsquad-logo-large.png` | config | image/icon asset | â€” |
| `Resources\elu_tools.png` | config | image/icon asset | â€” |
| `Resources\EluBar_16.png` | config | image/icon asset | â€” |
| `Resources\EluCad0.ico` | config | image/icon asset | â€” |
| `Resources\elucad1.png` | config | image/icon asset | â€” |
| `Resources\EluCircle_16.png` | config | image/icon asset | â€” |
| `Resources\EluCut_16.png` | config | image/icon asset | â€” |
| `Resources\EluDeburr_16.png` | config | image/icon asset | â€” |
| `Resources\EluDrill_16.png` | config | image/icon asset | â€” |
| `Resources\EluFreeForm_16.png` | config | image/icon asset | â€” |
| `Resources\EluGroup_16.png` | config | image/icon asset | â€” |
| `Resources\EluInactive_16.png` | config | image/icon asset | â€” |
| `Resources\EluJob_16.png` | config | image/icon asset | â€” |
| `Resources\EluLine_16.png` | config | image/icon asset | â€” |
| `Resources\EluMacro_16.png` | config | image/icon asset | â€” |
| `Resources\EluRectangle_16.png` | config | image/icon asset | â€” |
| `Resources\EluSawCut_16.png` | config | image/icon asset | â€” |
| `Resources\EluSlottedHole_16.png` | config | image/icon asset | â€” |
| `Resources\Email_simple_16.png` | config | image/icon asset | â€” |
| `Resources\Email_simple_32.png` | config | image/icon asset | â€” |
| `Resources\Email-32.png` | config | image/icon asset | â€” |
| `Resources\Email-48.png` | config | image/icon asset | â€” |
| `Resources\emmegi_32.png` | config | image/icon asset | â€” |
| `Resources\Empty_Box_16.png` | config | image/icon asset | â€” |
| `Resources\empty_box_32.png` | config | image/icon asset | â€” |
| `Resources\empty_box_disabled_32.png` | config | image/icon asset | â€” |
| `Resources\Error_16_NW.png` | config | image/icon asset | â€” |
| `Resources\Error-32.png` | config | image/icon asset | â€” |
| `Resources\Error-48.png` | config | image/icon asset | â€” |
| `Resources\Error-64.png` | config | image/icon asset | â€” |
| `Resources\ErrorX-32.png` | config | image/icon asset | â€” |
| `Resources\evaluate.png` | config | image/icon asset | â€” |
| `Resources\excel_24.png` | config | image/icon asset | â€” |
| `Resources\excel_32.png` | config | image/icon asset | â€” |
| `Resources\excel_64.png` | config | image/icon asset | â€” |
| `Resources\F2-65x65.png` | config | image/icon asset | â€” |
| `Resources\Female_64.png` | config | image/icon asset | â€” |
| `Resources\FG LOGO 16X16.png` | config | image/icon asset | â€” |
| `Resources\FG LOGO 48x48.png` | config | image/icon asset | â€” |
| `Resources\file-64.png` | config | image/icon asset | â€” |
| `Resources\File-copy-32.png` | config | image/icon asset | â€” |
| `Resources\File-copy-48.png` | config | image/icon asset | â€” |
| `Resources\File-new-48.png` | config | image/icon asset | â€” |
| `Resources\fileopen-32.png` | config | image/icon asset | â€” |
| `Resources\Files-add-32.png` | config | image/icon asset | â€” |
| `Resources\Files-add-48.png` | config | image/icon asset | â€” |
| `Resources\filesave-16.png` | config | image/icon asset | â€” |
| `Resources\filesave-32.png` | config | image/icon asset | â€” |
| `Resources\filter-32.png` | config | image/icon asset | â€” |
| `Resources\filter-del-32.png` | config | image/icon asset | â€” |
| `Resources\Folder-32.png` | config | image/icon asset | â€” |
| `Resources\Folder-64.png` | config | image/icon asset | â€” |
| `Resources\Folder-blank-file-48.png` | config | image/icon asset | â€” |
| `Resources\Folder-light-32.png` | config | image/icon asset | â€” |
| `Resources\forklift_yellow_64.png` | config | image/icon asset | â€” |
| `Resources\Gauge-32.png` | config | image/icon asset | â€” |
| `Resources\gauge-type1-30-32px.png` | config | image/icon asset | â€” |
| `Resources\geo_16.png` | config | image/icon asset | â€” |
| `Resources\geo_open_contour_16.png` | config | image/icon asset | â€” |
| `Resources\glass_grey_32.png` | config | image/icon asset | â€” |
| `Resources\Glass-32.png` | config | image/icon asset | â€” |
| `Resources\Gnome-Stock-Person-64.png` | config | image/icon asset | â€” |
| `Resources\Golden_Star_NW_32.png` | config | image/icon asset | â€” |
| `Resources\Golden-Star-32.png` | config | image/icon asset | â€” |
| `Resources\Gray-Dossier-64.png` | config | image/icon asset | â€” |
| `Resources\Green-Dossier-16.png` | config | image/icon asset | â€” |
| `Resources\Green-Dossier-32.png` | config | image/icon asset | â€” |
| `Resources\Green-Dossier-64.png` | config | image/icon asset | â€” |
| `Resources\gridview.jpg` | config | image/icon asset | â€” |
| `Resources\hammer-32.png` | config | image/icon asset | â€” |
| `Resources\hammer-64.png` | config | image/icon asset | â€” |
| `Resources\hand_yellow_64.png` | config | image/icon asset | â€” |
| `Resources\Help_32.png` | config | image/icon asset | â€” |
| `Resources\Help_64.png` | config | image/icon asset | â€” |
| `Resources\home-32.png` | config | image/icon asset | â€” |
| `Resources\Hourglass-icon.png` | config | image/icon asset | â€” |
| `Resources\iCenter.ico` | config | image/icon asset | â€” |
| `Resources\iCenter.png` | config | image/icon asset | â€” |
| `Resources\import_24.png` | config | image/icon asset | â€” |
| `Resources\import-37-16.png` | config | image/icon asset | â€” |
| `Resources\import-37-32.png` | config | image/icon asset | â€” |
| `Resources\import-37-48.png` | config | image/icon asset | â€” |
| `Resources\info_blue_16.png` | config | image/icon asset | â€” |
| `Resources\info_blue-32.png` | config | image/icon asset | â€” |
| `Resources\info_blue-64.png` | config | image/icon asset | â€” |
| `Resources\Info-button-32.png` | config | image/icon asset | â€” |
| `Resources\Interface-builder.ico` | config | image/icon asset | â€” |
| `Resources\Interface-builder.png` | config | image/icon asset | â€” |
| `Resources\Interface-builder_16.png` | config | image/icon asset | â€” |
| `Resources\Isah_edit_16.png` | config | image/icon asset | â€” |
| `Resources\Isah_edit_32.png` | config | image/icon asset | â€” |
| `Resources\Isah_SW_16.png` | config | image/icon asset | â€” |
| `Resources\isah_table_edit_16.png` | config | image/icon asset | â€” |
| `Resources\JZ_logo_2012.png` | config | image/icon asset | â€” |
| `Resources\jzCadConnector.dll` | config | binary dependency | â€” |
| `Resources\JZ-logo-10x10mm.png` | config | image/icon asset | â€” |
| `Resources\JZSmall_50.png` | config | image/icon asset | â€” |
| `Resources\Kaltenbach_32.png` | config | image/icon asset | â€” |
| `Resources\Key-32.png` | config | image/icon asset | â€” |
| `Resources\kilogram-weight_64.png` | config | image/icon asset | â€” |
| `Resources\kilogram-weight_red_64.png` | config | image/icon asset | â€” |
| `Resources\layers_32.png` | config | image/icon asset | â€” |
| `Resources\LayerThicknessMeter_48.png` | config | image/icon asset | â€” |
| `Resources\length_32.png` | config | image/icon asset | â€” |
| `Resources\Length-32.png` | config | image/icon asset | â€” |
| `Resources\LightBlue-Dossier-32.png` | config | image/icon asset | â€” |
| `Resources\LightBlue-Dossier-64.png` | config | image/icon asset | â€” |
| `Resources\Link-Add-32.png` | config | image/icon asset | â€” |
| `Resources\Link-Break-32.png` | config | image/icon asset | â€” |
| `Resources\Link-Delete-32.png` | config | image/icon asset | â€” |
| `Resources\Link-Edit-32.png` | config | image/icon asset | â€” |
| `Resources\Locked-32.png` | config | image/icon asset | â€” |
| `Resources\maintenance_64.png` | config | image/icon asset | â€” |
| `Resources\manufacturing_16.png` | config | image/icon asset | â€” |
| `Resources\mcm_logo_blue_large.png` | config | image/icon asset | â€” |
| `Resources\merge-icon-16x16.png` | config | image/icon asset | â€” |
| `Resources\mill_16.png` | config | image/icon asset | â€” |
| `Resources\mill_alu_64.png` | config | image/icon asset | â€” |
| `Resources\Minus Button-24.png` | config | image/icon asset | â€” |
| `Resources\Minus Button-64.png` | config | image/icon asset | â€” |
| `Resources\New-32.png` | config | image/icon asset | â€” |
| `Resources\New-Year-Champagne-new-year-holiday-celebration-smiley-emoticon-000764-large.gif` | config | image/icon asset | â€” |
| `Resources\notify_16_NE.png` | config | image/icon asset | â€” |
| `Resources\notify_16_SW.png` | config | image/icon asset | â€” |
| `Resources\Ok-48.png` | config | image/icon asset | â€” |
| `Resources\Ok-64.png` | config | image/icon asset | â€” |
| `Resources\Open-32.png` | config | image/icon asset | â€” |
| `Resources\Orange-Dossier-32.png` | config | image/icon asset | â€” |
| `Resources\Orange-Dossier-64.png` | config | image/icon asset | â€” |
| `Resources\Packages-26.png` | config | image/icon asset | â€” |
| `Resources\paper_plane_16.png` | config | image/icon asset | â€” |
| `Resources\paper_plane_512_dqb_1-2.png` | config | image/icon asset | â€” |
| `Resources\Part_16.png` | config | image/icon asset | â€” |
| `Resources\part_link_16.png` | config | image/icon asset | â€” |
| `Resources\part_mill_16.png` | config | image/icon asset | â€” |
| `Resources\Paste-32.png` | config | image/icon asset | â€” |
| `Resources\pdf.png` | config | image/icon asset | â€” |
| `Resources\PDF-XChange_Icon_128.png` | config | image/icon asset | â€” |
| `Resources\Person-Undefined-Female-Light-64.png` | config | image/icon asset | â€” |
| `Resources\Person-Undefined-Male-Light-64.png` | config | image/icon asset | â€” |
| `Resources\phone_16.png` | config | image/icon asset | â€” |
| `Resources\phone_orange_16.png` | config | image/icon asset | â€” |
| `Resources\phone-32x32.png` | config | image/icon asset | â€” |
| `Resources\Phone-Blue-16.png` | config | image/icon asset | â€” |
| `Resources\Phone-Blue-32.png` | config | image/icon asset | â€” |
| `Resources\Planregels_RMB.png` | config | image/icon asset | â€” |
| `Resources\play_green_32.png` | config | image/icon asset | â€” |
| `Resources\player_pause-32.png` | config | image/icon asset | â€” |
| `Resources\player_play-32.png` | config | image/icon asset | â€” |
| `Resources\Plus-48.png` | config | image/icon asset | â€” |
| `Resources\pmi_16.png` | config | image/icon asset | â€” |
| `Resources\Printer-32x32.png` | config | image/icon asset | â€” |
| `Resources\printer-icon-32.gif` | config | image/icon asset | â€” |
| `Resources\Product-documentation-32.png` | config | image/icon asset | â€” |
| `Resources\Product-documentation-64.png` | config | image/icon asset | â€” |
| `Resources\ProeGlobalInterference_16.png` | config | image/icon asset | â€” |
| `Resources\proelogo.bmp` | config | image/icon asset | â€” |
| `Resources\Purple-Dossier-64.png` | config | image/icon asset | â€” |
| `Resources\rabbit_24.png` | config | image/icon asset | â€” |
| `Resources\Recycle-32.png` | config | image/icon asset | â€” |
| `Resources\Red-Dossier-32.png` | config | image/icon asset | â€” |
| `Resources\Red-Dossier-64.png` | config | image/icon asset | â€” |
| `Resources\Redo-32.png` | config | image/icon asset | â€” |
| `Resources\Redo-64.png` | config | image/icon asset | â€” |
| `Resources\Refresh-32.png` | config | image/icon asset | â€” |
| `Resources\Refresh-Orange-32.png` | config | image/icon asset | â€” |
| `Resources\reload_32.png` | config | image/icon asset | â€” |
| `Resources\Reminders-Wooden-32.png` | config | image/icon asset | â€” |
| `Resources\Remove_16.png` | config | image/icon asset | â€” |
| `Resources\Remove-from-database-32.png` | config | image/icon asset | â€” |
| `Resources\Resources-32.png` | config | image/icon asset | â€” |
| `Resources\rotate_blue_cw-48.png` | config | image/icon asset | â€” |
| `Resources\save_all-32.png` | config | image/icon asset | â€” |
| `Resources\Saw_Blade_26.jpg` | config | image/icon asset | â€” |
| `Resources\sawblad_32.png` | config | image/icon asset | â€” |
| `Resources\sawblad_47.png` | config | image/icon asset | â€” |
| `Resources\sawblade_32_anim.gif` | config | image/icon asset | â€” |
| `Resources\Search2-32.png` | config | image/icon asset | â€” |
| `Resources\Setting-64.png` | config | image/icon asset | â€” |
| `Resources\settings_gear_blue_32.png` | config | image/icon asset | â€” |
| `Resources\shopping_cart_16.png` | config | image/icon asset | â€” |
| `Resources\shopping_cart_32.png` | config | image/icon asset | â€” |
| `Resources\shopping_cart_64.png` | config | image/icon asset | â€” |
| `Resources\shopping_cart_yellow_64.png` | config | image/icon asset | â€” |
| `Resources\Signal-Stop-48.png` | config | image/icon asset | â€” |
| `Resources\sintlinks.gif` | config | image/icon asset | â€” |
| `Resources\Smiley_14.png` | config | image/icon asset | â€” |
| `Resources\Smiley-01.jpg` | config | image/icon asset | â€” |
| `Resources\Source-Code-32.png` | config | image/icon asset | â€” |
| `Resources\spyglass-32.png` | config | image/icon asset | â€” |
| `Resources\spyglass-64.png` | config | image/icon asset | â€” |
| `Resources\stack_32.png` | config | image/icon asset | â€” |
| `Resources\Started-cropped-16.png` | config | image/icon asset | â€” |
| `Resources\Started-cropped-64.png` | config | image/icon asset | â€” |
| `Resources\stop_32.png` | config | image/icon asset | â€” |
| `Resources\Stop-32.png` | config | image/icon asset | â€” |
| `Resources\Stop-64.png` | config | image/icon asset | â€” |
| `Resources\Table-Gear-32.png` | config | image/icon asset | â€” |
| `Resources\Taf_16.png` | config | image/icon asset | â€” |
| `Resources\Tag-Red-32 (1).png` | config | image/icon asset | â€” |
| `Resources\Tag-Red-32.png` | config | image/icon asset | â€” |
| `Resources\TCT_Carbide_Circular_Saw_Blade.jpg` | config | image/icon asset | â€” |
| `Resources\text_bold_16.png` | config | image/icon asset | â€” |
| `Resources\text_italic_16.png` | config | image/icon asset | â€” |
| `Resources\text_strikethrough_16.png` | config | image/icon asset | â€” |
| `Resources\text_underline_16.png` | config | image/icon asset | â€” |
| `Resources\time.png` | config | image/icon asset | â€” |
| `Resources\Time_64.png` | config | image/icon asset | â€” |
| `Resources\Time-Clock-16.png` | config | image/icon asset | â€” |
| `Resources\Time-Clock-16_nw.png` | config | image/icon asset | â€” |
| `Resources\Time-Clock-64.png` | config | image/icon asset | â€” |
| `Resources\Tip-32.png` | config | image/icon asset | â€” |
| `Resources\todo-icon-64.gif` | config | image/icon asset | â€” |
| `Resources\Tools_hammer_wrench_16.png` | config | image/icon asset | â€” |
| `Resources\Tools-32.png` | config | image/icon asset | â€” |
| `Resources\truck_darkgreen_64.png` | config | image/icon asset | â€” |
| `Resources\truck_green_32.png` | config | image/icon asset | â€” |
| `Resources\truck_green_64.png` | config | image/icon asset | â€” |
| `Resources\Trumpf.TruTops.Control.Shell.png` | config | image/icon asset | â€” |
| `Resources\trumpf_16.png` | config | image/icon asset | â€” |
| `Resources\trumpf_32.png` | config | image/icon asset | â€” |
| `Resources\trumpf_48.png` | config | image/icon asset | â€” |
| `Resources\turtle_24.png` | config | image/icon asset | â€” |
| `Resources\UniLink.png` | config | image/icon asset | â€” |
| `Resources\Unlocked-32.png` | config | image/icon asset | â€” |
| `Resources\User-add-32.png` | config | image/icon asset | â€” |
| `Resources\Users-info-48.png` | config | image/icon asset | â€” |
| `Resources\Users-mixed-gender-32.png` | config | image/icon asset | â€” |
| `Resources\us-proj_64.png` | config | image/icon asset | â€” |
| `Resources\waitCursor.gif` | config | image/icon asset | â€” |
| `Resources\Warning_16_NW.png` | config | image/icon asset | â€” |
| `Resources\Warning_16_SE.png` | config | image/icon asset | â€” |
| `Resources\warning_anim_32.gif` | config | image/icon asset | â€” |
| `Resources\warning_black_32.png` | config | image/icon asset | â€” |
| `Resources\warning_black_64.png` | config | image/icon asset | â€” |
| `Resources\Warning-32 (1).png` | config | image/icon asset | â€” |
| `Resources\Warning-64.png` | config | image/icon asset | â€” |
| `Resources\Wikipedia-32.png` | config | image/icon asset | â€” |
| `Resources\Windchill_Product_16.png` | config | image/icon asset | â€” |
| `Resources\Windchill_search_16.png` | config | image/icon asset | â€” |
| `Resources\Windchill11.png` | config | image/icon asset | â€” |
| `Resources\ws_where_used_report.gif` | config | image/icon asset | â€” |
| `Resources\Yellow-Dossier-64.png` | config | image/icon asset | â€” |
| `Resources\Zammad_icon_24.png` | config | image/icon asset | â€” |
| `Resources\Zammad_icon_32.png` | config | image/icon asset | â€” |
| `Resources\Zammad_icon_64.png` | config | image/icon asset | â€” |

### Sales

**Total: 3** &nbsp; | &nbsp; .vb: 2 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Sales\FrmCustomerTeam.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Sales\FrmCustomerTeam.resx` | generated | resource bundle (designer-managed) | â€” |
| `Sales\FrmCustomerTeam.vb` | done | [[modules/sales-customer-team]] | 2026-06-18 |

### SmtManufacturing

**Total: 78** &nbsp; | &nbsp; .vb: 63 | .resx: 15 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `SmtManufacturing\BendNote.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\BendPart.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\BendTool.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\BendToolGroup.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\BendToolStation.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\BncInterpreter\BNC.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\BoostMigrator.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\BoostPartViewerControl.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\BoostPartViewerControl.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\BoostPartViewerControl.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\ContourCheck.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\ControlSmtCut.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\ControlSmtCut.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\ControlSmtCut.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\CtrlBendBoost.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\CtrlBendBoost.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\CtrlBendBoost.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\CtrlBendCalcDetails.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\CtrlBendCalcDetails.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\CtrlMachinePartBendSolutions.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\CtrlMachinePartBendSolutions.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\CtrlMachinePartBendSolutions.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\CtrlPartBendSolution.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\CtrlPartBendSolution.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\CtrlPartBendSolution.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\CutSheetLabelPrintHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\DxfContour.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\FileMerger.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\FlatPatternConverter.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\FrmBendLicense.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\FrmBendLicense.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\FrmBendLicense.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\FrmCalcCycleTimeManagement.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\FrmCalcCycleTimeManagement.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\FrmCalcCycleTimeManagement.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\FrmNestedSheet.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\FrmNestedSheet.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\FrmNestedSheet.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\FrmPartIdentifier.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\FrmPartIdentifier.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\FrmPartIdentifier.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\FrmSmtMaterialManagement.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\FrmSmtMaterialManagement.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\FrmSmtMaterialManagement.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\frmTopsLicenses.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\frmTopsLicenses.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\frmTopsLicenses.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\GeoViewerControl.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\GeoViewerControl.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\GeoViewerControl.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\ISmtFileViewer.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\Job.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\JPLT_DistrSticker.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\JPLT_PartIdentSticker.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\JPLT_SheetIdentSticker.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\LaserCalc.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\NestedSheet.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\NestPart.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\Oid.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\Part.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\PreProcessorTest.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\PreProcessorTest.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\PreProcessorTest.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\ProductionOrderCleanupHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\Smt3DImportExclusionHelper.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\SmtFileViewerFactory.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\SmtPartViewer.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\SmtPartViewer.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\SmtPartViewer.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\TafInterpreter\Part.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\TafInterpreter\PartInstance.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\TafInterpreter\TAF.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\TmtInterpreter\TMT.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\TruTops.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\UCTAFViewer.designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\UCTAFViewer.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\UCTAFViewer.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SmtManufacturing\WorkViewBoost.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |

### SolaDataConnector

**Total: 2** &nbsp; | &nbsp; .vb: 2 | .resx: 0 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `SolaDataConnector\AppHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `SolaDataConnector\AppWindowHelper.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |

### UniLink

**Total: 30** &nbsp; | &nbsp; .vb: 29 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `UniLink\ApplicationHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\ApplicationService.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\CSVImportHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\Entities\List.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\Entities\Profile.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\ExceptionHandlers\ProfileSeriesExceptionHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\ExceptionHandlers\SourceFilesExceptionHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\Exceptions\ProfileSeriesException.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\Exceptions\SourceFilesException.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\ExportConverter.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\ExportHelper.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\ExportInstructions.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\Factories\ListFactory.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\ListDataService.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\Machine.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\MachineHelper.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\MecalAriel4Export.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\MultiStepReader\CSVHandler.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\MultiStepReader\LogDetailContent.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\MultiStepReader\MasterLog.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\MultiStepReader\MasterLogDetail.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\MultiStepReader\MasterLogDetailFactory.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\MultiStepReader\MasterLogDetailFile.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\MultiStepReader\MasterLogDetailOptimiser.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\MultiStepReader\MasterLogFactory.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\MultiStepReader\MasterLogService.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\Settings.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `UniLink\UI\TabControl.Designer.vb` | generated | VS Forms designer partial | â€” |
| `UniLink\UI\TabControl.resx` | generated | resource bundle (designer-managed) | â€” |
| `UniLink\UI\TabControl.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |

### VentDuctConfigurator

**Total: 4** &nbsp; | &nbsp; .vb: 3 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `VentDuctConfigurator\FrmVentDuctConfigurator.Designer.vb` | generated | VS Forms designer partial | â€” |
| `VentDuctConfigurator\FrmVentDuctConfigurator.resx` | generated | resource bundle (designer-managed) | â€” |
| `VentDuctConfigurator\FrmVentDuctConfigurator.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `VentDuctConfigurator\VentDuct.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |

### WebClock

**Total: 13** &nbsp; | &nbsp; .vb: 9 | .resx: 4 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `WebClock\FrmChangedTimeReg.Designer.vb` | generated | VS Forms designer partial | â€” |
| `WebClock\FrmChangedTimeReg.resx` | generated | resource bundle (designer-managed) | â€” |
| `WebClock\FrmChangedTimeReg.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `WebClock\FrmCurrentTimeReg.Designer.vb` | generated | VS Forms designer partial | â€” |
| `WebClock\FrmCurrentTimeReg.resx` | generated | resource bundle (designer-managed) | â€” |
| `WebClock\FrmCurrentTimeReg.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `WebClock\FrmOfficeClockDetailLines.Designer.vb` | generated | VS Forms designer partial | â€” |
| `WebClock\FrmOfficeClockDetailLines.resx` | generated | resource bundle (designer-managed) | â€” |
| `WebClock\FrmOfficeClockDetailLines.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `WebClock\FrmTimeRegistration.designer.vb` | generated | VS Forms designer partial | â€” |
| `WebClock\FrmTimeRegistration.resx` | generated | resource bundle (designer-managed) | â€” |
| `WebClock\FrmTimeRegistration.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |
| `WebClock\WebClock.vb` | done | [[mocs/icenter-remaining]] | 2026-06-18 |

### WorkPreparation

**Total: 6** &nbsp; | &nbsp; .vb: 5 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `WorkPreparation\FrmOutsourceOperations.Designer.vb` | generated | VS Forms designer partial | â€” |
| `WorkPreparation\FrmOutsourceOperations.resx` | generated | resource bundle (designer-managed) | â€” |
| `WorkPreparation\FrmOutsourceOperations.vb` | done | [[modules/workprep-outsource-operations]] | 2026-06-18 |
| `WorkPreparation\IPBatchCollector.vb` | done | [[modules/workprep-ipbatch-collector]] | 2026-06-18 |
| `WorkPreparation\OperationSubstitutionHandler.vb` | done | [[modules/workprep-operation-substitution]] | 2026-06-18 |
| `WorkPreparation\OutsourceOperationsHandler.vb` | done | [[modules/workprep-outsource-operations]] | 2026-06-18 |



---


---

# Project: TruTopsLib

**Total files: 65**

### TruTopsLib / (root)

**Total: 16** &nbsp; | &nbsp; .vb: 3 | .cs: 5 | .resx: 

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `app.config` | config | config / project metadata | â€” |
| `FileReaderHelper.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `FileReaderHelper.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `JAZO Zevenaar bv.snk` | config | signing key | â€” |
| `LayerConverter.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `LayerConverter.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `MigrationFix.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `MigrationFix.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `PMILabel.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `PMILabelCollection.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `Resources.Designer.cs` | generated | VS designer partial | â€” |
| `Resources.resx` | generated | resource bundle (designer-managed) | â€” |
| `TruTopsLib.csproj` | config | config / project metadata | â€” |
| `TruTopsLib.vbproj` | config | config / project metadata | â€” |
| `TruTopsLib.vbproj.vspscc` | config | config / project metadata | â€” |
| `TruTopsLib2.sln` | config | config / project metadata | â€” |

### TruTopsLib / GeoInterpreter

**Total: 10** &nbsp; | &nbsp; .vb: 5 | .cs: 5 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `GeoInterpreter\FlatGeometry.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `GeoInterpreter\FlatGeometry.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `GeoInterpreter\FlatGeometryDxfExport.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `GeoInterpreter\FlatGeometryDxfExport.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `GeoInterpreter\FlatGeometryGeoExport.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `GeoInterpreter\FlatGeometryGeoExport.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `GeoInterpreter\FlatGeometryReader.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `GeoInterpreter\FlatGeometryReader.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `GeoInterpreter\IFlatGeometryExport.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `GeoInterpreter\IFlatGeometryExport.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |

### TruTopsLib / PMI

**Total: 5** &nbsp; | &nbsp; .vb: 5 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `PMI\IPmiLabel.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `PMI\PMILabel.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `PMI\PMILabelCollection.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `PMI\PmiLabelCountersink.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `PMI\PmiLabelThreadNote.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |

### TruTopsLib / TopsFile

**Total: 34** &nbsp; | &nbsp; .vb: 17 | .cs: 17 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `TopsFile\BendLine.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\BendLine.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\Body.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\Body.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\Bounds.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\Bounds.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\Contour.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\Contour.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\DataTypeHandler.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\DataTypeHandler.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\Parameters.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\Parameters.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\Point.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\Point.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\PointCollection.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\PointCollection.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\Properties.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\Properties.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\SubContour\Arc.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\SubContour\Arc.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\SubContour\Circle.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\SubContour\Circle.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\SubContour\Fillet.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\SubContour\Fillet.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\SubContour\Line.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\SubContour\Line.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\SubContour\SubContour.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\SubContour\SubContour.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\SubContour\Text.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\SubContour\Text.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\TextCollection.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\TextCollection.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\TTInfo.cs` | done | [[mocs/trutopslib]] | 2026-06-18 |
| `TopsFile\TTInfo.vb` | done | [[mocs/trutopslib]] | 2026-06-18 |



---

# Project: ICenterLib

**Total files: 723**

### ICenterLib / (root)

**Total: 15** &nbsp; | &nbsp; .vb: 9 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `app.config` | config | config / project metadata | â€” |
| `AppSettings.vb` | done | [[modules/icenterlib-appsettings]] | 2026-06-18 |
| `Common.vb` | done | [[modules/icenterlib-common]] | 2026-06-18 |
| `ComputerSetting.vb` | done | [[mocs/icenterlib]] | 2026-06-18 |
| `Connections.vb` | done | [[modules/icenterlib-connections]] | 2026-06-18 |
| `ICenterLib.vbproj` | config | config / project metadata | â€” |
| `ICenterLib.vbproj.user` | config | config / project metadata | â€” |
| `ICenterLib.vbproj.vspscc` | config | config / project metadata | â€” |
| `Images.vb` | done | [[mocs/icenterlib]] | 2026-06-18 |
| `JAZO Zevenaar bv.snk` | config | signing key | â€” |
| `Log.vb` | done | [[mocs/icenterlib]] | 2026-06-18 |
| `Main.vb` | done | [[mocs/icenterlib]] | 2026-06-18 |
| `packages.config` | config | config / project metadata | â€” |
| `PdfTools.vb` | done | [[mocs/icenterlib]] | 2026-06-18 |
| `UserSetting.vb` | done | [[mocs/icenterlib]] | 2026-06-18 |

### ICenterLib / CAD

**Total: 127** &nbsp; | &nbsp; .vb: 116 | .cs: 0 | .resx: 5

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `CAD\Creo\AppManager.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\AppVersion.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\CadApp.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\CadAppVersion.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\CtrlAppManager.Designer.vb` | generated | VS designer partial | â€” |
| `CAD\Creo\CtrlAppManager.resx` | generated | resource bundle (designer-managed) | â€” |
| `CAD\Creo\CtrlAppManager.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\Dimension.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\Environment.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\Feature.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\Features.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\FrmAppManager.Designer.vb` | generated | VS designer partial | â€” |
| `CAD\Creo\FrmAppManager.resx` | generated | resource bundle (designer-managed) | â€” |
| `CAD\Creo\FrmAppManager.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\FrmCreoLicense.Designer.vb` | generated | VS designer partial | â€” |
| `CAD\Creo\FrmCreoLicense.resx` | generated | resource bundle (designer-managed) | â€” |
| `CAD\Creo\FrmCreoLicense.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\License\License.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\License\LicenseResource.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\License\LicenseResourceHandler.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\License\LicenseSelectionHandler.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\Material.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\Materials.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\ModelInformation.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\ModelItem.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\Parameter.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\ParameterCollection.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\ParamValue\ParamValue.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\ParamValue\ParamValueBoolean.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\ParamValue\ParamValueDouble.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\ParamValue\ParamValueInteger.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\ParamValue\ParamValueString.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\PlmAppVersion.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\ProProgram\Design.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\ProProgram\ExecuteStatement.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\ProProgram\Functions.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\ProProgram\Input.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\ProProgram\VBCodeConverter.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\RegenerationInput.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\StartupFile.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\Toolbox.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\Tools\StpAssySplitter.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Creo\Trailfile.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\CreoView\Application.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\CreoView\Configuration.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\CreoView\Converter.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\DXF\SvgConverter\DxfHelper.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\DXF\SvgConverter\DxfToSvgConverter.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\DXF\SvgConverter\Export.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\DXF\SvgConverter\Modifier.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Geometry\BoundingBox.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Geometry\BoundingBoxFactory.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Geometry\Dxf3DProfileMill.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Geometry\DxfEntityColor.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Geometry\Earcut.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Geometry\Earcut_CSharp.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Geometry\GraphicsPathHelper.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\Configuration.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\Definition.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\DefinitionCollection.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\Exceptions\GenericObjectNotFoundException.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\GenericModelBackupConfiguration.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\ModelGeneratorOption.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\ModelgeneratorRequests\DesignDuplicationConfiguration.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\ModelgeneratorRequests\ElfsquadModelgeneratorRequest.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\ModelgeneratorRequests\IrisModelgeneratorInstructions.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\ModelgeneratorRequests\IrisModelgeneratorTaskResult.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\Operations\Checkin.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\Operations\CreateWS.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\Operations\DeleteWS.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\Operations\Download.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\Operations\EraseUndisplayedModels.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\Operations\ExportDocumentOperation.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\Operations\JZCheckoutFolders.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\Operations\JZExportByNumber.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\Operations\JZImportByNumber.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\Operations\JZRenameObjectNoServer.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\Operations\OpenInProE.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\Operations\Operation.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\Operations\OperationCollection.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\Operations\RegenReadPar.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\Operations\Register.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\Operations\Rename.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\Operations\Save.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\Operations\SetWorkingDirectory.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\Operations\Unregister.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\RenameRule.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\RenameRuleCollection.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Modelgenerator\TriggerFile.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\OpenGL\CtrlOpenGLViewer.Designer.vb` | generated | VS designer partial | â€” |
| `CAD\OpenGL\CtrlOpenGLViewer.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\OpenGL\GLUtil.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\Archive.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\AutoPromotionRequest.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\AutoPromotionRequestHandler.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\AutoPromotionRequestParameters.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\Container.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\EPMDocument.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\FileServer.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\Folder.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\FrmAutomaticPromotionRequestInput.Designer.vb` | generated | VS designer partial | â€” |
| `CAD\PLM\FrmAutomaticPromotionRequestInput.resx` | generated | resource bundle (designer-managed) | â€” |
| `CAD\PLM\FrmAutomaticPromotionRequestInput.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\FrmVaultFolderChart.Designer.vb` | generated | VS designer partial | â€” |
| `CAD\PLM\FrmVaultFolderChart.resx` | generated | resource bundle (designer-managed) | â€” |
| `CAD\PLM\FrmVaultFolderChart.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\InfoEngineParameter.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\InfoEngineParameterCollection.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\LifeCycleState.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\Product.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\PromotionNotice.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\ServerManagement.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\Task\AddVaultFolderDetailsLogEntry.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\Task\GenericModelBackup.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\Task\PurgePromotionNotices.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\Task\Task.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\Toolbox.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\User.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\VaultCleanupAuditLogs.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\Version.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\PLM\WCObject.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\Publisher\Common.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\SolidEdge\IfcExport.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\SolidEdge\Importer.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\SolidEdge\Settings.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\SolidEdge\StepExport.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |
| `CAD\SolidEdge\Toolkit.vb` | done | [[mocs/icenterlib-cad]] | 2026-06-18 |

### ICenterLib / CadBatchServer

**Total: 17** &nbsp; | &nbsp; .vb: 17 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `CadBatchServer\CadBatchserverDataService.vb` | done | [[mocs/icenterlib-cadbatchserver]] | 2026-06-18 |
| `CadBatchServer\CadBatchserverStatus.vb` | done | [[mocs/icenterlib-cadbatchserver]] | 2026-06-18 |
| `CadBatchServer\CadBatchserverStatusCollection.vb` | done | [[mocs/icenterlib-cadbatchserver]] | 2026-06-18 |
| `CadBatchServer\CadBatchserverStatusDataService.vb` | done | [[mocs/icenterlib-cadbatchserver]] | 2026-06-18 |
| `CadBatchServer\DistributedLockCreoPublish.vb` | done | [[mocs/icenterlib-cadbatchserver]] | 2026-06-18 |
| `CadBatchServer\JobAlreadyExistsException.vb` | done | [[mocs/icenterlib-cadbatchserver]] | 2026-06-18 |
| `CadBatchServer\JobDataService.vb` | done | [[mocs/icenterlib-cadbatchserver]] | 2026-06-18 |
| `CadBatchServer\JobParameters.vb` | done | [[mocs/icenterlib-cadbatchserver]] | 2026-06-18 |
| `CadBatchServer\JobToolbox.vb` | done | [[mocs/icenterlib-cadbatchserver]] | 2026-06-18 |
| `CadBatchServer\Modelgenerator\CadInputParameters.vb` | done | [[mocs/icenterlib-cadbatchserver]] | 2026-06-18 |
| `CadBatchServer\Modelgenerator\DuplicateInstruction.vb` | done | [[mocs/icenterlib-cadbatchserver]] | 2026-06-18 |
| `CadBatchServer\Modelgenerator\ModelgeneratorInstructions.vb` | done | [[mocs/icenterlib-cadbatchserver]] | 2026-06-18 |
| `CadBatchServer\Modelgenerator\ModelgeneratorTask.vb` | done | [[mocs/icenterlib-cadbatchserver]] | 2026-06-18 |
| `CadBatchServer\ModelgeneratorDataService.vb` | done | [[mocs/icenterlib-cadbatchserver]] | 2026-06-18 |
| `CadBatchServer\PublishJobInstructions.vb` | done | [[mocs/icenterlib-cadbatchserver]] | 2026-06-18 |
| `CadBatchServer\PublishMonitor.vb` | done | [[mocs/icenterlib-cadbatchserver]] | 2026-06-18 |
| `CadBatchServer\PublishWatchDirProcessor.vb` | done | [[mocs/icenterlib-cadbatchserver]] | 2026-06-18 |

### ICenterLib / Comparer

**Total: 1** &nbsp; | &nbsp; .vb:  | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Comparer\DateComparer.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |

### ICenterLib / Connections

**Total: 1** &nbsp; | &nbsp; .vb:  | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Connections\WebClients\WindchillWebClientProvider.vb` | done | [[modules/icenterlib-connections]] | 2026-06-18 |

### ICenterLib / CrystalReport

**Total: 5** &nbsp; | &nbsp; .vb: 5 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `CrystalReport\ExportInstruction.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `CrystalReport\ExportRequest.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `CrystalReport\ExportResult.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `CrystalReport\PrintingInstruction.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `CrystalReport\ReportParameter.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |

### ICenterLib / DataHandler

**Total: 23** &nbsp; | &nbsp; .vb: 19 | .cs: 0 | .resx: 2

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `DataHandler\BetterDataGridView.vb` | done | [[mocs/icenterlib-datahandler]] | 2026-06-18 |
| `DataHandler\DataSetComparer.vb` | done | [[mocs/icenterlib-datahandler]] | 2026-06-18 |
| `DataHandler\DataSetCompareResult.vb` | done | [[mocs/icenterlib-datahandler]] | 2026-06-18 |
| `DataHandler\DataSetCompareTolerance.vb` | done | [[mocs/icenterlib-datahandler]] | 2026-06-18 |
| `DataHandler\DataTableColumnSchema.vb` | done | [[mocs/icenterlib-datahandler]] | 2026-06-18 |
| `DataHandler\DataTableColumnSchemaHandler.vb` | done | [[mocs/icenterlib-datahandler]] | 2026-06-18 |
| `DataHandler\DataTableColumnSchemaRecord.vb` | done | [[mocs/icenterlib-datahandler]] | 2026-06-18 |
| `DataHandler\ExportExcel.vb` | done | [[mocs/icenterlib-datahandler]] | 2026-06-18 |
| `DataHandler\FrmDataGridView.Designer.vb` | generated | VS designer partial | â€” |
| `DataHandler\FrmDataGridView.resx` | generated | resource bundle (designer-managed) | â€” |
| `DataHandler\FrmDataGridView.vb` | done | [[mocs/icenterlib-datahandler]] | 2026-06-18 |
| `DataHandler\FrmDataViewer.Designer.vb` | generated | VS designer partial | â€” |
| `DataHandler\FrmDataViewer.resx` | generated | resource bundle (designer-managed) | â€” |
| `DataHandler\FrmDataViewer.vb` | done | [[mocs/icenterlib-datahandler]] | 2026-06-18 |
| `DataHandler\GenericQuery.vb` | done | [[mocs/icenterlib-datahandler]] | 2026-06-18 |
| `DataHandler\IStreamWrapper.vb` | done | [[mocs/icenterlib-datahandler]] | 2026-06-18 |
| `DataHandler\JsonHelper.vb` | done | [[mocs/icenterlib-datahandler]] | 2026-06-18 |
| `DataHandler\OpenXml\Excel.vb` | done | [[mocs/icenterlib-datahandler]] | 2026-06-18 |
| `DataHandler\OpenXml\OpenXmlSpreadsheet.vb` | done | [[mocs/icenterlib-datahandler]] | 2026-06-18 |
| `DataHandler\QrCode.vb` | done | [[mocs/icenterlib-datahandler]] | 2026-06-18 |
| `DataHandler\Selection.vb` | done | [[mocs/icenterlib-datahandler]] | 2026-06-18 |
| `DataHandler\Toolbox.vb` | done | [[mocs/icenterlib-datahandler]] | 2026-06-18 |
| `DataHandler\ZeroCode.vb` | done | [[mocs/icenterlib-datahandler]] | 2026-06-18 |

### ICenterLib / DataServices

**Total: 10** &nbsp; | &nbsp; .vb: 10 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `DataServices\Contracts\IProdexConfiguratorDataService.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `DataServices\Contracts\IProdexDossierDetailDesignDataService.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `DataServices\Contracts\IProdexModelgeneratorDataService.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `DataServices\CrystalReportDataService.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `DataServices\DistributedLockDataService.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `DataServices\ElfsquadDataService.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `DataServices\ProdexConfiguratorDataService.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `DataServices\ProdexDossierDetailDesignDataService.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `DataServices\ProdexModelgeneratorDataService.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `DataServices\ProductDbDataService.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |

### ICenterLib / Debug

**Total: 1** &nbsp; | &nbsp; .vb:  | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Debug\Debug1.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |

### ICenterLib / DistributedLock

**Total: 1** &nbsp; | &nbsp; .vb:  | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `DistributedLock\DistributedLock.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |

### ICenterLib / Elfsquad

**Total: 9** &nbsp; | &nbsp; .vb: 5 | .cs: 0 | .resx: 2

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Elfsquad\ConfigurationMappingMismatchException.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `Elfsquad\ConfigurationRequestType.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `Elfsquad\ModelgeneratorHelper.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `Elfsquad\UCConfigurationManager.Designer.vb` | generated | VS designer partial | â€” |
| `Elfsquad\UCConfigurationManager.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elfsquad\UCConfigurationManager.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `Elfsquad\UCModelgenerator.Designer.vb` | generated | VS designer partial | â€” |
| `Elfsquad\UCModelgenerator.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elfsquad\UCModelgenerator.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |

### ICenterLib / Enums

**Total: 2** &nbsp; | &nbsp; .vb: 2 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Enums\Application.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `Enums\ISAH.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |

### ICenterLib / GUI

**Total: 3** &nbsp; | &nbsp; .vb: 3 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `GUI\MyMenuItem.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `GUI\MyMenuItemConverter.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `GUI\ToolStripMenuItemHelper.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |

### ICenterLib / Helpers

**Total: 1** &nbsp; | &nbsp; .vb:  | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Helpers\UrlHelper.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |

### ICenterLib / iCenter

**Total: 29** &nbsp; | &nbsp; .vb: 24 | .cs: 0 | .resx: 2

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `iCenter\BillOfOper.vb` | done | [[modules/icenterlib-icenter-batch-hierarchy]] | 2026-06-18 |
| `iCenter\Client.vb` | done | [[modules/icenterlib-icenter-production-machines]] | 2026-06-18 |
| `iCenter\CtrlImageIdentification.designer.vb` | generated | VS designer partial | â€” |
| `iCenter\CtrlImageIdentification.resx` | generated | resource bundle (designer-managed) | â€” |
| `iCenter\CtrlImageIdentification.vb` | done | [[modules/icenterlib-icenter-identification]] | 2026-06-18 |
| `iCenter\CtrlRadButtonIdent.designer.vb` | generated | VS designer partial | â€” |
| `iCenter\CtrlRadButtonIdent.vb` | done | [[modules/icenterlib-icenter-identification]] | 2026-06-18 |
| `iCenter\DataServices\CoatingLayerThicknessDataService.vb` | done | [[modules/icenterlib-icenter-leaves]] | 2026-06-18 |
| `iCenter\DossierContactFavorite.vb` | done | [[modules/icenterlib-icenter-leaves]] | 2026-06-18 |
| `iCenter\ExternalReference.vb` | done | [[modules/icenterlib-icenter-leaves]] | 2026-06-18 |
| `iCenter\ExternalReferences.vb` | done | [[modules/icenterlib-icenter-leaves]] | 2026-06-18 |
| `iCenter\FrmIdentification.designer.vb` | generated | VS designer partial | â€” |
| `iCenter\FrmIdentification.resx` | generated | resource bundle (designer-managed) | â€” |
| `iCenter\FrmIdentification.vb` | done | [[modules/icenterlib-icenter-identification]] | 2026-06-18 |
| `iCenter\IPBatch.vb` | done | [[modules/icenterlib-icenter-batch-hierarchy]] | 2026-06-18 |
| `iCenter\IPOrder.vb` | done | [[modules/icenterlib-icenter-batch-hierarchy]] | 2026-06-18 |
| `iCenter\IPPacket.vb` | done | [[modules/icenterlib-icenter-batch-hierarchy]] | 2026-06-18 |
| `iCenter\IPPart.vb` | done | [[modules/icenterlib-icenter-batch-hierarchy]] | 2026-06-18 |
| `iCenter\Part.vb` | done | [[modules/icenterlib-icenter-leaves]] | 2026-06-18 |
| `iCenter\ProductionMachineMultiPurpose.vb` | done | [[modules/icenterlib-icenter-production-machines]] | 2026-06-18 |
| `iCenter\ProductionMachines.vb` | done | [[modules/icenterlib-icenter-production-machines]] | 2026-06-18 |
| `iCenter\Servicedesk.vb` | done | [[modules/icenterlib-icenter-leaves]] | 2026-06-18 |
| `iCenter\SurfaceTreatmentDefinition.vb` | done | [[modules/icenterlib-icenter-leaves]] | 2026-06-18 |
| `iCenter\TimeRegistration.vb` | done | [[modules/icenterlib-icenter-leaves]] | 2026-06-18 |
| `iCenter\WebClockAssistant\WebClockAssistantAnonymousHandler.vb` | done | [[modules/icenterlib-icenter-identification]] | 2026-06-18 |
| `iCenter\WebClockAssistant\WebClockAssistantGenericHandler.vb` | done | [[modules/icenterlib-icenter-identification]] | 2026-06-18 |
| `iCenter\WebClockAssistant\WebClockAssistantRepository.vb` | done | [[modules/icenterlib-icenter-identification]] | 2026-06-18 |
| `iCenter\WebClockAssistant\WebClockAssistantUserSelectionHandler.vb` | done | [[modules/icenterlib-icenter-identification]] | 2026-06-18 |
| `iCenter\XmlFile.vb` | done | [[modules/icenterlib-icenter-leaves]] | 2026-06-18 |

### ICenterLib / ISAH

**Total: 65** &nbsp; | &nbsp; .vb: 63 | .cs: 0 | .resx: 

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `ISAH\BillOfMat.vb` | done | [[modules/isah-production-hierarchy]] | 2026-06-18 |
| `ISAH\BillOfOper.vb` | done | [[modules/isah-production-hierarchy]] | 2026-06-18 |
| `ISAH\CallRegistration.vb` | done | [[modules/isah-leaves]] | 2026-06-18 |
| `ISAH\Company.vb` | done | [[modules/isah-identity]] | 2026-06-18 |
| `ISAH\Contact.vb` | done | [[modules/isah-leaves]] | 2026-06-18 |
| `ISAH\Customer.vb` | done | [[modules/isah-leaves]] | 2026-06-18 |
| `ISAH\CustomerRelation.vb` | done | [[modules/isah-leaves]] | 2026-06-18 |
| `ISAH\CustomerSelection.vb` | done | [[modules/isah-leaves]] | 2026-06-18 |
| `ISAH\Database.vb` | done | [[modules/isah-leaves]] | 2026-06-18 |
| `ISAH\DataServices\EmployeeDataService.vb` | done | [[modules/isah-sub-services]] | 2026-06-18 |
| `ISAH\DataServices\IsahCustomisingElfsquadDataService.vb` | done | [[modules/isah-sub-services]] | 2026-06-18 |
| `ISAH\DataServices\MemoDetailDataService.vb` | done | [[modules/isah-sub-services]] | 2026-06-18 |
| `ISAH\DataServices\PartDataService.vb` | done | [[modules/isah-part-and-dispatch]] | 2026-06-18 |
| `ISAH\DataServices\PartDispatchCollectorDataService.vb` | done | [[modules/isah-part-and-dispatch]] | 2026-06-18 |
| `ISAH\DataServices\ToolboxDataService.vb` | done | [[modules/isah-sub-services]] | 2026-06-18 |
| `ISAH\DataServices\UpdateProdLeadTimeDataService.vb` | done | [[modules/isah-sub-services]] | 2026-06-18 |
| `ISAH\DateDimension.vb` | done | [[modules/isah-leaves]] | 2026-06-18 |
| `ISAH\DeliveryLine.vb` | done | [[modules/isah-leaves]] | 2026-06-18 |
| `ISAH\Design.vb` | done | [[modules/isah-leaves]] | 2026-06-18 |
| `ISAH\DossierDetail.vb` | done | [[modules/isah-dossier]] | 2026-06-18 |
| `ISAH\DossierDetailExtra.vb` | done | [[modules/isah-dossier]] | 2026-06-18 |
| `ISAH\DossierDetailExtraDto.vb` | done | [[modules/isah-dossier]] | 2026-06-18 |
| `ISAH\DossierDocFolder.vb` | done | [[modules/isah-dossier]] | 2026-06-18 |
| `ISAH\DossierMain.vb` | done | [[modules/isah-dossier]] | 2026-06-18 |
| `ISAH\Employee.vb` | done | [[modules/isah-identity]] | 2026-06-18 |
| `ISAH\FrmJConfigParamDesignCode.Designer.vb` | generated | VS designer partial | â€” |
| `ISAH\FrmJConfigParamDesignCode.resx` | generated | resource bundle (designer-managed) | â€” |
| `ISAH\FrmJConfigParamDesignCode.vb` | done | [[modules/isah-leaves]] | 2026-06-18 |
| `ISAH\Handlers\UpdateProdLeadTimeHandler.vb` | done | [[modules/isah-sub-services]] | 2026-06-18 |
| `ISAH\Helpers\DossierDetailExtraHelper.vb` | done | [[modules/isah-dossier]] | 2026-06-18 |
| `ISAH\Helpers\EncryptionHelper.vb` | done | [[modules/isah-sub-services]] | 2026-06-18 |
| `ISAH\Helpers\TextStyling\HtmlPlainTextHelper.vb` | done | [[modules/isah-sub-services]] | 2026-06-18 |
| `ISAH\Helpers\TextStyling\IPlainTextHelper.vb` | done | [[modules/isah-sub-services]] | 2026-06-18 |
| `ISAH\Helpers\TextStyling\PlainTextHelper.vb` | done | [[modules/isah-sub-services]] | 2026-06-18 |
| `ISAH\Helpers\TextStyling\RtfPlainTextHelper.vb` | done | [[modules/isah-sub-services]] | 2026-06-18 |
| `ISAH\Icenter2Isah.vb` | done | [[modules/isah-icenter-to-isah]] | 2026-06-18 |
| `ISAH\IsahFieldML.vb` | done | [[modules/isah-leaves]] | 2026-06-18 |
| `ISAH\JConfigParam.vb` | done | [[modules/isah-leaves]] | 2026-06-18 |
| `ISAH\Language.vb` | done | [[modules/isah-leaves]] | 2026-06-18 |
| `ISAH\MachGrp.vb` | done | [[modules/isah-machgrp]] | 2026-06-18 |
| `ISAH\MemoDetailElfsquadConfiguration.vb` | done | [[modules/isah-leaves]] | 2026-06-18 |
| `ISAH\MultiFinance.vb` | done | [[modules/isah-leaves]] | 2026-06-18 |
| `ISAH\Part.vb` | done | [[modules/isah-part-and-dispatch]] | 2026-06-18 |
| `ISAH\PartDispatch.vb` | done | [[modules/isah-part-and-dispatch]] | 2026-06-18 |
| `ISAH\PartDispatchCollectorDataService.vb` | done | [[modules/isah-part-and-dispatch]] | 2026-06-18 |
| `ISAH\PartSelection.vb` | done | [[modules/isah-part-and-dispatch]] | 2026-06-18 |
| `ISAH\PartVendor.vb` | done | [[modules/isah-part-and-dispatch]] | 2026-06-18 |
| `ISAH\PBOM.vb` | done | [[modules/isah-production-hierarchy]] | 2026-06-18 |
| `ISAH\PBOO.vb` | done | [[modules/isah-production-hierarchy]] | 2026-06-18 |
| `ISAH\PBOS.vb` | done | [[modules/isah-production-hierarchy]] | 2026-06-18 |
| `ISAH\ProductionHeader.vb` | done | [[modules/isah-production-hierarchy]] | 2026-06-18 |
| `ISAH\PurchaseDocumentPartLine.vb` | done | [[modules/isah-leaves]] | 2026-06-18 |
| `ISAH\PurDoc.vb` | done | [[modules/isah-shop-and-pur-doc]] | 2026-06-18 |
| `ISAH\Selection.vb` | done | [[modules/isah-lookups]] | 2026-06-18 |
| `ISAH\Setting.vb` | done | [[modules/isah-lookups]] | 2026-06-18 |
| `ISAH\ShopDoc.vb` | done | [[modules/isah-shop-and-pur-doc]] | 2026-06-18 |
| `ISAH\ShopDocCollection.vb` | done | [[modules/isah-leaves]] | 2026-06-18 |
| `ISAH\TimeRegCollector.vb` | done | [[modules/isah-time-registration]] | 2026-06-18 |
| `ISAH\TimeRegistration.vb` | done | [[modules/isah-time-registration]] | 2026-06-18 |
| `ISAH\User.vb` | done | [[modules/isah-identity]] | 2026-06-18 |
| `ISAH\Vendor.vb` | done | [[modules/isah-leaves]] | 2026-06-18 |
| `ISAH\ViewModels\CustomerAddressViewModel.vb` | done | [[modules/isah-sub-services]] | 2026-06-18 |
| `ISAH\ViewModels\PartBasicViewModel.vb` | done | [[modules/isah-sub-services]] | 2026-06-18 |
| `ISAH\WeighingFactor.vb` | done | [[modules/isah-leaves]] | 2026-06-18 |
| `ISAH\WorkView.vb` | done | [[modules/isah-leaves]] | 2026-06-18 |

### ICenterLib / JIBA

**Total: 14** &nbsp; | &nbsp; .vb: 14 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `JIBA\AppParameter.vb` | done | [[mocs/icenterlib-jiba]] | 2026-06-18 |
| `JIBA\Asset.vb` | done | [[mocs/icenterlib-jiba]] | 2026-06-18 |
| `JIBA\Company.vb` | done | [[mocs/icenterlib-jiba]] | 2026-06-18 |
| `JIBA\ConfigPart.vb` | done | [[mocs/icenterlib-jiba]] | 2026-06-18 |
| `JIBA\CustSat.vb` | done | [[mocs/icenterlib-jiba]] | 2026-06-18 |
| `JIBA\Employee.vb` | done | [[mocs/icenterlib-jiba]] | 2026-06-18 |
| `JIBA\Encryption.vb` | done | [[mocs/icenterlib-jiba]] | 2026-06-18 |
| `JIBA\Enums.vb` | done | [[mocs/icenterlib-jiba]] | 2026-06-18 |
| `JIBA\LinkItem.vb` | done | [[mocs/icenterlib-jiba]] | 2026-06-18 |
| `JIBA\Log.vb` | done | [[mocs/icenterlib-jiba]] | 2026-06-18 |
| `JIBA\Menu.vb` | done | [[mocs/icenterlib-jiba]] | 2026-06-18 |
| `JIBA\NavigationGroupItem.vb` | done | [[mocs/icenterlib-jiba]] | 2026-06-18 |
| `JIBA\SubMenu.vb` | done | [[mocs/icenterlib-jiba]] | 2026-06-18 |
| `JIBA\XmlData.vb` | done | [[mocs/icenterlib-jiba]] | 2026-06-18 |

### ICenterLib / JMail

**Total: 6** &nbsp; | &nbsp; .vb: 6 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `JMail\FileHelper.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `JMail\Message.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `JMail\SpecificationReportData.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `JMail\SpecificationReportHandler.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `JMail\StartOptions.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `JMail\StartOptionsHelper.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |

### ICenterLib / LaserWork

**Total: 4** &nbsp; | &nbsp; .vb: 2 | .cs: 0 | .resx: 

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `LaserWork\AppWrapper.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `LaserWork\UCLaserWork.Designer.vb` | generated | VS designer partial | â€” |
| `LaserWork\UCLaserWork.resx` | generated | resource bundle (designer-managed) | â€” |
| `LaserWork\UCLaserWork.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |

### ICenterLib / Metabase

**Total: 4** &nbsp; | &nbsp; .vb: 4 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Metabase\ParameterParsing.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `Metabase\ParameterResolver.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `Metabase\QueryHelper.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `Metabase\UrlHelper.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |

### ICenterLib / ModelDefinition

**Total: 3** &nbsp; | &nbsp; .vb: 3 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `ModelDefinition\Dimension.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `ModelDefinition\DimensionCollection.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `ModelDefinition\DimensionType.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |

### ICenterLib / MySystem

**Total: 22** &nbsp; | &nbsp; .vb: 20 | .cs: 0 | .resx: 

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `MySystem\Computer.vb` | done | [[mocs/icenterlib-mysystem]] | 2026-06-18 |
| `MySystem\Encryption\FrmEncrypt.Designer.vb` | generated | VS designer partial | â€” |
| `MySystem\Encryption\FrmEncrypt.resx` | generated | resource bundle (designer-managed) | â€” |
| `MySystem\Encryption\FrmEncrypt.vb` | done | [[mocs/icenterlib-mysystem]] | 2026-06-18 |
| `MySystem\Encryption\SecurityController.vb` | done | [[mocs/icenterlib-mysystem]] | 2026-06-18 |
| `MySystem\Environment.vb` | done | [[mocs/icenterlib-mysystem]] | 2026-06-18 |
| `MySystem\ExceptionList.vb` | done | [[mocs/icenterlib-mysystem]] | 2026-06-18 |
| `MySystem\FileSystem.vb` | done | [[mocs/icenterlib-mysystem]] | 2026-06-18 |
| `MySystem\HealthMonitorClient.vb` | done | [[mocs/icenterlib-mysystem]] | 2026-06-18 |
| `MySystem\HelpHandler.vb` | done | [[mocs/icenterlib-mysystem]] | 2026-06-18 |
| `MySystem\ICenterObjectNotFoundException.vb` | done | [[mocs/icenterlib-mysystem]] | 2026-06-18 |
| `MySystem\Math.vb` | done | [[mocs/icenterlib-mysystem]] | 2026-06-18 |
| `MySystem\MyProcess.vb` | done | [[mocs/icenterlib-mysystem]] | 2026-06-18 |
| `MySystem\Net\TcpServer.vb` | done | [[mocs/icenterlib-mysystem]] | 2026-06-18 |
| `MySystem\Network.vb` | done | [[mocs/icenterlib-mysystem]] | 2026-06-18 |
| `MySystem\OpenWindowGetter.vb` | done | [[mocs/icenterlib-mysystem]] | 2026-06-18 |
| `MySystem\PowerShellWrapper.vb` | done | [[mocs/icenterlib-mysystem]] | 2026-06-18 |
| `MySystem\Printer.vb` | done | [[mocs/icenterlib-mysystem]] | 2026-06-18 |
| `MySystem\Registry.vb` | done | [[mocs/icenterlib-mysystem]] | 2026-06-18 |
| `MySystem\TerminalServerSessions.vb` | done | [[mocs/icenterlib-mysystem]] | 2026-06-18 |
| `MySystem\Window.vb` | done | [[mocs/icenterlib-mysystem]] | 2026-06-18 |
| `MySystem\WindowsUser.vb` | done | [[mocs/icenterlib-mysystem]] | 2026-06-18 |

### ICenterLib / PCFNet

**Total: 45** &nbsp; | &nbsp; .vb: 39 | .cs: 0 | .resx: 3

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `PCFNet\BOM.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\BOO.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\Calculation.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\ControlDefinition.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\ControlDefinitionComparer.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\ControlMapping\ControlMap.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\ControlMapping\Mapping.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\ControlMapping\MappingCollection.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\ControlMapping\MappingQuery.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\ControlMapping\Parameter.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\ControlMappingDesigner\UCControlMappingDesigner.Designer.vb` | generated | VS designer partial | â€” |
| `PCFNet\ControlMappingDesigner\UCControlMappingDesigner.resx` | generated | resource bundle (designer-managed) | â€” |
| `PCFNet\ControlMappingDesigner\UCControlMappingDesigner.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\ControlMappingImport.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\CPart.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\ExcelObject.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\ExcelObjectCalculator.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\FastenerCalculator.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\GenericPart.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\GenericPartTemplate.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\JConfigurator.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\ManualOperation.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\ManualProperty.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\NameMapping.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\NameMappingRule.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\ParametersMapping\AluBasicWallLouver001.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\ParametersMapping\AluDoor001.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\ParametersMapping\AluLouver001.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\ParametersMapping\ParametersMappingBase.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\ParametersMapping\PlankAssembly.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\ParametersMapping\StlDoor001.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\PcfCompiler.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\PcfNetDataSet.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\ProductValidation.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\PropertyDefinition.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\PropertyDefinitionType.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\SmtCalculator.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\TcpClientConfiguration.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\UCInputControls.Designer.vb` | generated | VS designer partial | â€” |
| `PCFNet\UCInputControls.resx` | generated | resource bundle (designer-managed) | â€” |
| `PCFNet\UCInputControls.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\UCOption.Designer.vb` | generated | VS designer partial | â€” |
| `PCFNet\UCOption.resx` | generated | resource bundle (designer-managed) | â€” |
| `PCFNet\UCOption.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |
| `PCFNet\WAASHandler.vb` | done | [[mocs/icenterlib-pcfnet]] | 2026-06-18 |

### ICenterLib / Prodex

**Total: 6** &nbsp; | &nbsp; .vb: 6 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Prodex\Models\DesignConfiguration.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `Prodex\Models\DesignParameter.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `Prodex\Models\DesignParameters.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `Prodex\Models\DossierDetailDesignCalculationRequest.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `Prodex\Models\DossierDetailDesignUpdateRequest.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `Prodex\ViewModels\DossierDetailDesignCalculationVM.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |

### ICenterLib / ProductDb

**Total: 31** &nbsp; | &nbsp; .vb: 21 | .cs: 0 | .resx: 5

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `ProductDb\CloneProductHandler.vb` | done | [[mocs/icenterlib-productdb]] | 2026-06-18 |
| `ProductDb\CommonDb.vb` | done | [[mocs/icenterlib-productdb]] | 2026-06-18 |
| `ProductDb\DeclarationOfPerformance.vb` | done | [[mocs/icenterlib-productdb]] | 2026-06-18 |
| `ProductDb\ExcelTemplate.vb` | done | [[mocs/icenterlib-productdb]] | 2026-06-18 |
| `ProductDb\ExcelWorkBookHelper.vb` | done | [[mocs/icenterlib-productdb]] | 2026-06-18 |
| `ProductDb\FrmCloneProduct.Designer.vb` | generated | VS designer partial | â€” |
| `ProductDb\FrmCloneProduct.resx` | generated | resource bundle (designer-managed) | â€” |
| `ProductDb\FrmCloneProduct.vb` | done | [[mocs/icenterlib-productdb]] | 2026-06-18 |
| `ProductDb\FrmProductDbPriceList.Designer.vb` | generated | VS designer partial | â€” |
| `ProductDb\FrmProductDbPriceList.resx` | generated | resource bundle (designer-managed) | â€” |
| `ProductDb\FrmProductDbPriceList.vb` | done | [[mocs/icenterlib-productdb]] | 2026-06-18 |
| `ProductDb\FrmProductPricePart.Designer.vb` | generated | VS designer partial | â€” |
| `ProductDb\FrmProductPricePart.resx` | generated | resource bundle (designer-managed) | â€” |
| `ProductDb\FrmProductPricePart.vb` | done | [[mocs/icenterlib-productdb]] | 2026-06-18 |
| `ProductDb\Language.vb` | done | [[mocs/icenterlib-productdb]] | 2026-06-18 |
| `ProductDb\PriceListHelper.vb` | done | [[mocs/icenterlib-productdb]] | 2026-06-18 |
| `ProductDb\PricePart.vb` | done | [[mocs/icenterlib-productdb]] | 2026-06-18 |
| `ProductDb\Product.vb` | done | [[mocs/icenterlib-productdb]] | 2026-06-18 |
| `ProductDb\ProductConfiguration.vb` | done | [[mocs/icenterlib-productdb]] | 2026-06-18 |
| `ProductDb\ProductConfigurationDataService.vb` | done | [[mocs/icenterlib-productdb]] | 2026-06-18 |
| `ProductDb\ProductConfiguratorMapping.vb` | done | [[mocs/icenterlib-productdb]] | 2026-06-18 |
| `ProductDb\ProductFilter.vb` | done | [[mocs/icenterlib-productdb]] | 2026-06-18 |
| `ProductDb\ProductFilterGroup.vb` | done | [[mocs/icenterlib-productdb]] | 2026-06-18 |
| `ProductDb\ProductGroup.vb` | done | [[mocs/icenterlib-productdb]] | 2026-06-18 |
| `ProductDb\ProductPricePart.vb` | done | [[mocs/icenterlib-productdb]] | 2026-06-18 |
| `ProductDb\UCPropertyDefinitionEditor.Designer.vb` | generated | VS designer partial | â€” |
| `ProductDb\UCPropertyDefinitionEditor.resx` | generated | resource bundle (designer-managed) | â€” |
| `ProductDb\UCPropertyDefinitionEditor.vb` | done | [[mocs/icenterlib-productdb]] | 2026-06-18 |
| `ProductDb\UCPropertyDefinitionManager.Designer.vb` | generated | VS designer partial | â€” |
| `ProductDb\UCPropertyDefinitionManager.resx` | generated | resource bundle (designer-managed) | â€” |
| `ProductDb\UCPropertyDefinitionManager.vb` | done | [[mocs/icenterlib-productdb]] | 2026-06-18 |

### ICenterLib / Production

**Total: 23** &nbsp; | &nbsp; .vb: 19 | .cs: 0 | .resx: 2

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Production\AutoIPOrderSelection.vb` | done | [[mocs/icenterlib-production]] | 2026-06-18 |
| `Production\BaseProductionItem.vb` | done | [[mocs/icenterlib-production]] | 2026-06-18 |
| `Production\BOMFilter.vb` | done | [[mocs/icenterlib-production]] | 2026-06-18 |
| `Production\CEChecklist.vb` | done | [[mocs/icenterlib-production]] | 2026-06-18 |
| `Production\FGQualityControlPart.vb` | done | [[mocs/icenterlib-production]] | 2026-06-18 |
| `Production\FrmProdMachineSelector.Designer.vb` | generated | VS designer partial | â€” |
| `Production\FrmProdMachineSelector.resx` | generated | resource bundle (designer-managed) | â€” |
| `Production\FrmProdMachineSelector.vb` | done | [[mocs/icenterlib-production]] | 2026-06-18 |
| `Production\KanbanBin.vb` | done | [[mocs/icenterlib-production]] | 2026-06-18 |
| `Production\KeyPerformanceIndicator.vb` | done | [[mocs/icenterlib-production]] | 2026-06-18 |
| `Production\LabelLog.vb` | done | [[mocs/icenterlib-production]] | 2026-06-18 |
| `Production\ProdChecklist.vb` | done | [[mocs/icenterlib-production]] | 2026-06-18 |
| `Production\ProdChecklistTag.vb` | done | [[mocs/icenterlib-production]] | 2026-06-18 |
| `Production\ProductionLine.vb` | done | [[mocs/icenterlib-production]] | 2026-06-18 |
| `Production\ProductionLog.vb` | done | [[mocs/icenterlib-production]] | 2026-06-18 |
| `Production\ProductionProfileCutItem.vb` | done | [[mocs/icenterlib-production]] | 2026-06-18 |
| `Production\ProductionProfileCutItemHandler.vb` | done | [[mocs/icenterlib-production]] | 2026-06-18 |
| `Production\ProductionProfileCutItemsHandler.vb` | done | [[modules/production-profile-cut-items]] | 2026-06-18 |
| `Production\ProductionRegistrationAnalysisRange.vb` | done | [[mocs/icenterlib-production]] | 2026-06-18 |
| `Production\SmtBendQueue.vb` | done | [[mocs/icenterlib-production]] | 2026-06-18 |
| `Production\UCProdLineLeanStatus.Designer.vb` | generated | VS designer partial | â€” |
| `Production\UCProdLineLeanStatus.resx` | generated | resource bundle (designer-managed) | â€” |
| `Production\UCProdLineLeanStatus.vb` | done | [[mocs/icenterlib-production]] | 2026-06-18 |

### ICenterLib / Resources

**Total: 75** &nbsp; | &nbsp; .vb: 0 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Resources\1downarrow1-32.png` | config | image/icon asset | â€” |
| `Resources\1leftarrow-32.png` | config | image/icon asset | â€” |
| `Resources\1rightarrow-32.png` | config | image/icon asset | â€” |
| `Resources\1uparrow-32.png` | config | image/icon asset | â€” |
| `Resources\2008-09-23 JAZO witte omranding 3cm.jpg` | config | image/icon asset | â€” |
| `Resources\2dowarrow-32.png` | config | image/icon asset | â€” |
| `Resources\2leftarrow-32.png` | config | image/icon asset | â€” |
| `Resources\2rightarrow-32.png` | config | image/icon asset | â€” |
| `Resources\2uparrow-32.png` | config | image/icon asset | â€” |
| `Resources\Add Button-24.png` | config | image/icon asset | â€” |
| `Resources\Add_green_16.png` | config | image/icon asset | â€” |
| `Resources\Batch_128.png` | config | image/icon asset | â€” |
| `Resources\Bullet-Black-16.png` | config | image/icon asset | â€” |
| `Resources\button_cancel-24.png` | config | image/icon asset | â€” |
| `Resources\Check-16.png` | config | image/icon asset | â€” |
| `Resources\checkbox_checked.gif` | config | image/icon asset | â€” |
| `Resources\checkbox_checked_png.png` | config | image/icon asset | â€” |
| `Resources\checkbox_unchecked.gif` | config | image/icon asset | â€” |
| `Resources\checkbox_unchecked_png.png` | config | image/icon asset | â€” |
| `Resources\Copy_32.png` | config | image/icon asset | â€” |
| `Resources\creo_logo_16.png` | config | image/icon asset | â€” |
| `Resources\creo_logo_32.jpg` | config | image/icon asset | â€” |
| `Resources\creo_regen_jz_16.png` | config | image/icon asset | â€” |
| `Resources\creo_regen_jz_24.png` | config | image/icon asset | â€” |
| `Resources\creo_with_regen_jz_16.png` | config | image/icon asset | â€” |
| `Resources\Edit-32.png` | config | image/icon asset | â€” |
| `Resources\elfsquad-logo-16.png` | config | image/icon asset | â€” |
| `Resources\elfsquad-logo-large.png` | config | image/icon asset | â€” |
| `Resources\ErrorX-32.png` | config | image/icon asset | â€” |
| `Resources\excel_16.png` | config | image/icon asset | â€” |
| `Resources\excel_24.png` | config | image/icon asset | â€” |
| `Resources\excel_32.png` | config | image/icon asset | â€” |
| `Resources\excel_64.png` | config | image/icon asset | â€” |
| `Resources\fabpartseverity_1.png` | config | image/icon asset | â€” |
| `Resources\fabpartseverity_2.png` | config | image/icon asset | â€” |
| `Resources\fabpartstatus_1.png` | config | image/icon asset | â€” |
| `Resources\fabpartstatus_12.png` | config | image/icon asset | â€” |
| `Resources\fabpartstatus_13.png` | config | image/icon asset | â€” |
| `Resources\fabpartstatus_14.png` | config | image/icon asset | â€” |
| `Resources\fabpartstatus_2.png` | config | image/icon asset | â€” |
| `Resources\fabpartstatus_20.png` | config | image/icon asset | â€” |
| `Resources\fabpartstatus_3.png` | config | image/icon asset | â€” |
| `Resources\fabpartstatus_4.png` | config | image/icon asset | â€” |
| `Resources\fabpartstatus_5.png` | config | image/icon asset | â€” |
| `Resources\fabpartstatus_6.png` | config | image/icon asset | â€” |
| `Resources\fabpartstatus_7.png` | config | image/icon asset | â€” |
| `Resources\fabpartstatus_70.png` | config | image/icon asset | â€” |
| `Resources\fabpartstatus_8.png` | config | image/icon asset | â€” |
| `Resources\Female_64.png` | config | image/icon asset | â€” |
| `Resources\filesave-16.png` | config | image/icon asset | â€” |
| `Resources\filesave-32.png` | config | image/icon asset | â€” |
| `Resources\Gnome-Stock-Person-64.png` | config | image/icon asset | â€” |
| `Resources\gridview.jpg` | config | image/icon asset | â€” |
| `Resources\iCenter.ico` | config | image/icon asset | â€” |
| `Resources\import_24.png` | config | image/icon asset | â€” |
| `Resources\info_blue_16.png` | config | image/icon asset | â€” |
| `Resources\info_blue_161.png` | config | image/icon asset | â€” |
| `Resources\info_blue-32.png` | config | image/icon asset | â€” |
| `Resources\Interface-builder_16.png` | config | image/icon asset | â€” |
| `Resources\merge-icon-16x16.png` | config | image/icon asset | â€” |
| `Resources\merge-icon-32x32.png` | config | image/icon asset | â€” |
| `Resources\Modelgenerator-creo-icon.png` | config | image/icon asset | â€” |
| `Resources\Person-Undefined-Male-Light-64.png` | config | image/icon asset | â€” |
| `Resources\Refresh-32.png` | config | image/icon asset | â€” |
| `Resources\Resources-32.png` | config | image/icon asset | â€” |
| `Resources\Setting-16.png` | config | image/icon asset | â€” |
| `Resources\Setting-64.png` | config | image/icon asset | â€” |
| `Resources\texture_alu01.bmp` | config | image/icon asset | â€” |
| `Resources\warning_anim_32.gif` | config | image/icon asset | â€” |
| `Resources\Warning-32 (1).png` | config | image/icon asset | â€” |
| `Resources\Warning-64.png` | config | image/icon asset | â€” |
| `Resources\Windchill11.png` | config | image/icon asset | â€” |
| `Resources\Zammad_icon_24.png` | config | image/icon asset | â€” |
| `Resources\Zammad_icon_32.png` | config | image/icon asset | â€” |
| `Resources\Zammad_icon_64.png` | config | image/icon asset | â€” |

### ICenterLib / SmartForms

**Total: 1** &nbsp; | &nbsp; .vb:  | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `SmartForms\SmartForm.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |

### ICenterLib / SmtCadCam

**Total: 4** &nbsp; | &nbsp; .vb: 4 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `SmtCadCam\PreProcessorHandler.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `SmtCadCam\SpaceClaimApiHelper.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `SmtCadCam\SpaceClaimServerRequest.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `SmtCadCam\SpaceClaimServerRequestDataService.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |

### ICenterLib / SmtProduction

**Total: 128** &nbsp; | &nbsp; .vb: 126 | .cs: 0 | .resx: 

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `SmtProduction\DataServices\CutSheetOperRegistrationDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\DataServices\OseonAppContextDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\DataServices\PartDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\DataServices\ProductionOrderDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\DataServices\WorkplaceEmployeeLinkDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\Entities\CutSheetOperRegistration.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\Entities\ImportSettings2D.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\Entities\ImportSettings3D.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\Entities\ImportSettingsBase.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\Entities\OseonAppContext.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\Entities\WorkplaceEmployeeLink.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\ImportFileValidator.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\ProgrammingEnvironment.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Client\Application.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Collections\CadCamDocumentCollection.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Contracts\IAppMerkerDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Contracts\IBasicMaterialDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Contracts\IBendSolutionDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Contracts\ICadCamDocumentDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Contracts\ICutSheetDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Contracts\IDataTableColumnSchemaDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Contracts\IMasterWorkPlanDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Contracts\IPartBendSolutionDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Contracts\IPartDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Contracts\IPartOnTableDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Contracts\IPartStatusDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Contracts\IPdmDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Contracts\IProductionOperationDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Contracts\IProductionOrderDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Contracts\IRawMaterialDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Contracts\ITTNGActionDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Contracts\IWorkplaceDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\DataServices\AppMerkerDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\DataServices\BasicMaterialDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\DataServices\BendSolutionDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\DataServices\CadCamDocumentDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\DataServices\CutSheetDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\DataServices\MasterWorkPlanDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\DataServices\PartBendSolutionDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\DataServices\PartDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\DataServices\PartOnTableDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\DataServices\PartStatusDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\DataServices\PdmDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\DataServices\ProductionOperationDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\DataServices\ProductionOrderDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\DataServices\RawMaterialDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\DataServices\TTNGActionDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\DataServices\WorkplaceDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Entities\AppMerker.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Entities\BasicMaterial.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Entities\CadCamDocument.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Entities\CadCamDocumentType.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Entities\CutSheet.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Entities\MasterWorkPlan.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Entities\Operation.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Entities\Part.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Entities\PartBendSolution.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Entities\PartDisplayStatus.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Entities\PartOnTable.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Entities\PartStatus.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Entities\PartStatusMaster.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Entities\ProductionOperation.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Entities\ProductionOrder.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Entities\RawMaterial.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Entities\TTNGBendToolList.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\Entities\Workplace.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Oseon\PDM.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\DataServices\ImportLogDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\DataServices\ImportResultDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\DataServices\PPSInterfaceDataService.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Exceptions\PropertyNotSpecifiedException.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjectItems\FeedbackObjectItemBase.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjectItems\FeedbackObjectItemDimensions.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjectItems\FeedbackObjectItemPartOnSheet.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjectItems\FeedbackObjectItemSheet.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjects\FeedbackObjectBase.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjects\FeedbackObjectPDAMessage.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjects\FeedbackObjectProcessedSheetReport.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjects\FeedbackObjectProductionOperation.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjects\FeedbackObjectProductionOrder.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Export\PPSExport\PPSExportBase.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Export\PPSExport\PPSExportConsumptionReport.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Export\PPSExport\PPSExportManufacturedSheet.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Export\PPSExport\PPSExportOperation.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Export\PPSExport\PPSExportPDAMessage.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Export\PPSExport\PPSExportPPSExportManufacturedSheetOper.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Export\PPSExport\PPSExportProductionOrder.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Export\ProductionOrderExportHandler.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Export\ProductionQuantityReport.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\Collections\BendSolutionCollection.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\Collections\DeleteProductionOrderCollection.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\Collections\DeleteProductionOrderCollectionXmlHandler.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\Collections\DocumentCollection.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\Collections\FinishProductionOrderCollection.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\Collections\PartCollection.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\Collections\ProductionOrderCollection.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\BendSolution.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\DeleteProductionOrder.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\Document.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\FinishProductionOrder.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\ImportResult.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\Part.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\PPSImport.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\PPSInterface.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\PPSInterfaceDate.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\ProductionOrder.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\WorkingPlan.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\WorkingStep.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\ProductionOrderImportHandler.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\BendSolutionXmlHandler.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\DeleteProductionOrderXmlHandler.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\DocumentXmlHandler.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\FinishProductionOrderCollectionXmlHandler.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\FinishProductionOrderXmlHandler.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\PartCollectionXmlHandler.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\PartXmlHandler.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\PPSImportXmlHandler.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\ProductionOrderCollectionXmlHandler.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\ProductionOrderXmlHandler.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\WorkingPlanXmlHandler.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\WorkingStepXmlHandler.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\PPSInterface\Utils\TimeConversionHelper.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Utils\ProductionOrderNumberHelper.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\TruTops\Utils\TruTopsConvertHandler.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\UI\CtrlTTNGProcessingErrorInfo.Designer.vb` | generated | VS designer partial | â€” |
| `SmtProduction\UI\CtrlTTNGProcessingErrorInfo.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtProduction\UI\CtrlTTNGProcessingErrorInfo.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |
| `SmtProduction\UI\OseonAppContextSelectionWrapper.vb` | done | [[mocs/icenterlib-smtproduction]] | 2026-06-18 |

### ICenterLib / STEP3D

**Total: 5** &nbsp; | &nbsp; .vb: 5 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `STEP3D\AssySplitter.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `STEP3D\DefinitionAnalyser.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `STEP3D\Model.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `STEP3D\Product.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `STEP3D\Reader.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |

### ICenterLib / Ticketing

**Total: 5** &nbsp; | &nbsp; .vb: 5 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Ticketing\ApiConnector.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `Ticketing\Article.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `Ticketing\Attachment.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `Ticketing\Ticket.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `Ticketing\TicketHandler.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |

### ICenterLib / TimeRegistration

**Total: 2** &nbsp; | &nbsp; .vb: 2 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `TimeRegistration\MovingEmployeeTimeRegistration.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `TimeRegistration\WebClock.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |

### ICenterLib / UserControls

**Total: 34** &nbsp; | &nbsp; .vb: 15 | .cs: 0 | .resx: 8

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `UserControls\DataGridViewFormatter.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `UserControls\FriendlyComboBox.Designer.vb` | generated | VS designer partial | â€” |
| `UserControls\FriendlyComboBox.resx` | generated | resource bundle (designer-managed) | â€” |
| `UserControls\FriendlyComboBox.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `UserControls\FrmRemoteDesktop.Designer.vb` | generated | VS designer partial | â€” |
| `UserControls\FrmRemoteDesktop.resx` | generated | resource bundle (designer-managed) | â€” |
| `UserControls\FrmRemoteDesktop.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `UserControls\FrmUserSelection.Designer.vb` | generated | VS designer partial | â€” |
| `UserControls\FrmUserSelection.resx` | generated | resource bundle (designer-managed) | â€” |
| `UserControls\FrmUserSelection.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `UserControls\FrmWebView.Designer.vb` | generated | VS designer partial | â€” |
| `UserControls\FrmWebView.resx` | generated | resource bundle (designer-managed) | â€” |
| `UserControls\FrmWebView.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `UserControls\HintComboBox.Designer.vb` | generated | VS designer partial | â€” |
| `UserControls\HintComboBox.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `UserControls\ListViewWithReordering.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `UserControls\ProProgramEditor.Designer.vb` | generated | VS designer partial | â€” |
| `UserControls\ProProgramEditor.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `UserControls\Ticker.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `UserControls\UCDossierDetailDesign.Designer.vb` | generated | VS designer partial | â€” |
| `UserControls\UCDossierDetailDesign.resx` | generated | resource bundle (designer-managed) | â€” |
| `UserControls\UCDossierDetailDesign.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `UserControls\UCFastColoredTextBox.Designer.vb` | generated | VS designer partial | â€” |
| `UserControls\UCFastColoredTextBox.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `UserControls\UCNoAccess.Designer.vb` | generated | VS designer partial | â€” |
| `UserControls\UCNoAccess.resx` | generated | resource bundle (designer-managed) | â€” |
| `UserControls\UCNoAccess.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `UserControls\UCTicker.Designer.vb` | generated | VS designer partial | â€” |
| `UserControls\UCTicker.resx` | generated | resource bundle (designer-managed) | â€” |
| `UserControls\UCTicker.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `UserControls\UCWebView.Designer.vb` | generated | VS designer partial | â€” |
| `UserControls\UCWebView.resx` | generated | resource bundle (designer-managed) | â€” |
| `UserControls\UCWebView.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |
| `UserControls\XWikiForm.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |

### ICenterLib / Zabbix

**Total: 1** &nbsp; | &nbsp; .vb:  | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Zabbix\ZabbixHandler.vb` | done | [[mocs/icenterlib-small-folders]] | 2026-06-18 |


## Roll-up

_Heuristic snapshot updated whenever new projects come into scope or files transition out of `todo`._

### By project

| Project | Total | Done | Todo | Config | Generated | Dead | Needs-review |
|---------|------:|-----:|-----:|-------:|----------:|-----:|-------------:|
| iCENTER | 1237 | 531 | 0 | 358 | 341 | 0 | 7 |
| TruTopsLib | 65 | 57 | 0 | 6 | 2 | 0 | 0 |
| ICenterLib | 723 | 571 | 0 | 81 | 71 | 0 | 0 |
| **TOTAL** | **2025** | **1159** | **0** | **445** | **414** | **0** | **7** |

### Per iCENTER sub-folder

| Folder | Total | Done | Todo | Config | Generated | Dead | Needs-review |
|--------|------:|-----:|-----:|-------:|----------:|-----:|-------------:|
| (root) | 11 | 4 | 0 | 5 | 2 | 0 | 1 |
| Batchserver | 4 | 0 | 2 | 0 | 2 | 0 | 0 |
| CAD | 3 | 0 | 1 | 0 | 2 | 0 | 0 |
| CadBatchserver | 29 | 0 | 25 | 0 | 4 | 0 | 0 |
| CAM | 4 | 0 | 4 | 0 | 0 | 0 | 0 |
| Classes | 111 | 0 | 91 | 0 | 20 | 0 | 0 |
| Comparers | 4 | 0 | 4 | 0 | 0 | 0 | 0 |
| Controls | 114 | 0 | 42 | 0 | 72 | 0 | 0 |
| DataMigration | 50 | 0 | 50 | 0 | 0 | 0 | 0 |
| DesignComments | 10 | 0 | 4 | 0 | 6 | 0 | 0 |
| Elumatec | 157 | 31 | 87 | 0 | 34 | 0 | 5 |
| Engineering | 21 | 8 | 0 | 0 | 12 | 0 | 1 |
| Forms | 188 | 0 | 64 | 0 | 124 | 0 | 0 |
| IcImporter | 6 | 0 | 4 | 0 | 2 | 0 | 0 |
| Kardex | 4 | 0 | 2 | 0 | 2 | 0 | 0 |
| MarkTool | 5 | 0 | 3 | 0 | 2 | 0 | 0 |
| Modules | 3 | 1 | 2 | 0 | 0 | 0 | 0 |
| PCFNetStudio | 17 | 0 | 7 | 0 | 10 | 0 | 0 |
| Production | 6 | 6 | 0 | 0 | 0 | 0 | 0 |
| Resources | 354 | 0 | 0 | 354 | 0 | 0 | 0 |
| Sales | 3 | 1 | 0 | 0 | 2 | 0 | 0 |
| SmtManufacturing | 78 | 0 | 47 | 0 | 31 | 0 | 0 |
| SolaDataConnector | 2 | 0 | 2 | 0 | 0 | 0 | 0 |
| UniLink | 30 | 0 | 28 | 0 | 2 | 0 | 0 |
| VentDuctConfigurator | 4 | 0 | 2 | 0 | 2 | 0 | 0 |
| WebClock | 13 | 0 | 5 | 0 | 8 | 0 | 0 |
| WorkPreparation | 6 | 4 | 0 | 0 | 2 | 0 | 0 |
| iCENTER **TOTAL** | **1237** | **56** | **475** | **358** | **341** | **0** | **7** |
