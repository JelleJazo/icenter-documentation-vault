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
| `FrmMain.Designer.vb` | generated | VS Forms designer partial | — |
| `FrmMain.resx` | generated | resource bundle (designer-managed) | — |
| `FrmMain.vb` | needs-review | 15 216-line god-form; Phase 3 will split into multiple notes. Briefly characterized in [[architecture/entry-points]] + [[architecture/global-state]]. | — |
| `iCenter.vbproj` | done | covered in [[architecture/build-and-deploy]] + [[architecture/project-references]] | 2026-06-18 |
| `iCenter.vbproj.user` | config | per-user VS metadata | — |
| `iCenter.vbproj.vspscc` | config | source-control plugin metadata | — |
| `JAZO Comodo Code Signing Certificate.pfx` | config | signing key (Authenticode; expired 2023-07 per inline comment in vbproj) | — |
| `JAZO Zevenaar bv.snk` | config | strong-name signing key | — |
| `packages.config` | done | covered in [[architecture/build-and-deploy]] | 2026-06-18 |

### Batchserver

**Total: 4** &nbsp; | &nbsp; .vb: 3 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Batchserver\BatchserverToolkit.vb` | todo | — | — |
| `Batchserver\FrmBatchServer.Designer.vb` | generated | VS Forms designer partial | — |
| `Batchserver\FrmBatchServer.resx` | generated | resource bundle (designer-managed) | — |
| `Batchserver\FrmBatchServer.vb` | todo | — | — |

### CAD

**Total: 3** &nbsp; | &nbsp; .vb: 2 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `CAD\PLM\FrmPDMLinkOrdTypeContext.Designer.vb` | generated | VS Forms designer partial | — |
| `CAD\PLM\FrmPDMLinkOrdTypeContext.resx` | generated | resource bundle (designer-managed) | — |
| `CAD\PLM\FrmPDMLinkOrdTypeContext.vb` | todo | — | — |

### CadBatchserver

**Total: 29** &nbsp; | &nbsp; .vb: 27 | .resx: 2 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `CadBatchserver\BatchServerWatch.vb` | todo | — | — |
| `CadBatchserver\CadBatchserverTools.vb` | todo | — | — |
| `CadBatchserver\FormRunJob.Designer.vb` | generated | VS Forms designer partial | — |
| `CadBatchserver\FormRunJob.resx` | generated | resource bundle (designer-managed) | — |
| `CadBatchserver\FormRunJob.vb` | todo | — | — |
| `CadBatchserver\FrmCadBatchServer.Designer.vb` | generated | VS Forms designer partial | — |
| `CadBatchserver\FrmCadBatchServer.resx` | generated | resource bundle (designer-managed) | — |
| `CadBatchserver\FrmCadBatchServer.vb` | todo | — | — |
| `CadBatchserver\Job.vb` | todo | — | — |
| `CadBatchserver\JobArchive.vb` | todo | — | — |
| `CadBatchserver\JobAutoManufacturing.vb` | todo | — | — |
| `CadBatchserver\JobCreateProdOrd.vb` | todo | — | — |
| `CadBatchserver\JobCreatePurOrdDocs.vb` | todo | — | — |
| `CadBatchserver\JobEngOrdFinNotification.vb` | todo | — | — |
| `CadBatchserver\JobGenerateAndReleaseModel.vb` | todo | — | — |
| `CadBatchserver\JobGenericModelBackup.vb` | todo | — | — |
| `CadBatchserver\JobGeo2Dxf.vb` | todo | — | — |
| `CadBatchserver\JobModelGeneratorCreo.vb` | todo | — | — |
| `CadBatchserver\JobPartDispatch.vb` | todo | — | — |
| `CadBatchserver\JobProductionRegistration.vb` | todo | — | — |
| `CadBatchserver\JobPublishCreo.vb` | todo | — | — |
| `CadBatchserver\JobRebootMonitor.vb` | todo | — | — |
| `CadBatchserver\JobSendEmail.vb` | todo | — | — |
| `CadBatchserver\JobSmtOperSync.vb` | todo | — | — |
| `CadBatchserver\JobSmtPartDispatch.vb` | todo | — | — |
| `CadBatchserver\Modelgenerator\ModelgeneratorTask.vb` | todo | — | — |
| `CadBatchserver\Publisher\Trailfile.vb` | todo | — | — |
| `CadBatchserver\PublishWatchDirProcessor.vb` | todo | — | — |
| `CadBatchserver\ServerReboot.vb` | todo | — | — |

### CAM

**Total: 4** &nbsp; | &nbsp; .vb: 4 | .resx: 0 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `CAM\ManufMachine.vb` | todo | — | — |
| `CAM\ManufPart.vb` | todo | — | — |
| `CAM\ManufPartItem.vb` | todo | — | — |
| `CAM\ManufPartMember.vb` | todo | — | — |

### Classes

**Total: 111** &nbsp; | &nbsp; .vb: 101 | .resx: 10 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Classes\ApplicationLog.vb` | todo | — | — |
| `Classes\AutoDocAttach.vb` | todo | — | — |
| `Classes\AxControl.vb` | todo | — | — |
| `Classes\BriefcaseCollection.vb` | todo | — | — |
| `Classes\BriefcaseItem.vb` | todo | — | — |
| `Classes\ClsEmailEBTV.vb` | todo | — | — |
| `Classes\ClsPdfCommentLines.vb` | todo | — | — |
| `Classes\ClsProdObjects.vb` | todo | — | — |
| `Classes\ClsRecOrdNr.vb` | todo | — | — |
| `Classes\ClsXMLfile.vb` | todo | — | — |
| `Classes\Coating\Coating.vb` | todo | — | — |
| `Classes\Coating\CoatingPickLabel_v3.vb` | todo | — | — |
| `Classes\Coating\ControlCoating.Designer.vb` | generated | VS Forms designer partial | — |
| `Classes\Coating\ControlCoating.resx` | generated | resource bundle (designer-managed) | — |
| `Classes\Coating\ControlCoating.vb` | todo | — | — |
| `Classes\Coating\ControlCoatingGrouped.Designer.vb` | generated | VS Forms designer partial | — |
| `Classes\Coating\ControlCoatingGrouped.resx` | generated | resource bundle (designer-managed) | — |
| `Classes\Coating\ControlCoatingGrouped.vb` | todo | — | — |
| `Classes\Coating\CtrlCoatingPickLocation.Designer.vb` | generated | VS Forms designer partial | — |
| `Classes\Coating\CtrlCoatingPickLocation.resx` | generated | resource bundle (designer-managed) | — |
| `Classes\Coating\CtrlCoatingPickLocation.vb` | todo | — | — |
| `Classes\Coating\DataGridViewProgressColumn.vb` | todo | — | — |
| `Classes\Coating\DebugLog.vb` | todo | — | — |
| `Classes\Coating\EbtvSticker.vb` | todo | — | — |
| `Classes\Coating\FrmAddLayerThickness.Designer.vb` | generated | VS Forms designer partial | — |
| `Classes\Coating\FrmAddLayerThickness.resx` | generated | resource bundle (designer-managed) | — |
| `Classes\Coating\FrmAddLayerThickness.vb` | todo | — | — |
| `Classes\Coating\FrmCoatingLayerThickness.Designer.vb` | generated | VS Forms designer partial | — |
| `Classes\Coating\FrmCoatingLayerThickness.resx` | generated | resource bundle (designer-managed) | — |
| `Classes\Coating\FrmCoatingLayerThickness.vb` | todo | — | — |
| `Classes\Coating\FrmCoatingPick.Designer.vb` | generated | VS Forms designer partial | — |
| `Classes\Coating\FrmCoatingPick.resx` | generated | resource bundle (designer-managed) | — |
| `Classes\Coating\FrmCoatingPick.vb` | todo | — | — |
| `Classes\Coating\FrmGetCoatingMaterials.Designer.vb` | generated | VS Forms designer partial | — |
| `Classes\Coating\FrmGetCoatingMaterials.resx` | generated | resource bundle (designer-managed) | — |
| `Classes\Coating\FrmGetCoatingMaterials.vb` | todo | — | — |
| `Classes\Coating\FrmGetCoatingNextJob.Designer.vb` | generated | VS Forms designer partial | — |
| `Classes\Coating\FrmGetCoatingNextJob.resx` | generated | resource bundle (designer-managed) | — |
| `Classes\Coating\FrmGetCoatingNextJob.vb` | todo | — | — |
| `Classes\Coating\frmKardexJobIncomplete.designer.vb` | generated | VS Forms designer partial | — |
| `Classes\Coating\frmKardexJobIncomplete.resx` | generated | resource bundle (designer-managed) | — |
| `Classes\Coating\frmKardexJobIncomplete.vb` | todo | — | — |
| `Classes\Coating\frmScanNext.Designer.vb` | generated | VS Forms designer partial | — |
| `Classes\Coating\frmScanNext.resx` | generated | resource bundle (designer-managed) | — |
| `Classes\Coating\frmScanNext.vb` | todo | — | — |
| `Classes\Coating\IControlCoating.vb` | todo | — | — |
| `Classes\Connectivity\CalcExcelExport.vb` | todo | — | — |
| `Classes\Connectivity\ClsActiveDirectory.vb` | todo | — | — |
| `Classes\Connectivity\ClsICenter.vb` | todo | — | — |
| `Classes\Connectivity\ClsISAH.vb` | todo | — | — |
| `Classes\Connectivity\ClsJIBA.vb` | todo | — | — |
| `Classes\Connectivity\DeviceInfo.vb` | todo | — | — |
| `Classes\Connectivity\LabelWriter.vb` | todo | — | — |
| `Classes\Connectivity\NetworkPrinter.vb` | todo | — | — |
| `Classes\DocElementConverter.vb` | todo | — | — |
| `Classes\DossierDetail.vb` | todo | — | — |
| `Classes\DossierDocFolder.vb` | todo | — | — |
| `Classes\EngineeringOrders.vb` | todo | — | — |
| `Classes\ExtOperPart.vb` | todo | — | — |
| `Classes\ICenterDoc.vb` | todo | — | — |
| `Classes\ICenterObject.vb` | todo | — | — |
| `Classes\ICenterPartBasicXML.vb` | todo | — | — |
| `Classes\InvtOrd.vb` | todo | — | — |
| `Classes\IsahDoc.vb` | todo | — | — |
| `Classes\MachGrp.vb` | todo | — | — |
| `Classes\MachineFilter.vb` | todo | — | — |
| `Classes\ModelTree.vb` | todo | — | — |
| `Classes\OrdRefNrStatusUpdate.vb` | todo | — | — |
| `Classes\PartCalculation.vb` | todo | — | — |
| `Classes\PartOptimisationExcelExport.vb` | todo | — | — |
| `Classes\PreSelectMachGrpCodes\PreSelectMachGrpCode.vb` | todo | — | — |
| `Classes\PreSelectMachGrpCodes\PreSelectMachGrpCodeOrdType.vb` | todo | — | — |
| `Classes\PreSelectMachGrpCodes\PreSelectMachGrpCodeOrdTypeHandler.vb` | todo | — | — |
| `Classes\PreSelectMachGrpCodes\PreSelectMachGrpCodes.vb` | todo | — | — |
| `Classes\PreSelectMachGrpCodes\PreSelectMachGrpCodesHandler.vb` | todo | — | — |
| `Classes\ProcessWatch.vb` | todo | — | — |
| `Classes\Production\ClsIPbatch.vb` | todo | — | — |
| `Classes\Production\ClsIPorder.vb` | todo | — | — |
| `Classes\Production\ClsIPpacket.vb` | todo | — | — |
| `Classes\Production\ClsIPpart.vb` | todo | — | — |
| `Classes\Production\Icenter2IsahJob.vb` | todo | — | — |
| `Classes\Production\LeanJob.vb` | todo | — | — |
| `Classes\Production\LeanWorkTime.vb` | todo | — | — |
| `Classes\Production\MachGrpPerformance.vb` | todo | — | — |
| `Classes\Production\PartProgress.vb` | todo | — | — |
| `Classes\Production\ProdPlanView.vb` | todo | — | — |
| `Classes\Production\ProdStatus.vb` | todo | — | — |
| `Classes\Production\ProductionRegistration.vb` | todo | — | — |
| `Classes\Production\WorkView.vb` | todo | — | — |
| `Classes\PurOrd.vb` | todo | — | — |
| `Classes\RecurrenceCheck.vb` | todo | — | — |
| `Classes\StandardPartUpdater.vb` | todo | — | — |
| `Classes\StickersAndLabels\AltecLabel.vb` | todo | — | — |
| `Classes\StickersAndLabels\DymoLabelTest.vb` | todo | — | — |
| `Classes\StickersAndLabels\FG_ProductLabel.vb` | todo | — | — |
| `Classes\StickersAndLabels\InvtPartSticker.vb` | todo | — | — |
| `Classes\StickersAndLabels\KanbanBinLabel.vb` | todo | — | — |
| `Classes\StickersAndLabels\LeanBatchSticker.vb` | todo | — | — |
| `Classes\StickersAndLabels\LeanSticker_v1.vb` | todo | — | — |
| `Classes\StickersAndLabels\LeanStickerCopy.vb` | todo | — | — |
| `Classes\StickersAndLabels\OrdRefLabel.vb` | todo | — | — |
| `Classes\StickersAndLabels\PartIdent.vb` | todo | — | — |
| `Classes\StickersAndLabels\PartIdentProfMill.vb` | todo | — | — |
| `Classes\Toolbox\ApplicationHelper.vb` | todo | — | — |
| `Classes\Toolbox\ClipboardHelper.vb` | todo | — | — |
| `Classes\Toolbox\HelpHandler.vb` | todo | — | — |
| `Classes\Toolbox\JzWindow.vb` | todo | — | — |
| `Classes\Toolbox\RtfBuilder.vb` | todo | — | — |
| `Classes\Toolbox\SessionHelper.vb` | todo | — | — |
| `Classes\Toolbox\WindowHandler.vb` | todo | — | — |
| `Classes\Toolbox\Zip.vb` | todo | — | — |

### Comparers

**Total: 4** &nbsp; | &nbsp; .vb: 4 | .resx: 0 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Comparers\BewerkingsGroepComparer.vb` | todo | — | — |
| `Comparers\DefaultNodeSorter.vb` | todo | — | — |
| `Comparers\DPrintDocumentComparer.vb` | todo | — | — |
| `Comparers\JAZOTreeViewNodeSorter.vb` | todo | — | — |

### Controls

**Total: 114** &nbsp; | &nbsp; .vb: 78 | .resx: 36 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Controls\BomControl.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\BomControl.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\BomControl.vb` | todo | — | — |
| `Controls\ButtonExtended.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\ButtonExtended.vb` | todo | — | — |
| `Controls\CheckBoxExtended.vb` | todo | — | — |
| `Controls\CompBriefcaseButton.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CompBriefcaseButton.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CompBriefcaseButton.vb` | todo | — | — |
| `Controls\CreoViewControl.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CreoViewControl.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CreoViewControl.vb` | todo | — | — |
| `Controls\CtrlCeChecklistViewer.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CtrlCeChecklistViewer.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CtrlCeChecklistViewer.vb` | todo | — | — |
| `Controls\CtrlDocumentReplace.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CtrlDocumentReplace.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CtrlDocumentReplace.vb` | todo | — | — |
| `Controls\CtrlDosDesignCodes.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CtrlDosDesignCodes.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CtrlDosDesignCodes.vb` | todo | — | — |
| `Controls\CtrlDossierDetailProdDosCompare.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CtrlDossierDetailProdDosCompare.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CtrlDossierDetailProdDosCompare.vb` | todo | — | — |
| `Controls\CtrlDossierDetailProperties.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CtrlDossierDetailProperties.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CtrlDossierDetailProperties.vb` | todo | — | — |
| `Controls\CtrlDossierDetailText.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CtrlDossierDetailText.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CtrlDossierDetailText.vb` | todo | — | — |
| `Controls\CtrlEditOperations.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CtrlEditOperations.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CtrlEditOperations.vb` | todo | — | — |
| `Controls\CtrlExplorer.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CtrlExplorer.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CtrlExplorer.vb` | todo | — | — |
| `Controls\CtrlFgProductLabel.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CtrlFgProductLabel.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CtrlFgProductLabel.vb` | todo | — | — |
| `Controls\CtrlIpInfo.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CtrlIpInfo.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CtrlIpInfo.vb` | todo | — | — |
| `Controls\CtrlIsahContacts.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CtrlIsahContacts.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CtrlIsahContacts.vb` | todo | — | — |
| `Controls\CtrlModelgenerator.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CtrlModelgenerator.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CtrlModelgenerator.vb` | todo | — | — |
| `Controls\CtrlModelgenerators.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CtrlModelgenerators.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CtrlModelgenerators.vb` | todo | — | — |
| `Controls\CtrlOfficeClockFavorites.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CtrlOfficeClockFavorites.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CtrlOfficeClockFavorites.vb` | todo | — | — |
| `Controls\CtrlOfficeClockStatus.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CtrlOfficeClockStatus.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CtrlOfficeClockStatus.vb` | todo | — | — |
| `Controls\CtrlPhoneNr.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CtrlPhoneNr.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CtrlPhoneNr.vb` | todo | — | — |
| `Controls\CtrlProdChecklistViewer.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CtrlProdChecklistViewer.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CtrlProdChecklistViewer.vb` | todo | — | — |
| `Controls\CtrlProdRoutingDetail.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CtrlProdRoutingDetail.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CtrlProdRoutingDetail.vb` | todo | — | — |
| `Controls\CtrlProductConfiguration.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CtrlProductConfiguration.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CtrlProductConfiguration.vb` | todo | — | — |
| `Controls\CtrlSalesFavorites.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CtrlSalesFavorites.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CtrlSalesFavorites.vb` | todo | — | — |
| `Controls\CtrlTreeNodeDetails.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CtrlTreeNodeDetails.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CtrlTreeNodeDetails.vb` | todo | — | — |
| `Controls\CtrlTreeNodeProperties.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CtrlTreeNodeProperties.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CtrlTreeNodeProperties.vb` | todo | — | — |
| `Controls\CtrlWebClockAssistant.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\CtrlWebClockAssistant.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\CtrlWebClockAssistant.vb` | todo | — | — |
| `Controls\DxfView\DxfViewControl.designer.vb` | generated | VS Forms designer partial | — |
| `Controls\DxfView\DxfViewControl.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\DxfView\DxfViewControl.vb` | todo | — | — |
| `Controls\DxfView\DxfViewDisplayControl.designer.vb` | generated | VS Forms designer partial | — |
| `Controls\DxfView\DxfViewDisplayControl.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\DxfView\DxfViewDisplayControl.vb` | todo | — | — |
| `Controls\DxfView\PolygonWireframeGraphicsFactory.vb` | todo | — | — |
| `Controls\ExtendedDateTimePicker.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\ExtendedDateTimePicker.vb` | todo | — | — |
| `Controls\Isah\CtrlDosDetailExtra.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\Isah\CtrlDosDetailExtra.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\Isah\CtrlDosDetailExtra.vb` | todo | — | — |
| `Controls\Isah\DossierDetailExtraAuthorizationHelper.vb` | todo | — | — |
| `Controls\Isah\FrmDosDetailExtra.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\Isah\FrmDosDetailExtra.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\Isah\FrmDosDetailExtra.vb` | todo | — | — |
| `Controls\Isah\ICtrlDosDetailExtra.vb` | todo | — | — |
| `Controls\ModelOpers\ControlModelOper.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\ModelOpers\ControlModelOper.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\ModelOpers\ControlModelOper.vb` | todo | — | — |
| `Controls\ModelOpers\ControlModelOpers.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\ModelOpers\ControlModelOpers.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\ModelOpers\ControlModelOpers.vb` | todo | — | — |
| `Controls\MSVistaPBar.vb` | todo | — | — |
| `Controls\RichTextBoxEditor.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\RichTextBoxEditor.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\RichTextBoxEditor.vb` | todo | — | — |
| `Controls\TabPageExtended.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\TabPageExtended.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\TabPageExtended.vb` | todo | — | — |
| `Controls\UCKanbanPart.Designer.vb` | generated | VS Forms designer partial | — |
| `Controls\UCKanbanPart.resx` | generated | resource bundle (designer-managed) | — |
| `Controls\UCKanbanPart.vb` | todo | — | — |

### DataMigration

**Total: 50** &nbsp; | &nbsp; .vb: 50 | .resx: 0 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `DataMigration\BasicDataHandler.vb` | todo | — | — |
| `DataMigration\BasicMigrationHandler.vb` | todo | — | — |
| `DataMigration\CacheHandler.vb` | todo | — | — |
| `DataMigration\ContextMenuHelper.vb` | todo | — | — |
| `DataMigration\DataBaseConnection.vb` | todo | — | — |
| `DataMigration\DataBaseConnectionHandler.vb` | todo | — | — |
| `DataMigration\DataHandlerToolbox.vb` | todo | — | — |
| `DataMigration\DossierItemMigrationHandler.vb` | todo | — | — |
| `DataMigration\DossierMigrationHandler.vb` | todo | — | — |
| `DataMigration\Entities\AuditableEntity.vb` | todo | — | — |
| `DataMigration\Entities\BillOfMaterialItem.vb` | todo | — | — |
| `DataMigration\Entities\BillOfOperation.vb` | todo | — | — |
| `DataMigration\Entities\BillOfOperationItem.vb` | todo | — | — |
| `DataMigration\Entities\Document.vb` | todo | — | — |
| `DataMigration\Entities\Dossier.vb` | todo | — | — |
| `DataMigration\Entities\DossierItem.vb` | todo | — | — |
| `DataMigration\Entities\IEntity.vb` | todo | — | — |
| `DataMigration\Entities\Material.vb` | todo | — | — |
| `DataMigration\Entities\Operation.vb` | todo | — | — |
| `DataMigration\Entities\Part.vb` | todo | — | — |
| `DataMigration\Entities\ProductionDossier.vb` | todo | — | — |
| `DataMigration\Entities\Project.vb` | todo | — | — |
| `DataMigration\Entities\User.vb` | todo | — | — |
| `DataMigration\Features\BillOfMaterialItemFactory.vb` | todo | — | — |
| `DataMigration\Features\BillOfMaterialItemHandler.vb` | todo | — | — |
| `DataMigration\Features\BillOfOperationItemFactory.vb` | todo | — | — |
| `DataMigration\Features\BillOfOperationItemHandler.vb` | todo | — | — |
| `DataMigration\Features\DocumentFactory.vb` | todo | — | — |
| `DataMigration\Features\DocumentHandler.vb` | todo | — | — |
| `DataMigration\Features\DossierFactory.vb` | todo | — | — |
| `DataMigration\Features\DossierHandler.vb` | todo | — | — |
| `DataMigration\Features\DossierItemFactory.vb` | todo | — | — |
| `DataMigration\Features\DossierItemHandler.vb` | todo | — | — |
| `DataMigration\Features\MaterialFactory.vb` | todo | — | — |
| `DataMigration\Features\MaterialHandler.vb` | todo | — | — |
| `DataMigration\Features\OperationFactory.vb` | todo | — | — |
| `DataMigration\Features\OperationHandler.vb` | todo | — | — |
| `DataMigration\Features\PartFactory.vb` | todo | — | — |
| `DataMigration\Features\PartHandler.vb` | todo | — | — |
| `DataMigration\Features\ProductionDossierFactory.vb` | todo | — | — |
| `DataMigration\Features\ProductionDossierHandler.vb` | todo | — | — |
| `DataMigration\Features\ProjectFactory.vb` | todo | — | — |
| `DataMigration\Features\ProjectHandler.vb` | todo | — | — |
| `DataMigration\Features\UserFactory.vb` | todo | — | — |
| `DataMigration\Features\UserHandler.vb` | todo | — | — |
| `DataMigration\IMigrationHandler.vb` | todo | — | — |
| `DataMigration\MockDataHandler.vb` | todo | — | — |
| `DataMigration\PartMigrationHandler.vb` | todo | — | — |
| `DataMigration\ProductionDossierMigrationHandler.vb` | todo | — | — |
| `DataMigration\UserMigrationHandler.vb` | todo | — | — |

### DesignComments

**Total: 10** &nbsp; | &nbsp; .vb: 7 | .resx: 3 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `DesignComments\ClsDesignComments.vb` | todo | — | — |
| `DesignComments\dgCommentLines.designer.vb` | generated | VS Forms designer partial | — |
| `DesignComments\dgCommentLines.resx` | generated | resource bundle (designer-managed) | — |
| `DesignComments\dgCommentLines.vb` | todo | — | — |
| `DesignComments\frmCommentLine.designer.vb` | generated | VS Forms designer partial | — |
| `DesignComments\frmCommentLine.resx` | generated | resource bundle (designer-managed) | — |
| `DesignComments\frmCommentLine.vb` | todo | — | — |
| `DesignComments\FrmDesigncomments.designer.vb` | generated | VS Forms designer partial | — |
| `DesignComments\FrmDesigncomments.resx` | generated | resource bundle (designer-managed) | — |
| `DesignComments\FrmDesigncomments.vb` | todo | — | — |

### Elumatec

**Total: 157** &nbsp; | &nbsp; .vb: 140 | .resx: 17 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Elumatec\AppVersion.vb` | todo | — | — |
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
| `Elumatec\AutoProfMillProgApproval.vb` | todo | — | — |
| `Elumatec\ClsComWatcher.vb` | done | [[modules/elumatec-com-watcher]] | 2026-06-18 |
| `Elumatec\ClsDgxShoppingList.vb` | todo | — | — |
| `Elumatec\ClsDgxStickerPrinter.vb` | todo | — | — |
| `Elumatec\ClsEluLanguage.vb` | todo | — | — |
| `Elumatec\ClsSawList.vb` | todo | — | — |
| `Elumatec\ControlProfSaw.Designer.vb` | generated | VS Forms designer partial | — |
| `Elumatec\ControlProfSaw.resx` | generated | resource bundle (designer-managed) | — |
| `Elumatec\ControlProfSaw.vb` | todo | — | — |
| `Elumatec\CRs232.vb` | done | third-party (Corrado Cavalli ©2003); [[modules/elumatec-com-watcher]] | 2026-06-18 |
| `Elumatec\CtrlEluOpenGLViewer.Designer.vb` | generated | VS Forms designer partial | — |
| `Elumatec\CtrlEluOpenGLViewer.resx` | generated | resource bundle (designer-managed) | — |
| `Elumatec\CtrlEluOpenGLViewer.vb` | todo | — | — |
| `Elumatec\CtrlProfMillCam.Designer.vb` | generated | VS Forms designer partial | — |
| `Elumatec\CtrlProfMillCam.resx` | generated | resource bundle (designer-managed) | — |
| `Elumatec\CtrlProfMillCam.vb` | todo | — | — |
| `Elumatec\CtrlProfMillElu.Designer.vb` | generated | VS Forms designer partial | — |
| `Elumatec\CtrlProfMillElu.resx` | generated | resource bundle (designer-managed) | — |
| `Elumatec\CtrlProfMillElu.vb` | todo | — | — |
| `Elumatec\CutFactory.vb` | todo | — | — |
| `Elumatec\CycleTime.vb` | todo | — | — |
| `Elumatec\Database\Fixture.vb` | todo | — | — |
| `Elumatec\Database\FixtureCollection.vb` | todo | — | — |
| `Elumatec\Database\Offset.vb` | todo | — | — |
| `Elumatec\Database\OffsetCollection.vb` | todo | — | — |
| `Elumatec\Database\OffsetFile.vb` | todo | — | — |
| `Elumatec\Database\Offsets.vb` | todo | — | — |
| `Elumatec\Database\Profile.vb` | todo | — | — |
| `Elumatec\Database\ProfileExportHandler.vb` | todo | — | — |
| `Elumatec\Database\ProfileMachineSetting.vb` | todo | — | — |
| `Elumatec\Database\ToolDbSimplified.vb` | todo | — | — |
| `Elumatec\DXF\EluDxf.vb` | todo | — | — |
| `Elumatec\DXF\EluDxfEntity.vb` | todo | — | — |
| `Elumatec\DXF\EluDxfPolyline.vb` | todo | — | — |
| `Elumatec\DXF\EluDxfVertex.vb` | todo | — | — |
| `Elumatec\EluCadApp.vb` | needs-review | overview only in [[modules/elumatec-cad-app]]; per-cluster sub-notes pending | — |
| `Elumatec\EluCadSetting.vb` | todo | — | — |
| `Elumatec\FileFormat.vb` | todo | — | — |
| `Elumatec\FrmComWatcher.Designer.vb` | generated | VS Forms designer partial | — |
| `Elumatec\FrmComWatcher.resx` | generated | resource bundle (designer-managed) | — |
| `Elumatec\FrmComWatcher.vb` | done | [[modules/elumatec-com-watcher]] | 2026-06-18 |
| `Elumatec\frmDgxStack.Designer.vb` | generated | VS Forms designer partial | — |
| `Elumatec\frmDgxStack.resx` | generated | resource bundle (designer-managed) | — |
| `Elumatec\frmDgxStack.vb` | todo | — | — |
| `Elumatec\FrmEluMissingProfile.Designer.vb` | generated | VS Forms designer partial | — |
| `Elumatec\FrmEluMissingProfile.resx` | generated | resource bundle (designer-managed) | — |
| `Elumatec\FrmEluMissingProfile.vb` | todo | — | — |
| `Elumatec\frmExportDgx.Designer.vb` | generated | VS Forms designer partial | — |
| `Elumatec\frmExportDgx.resx` | generated | resource bundle (designer-managed) | — |
| `Elumatec\frmExportDgx.vb` | todo | — | — |
| `Elumatec\frmManualProfile.Designer.vb` | generated | VS Forms designer partial | — |
| `Elumatec\frmManualProfile.resx` | generated | resource bundle (designer-managed) | — |
| `Elumatec\frmManualProfile.vb` | todo | — | — |
| `Elumatec\FrmNcxErrors.Designer.vb` | generated | VS Forms designer partial | — |
| `Elumatec\FrmNcxErrors.resx` | generated | resource bundle (designer-managed) | — |
| `Elumatec\FrmNcxErrors.vb` | todo | — | — |
| `Elumatec\FrmProfileView.Designer.vb` | generated | VS Forms designer partial | — |
| `Elumatec\FrmProfileView.resx` | generated | resource bundle (designer-managed) | — |
| `Elumatec\FrmProfileView.vb` | todo | — | — |
| `Elumatec\frmSawQtyDone.Designer.vb` | generated | VS Forms designer partial | — |
| `Elumatec\frmSawQtyDone.resx` | generated | resource bundle (designer-managed) | — |
| `Elumatec\frmSawQtyDone.vb` | todo | — | — |
| `Elumatec\frmSawQtyDoneExt.Designer.vb` | generated | VS Forms designer partial | — |
| `Elumatec\frmSawQtyDoneExt.resx` | generated | resource bundle (designer-managed) | — |
| `Elumatec\frmSawQtyDoneExt.vb` | todo | — | — |
| `Elumatec\LicenseHelper.vb` | todo | — | — |
| `Elumatec\LicenseManagementCenter.vb` | todo | — | — |
| `Elumatec\Machine\Sbz140Alu.vb` | done | [[modules/elumatec-machine-base]] | 2026-06-18 |
| `Elumatec\Machine\Sbz140Rvs.vb` | needs-review | [[modules/elumatec-machine-base]] (values not transcribed; Q-027) | — |
| `Elumatec\Machine\Sbz140Stl.vb` | needs-review | [[modules/elumatec-machine-base]] (values not transcribed; Q-027) | — |
| `Elumatec\Machine\Sbz141Alu.vb` | needs-review | [[modules/elumatec-machine-base]] (values not transcribed) | — |
| `Elumatec\Machine\Sbz14x.vb` | done | [[modules/elumatec-machine-base]] | 2026-06-18 |
| `Elumatec\MacroDatabase.vb` | todo | — | — |
| `Elumatec\NcStructure\Bar.vb` | done | [[modules/elumatec-ncstructure-hierarchy]] | 2026-06-18 |
| `Elumatec\NcStructure\Cut.vb` | done | [[modules/elumatec-ncstructure-hierarchy]] | 2026-06-18 |
| `Elumatec\NcStructure\EluCadFile.vb` | done | [[modules/elumatec-elucadfile]] | 2026-06-18 |
| `Elumatec\NcStructure\Job.vb` | done | [[modules/elumatec-ncstructure-hierarchy]] | 2026-06-18 |
| `Elumatec\NcStructure\Plane.vb` | done | [[modules/elumatec-ncstructure-hierarchy]] | 2026-06-18 |
| `Elumatec\NcStructure\PlaneCollection.vb` | done | [[modules/elumatec-ncstructure-hierarchy]] | 2026-06-18 |
| `Elumatec\NcVersionHandler.vb` | todo | — | — |
| `Elumatec\NcwExportProfile.vb` | todo | — | — |
| `Elumatec\NcwViewer.vb` | todo | — | — |
| `Elumatec\NcxContainer.vb` | todo | — | — |
| `Elumatec\NumberLib.vb` | todo | — | — |
| `Elumatec\Optimizer\CutOptimizer.vb` | todo | — | — |
| `Elumatec\Optimizer\frmSawJobOptimizer.Designer.vb` | generated | VS Forms designer partial | — |
| `Elumatec\Optimizer\frmSawJobOptimizer.resx` | generated | resource bundle (designer-managed) | — |
| `Elumatec\Optimizer\frmSawJobOptimizer.vb` | todo | — | — |
| `Elumatec\ProfileMatcher.vb` | todo | — | — |
| `Elumatec\ProfMillConverter.vb` | done | [[modules/elumatec-profmill-converter]] | 2026-06-18 |
| `Elumatec\ProfMillJob.vb` | done | [[modules/elumatec-profmill-job]] | 2026-06-18 |
| `Elumatec\ReferenceDxf.vb` | todo | — | — |
| `Elumatec\SawListReport.vb` | todo | — | — |
| `Elumatec\SectionCutOffBox.vb` | todo | — | — |
| `Elumatec\UCDgxWorksheet.Designer.vb` | generated | VS Forms designer partial | — |
| `Elumatec\UCDgxWorksheet.resx` | generated | resource bundle (designer-managed) | — |
| `Elumatec\UCDgxWorksheet.vb` | todo | — | — |
| `Elumatec\Workpiece.vb` | todo | — | — |
| `Elumatec\Works\Circle.vb` | todo | — | — |
| `Elumatec\Works\Deburr.vb` | todo | — | — |
| `Elumatec\Works\Drill.vb` | todo | — | — |
| `Elumatec\Works\DxfFreeForm.vb` | todo | — | — |
| `Elumatec\Works\FreeForm.vb` | todo | — | — |
| `Elumatec\Works\FreeFormPoint.vb` | todo | — | — |
| `Elumatec\Works\FrmTestDrwProfile.Designer.vb` | generated | VS Forms designer partial | — |
| `Elumatec\Works\FrmTestDrwProfile.resx` | generated | resource bundle (designer-managed) | — |
| `Elumatec\Works\FrmTestDrwProfile.vb` | todo | — | — |
| `Elumatec\Works\Group.vb` | todo | — | — |
| `Elumatec\Works\Line.vb` | todo | — | — |
| `Elumatec\Works\Macro.vb` | todo | — | — |
| `Elumatec\Works\Rectangle.vb` | todo | — | — |
| `Elumatec\Works\Replacements\AluGeneral.vb` | needs-review | [[modules/elumatec-replacement-alu-general]] (overview only; per-branch notes pending) | — |
| `Elumatec\Works\Replacements\AluHinge.vb` | todo | — | — |
| `Elumatec\Works\Replacements\AluHUPO.vb` | todo | — | — |
| `Elumatec\Works\Replacements\AluSinglePnotch.vb` | todo | — | — |
| `Elumatec\Works\Replacements\AluSRkom.vb` | todo | — | — |
| `Elumatec\Works\Replacements\DoorPlankCalibration.vb` | todo | — | — |
| `Elumatec\Works\Replacements\DoorPlankCalibrationMessageCutOff.vb` | todo | — | — |
| `Elumatec\Works\Replacements\DoublePnotch.vb` | todo | — | — |
| `Elumatec\Works\Replacements\ExtraLength.vb` | todo | — | — |
| `Elumatec\Works\Replacements\ExtraLengthMacro.vb` | todo | — | — |
| `Elumatec\Works\Replacements\ExtraLengthMacroFactory.vb` | todo | — | — |
| `Elumatec\Works\Replacements\Flowdrill.vb` | done | [[modules/elumatec-replacement-flowdrill]] | 2026-06-18 |
| `Elumatec\Works\Replacements\IDoorPlankCalibrationMessage.vb` | todo | — | — |
| `Elumatec\Works\Replacements\LargeRectangle.vb` | done | [[modules/elumatec-replacement-large-rectangle]] | 2026-06-18 |
| `Elumatec\Works\Replacements\OpdekH.vb` | todo | — | — |
| `Elumatec\Works\Replacements\RDHS27Notch.vb` | todo | — | — |
| `Elumatec\Works\Replacements\RuntimeManipulation.vb` | todo | — | — |
| `Elumatec\Works\Replacements\RuntimeManipulationInstruction.vb` | todo | — | — |
| `Elumatec\Works\Replacements\RuntimeManipulations.vb` | todo | — | — |
| `Elumatec\Works\Replacements\StlDoublePnotch.vb` | todo | — | — |
| `Elumatec\Works\Replacements\StlFlowDrill.vb` | todo | — | — |
| `Elumatec\Works\Replacements\StlGeneral.vb` | todo | — | — |
| `Elumatec\Works\Replacements\StlHinge.vb` | todo | — | — |
| `Elumatec\Works\Replacements\UCDoorPlankCalibration.Designer.vb` | generated | VS Forms designer partial | — |
| `Elumatec\Works\Replacements\UCDoorPlankCalibration.resx` | generated | resource bundle (designer-managed) | — |
| `Elumatec\Works\Replacements\UCDoorPlankCalibration.vb` | todo | — | — |
| `Elumatec\Works\Replacements\WorksReplacement.vb` | done | [[modules/elumatec-works-replacement-base]] | 2026-06-18 |
| `Elumatec\Works\Replacements\WorksTranslation.vb` | todo | — | — |
| `Elumatec\Works\Sawcut.vb` | todo | — | — |
| `Elumatec\Works\SlottedHole.vb` | todo | — | — |
| `Elumatec\Works\Work.vb` | done | [[modules/elumatec-work-base]] | 2026-06-18 |

### Engineering

**Total: 21** &nbsp; | &nbsp; .vb: 15 | .resx: 6 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Engineering\FrmDesignCodeTool.designer.vb` | generated | VS Forms designer partial | — |
| `Engineering\FrmDesignCodeTool.resx` | generated | resource bundle (designer-managed) | — |
| `Engineering\FrmDesignCodeTool.vb` | todo | — | — |
| `Engineering\FrmDrwCheck.Designer.vb` | generated | VS Forms designer partial | — |
| `Engineering\FrmDrwCheck.resx` | generated | resource bundle (designer-managed) | — |
| `Engineering\FrmDrwCheck.vb` | todo | — | — |
| `Engineering\frmEngGeneratedProductOverview.Designer.vb` | generated | VS Forms designer partial | — |
| `Engineering\frmEngGeneratedProductOverview.resx` | generated | resource bundle (designer-managed) | — |
| `Engineering\frmEngGeneratedProductOverview.vb` | todo | — | — |
| `Engineering\frmGenericStatus.Designer.vb` | generated | VS Forms designer partial | — |
| `Engineering\frmGenericStatus.resx` | generated | resource bundle (designer-managed) | — |
| `Engineering\frmGenericStatus.vb` | todo | — | — |
| `Engineering\FrmOrdersAsBuilt.designer.vb` | generated | VS Forms designer partial | — |
| `Engineering\FrmOrdersAsBuilt.resx` | generated | resource bundle (designer-managed) | — |
| `Engineering\FrmOrdersAsBuilt.vb` | todo | — | — |
| `Engineering\frmSelectAnnotGenericStatus.Designer.vb` | generated | VS Forms designer partial | — |
| `Engineering\frmSelectAnnotGenericStatus.resx` | generated | resource bundle (designer-managed) | — |
| `Engineering\frmSelectAnnotGenericStatus.vb` | todo | — | — |
| `Engineering\ModelCopies\CopyLocalizer.vb` | todo | — | — |
| `Engineering\ModelCopies\GenericSmtPartFinder.vb` | todo | — | — |
| `Engineering\ModelCopies\UitsparingVoorplaatMeerpslAlu.vb` | todo | — | — |

### Forms

**Total: 188** &nbsp; | &nbsp; .vb: 127 | .resx: 61 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Forms\FrmProdChecklistViewer.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\FrmProdChecklistViewer.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\FrmProdChecklistViewer.vb` | todo | — | — |
| `Forms\FrmProdObjects.designer.vb` | generated | VS Forms designer partial | — |
| `Forms\FrmProdObjects.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\FrmProdObjects.vb` | todo | — | — |
| `Forms\FrmProdObjectsSelectCoating.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\FrmProdObjectsSelectCoating.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\FrmProdObjectsSelectCoating.vb` | todo | — | — |
| `Forms\ISAH\FrmPartBrowse.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\ISAH\FrmPartBrowse.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\ISAH\FrmPartBrowse.vb` | todo | — | — |
| `Forms\Management\ClsApplRevisions.vb` | todo | — | — |
| `Forms\Management\frmAddLeanProdTraject.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Management\frmAddLeanProdTraject.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Management\frmAddLeanProdTraject.vb` | todo | — | — |
| `Forms\Management\frmAdminActivePdfMarkups.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Management\frmAdminActivePdfMarkups.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Management\frmAdminActivePdfMarkups.vb` | todo | — | — |
| `Forms\Management\FrmApplNewRelease.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Management\FrmApplNewRelease.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Management\FrmApplNewRelease.vb` | todo | — | — |
| `Forms\Management\FrmApplRevisions.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Management\FrmApplRevisions.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Management\FrmApplRevisions.vb` | todo | — | — |
| `Forms\Management\frmCoatManagement.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Management\frmCoatManagement.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Management\frmCoatManagement.vb` | todo | — | — |
| `Forms\Management\frmCoatManagementAdd.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Management\frmCoatManagementAdd.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Management\frmCoatManagementAdd.vb` | todo | — | — |
| `Forms\Management\frmColors.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Management\frmColors.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Management\frmColors.vb` | todo | — | — |
| `Forms\Management\frmConvertDocElement.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Management\frmConvertDocElement.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Management\frmConvertDocElement.vb` | todo | — | — |
| `Forms\Management\frmEditCadBatchserverRules.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Management\frmEditCadBatchserverRules.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Management\frmEditCadBatchserverRules.vb` | todo | — | — |
| `Forms\Management\frmEditCapacityTickets.designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Management\frmEditCapacityTickets.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Management\frmEditCapacityTickets.vb` | todo | — | — |
| `Forms\Management\frmEditCoatingDefPrimer.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Management\frmEditCoatingDefPrimer.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Management\frmEditCoatingDefPrimer.vb` | todo | — | — |
| `Forms\Management\frmEditKanbanBin.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Management\frmEditKanbanBin.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Management\frmEditKanbanBin.vb` | todo | — | — |
| `Forms\Management\FrmEditLeanMachines.designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Management\FrmEditLeanMachines.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Management\FrmEditLeanMachines.vb` | todo | — | — |
| `Forms\Management\frmEditMachGrps.designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Management\frmEditMachGrps.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Management\frmEditMachGrps.vb` | todo | — | — |
| `Forms\Management\FrmEditTable.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Management\FrmEditTable.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Management\FrmEditTable.vb` | todo | — | — |
| `Forms\Management\frmExtractIconFromFile.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Management\frmExtractIconFromFile.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Management\frmExtractIconFromFile.vb` | todo | — | — |
| `Forms\Management\frmFeedbackUpdate.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Management\frmFeedbackUpdate.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Management\frmFeedbackUpdate.vb` | todo | — | — |
| `Forms\Management\frmGenericsAdmin.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Management\frmGenericsAdmin.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Management\frmGenericsAdmin.vb` | todo | — | — |
| `Forms\Management\FrmHelp.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Management\FrmHelp.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Management\FrmHelp.vb` | todo | — | — |
| `Forms\Management\FrmUpdateProdTrackInWorkView.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Management\FrmUpdateProdTrackInWorkView.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Management\FrmUpdateProdTrackInWorkView.vb` | todo | — | — |
| `Forms\ShopProcess\FrmDgxPickJob.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\ShopProcess\FrmDgxPickJob.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\ShopProcess\FrmDgxPickJob.vb` | todo | — | — |
| `Forms\ShopProcess\FrmLeanAdmin.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\ShopProcess\FrmLeanAdmin.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\ShopProcess\FrmLeanAdmin.vb` | todo | — | — |
| `Forms\ShopProcess\FrmLeanDashboard.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\ShopProcess\FrmLeanDashboard.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\ShopProcess\FrmLeanDashboard.vb` | todo | — | — |
| `Forms\ShopProcess\FrmLeanFlowChart.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\ShopProcess\FrmLeanFlowChart.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\ShopProcess\FrmLeanFlowChart.vb` | todo | — | — |
| `Forms\ShopProcess\FrmLeanStatus.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\ShopProcess\FrmLeanStatus.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\ShopProcess\FrmLeanStatus.vb` | todo | — | — |
| `Forms\ShopProcess\FrmPartDispatchCollector.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\ShopProcess\FrmPartDispatchCollector.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\ShopProcess\FrmPartDispatchCollector.vb` | todo | — | — |
| `Forms\ShopProcess\FrmPartPick.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\ShopProcess\FrmPartPick.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\ShopProcess\FrmPartPick.vb` | todo | — | — |
| `Forms\ShopProcess\FrmWorkChange.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\ShopProcess\FrmWorkChange.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\ShopProcess\FrmWorkChange.vb` | todo | — | — |
| `Forms\ShopProcess\FrmWorkViewWithStatus.designer.vb` | generated | VS Forms designer partial | — |
| `Forms\ShopProcess\FrmWorkViewWithStatus.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\ShopProcess\FrmWorkViewWithStatus.vb` | todo | — | — |
| `Forms\ShopProcess\FrmWorkViewWithStatusJCOA.designer.vb` | generated | VS Forms designer partial | — |
| `Forms\ShopProcess\FrmWorkViewWithStatusJCOA.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\ShopProcess\FrmWorkViewWithStatusJCOA.vb` | todo | — | — |
| `Forms\Template\AboutBox1.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Template\AboutBox1.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Template\AboutBox1.vb` | todo | — | — |
| `Forms\Template\FrmBulkRename.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Template\FrmBulkRename.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Template\FrmBulkRename.vb` | todo | — | — |
| `Forms\Template\FrmFadingMsgBox.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Template\FrmFadingMsgBox.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Template\FrmFadingMsgBox.vb` | todo | — | — |
| `Forms\Template\frmHappyNewYear.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Template\frmHappyNewYear.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Template\frmHappyNewYear.vb` | todo | — | — |
| `Forms\Template\FrmPrintPdf.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Template\FrmPrintPdf.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Template\FrmPrintPdf.vb` | todo | — | — |
| `Forms\Template\FrmTextBox.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Template\FrmTextBox.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Template\FrmTextBox.vb` | todo | — | — |
| `Forms\Template\LoginForm1.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Template\LoginForm1.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Template\LoginForm1.vb` | todo | — | — |
| `Forms\Template\SelectFromListDialog.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Template\SelectFromListDialog.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Template\SelectFromListDialog.vb` | todo | — | — |
| `Forms\Test3DSpace_Form1.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Test3DSpace_Form1.vb` | todo | — | — |
| `Forms\Test3DSpace_Form2.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Test3DSpace_Form2.vb` | todo | — | — |
| `Forms\Toolbox\FrmAddICenterDoc.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Toolbox\FrmAddICenterDoc.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Toolbox\FrmAddICenterDoc.vb` | todo | — | — |
| `Forms\Toolbox\FrmBatchProdOrd.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Toolbox\FrmBatchProdOrd.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Toolbox\FrmBatchProdOrd.vb` | todo | — | — |
| `Forms\Toolbox\FrmBomMember.designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Toolbox\FrmBomMember.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Toolbox\FrmBomMember.vb` | todo | — | — |
| `Forms\Toolbox\FrmCadLicensesStatus.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Toolbox\FrmCadLicensesStatus.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Toolbox\FrmCadLicensesStatus.vb` | todo | — | — |
| `Forms\Toolbox\FrmComPortScanner.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Toolbox\FrmComPortScanner.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Toolbox\FrmComPortScanner.vb` | todo | — | — |
| `Forms\Toolbox\FrmContact.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Toolbox\FrmContact.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Toolbox\FrmContact.vb` | todo | — | — |
| `Forms\Toolbox\FrmCrystalReport.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Toolbox\FrmCrystalReport.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Toolbox\FrmCrystalReport.vb` | todo | — | — |
| `Forms\Toolbox\FrmDossierFromPurOrd.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Toolbox\FrmDossierFromPurOrd.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Toolbox\FrmDossierFromPurOrd.vb` | todo | — | — |
| `Forms\Toolbox\frmGetValidPartcode.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Toolbox\frmGetValidPartcode.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Toolbox\frmGetValidPartcode.vb` | todo | — | — |
| `Forms\Toolbox\frmJumpToOrdNr.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Toolbox\frmJumpToOrdNr.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Toolbox\frmJumpToOrdNr.vb` | todo | — | — |
| `Forms\Toolbox\FrmLinkIsahToIPO.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Toolbox\FrmLinkIsahToIPO.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Toolbox\FrmLinkIsahToIPO.vb` | todo | — | — |
| `Forms\Toolbox\FrmModelGenerator.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Toolbox\FrmModelGenerator.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Toolbox\FrmModelGenerator.vb` | todo | — | — |
| `Forms\Toolbox\FrmNewDossierPos.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Toolbox\FrmNewDossierPos.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Toolbox\FrmNewDossierPos.vb` | todo | — | — |
| `Forms\Toolbox\FrmNewICenterObject.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Toolbox\FrmNewICenterObject.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Toolbox\FrmNewICenterObject.vb` | todo | — | — |
| `Forms\Toolbox\FrmPartCalculationUpdate.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Toolbox\FrmPartCalculationUpdate.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Toolbox\FrmPartCalculationUpdate.vb` | todo | — | — |
| `Forms\Toolbox\FrmPosGenerator.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Toolbox\FrmPosGenerator.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Toolbox\FrmPosGenerator.vb` | todo | — | — |
| `Forms\Toolbox\FrmPrintCeSticker.designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Toolbox\FrmPrintCeSticker.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Toolbox\FrmPrintCeSticker.vb` | todo | — | — |
| `Forms\Toolbox\frmQuickview.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Toolbox\frmQuickview.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Toolbox\frmQuickview.vb` | todo | — | — |
| `Forms\Toolbox\frmSelectProject.Designer.vb` | generated | VS Forms designer partial | — |
| `Forms\Toolbox\frmSelectProject.resx` | generated | resource bundle (designer-managed) | — |
| `Forms\Toolbox\frmSelectProject.vb` | todo | — | — |

### IcImporter

**Total: 6** &nbsp; | &nbsp; .vb: 5 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `IcImporter\FrmIcImport.Designer.vb` | generated | VS Forms designer partial | — |
| `IcImporter\FrmIcImport.resx` | generated | resource bundle (designer-managed) | — |
| `IcImporter\FrmIcImport.vb` | todo | — | — |
| `IcImporter\ICenterPart.vb` | todo | — | — |
| `IcImporter\IcImportHandler.vb` | todo | — | — |
| `IcImporter\IcImportStatus.vb` | todo | — | — |

### Kardex

**Total: 4** &nbsp; | &nbsp; .vb: 3 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Kardex\FrmKardexInterface.designer.vb` | generated | VS Forms designer partial | — |
| `Kardex\FrmKardexInterface.resx` | generated | resource bundle (designer-managed) | — |
| `Kardex\FrmKardexInterface.vb` | todo | — | — |
| `Kardex\KardexProcessor.vb` | todo | — | — |

### MarkTool

**Total: 5** &nbsp; | &nbsp; .vb: 4 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `MarkTool\ClsComPort.vb` | todo | — | — |
| `MarkTool\ClsMarkToolDb.vb` | todo | — | — |
| `MarkTool\frmMarkTool.Designer.vb` | generated | VS Forms designer partial | — |
| `MarkTool\frmMarkTool.resx` | generated | resource bundle (designer-managed) | — |
| `MarkTool\frmMarkTool.vb` | todo | — | — |

### Modules

**Total: 3** &nbsp; | &nbsp; .vb: 3 | .resx: 0 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Modules\Functions.vb` | todo | (referenced from [[modules/main-module]] for `GetApplicationArguments`, `UpdateIcenter`, `GetXMLWriteAccess`, `InitProfMillMachGrps`, etc. — Phase 3) | — |
| `Modules\MailMessageExt.vb` | todo | — | — |
| `Modules\Main.vb` | done | [[modules/main-module]] | 2026-06-18 |

### PCFNetStudio

**Total: 17** &nbsp; | &nbsp; .vb: 12 | .resx: 5 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `PCFNetStudio\CPartBuilder.vb` | todo | — | — |
| `PCFNetStudio\FrmCodeConverter.Designer.vb` | generated | VS Forms designer partial | — |
| `PCFNetStudio\FrmCodeConverter.resx` | generated | resource bundle (designer-managed) | — |
| `PCFNetStudio\FrmCodeConverter.vb` | todo | — | — |
| `PCFNetStudio\FrmProductComparer.Designer.vb` | generated | VS Forms designer partial | — |
| `PCFNetStudio\FrmProductComparer.resx` | generated | resource bundle (designer-managed) | — |
| `PCFNetStudio\FrmProductComparer.vb` | todo | — | — |
| `PCFNetStudio\PartEditor.Designer.vb` | generated | VS Forms designer partial | — |
| `PCFNetStudio\PartEditor.resx` | generated | resource bundle (designer-managed) | — |
| `PCFNetStudio\PartEditor.vb` | todo | — | — |
| `PCFNetStudio\ProductionSet.vb` | todo | — | — |
| `PCFNetStudio\UCCalculation.Designer.vb` | generated | VS Forms designer partial | — |
| `PCFNetStudio\UCCalculation.resx` | generated | resource bundle (designer-managed) | — |
| `PCFNetStudio\UCCalculation.vb` | todo | — | — |
| `PCFNetStudio\UCProductComparer.Designer.vb` | generated | VS Forms designer partial | — |
| `PCFNetStudio\UCProductComparer.resx` | generated | resource bundle (designer-managed) | — |
| `PCFNetStudio\UCProductComparer.vb` | todo | — | — |

### Production

**Total: 6** &nbsp; | &nbsp; .vb: 6 | .resx: 0 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Production\ProductionProfileCutItemsHandler.vb` | todo | — | — |
| `Production\ProfileMilling\ExternalReferenceHelper.vb` | todo | — | — |
| `Production\ProfileMilling\ICAMImport.vb` | todo | — | — |
| `Production\ProfileMilling\ImportData.vb` | todo | — | — |
| `Production\ProfileMilling\ImportHandler.vb` | todo | — | — |
| `Production\ProfileMilling\PMMExportHandler.vb` | todo | — | — |

### Resources

**Total: 354** &nbsp; | &nbsp; .vb: 0 | .resx: 0 | assets: 353

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Resources\1downarrow1-32.png` | config | image/icon asset | — |
| `Resources\1leftarrow-32.png` | config | image/icon asset | — |
| `Resources\1rightarrow-32.png` | config | image/icon asset | — |
| `Resources\1uparrow-32.png` | config | image/icon asset | — |
| `Resources\2008-09-23 JAZO witte omranding 3cm.jpg` | config | image/icon asset | — |
| `Resources\2113918.png` | config | image/icon asset | — |
| `Resources\2dowarrow-32.png` | config | image/icon asset | — |
| `Resources\2leftarrow-32.png` | config | image/icon asset | — |
| `Resources\2rightarrow-32.png` | config | image/icon asset | — |
| `Resources\2uparrow-32.png` | config | image/icon asset | — |
| `Resources\5x-icon-sm.png` | config | image/icon asset | — |
| `Resources\accept-32.png` | config | image/icon asset | — |
| `Resources\Add Button-24.png` | config | image/icon asset | — |
| `Resources\Add Button-64.png` | config | image/icon asset | — |
| `Resources\Add_green_16.png` | config | image/icon asset | — |
| `Resources\Add_green_32.png` | config | image/icon asset | — |
| `Resources\Add-32.png` | config | image/icon asset | — |
| `Resources\AddButton_32_SE.png` | config | image/icon asset | — |
| `Resources\address_book-32.png` | config | image/icon asset | — |
| `Resources\Add-to_32.png` | config | image/icon asset | — |
| `Resources\Add-to-database-32.png` | config | image/icon asset | — |
| `Resources\agt_action_success-32.png` | config | image/icon asset | — |
| `Resources\agt_reload-32.png` | config | image/icon asset | — |
| `Resources\apply-64.png` | config | image/icon asset | — |
| `Resources\ArcEye_48.png` | config | image/icon asset | — |
| `Resources\Arrow_white_NE_16.png` | config | image/icon asset | — |
| `Resources\ArrowLong_right_64.png` | config | image/icon asset | — |
| `Resources\Arrow-sans-down-32.png` | config | image/icon asset | — |
| `Resources\Arrow-sans-up-32.png` | config | image/icon asset | — |
| `Resources\Assembly_16.png` | config | image/icon asset | — |
| `Resources\Assembly_link_16.png` | config | image/icon asset | — |
| `Resources\Ball-black-32.png` | config | image/icon asset | — |
| `Resources\Ball-green-32.png` | config | image/icon asset | — |
| `Resources\Ball-green-64.png` | config | image/icon asset | — |
| `Resources\Ball-red-32.png` | config | image/icon asset | — |
| `Resources\Ball-red-64.png` | config | image/icon asset | — |
| `Resources\Ball-yellow-64.png` | config | image/icon asset | — |
| `Resources\barcode_16.jpg` | config | image/icon asset | — |
| `Resources\barcode_laser_64.png` | config | image/icon asset | — |
| `Resources\Batch_128.png` | config | image/icon asset | — |
| `Resources\Batch_32.png` | config | image/icon asset | — |
| `Resources\bend_3d_16.png` | config | image/icon asset | — |
| `Resources\bend_3d_32.png` | config | image/icon asset | — |
| `Resources\bend_3d_64.png` | config | image/icon asset | — |
| `Resources\bendDie_16.png` | config | image/icon asset | — |
| `Resources\bendPiston_16.png` | config | image/icon asset | — |
| `Resources\Billiard-Marker-32.png` | config | image/icon asset | — |
| `Resources\Bin-blue-16.png` | config | image/icon asset | — |
| `Resources\Bin-blue-32.png` | config | image/icon asset | — |
| `Resources\Blue-Dossier-32.png` | config | image/icon asset | — |
| `Resources\Blue-Dossier-64.png` | config | image/icon asset | — |
| `Resources\Blue-Dossier-New-64.png` | config | image/icon asset | — |
| `Resources\bmt_16.png` | config | image/icon asset | — |
| `Resources\Box_65_65.png` | config | image/icon asset | — |
| `Resources\button_cancel-24.cur` | config | image/icon asset | — |
| `Resources\button_cancel-24.png` | config | image/icon asset | — |
| `Resources\button_cancel-32.png` | config | image/icon asset | — |
| `Resources\ButtonStop.png` | config | image/icon asset | — |
| `Resources\Calculator-32.png` | config | image/icon asset | — |
| `Resources\Calendar_16x16.png` | config | image/icon asset | — |
| `Resources\change_blue_32.png` | config | image/icon asset | — |
| `Resources\Chart-Line-32.png` | config | image/icon asset | — |
| `Resources\Check-64.png` | config | image/icon asset | — |
| `Resources\checkbox_checked.gif` | config | image/icon asset | — |
| `Resources\checkbox_checked_32.png` | config | image/icon asset | — |
| `Resources\checkbox_checked_png.png` | config | image/icon asset | — |
| `Resources\checkbox_unchecked.gif` | config | image/icon asset | — |
| `Resources\checkbox_unchecked_png.png` | config | image/icon asset | — |
| `Resources\Check-cropped-16.png` | config | image/icon asset | — |
| `Resources\Check-cropped-64.png` | config | image/icon asset | — |
| `Resources\Cinema4D_32.png` | config | image/icon asset | — |
| `Resources\Close_text_cursor.cur` | config | image/icon asset | — |
| `Resources\Close_text_cursor.png` | config | image/icon asset | — |
| `Resources\Close-48.png` | config | image/icon asset | — |
| `Resources\CMD922.PNG` | config | image/icon asset | — |
| `Resources\Cog-Edit-32.png` | config | image/icon asset | — |
| `Resources\Color-Edit-16.png` | config | image/icon asset | — |
| `Resources\Color-Select-32.png` | config | image/icon asset | — |
| `Resources\Comment-delete-icon16.png` | config | image/icon asset | — |
| `Resources\Comment-delete-icon24.png` | config | image/icon asset | — |
| `Resources\comment-edit-icon24.png` | config | image/icon asset | — |
| `Resources\compile_32.png` | config | image/icon asset | — |
| `Resources\compile_64.png` | config | image/icon asset | — |
| `Resources\Compress-16.png` | config | image/icon asset | — |
| `Resources\cone_vlc_24.png` | config | image/icon asset | — |
| `Resources\Copy_barcode_16.png` | config | image/icon asset | — |
| `Resources\creo_logo_32.jpg` | config | image/icon asset | — |
| `Resources\Database-Edit-32.png` | config | image/icon asset | — |
| `Resources\Date-From-32.png` | config | image/icon asset | — |
| `Resources\Delete_16.png` | config | image/icon asset | — |
| `Resources\Delete_32.png` | config | image/icon asset | — |
| `Resources\delete-32.png` | config | image/icon asset | — |
| `Resources\Delete-from_32.png` | config | image/icon asset | — |
| `Resources\dgw_32.png` | config | image/icon asset | — |
| `Resources\Document Edit_32.png` | config | image/icon asset | — |
| `Resources\Document-32.png` | config | image/icon asset | — |
| `Resources\Document-Import-32.png` | config | image/icon asset | — |
| `Resources\dwg_16.png` | config | image/icon asset | — |
| `Resources\Dxf_34.png` | config | image/icon asset | — |
| `Resources\edit_green_16.png` | config | image/icon asset | — |
| `Resources\edit_green_16NW.png` | config | image/icon asset | — |
| `Resources\edit_green_32.png` | config | image/icon asset | — |
| `Resources\Edit-16.png` | config | image/icon asset | — |
| `Resources\Edit-16-nw.png` | config | image/icon asset | — |
| `Resources\Edit-32.png` | config | image/icon asset | — |
| `Resources\Edit-64.png` | config | image/icon asset | — |
| `Resources\Edit-No-32.png` | config | image/icon asset | — |
| `Resources\Edit-Yes-32.png` | config | image/icon asset | — |
| `Resources\elfsquad-logo-large.png` | config | image/icon asset | — |
| `Resources\elu_tools.png` | config | image/icon asset | — |
| `Resources\EluBar_16.png` | config | image/icon asset | — |
| `Resources\EluCad0.ico` | config | image/icon asset | — |
| `Resources\elucad1.png` | config | image/icon asset | — |
| `Resources\EluCircle_16.png` | config | image/icon asset | — |
| `Resources\EluCut_16.png` | config | image/icon asset | — |
| `Resources\EluDeburr_16.png` | config | image/icon asset | — |
| `Resources\EluDrill_16.png` | config | image/icon asset | — |
| `Resources\EluFreeForm_16.png` | config | image/icon asset | — |
| `Resources\EluGroup_16.png` | config | image/icon asset | — |
| `Resources\EluInactive_16.png` | config | image/icon asset | — |
| `Resources\EluJob_16.png` | config | image/icon asset | — |
| `Resources\EluLine_16.png` | config | image/icon asset | — |
| `Resources\EluMacro_16.png` | config | image/icon asset | — |
| `Resources\EluRectangle_16.png` | config | image/icon asset | — |
| `Resources\EluSawCut_16.png` | config | image/icon asset | — |
| `Resources\EluSlottedHole_16.png` | config | image/icon asset | — |
| `Resources\Email_simple_16.png` | config | image/icon asset | — |
| `Resources\Email_simple_32.png` | config | image/icon asset | — |
| `Resources\Email-32.png` | config | image/icon asset | — |
| `Resources\Email-48.png` | config | image/icon asset | — |
| `Resources\emmegi_32.png` | config | image/icon asset | — |
| `Resources\Empty_Box_16.png` | config | image/icon asset | — |
| `Resources\empty_box_32.png` | config | image/icon asset | — |
| `Resources\empty_box_disabled_32.png` | config | image/icon asset | — |
| `Resources\Error_16_NW.png` | config | image/icon asset | — |
| `Resources\Error-32.png` | config | image/icon asset | — |
| `Resources\Error-48.png` | config | image/icon asset | — |
| `Resources\Error-64.png` | config | image/icon asset | — |
| `Resources\ErrorX-32.png` | config | image/icon asset | — |
| `Resources\evaluate.png` | config | image/icon asset | — |
| `Resources\excel_24.png` | config | image/icon asset | — |
| `Resources\excel_32.png` | config | image/icon asset | — |
| `Resources\excel_64.png` | config | image/icon asset | — |
| `Resources\F2-65x65.png` | config | image/icon asset | — |
| `Resources\Female_64.png` | config | image/icon asset | — |
| `Resources\FG LOGO 16X16.png` | config | image/icon asset | — |
| `Resources\FG LOGO 48x48.png` | config | image/icon asset | — |
| `Resources\file-64.png` | config | image/icon asset | — |
| `Resources\File-copy-32.png` | config | image/icon asset | — |
| `Resources\File-copy-48.png` | config | image/icon asset | — |
| `Resources\File-new-48.png` | config | image/icon asset | — |
| `Resources\fileopen-32.png` | config | image/icon asset | — |
| `Resources\Files-add-32.png` | config | image/icon asset | — |
| `Resources\Files-add-48.png` | config | image/icon asset | — |
| `Resources\filesave-16.png` | config | image/icon asset | — |
| `Resources\filesave-32.png` | config | image/icon asset | — |
| `Resources\filter-32.png` | config | image/icon asset | — |
| `Resources\filter-del-32.png` | config | image/icon asset | — |
| `Resources\Folder-32.png` | config | image/icon asset | — |
| `Resources\Folder-64.png` | config | image/icon asset | — |
| `Resources\Folder-blank-file-48.png` | config | image/icon asset | — |
| `Resources\Folder-light-32.png` | config | image/icon asset | — |
| `Resources\forklift_yellow_64.png` | config | image/icon asset | — |
| `Resources\Gauge-32.png` | config | image/icon asset | — |
| `Resources\gauge-type1-30-32px.png` | config | image/icon asset | — |
| `Resources\geo_16.png` | config | image/icon asset | — |
| `Resources\geo_open_contour_16.png` | config | image/icon asset | — |
| `Resources\glass_grey_32.png` | config | image/icon asset | — |
| `Resources\Glass-32.png` | config | image/icon asset | — |
| `Resources\Gnome-Stock-Person-64.png` | config | image/icon asset | — |
| `Resources\Golden_Star_NW_32.png` | config | image/icon asset | — |
| `Resources\Golden-Star-32.png` | config | image/icon asset | — |
| `Resources\Gray-Dossier-64.png` | config | image/icon asset | — |
| `Resources\Green-Dossier-16.png` | config | image/icon asset | — |
| `Resources\Green-Dossier-32.png` | config | image/icon asset | — |
| `Resources\Green-Dossier-64.png` | config | image/icon asset | — |
| `Resources\gridview.jpg` | config | image/icon asset | — |
| `Resources\hammer-32.png` | config | image/icon asset | — |
| `Resources\hammer-64.png` | config | image/icon asset | — |
| `Resources\hand_yellow_64.png` | config | image/icon asset | — |
| `Resources\Help_32.png` | config | image/icon asset | — |
| `Resources\Help_64.png` | config | image/icon asset | — |
| `Resources\home-32.png` | config | image/icon asset | — |
| `Resources\Hourglass-icon.png` | config | image/icon asset | — |
| `Resources\iCenter.ico` | config | image/icon asset | — |
| `Resources\iCenter.png` | config | image/icon asset | — |
| `Resources\import_24.png` | config | image/icon asset | — |
| `Resources\import-37-16.png` | config | image/icon asset | — |
| `Resources\import-37-32.png` | config | image/icon asset | — |
| `Resources\import-37-48.png` | config | image/icon asset | — |
| `Resources\info_blue_16.png` | config | image/icon asset | — |
| `Resources\info_blue-32.png` | config | image/icon asset | — |
| `Resources\info_blue-64.png` | config | image/icon asset | — |
| `Resources\Info-button-32.png` | config | image/icon asset | — |
| `Resources\Interface-builder.ico` | config | image/icon asset | — |
| `Resources\Interface-builder.png` | config | image/icon asset | — |
| `Resources\Interface-builder_16.png` | config | image/icon asset | — |
| `Resources\Isah_edit_16.png` | config | image/icon asset | — |
| `Resources\Isah_edit_32.png` | config | image/icon asset | — |
| `Resources\Isah_SW_16.png` | config | image/icon asset | — |
| `Resources\isah_table_edit_16.png` | config | image/icon asset | — |
| `Resources\JZ_logo_2012.png` | config | image/icon asset | — |
| `Resources\jzCadConnector.dll` | config | binary dependency | — |
| `Resources\JZ-logo-10x10mm.png` | config | image/icon asset | — |
| `Resources\JZSmall_50.png` | config | image/icon asset | — |
| `Resources\Kaltenbach_32.png` | config | image/icon asset | — |
| `Resources\Key-32.png` | config | image/icon asset | — |
| `Resources\kilogram-weight_64.png` | config | image/icon asset | — |
| `Resources\kilogram-weight_red_64.png` | config | image/icon asset | — |
| `Resources\layers_32.png` | config | image/icon asset | — |
| `Resources\LayerThicknessMeter_48.png` | config | image/icon asset | — |
| `Resources\length_32.png` | config | image/icon asset | — |
| `Resources\Length-32.png` | config | image/icon asset | — |
| `Resources\LightBlue-Dossier-32.png` | config | image/icon asset | — |
| `Resources\LightBlue-Dossier-64.png` | config | image/icon asset | — |
| `Resources\Link-Add-32.png` | config | image/icon asset | — |
| `Resources\Link-Break-32.png` | config | image/icon asset | — |
| `Resources\Link-Delete-32.png` | config | image/icon asset | — |
| `Resources\Link-Edit-32.png` | config | image/icon asset | — |
| `Resources\Locked-32.png` | config | image/icon asset | — |
| `Resources\maintenance_64.png` | config | image/icon asset | — |
| `Resources\manufacturing_16.png` | config | image/icon asset | — |
| `Resources\mcm_logo_blue_large.png` | config | image/icon asset | — |
| `Resources\merge-icon-16x16.png` | config | image/icon asset | — |
| `Resources\mill_16.png` | config | image/icon asset | — |
| `Resources\mill_alu_64.png` | config | image/icon asset | — |
| `Resources\Minus Button-24.png` | config | image/icon asset | — |
| `Resources\Minus Button-64.png` | config | image/icon asset | — |
| `Resources\New-32.png` | config | image/icon asset | — |
| `Resources\New-Year-Champagne-new-year-holiday-celebration-smiley-emoticon-000764-large.gif` | config | image/icon asset | — |
| `Resources\notify_16_NE.png` | config | image/icon asset | — |
| `Resources\notify_16_SW.png` | config | image/icon asset | — |
| `Resources\Ok-48.png` | config | image/icon asset | — |
| `Resources\Ok-64.png` | config | image/icon asset | — |
| `Resources\Open-32.png` | config | image/icon asset | — |
| `Resources\Orange-Dossier-32.png` | config | image/icon asset | — |
| `Resources\Orange-Dossier-64.png` | config | image/icon asset | — |
| `Resources\Packages-26.png` | config | image/icon asset | — |
| `Resources\paper_plane_16.png` | config | image/icon asset | — |
| `Resources\paper_plane_512_dqb_1-2.png` | config | image/icon asset | — |
| `Resources\Part_16.png` | config | image/icon asset | — |
| `Resources\part_link_16.png` | config | image/icon asset | — |
| `Resources\part_mill_16.png` | config | image/icon asset | — |
| `Resources\Paste-32.png` | config | image/icon asset | — |
| `Resources\pdf.png` | config | image/icon asset | — |
| `Resources\PDF-XChange_Icon_128.png` | config | image/icon asset | — |
| `Resources\Person-Undefined-Female-Light-64.png` | config | image/icon asset | — |
| `Resources\Person-Undefined-Male-Light-64.png` | config | image/icon asset | — |
| `Resources\phone_16.png` | config | image/icon asset | — |
| `Resources\phone_orange_16.png` | config | image/icon asset | — |
| `Resources\phone-32x32.png` | config | image/icon asset | — |
| `Resources\Phone-Blue-16.png` | config | image/icon asset | — |
| `Resources\Phone-Blue-32.png` | config | image/icon asset | — |
| `Resources\Planregels_RMB.png` | config | image/icon asset | — |
| `Resources\play_green_32.png` | config | image/icon asset | — |
| `Resources\player_pause-32.png` | config | image/icon asset | — |
| `Resources\player_play-32.png` | config | image/icon asset | — |
| `Resources\Plus-48.png` | config | image/icon asset | — |
| `Resources\pmi_16.png` | config | image/icon asset | — |
| `Resources\Printer-32x32.png` | config | image/icon asset | — |
| `Resources\printer-icon-32.gif` | config | image/icon asset | — |
| `Resources\Product-documentation-32.png` | config | image/icon asset | — |
| `Resources\Product-documentation-64.png` | config | image/icon asset | — |
| `Resources\ProeGlobalInterference_16.png` | config | image/icon asset | — |
| `Resources\proelogo.bmp` | config | image/icon asset | — |
| `Resources\Purple-Dossier-64.png` | config | image/icon asset | — |
| `Resources\rabbit_24.png` | config | image/icon asset | — |
| `Resources\Recycle-32.png` | config | image/icon asset | — |
| `Resources\Red-Dossier-32.png` | config | image/icon asset | — |
| `Resources\Red-Dossier-64.png` | config | image/icon asset | — |
| `Resources\Redo-32.png` | config | image/icon asset | — |
| `Resources\Redo-64.png` | config | image/icon asset | — |
| `Resources\Refresh-32.png` | config | image/icon asset | — |
| `Resources\Refresh-Orange-32.png` | config | image/icon asset | — |
| `Resources\reload_32.png` | config | image/icon asset | — |
| `Resources\Reminders-Wooden-32.png` | config | image/icon asset | — |
| `Resources\Remove_16.png` | config | image/icon asset | — |
| `Resources\Remove-from-database-32.png` | config | image/icon asset | — |
| `Resources\Resources-32.png` | config | image/icon asset | — |
| `Resources\rotate_blue_cw-48.png` | config | image/icon asset | — |
| `Resources\save_all-32.png` | config | image/icon asset | — |
| `Resources\Saw_Blade_26.jpg` | config | image/icon asset | — |
| `Resources\sawblad_32.png` | config | image/icon asset | — |
| `Resources\sawblad_47.png` | config | image/icon asset | — |
| `Resources\sawblade_32_anim.gif` | config | image/icon asset | — |
| `Resources\Search2-32.png` | config | image/icon asset | — |
| `Resources\Setting-64.png` | config | image/icon asset | — |
| `Resources\settings_gear_blue_32.png` | config | image/icon asset | — |
| `Resources\shopping_cart_16.png` | config | image/icon asset | — |
| `Resources\shopping_cart_32.png` | config | image/icon asset | — |
| `Resources\shopping_cart_64.png` | config | image/icon asset | — |
| `Resources\shopping_cart_yellow_64.png` | config | image/icon asset | — |
| `Resources\Signal-Stop-48.png` | config | image/icon asset | — |
| `Resources\sintlinks.gif` | config | image/icon asset | — |
| `Resources\Smiley_14.png` | config | image/icon asset | — |
| `Resources\Smiley-01.jpg` | config | image/icon asset | — |
| `Resources\Source-Code-32.png` | config | image/icon asset | — |
| `Resources\spyglass-32.png` | config | image/icon asset | — |
| `Resources\spyglass-64.png` | config | image/icon asset | — |
| `Resources\stack_32.png` | config | image/icon asset | — |
| `Resources\Started-cropped-16.png` | config | image/icon asset | — |
| `Resources\Started-cropped-64.png` | config | image/icon asset | — |
| `Resources\stop_32.png` | config | image/icon asset | — |
| `Resources\Stop-32.png` | config | image/icon asset | — |
| `Resources\Stop-64.png` | config | image/icon asset | — |
| `Resources\Table-Gear-32.png` | config | image/icon asset | — |
| `Resources\Taf_16.png` | config | image/icon asset | — |
| `Resources\Tag-Red-32 (1).png` | config | image/icon asset | — |
| `Resources\Tag-Red-32.png` | config | image/icon asset | — |
| `Resources\TCT_Carbide_Circular_Saw_Blade.jpg` | config | image/icon asset | — |
| `Resources\text_bold_16.png` | config | image/icon asset | — |
| `Resources\text_italic_16.png` | config | image/icon asset | — |
| `Resources\text_strikethrough_16.png` | config | image/icon asset | — |
| `Resources\text_underline_16.png` | config | image/icon asset | — |
| `Resources\time.png` | config | image/icon asset | — |
| `Resources\Time_64.png` | config | image/icon asset | — |
| `Resources\Time-Clock-16.png` | config | image/icon asset | — |
| `Resources\Time-Clock-16_nw.png` | config | image/icon asset | — |
| `Resources\Time-Clock-64.png` | config | image/icon asset | — |
| `Resources\Tip-32.png` | config | image/icon asset | — |
| `Resources\todo-icon-64.gif` | config | image/icon asset | — |
| `Resources\Tools_hammer_wrench_16.png` | config | image/icon asset | — |
| `Resources\Tools-32.png` | config | image/icon asset | — |
| `Resources\truck_darkgreen_64.png` | config | image/icon asset | — |
| `Resources\truck_green_32.png` | config | image/icon asset | — |
| `Resources\truck_green_64.png` | config | image/icon asset | — |
| `Resources\Trumpf.TruTops.Control.Shell.png` | config | image/icon asset | — |
| `Resources\trumpf_16.png` | config | image/icon asset | — |
| `Resources\trumpf_32.png` | config | image/icon asset | — |
| `Resources\trumpf_48.png` | config | image/icon asset | — |
| `Resources\turtle_24.png` | config | image/icon asset | — |
| `Resources\UniLink.png` | config | image/icon asset | — |
| `Resources\Unlocked-32.png` | config | image/icon asset | — |
| `Resources\User-add-32.png` | config | image/icon asset | — |
| `Resources\Users-info-48.png` | config | image/icon asset | — |
| `Resources\Users-mixed-gender-32.png` | config | image/icon asset | — |
| `Resources\us-proj_64.png` | config | image/icon asset | — |
| `Resources\waitCursor.gif` | config | image/icon asset | — |
| `Resources\Warning_16_NW.png` | config | image/icon asset | — |
| `Resources\Warning_16_SE.png` | config | image/icon asset | — |
| `Resources\warning_anim_32.gif` | config | image/icon asset | — |
| `Resources\warning_black_32.png` | config | image/icon asset | — |
| `Resources\warning_black_64.png` | config | image/icon asset | — |
| `Resources\Warning-32 (1).png` | config | image/icon asset | — |
| `Resources\Warning-64.png` | config | image/icon asset | — |
| `Resources\Wikipedia-32.png` | config | image/icon asset | — |
| `Resources\Windchill_Product_16.png` | config | image/icon asset | — |
| `Resources\Windchill_search_16.png` | config | image/icon asset | — |
| `Resources\Windchill11.png` | config | image/icon asset | — |
| `Resources\ws_where_used_report.gif` | config | image/icon asset | — |
| `Resources\Yellow-Dossier-64.png` | config | image/icon asset | — |
| `Resources\Zammad_icon_24.png` | config | image/icon asset | — |
| `Resources\Zammad_icon_32.png` | config | image/icon asset | — |
| `Resources\Zammad_icon_64.png` | config | image/icon asset | — |

### Sales

**Total: 3** &nbsp; | &nbsp; .vb: 2 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Sales\FrmCustomerTeam.Designer.vb` | generated | VS Forms designer partial | — |
| `Sales\FrmCustomerTeam.resx` | generated | resource bundle (designer-managed) | — |
| `Sales\FrmCustomerTeam.vb` | todo | — | — |

### SmtManufacturing

**Total: 78** &nbsp; | &nbsp; .vb: 63 | .resx: 15 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `SmtManufacturing\BendNote.vb` | todo | — | — |
| `SmtManufacturing\BendPart.vb` | todo | — | — |
| `SmtManufacturing\BendTool.vb` | todo | — | — |
| `SmtManufacturing\BendToolGroup.vb` | todo | — | — |
| `SmtManufacturing\BendToolStation.vb` | todo | — | — |
| `SmtManufacturing\BncInterpreter\BNC.vb` | todo | — | — |
| `SmtManufacturing\BoostMigrator.vb` | todo | — | — |
| `SmtManufacturing\BoostPartViewerControl.Designer.vb` | generated | VS Forms designer partial | — |
| `SmtManufacturing\BoostPartViewerControl.resx` | generated | resource bundle (designer-managed) | — |
| `SmtManufacturing\BoostPartViewerControl.vb` | todo | — | — |
| `SmtManufacturing\ContourCheck.vb` | todo | — | — |
| `SmtManufacturing\ControlSmtCut.Designer.vb` | generated | VS Forms designer partial | — |
| `SmtManufacturing\ControlSmtCut.resx` | generated | resource bundle (designer-managed) | — |
| `SmtManufacturing\ControlSmtCut.vb` | todo | — | — |
| `SmtManufacturing\CtrlBendBoost.Designer.vb` | generated | VS Forms designer partial | — |
| `SmtManufacturing\CtrlBendBoost.resx` | generated | resource bundle (designer-managed) | — |
| `SmtManufacturing\CtrlBendBoost.vb` | todo | — | — |
| `SmtManufacturing\CtrlBendCalcDetails.Designer.vb` | generated | VS Forms designer partial | — |
| `SmtManufacturing\CtrlBendCalcDetails.vb` | todo | — | — |
| `SmtManufacturing\CtrlMachinePartBendSolutions.Designer.vb` | generated | VS Forms designer partial | — |
| `SmtManufacturing\CtrlMachinePartBendSolutions.resx` | generated | resource bundle (designer-managed) | — |
| `SmtManufacturing\CtrlMachinePartBendSolutions.vb` | todo | — | — |
| `SmtManufacturing\CtrlPartBendSolution.Designer.vb` | generated | VS Forms designer partial | — |
| `SmtManufacturing\CtrlPartBendSolution.resx` | generated | resource bundle (designer-managed) | — |
| `SmtManufacturing\CtrlPartBendSolution.vb` | todo | — | — |
| `SmtManufacturing\CutSheetLabelPrintHandler.vb` | todo | — | — |
| `SmtManufacturing\DxfContour.vb` | todo | — | — |
| `SmtManufacturing\FileMerger.vb` | todo | — | — |
| `SmtManufacturing\FlatPatternConverter.vb` | todo | — | — |
| `SmtManufacturing\FrmBendLicense.Designer.vb` | generated | VS Forms designer partial | — |
| `SmtManufacturing\FrmBendLicense.resx` | generated | resource bundle (designer-managed) | — |
| `SmtManufacturing\FrmBendLicense.vb` | todo | — | — |
| `SmtManufacturing\FrmCalcCycleTimeManagement.Designer.vb` | generated | VS Forms designer partial | — |
| `SmtManufacturing\FrmCalcCycleTimeManagement.resx` | generated | resource bundle (designer-managed) | — |
| `SmtManufacturing\FrmCalcCycleTimeManagement.vb` | todo | — | — |
| `SmtManufacturing\FrmNestedSheet.Designer.vb` | generated | VS Forms designer partial | — |
| `SmtManufacturing\FrmNestedSheet.resx` | generated | resource bundle (designer-managed) | — |
| `SmtManufacturing\FrmNestedSheet.vb` | todo | — | — |
| `SmtManufacturing\FrmPartIdentifier.Designer.vb` | generated | VS Forms designer partial | — |
| `SmtManufacturing\FrmPartIdentifier.resx` | generated | resource bundle (designer-managed) | — |
| `SmtManufacturing\FrmPartIdentifier.vb` | todo | — | — |
| `SmtManufacturing\FrmSmtMaterialManagement.Designer.vb` | generated | VS Forms designer partial | — |
| `SmtManufacturing\FrmSmtMaterialManagement.resx` | generated | resource bundle (designer-managed) | — |
| `SmtManufacturing\FrmSmtMaterialManagement.vb` | todo | — | — |
| `SmtManufacturing\frmTopsLicenses.Designer.vb` | generated | VS Forms designer partial | — |
| `SmtManufacturing\frmTopsLicenses.resx` | generated | resource bundle (designer-managed) | — |
| `SmtManufacturing\frmTopsLicenses.vb` | todo | — | — |
| `SmtManufacturing\GeoViewerControl.Designer.vb` | generated | VS Forms designer partial | — |
| `SmtManufacturing\GeoViewerControl.resx` | generated | resource bundle (designer-managed) | — |
| `SmtManufacturing\GeoViewerControl.vb` | todo | — | — |
| `SmtManufacturing\ISmtFileViewer.vb` | todo | — | — |
| `SmtManufacturing\Job.vb` | todo | — | — |
| `SmtManufacturing\JPLT_DistrSticker.vb` | todo | — | — |
| `SmtManufacturing\JPLT_PartIdentSticker.vb` | todo | — | — |
| `SmtManufacturing\JPLT_SheetIdentSticker.vb` | todo | — | — |
| `SmtManufacturing\LaserCalc.vb` | todo | — | — |
| `SmtManufacturing\NestedSheet.vb` | todo | — | — |
| `SmtManufacturing\NestPart.vb` | todo | — | — |
| `SmtManufacturing\Oid.vb` | todo | — | — |
| `SmtManufacturing\Part.vb` | todo | — | — |
| `SmtManufacturing\PreProcessorTest.Designer.vb` | generated | VS Forms designer partial | — |
| `SmtManufacturing\PreProcessorTest.resx` | generated | resource bundle (designer-managed) | — |
| `SmtManufacturing\PreProcessorTest.vb` | todo | — | — |
| `SmtManufacturing\ProductionOrderCleanupHandler.vb` | todo | — | — |
| `SmtManufacturing\Smt3DImportExclusionHelper.vb` | todo | — | — |
| `SmtManufacturing\SmtFileViewerFactory.vb` | todo | — | — |
| `SmtManufacturing\SmtPartViewer.Designer.vb` | generated | VS Forms designer partial | — |
| `SmtManufacturing\SmtPartViewer.resx` | generated | resource bundle (designer-managed) | — |
| `SmtManufacturing\SmtPartViewer.vb` | todo | — | — |
| `SmtManufacturing\TafInterpreter\Part.vb` | todo | — | — |
| `SmtManufacturing\TafInterpreter\PartInstance.vb` | todo | — | — |
| `SmtManufacturing\TafInterpreter\TAF.vb` | todo | — | — |
| `SmtManufacturing\TmtInterpreter\TMT.vb` | todo | — | — |
| `SmtManufacturing\TruTops.vb` | todo | — | — |
| `SmtManufacturing\UCTAFViewer.designer.vb` | generated | VS Forms designer partial | — |
| `SmtManufacturing\UCTAFViewer.resx` | generated | resource bundle (designer-managed) | — |
| `SmtManufacturing\UCTAFViewer.vb` | todo | — | — |
| `SmtManufacturing\WorkViewBoost.vb` | todo | — | — |

### SolaDataConnector

**Total: 2** &nbsp; | &nbsp; .vb: 2 | .resx: 0 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `SolaDataConnector\AppHandler.vb` | todo | — | — |
| `SolaDataConnector\AppWindowHelper.vb` | todo | — | — |

### UniLink

**Total: 30** &nbsp; | &nbsp; .vb: 29 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `UniLink\ApplicationHandler.vb` | todo | — | — |
| `UniLink\ApplicationService.vb` | todo | — | — |
| `UniLink\CSVImportHandler.vb` | todo | — | — |
| `UniLink\Entities\List.vb` | todo | — | — |
| `UniLink\Entities\Profile.vb` | todo | — | — |
| `UniLink\ExceptionHandlers\ProfileSeriesExceptionHandler.vb` | todo | — | — |
| `UniLink\ExceptionHandlers\SourceFilesExceptionHandler.vb` | todo | — | — |
| `UniLink\Exceptions\ProfileSeriesException.vb` | todo | — | — |
| `UniLink\Exceptions\SourceFilesException.vb` | todo | — | — |
| `UniLink\ExportConverter.vb` | todo | — | — |
| `UniLink\ExportHelper.vb` | todo | — | — |
| `UniLink\ExportInstructions.vb` | todo | — | — |
| `UniLink\Factories\ListFactory.vb` | todo | — | — |
| `UniLink\ListDataService.vb` | todo | — | — |
| `UniLink\Machine.vb` | todo | — | — |
| `UniLink\MachineHelper.vb` | todo | — | — |
| `UniLink\MecalAriel4Export.vb` | todo | — | — |
| `UniLink\MultiStepReader\CSVHandler.vb` | todo | — | — |
| `UniLink\MultiStepReader\LogDetailContent.vb` | todo | — | — |
| `UniLink\MultiStepReader\MasterLog.vb` | todo | — | — |
| `UniLink\MultiStepReader\MasterLogDetail.vb` | todo | — | — |
| `UniLink\MultiStepReader\MasterLogDetailFactory.vb` | todo | — | — |
| `UniLink\MultiStepReader\MasterLogDetailFile.vb` | todo | — | — |
| `UniLink\MultiStepReader\MasterLogDetailOptimiser.vb` | todo | — | — |
| `UniLink\MultiStepReader\MasterLogFactory.vb` | todo | — | — |
| `UniLink\MultiStepReader\MasterLogService.vb` | todo | — | — |
| `UniLink\Settings.vb` | todo | — | — |
| `UniLink\UI\TabControl.Designer.vb` | generated | VS Forms designer partial | — |
| `UniLink\UI\TabControl.resx` | generated | resource bundle (designer-managed) | — |
| `UniLink\UI\TabControl.vb` | todo | — | — |

### VentDuctConfigurator

**Total: 4** &nbsp; | &nbsp; .vb: 3 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `VentDuctConfigurator\FrmVentDuctConfigurator.Designer.vb` | generated | VS Forms designer partial | — |
| `VentDuctConfigurator\FrmVentDuctConfigurator.resx` | generated | resource bundle (designer-managed) | — |
| `VentDuctConfigurator\FrmVentDuctConfigurator.vb` | todo | — | — |
| `VentDuctConfigurator\VentDuct.vb` | todo | — | — |

### WebClock

**Total: 13** &nbsp; | &nbsp; .vb: 9 | .resx: 4 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `WebClock\FrmChangedTimeReg.Designer.vb` | generated | VS Forms designer partial | — |
| `WebClock\FrmChangedTimeReg.resx` | generated | resource bundle (designer-managed) | — |
| `WebClock\FrmChangedTimeReg.vb` | todo | — | — |
| `WebClock\FrmCurrentTimeReg.Designer.vb` | generated | VS Forms designer partial | — |
| `WebClock\FrmCurrentTimeReg.resx` | generated | resource bundle (designer-managed) | — |
| `WebClock\FrmCurrentTimeReg.vb` | todo | — | — |
| `WebClock\FrmOfficeClockDetailLines.Designer.vb` | generated | VS Forms designer partial | — |
| `WebClock\FrmOfficeClockDetailLines.resx` | generated | resource bundle (designer-managed) | — |
| `WebClock\FrmOfficeClockDetailLines.vb` | todo | — | — |
| `WebClock\FrmTimeRegistration.designer.vb` | generated | VS Forms designer partial | — |
| `WebClock\FrmTimeRegistration.resx` | generated | resource bundle (designer-managed) | — |
| `WebClock\FrmTimeRegistration.vb` | todo | — | — |
| `WebClock\WebClock.vb` | todo | — | — |

### WorkPreparation

**Total: 6** &nbsp; | &nbsp; .vb: 5 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `WorkPreparation\FrmOutsourceOperations.Designer.vb` | generated | VS Forms designer partial | — |
| `WorkPreparation\FrmOutsourceOperations.resx` | generated | resource bundle (designer-managed) | — |
| `WorkPreparation\FrmOutsourceOperations.vb` | todo | — | — |
| `WorkPreparation\IPBatchCollector.vb` | todo | — | — |
| `WorkPreparation\OperationSubstitutionHandler.vb` | todo | — | — |
| `WorkPreparation\OutsourceOperationsHandler.vb` | todo | — | — |



---


---

# Project: TruTopsLib

**Total files: 65**

### TruTopsLib / (root)

**Total: 16** &nbsp; | &nbsp; .vb: 3 | .cs: 5 | .resx: 

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `app.config` | config | config / project metadata | — |
| `FileReaderHelper.cs` | todo | — | — |
| `FileReaderHelper.vb` | todo | — | — |
| `JAZO Zevenaar bv.snk` | config | signing key | — |
| `LayerConverter.cs` | todo | — | — |
| `LayerConverter.vb` | todo | — | — |
| `MigrationFix.cs` | todo | — | — |
| `MigrationFix.vb` | todo | — | — |
| `PMILabel.cs` | todo | — | — |
| `PMILabelCollection.cs` | todo | — | — |
| `Resources.Designer.cs` | generated | VS designer partial | — |
| `Resources.resx` | generated | resource bundle (designer-managed) | — |
| `TruTopsLib.csproj` | config | config / project metadata | — |
| `TruTopsLib.vbproj` | config | config / project metadata | — |
| `TruTopsLib.vbproj.vspscc` | config | config / project metadata | — |
| `TruTopsLib2.sln` | config | config / project metadata | — |

### TruTopsLib / GeoInterpreter

**Total: 10** &nbsp; | &nbsp; .vb: 5 | .cs: 5 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `GeoInterpreter\FlatGeometry.cs` | todo | — | — |
| `GeoInterpreter\FlatGeometry.vb` | todo | — | — |
| `GeoInterpreter\FlatGeometryDxfExport.cs` | todo | — | — |
| `GeoInterpreter\FlatGeometryDxfExport.vb` | todo | — | — |
| `GeoInterpreter\FlatGeometryGeoExport.cs` | todo | — | — |
| `GeoInterpreter\FlatGeometryGeoExport.vb` | todo | — | — |
| `GeoInterpreter\FlatGeometryReader.cs` | todo | — | — |
| `GeoInterpreter\FlatGeometryReader.vb` | todo | — | — |
| `GeoInterpreter\IFlatGeometryExport.cs` | todo | — | — |
| `GeoInterpreter\IFlatGeometryExport.vb` | todo | — | — |

### TruTopsLib / PMI

**Total: 5** &nbsp; | &nbsp; .vb: 5 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `PMI\IPmiLabel.vb` | todo | — | — |
| `PMI\PMILabel.vb` | todo | — | — |
| `PMI\PMILabelCollection.vb` | todo | — | — |
| `PMI\PmiLabelCountersink.vb` | todo | — | — |
| `PMI\PmiLabelThreadNote.vb` | todo | — | — |

### TruTopsLib / TopsFile

**Total: 34** &nbsp; | &nbsp; .vb: 17 | .cs: 17 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `TopsFile\BendLine.cs` | todo | — | — |
| `TopsFile\BendLine.vb` | todo | — | — |
| `TopsFile\Body.cs` | todo | — | — |
| `TopsFile\Body.vb` | todo | — | — |
| `TopsFile\Bounds.cs` | todo | — | — |
| `TopsFile\Bounds.vb` | todo | — | — |
| `TopsFile\Contour.cs` | todo | — | — |
| `TopsFile\Contour.vb` | todo | — | — |
| `TopsFile\DataTypeHandler.cs` | todo | — | — |
| `TopsFile\DataTypeHandler.vb` | todo | — | — |
| `TopsFile\Parameters.cs` | todo | — | — |
| `TopsFile\Parameters.vb` | todo | — | — |
| `TopsFile\Point.cs` | todo | — | — |
| `TopsFile\Point.vb` | todo | — | — |
| `TopsFile\PointCollection.cs` | todo | — | — |
| `TopsFile\PointCollection.vb` | todo | — | — |
| `TopsFile\Properties.cs` | todo | — | — |
| `TopsFile\Properties.vb` | todo | — | — |
| `TopsFile\SubContour\Arc.cs` | todo | — | — |
| `TopsFile\SubContour\Arc.vb` | todo | — | — |
| `TopsFile\SubContour\Circle.cs` | todo | — | — |
| `TopsFile\SubContour\Circle.vb` | todo | — | — |
| `TopsFile\SubContour\Fillet.cs` | todo | — | — |
| `TopsFile\SubContour\Fillet.vb` | todo | — | — |
| `TopsFile\SubContour\Line.cs` | todo | — | — |
| `TopsFile\SubContour\Line.vb` | todo | — | — |
| `TopsFile\SubContour\SubContour.cs` | todo | — | — |
| `TopsFile\SubContour\SubContour.vb` | todo | — | — |
| `TopsFile\SubContour\Text.cs` | todo | — | — |
| `TopsFile\SubContour\Text.vb` | todo | — | — |
| `TopsFile\TextCollection.cs` | todo | — | — |
| `TopsFile\TextCollection.vb` | todo | — | — |
| `TopsFile\TTInfo.cs` | todo | — | — |
| `TopsFile\TTInfo.vb` | todo | — | — |



---

# Project: ICenterLib

**Total files: 723**

### ICenterLib / (root)

**Total: 15** &nbsp; | &nbsp; .vb: 9 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `app.config` | config | config / project metadata | — |
| `AppSettings.vb` | todo | — | — |
| `Common.vb` | todo | — | — |
| `ComputerSetting.vb` | todo | — | — |
| `Connections.vb` | todo | — | — |
| `ICenterLib.vbproj` | config | config / project metadata | — |
| `ICenterLib.vbproj.user` | config | config / project metadata | — |
| `ICenterLib.vbproj.vspscc` | config | config / project metadata | — |
| `Images.vb` | todo | — | — |
| `JAZO Zevenaar bv.snk` | config | signing key | — |
| `Log.vb` | todo | — | — |
| `Main.vb` | todo | — | — |
| `packages.config` | config | config / project metadata | — |
| `PdfTools.vb` | todo | — | — |
| `UserSetting.vb` | todo | — | — |

### ICenterLib / CAD

**Total: 127** &nbsp; | &nbsp; .vb: 116 | .cs: 0 | .resx: 5

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `CAD\Creo\AppManager.vb` | todo | — | — |
| `CAD\Creo\AppVersion.vb` | todo | — | — |
| `CAD\Creo\CadApp.vb` | todo | — | — |
| `CAD\Creo\CadAppVersion.vb` | todo | — | — |
| `CAD\Creo\CtrlAppManager.Designer.vb` | generated | VS designer partial | — |
| `CAD\Creo\CtrlAppManager.resx` | generated | resource bundle (designer-managed) | — |
| `CAD\Creo\CtrlAppManager.vb` | todo | — | — |
| `CAD\Creo\Dimension.vb` | todo | — | — |
| `CAD\Creo\Environment.vb` | todo | — | — |
| `CAD\Creo\Feature.vb` | todo | — | — |
| `CAD\Creo\Features.vb` | todo | — | — |
| `CAD\Creo\FrmAppManager.Designer.vb` | generated | VS designer partial | — |
| `CAD\Creo\FrmAppManager.resx` | generated | resource bundle (designer-managed) | — |
| `CAD\Creo\FrmAppManager.vb` | todo | — | — |
| `CAD\Creo\FrmCreoLicense.Designer.vb` | generated | VS designer partial | — |
| `CAD\Creo\FrmCreoLicense.resx` | generated | resource bundle (designer-managed) | — |
| `CAD\Creo\FrmCreoLicense.vb` | todo | — | — |
| `CAD\Creo\License\License.vb` | todo | — | — |
| `CAD\Creo\License\LicenseResource.vb` | todo | — | — |
| `CAD\Creo\License\LicenseResourceHandler.vb` | todo | — | — |
| `CAD\Creo\License\LicenseSelectionHandler.vb` | todo | — | — |
| `CAD\Creo\Material.vb` | todo | — | — |
| `CAD\Creo\Materials.vb` | todo | — | — |
| `CAD\Creo\ModelInformation.vb` | todo | — | — |
| `CAD\Creo\ModelItem.vb` | todo | — | — |
| `CAD\Creo\Parameter.vb` | todo | — | — |
| `CAD\Creo\ParameterCollection.vb` | todo | — | — |
| `CAD\Creo\ParamValue\ParamValue.vb` | todo | — | — |
| `CAD\Creo\ParamValue\ParamValueBoolean.vb` | todo | — | — |
| `CAD\Creo\ParamValue\ParamValueDouble.vb` | todo | — | — |
| `CAD\Creo\ParamValue\ParamValueInteger.vb` | todo | — | — |
| `CAD\Creo\ParamValue\ParamValueString.vb` | todo | — | — |
| `CAD\Creo\PlmAppVersion.vb` | todo | — | — |
| `CAD\Creo\ProProgram\Design.vb` | todo | — | — |
| `CAD\Creo\ProProgram\ExecuteStatement.vb` | todo | — | — |
| `CAD\Creo\ProProgram\Functions.vb` | todo | — | — |
| `CAD\Creo\ProProgram\Input.vb` | todo | — | — |
| `CAD\Creo\ProProgram\VBCodeConverter.vb` | todo | — | — |
| `CAD\Creo\RegenerationInput.vb` | todo | — | — |
| `CAD\Creo\StartupFile.vb` | todo | — | — |
| `CAD\Creo\Toolbox.vb` | todo | — | — |
| `CAD\Creo\Tools\StpAssySplitter.vb` | todo | — | — |
| `CAD\Creo\Trailfile.vb` | todo | — | — |
| `CAD\CreoView\Application.vb` | todo | — | — |
| `CAD\CreoView\Configuration.vb` | todo | — | — |
| `CAD\CreoView\Converter.vb` | todo | — | — |
| `CAD\DXF\SvgConverter\DxfHelper.vb` | todo | — | — |
| `CAD\DXF\SvgConverter\DxfToSvgConverter.vb` | todo | — | — |
| `CAD\DXF\SvgConverter\Export.vb` | todo | — | — |
| `CAD\DXF\SvgConverter\Modifier.vb` | todo | — | — |
| `CAD\Geometry\BoundingBox.vb` | todo | — | — |
| `CAD\Geometry\BoundingBoxFactory.vb` | todo | — | — |
| `CAD\Geometry\Dxf3DProfileMill.vb` | todo | — | — |
| `CAD\Geometry\DxfEntityColor.vb` | todo | — | — |
| `CAD\Geometry\Earcut.vb` | todo | — | — |
| `CAD\Geometry\Earcut_CSharp.vb` | todo | — | — |
| `CAD\Geometry\GraphicsPathHelper.vb` | todo | — | — |
| `CAD\Modelgenerator\Configuration.vb` | todo | — | — |
| `CAD\Modelgenerator\Definition.vb` | todo | — | — |
| `CAD\Modelgenerator\DefinitionCollection.vb` | todo | — | — |
| `CAD\Modelgenerator\Exceptions\GenericObjectNotFoundException.vb` | todo | — | — |
| `CAD\Modelgenerator\GenericModelBackupConfiguration.vb` | todo | — | — |
| `CAD\Modelgenerator\ModelGeneratorOption.vb` | todo | — | — |
| `CAD\Modelgenerator\ModelgeneratorRequests\DesignDuplicationConfiguration.vb` | todo | — | — |
| `CAD\Modelgenerator\ModelgeneratorRequests\ElfsquadModelgeneratorRequest.vb` | todo | — | — |
| `CAD\Modelgenerator\ModelgeneratorRequests\IrisModelgeneratorInstructions.vb` | todo | — | — |
| `CAD\Modelgenerator\ModelgeneratorRequests\IrisModelgeneratorTaskResult.vb` | todo | — | — |
| `CAD\Modelgenerator\Operations\Checkin.vb` | todo | — | — |
| `CAD\Modelgenerator\Operations\CreateWS.vb` | todo | — | — |
| `CAD\Modelgenerator\Operations\DeleteWS.vb` | todo | — | — |
| `CAD\Modelgenerator\Operations\Download.vb` | todo | — | — |
| `CAD\Modelgenerator\Operations\EraseUndisplayedModels.vb` | todo | — | — |
| `CAD\Modelgenerator\Operations\ExportDocumentOperation.vb` | todo | — | — |
| `CAD\Modelgenerator\Operations\JZCheckoutFolders.vb` | todo | — | — |
| `CAD\Modelgenerator\Operations\JZExportByNumber.vb` | todo | — | — |
| `CAD\Modelgenerator\Operations\JZImportByNumber.vb` | todo | — | — |
| `CAD\Modelgenerator\Operations\JZRenameObjectNoServer.vb` | todo | — | — |
| `CAD\Modelgenerator\Operations\OpenInProE.vb` | todo | — | — |
| `CAD\Modelgenerator\Operations\Operation.vb` | todo | — | — |
| `CAD\Modelgenerator\Operations\OperationCollection.vb` | todo | — | — |
| `CAD\Modelgenerator\Operations\RegenReadPar.vb` | todo | — | — |
| `CAD\Modelgenerator\Operations\Register.vb` | todo | — | — |
| `CAD\Modelgenerator\Operations\Rename.vb` | todo | — | — |
| `CAD\Modelgenerator\Operations\Save.vb` | todo | — | — |
| `CAD\Modelgenerator\Operations\SetWorkingDirectory.vb` | todo | — | — |
| `CAD\Modelgenerator\Operations\Unregister.vb` | todo | — | — |
| `CAD\Modelgenerator\RenameRule.vb` | todo | — | — |
| `CAD\Modelgenerator\RenameRuleCollection.vb` | todo | — | — |
| `CAD\Modelgenerator\TriggerFile.vb` | todo | — | — |
| `CAD\OpenGL\CtrlOpenGLViewer.Designer.vb` | generated | VS designer partial | — |
| `CAD\OpenGL\CtrlOpenGLViewer.vb` | todo | — | — |
| `CAD\OpenGL\GLUtil.vb` | todo | — | — |
| `CAD\PLM\Archive.vb` | todo | — | — |
| `CAD\PLM\AutoPromotionRequest.vb` | todo | — | — |
| `CAD\PLM\AutoPromotionRequestHandler.vb` | todo | — | — |
| `CAD\PLM\AutoPromotionRequestParameters.vb` | todo | — | — |
| `CAD\PLM\Container.vb` | todo | — | — |
| `CAD\PLM\EPMDocument.vb` | todo | — | — |
| `CAD\PLM\FileServer.vb` | todo | — | — |
| `CAD\PLM\Folder.vb` | todo | — | — |
| `CAD\PLM\FrmAutomaticPromotionRequestInput.Designer.vb` | generated | VS designer partial | — |
| `CAD\PLM\FrmAutomaticPromotionRequestInput.resx` | generated | resource bundle (designer-managed) | — |
| `CAD\PLM\FrmAutomaticPromotionRequestInput.vb` | todo | — | — |
| `CAD\PLM\FrmVaultFolderChart.Designer.vb` | generated | VS designer partial | — |
| `CAD\PLM\FrmVaultFolderChart.resx` | generated | resource bundle (designer-managed) | — |
| `CAD\PLM\FrmVaultFolderChart.vb` | todo | — | — |
| `CAD\PLM\InfoEngineParameter.vb` | todo | — | — |
| `CAD\PLM\InfoEngineParameterCollection.vb` | todo | — | — |
| `CAD\PLM\LifeCycleState.vb` | todo | — | — |
| `CAD\PLM\Product.vb` | todo | — | — |
| `CAD\PLM\PromotionNotice.vb` | todo | — | — |
| `CAD\PLM\ServerManagement.vb` | todo | — | — |
| `CAD\PLM\Task\AddVaultFolderDetailsLogEntry.vb` | todo | — | — |
| `CAD\PLM\Task\GenericModelBackup.vb` | todo | — | — |
| `CAD\PLM\Task\PurgePromotionNotices.vb` | todo | — | — |
| `CAD\PLM\Task\Task.vb` | todo | — | — |
| `CAD\PLM\Toolbox.vb` | todo | — | — |
| `CAD\PLM\User.vb` | todo | — | — |
| `CAD\PLM\VaultCleanupAuditLogs.vb` | todo | — | — |
| `CAD\PLM\Version.vb` | todo | — | — |
| `CAD\PLM\WCObject.vb` | todo | — | — |
| `CAD\Publisher\Common.vb` | todo | — | — |
| `CAD\SolidEdge\IfcExport.vb` | todo | — | — |
| `CAD\SolidEdge\Importer.vb` | todo | — | — |
| `CAD\SolidEdge\Settings.vb` | todo | — | — |
| `CAD\SolidEdge\StepExport.vb` | todo | — | — |
| `CAD\SolidEdge\Toolkit.vb` | todo | — | — |

### ICenterLib / CadBatchServer

**Total: 17** &nbsp; | &nbsp; .vb: 17 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `CadBatchServer\CadBatchserverDataService.vb` | todo | — | — |
| `CadBatchServer\CadBatchserverStatus.vb` | todo | — | — |
| `CadBatchServer\CadBatchserverStatusCollection.vb` | todo | — | — |
| `CadBatchServer\CadBatchserverStatusDataService.vb` | todo | — | — |
| `CadBatchServer\DistributedLockCreoPublish.vb` | todo | — | — |
| `CadBatchServer\JobAlreadyExistsException.vb` | todo | — | — |
| `CadBatchServer\JobDataService.vb` | todo | — | — |
| `CadBatchServer\JobParameters.vb` | todo | — | — |
| `CadBatchServer\JobToolbox.vb` | todo | — | — |
| `CadBatchServer\Modelgenerator\CadInputParameters.vb` | todo | — | — |
| `CadBatchServer\Modelgenerator\DuplicateInstruction.vb` | todo | — | — |
| `CadBatchServer\Modelgenerator\ModelgeneratorInstructions.vb` | todo | — | — |
| `CadBatchServer\Modelgenerator\ModelgeneratorTask.vb` | todo | — | — |
| `CadBatchServer\ModelgeneratorDataService.vb` | todo | — | — |
| `CadBatchServer\PublishJobInstructions.vb` | todo | — | — |
| `CadBatchServer\PublishMonitor.vb` | todo | — | — |
| `CadBatchServer\PublishWatchDirProcessor.vb` | todo | — | — |

### ICenterLib / Comparer

**Total: 1** &nbsp; | &nbsp; .vb:  | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Comparer\DateComparer.vb` | todo | — | — |

### ICenterLib / Connections

**Total: 1** &nbsp; | &nbsp; .vb:  | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Connections\WebClients\WindchillWebClientProvider.vb` | todo | — | — |

### ICenterLib / CrystalReport

**Total: 5** &nbsp; | &nbsp; .vb: 5 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `CrystalReport\ExportInstruction.vb` | todo | — | — |
| `CrystalReport\ExportRequest.vb` | todo | — | — |
| `CrystalReport\ExportResult.vb` | todo | — | — |
| `CrystalReport\PrintingInstruction.vb` | todo | — | — |
| `CrystalReport\ReportParameter.vb` | todo | — | — |

### ICenterLib / DataHandler

**Total: 23** &nbsp; | &nbsp; .vb: 19 | .cs: 0 | .resx: 2

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `DataHandler\BetterDataGridView.vb` | todo | — | — |
| `DataHandler\DataSetComparer.vb` | todo | — | — |
| `DataHandler\DataSetCompareResult.vb` | todo | — | — |
| `DataHandler\DataSetCompareTolerance.vb` | todo | — | — |
| `DataHandler\DataTableColumnSchema.vb` | todo | — | — |
| `DataHandler\DataTableColumnSchemaHandler.vb` | todo | — | — |
| `DataHandler\DataTableColumnSchemaRecord.vb` | todo | — | — |
| `DataHandler\ExportExcel.vb` | todo | — | — |
| `DataHandler\FrmDataGridView.Designer.vb` | generated | VS designer partial | — |
| `DataHandler\FrmDataGridView.resx` | generated | resource bundle (designer-managed) | — |
| `DataHandler\FrmDataGridView.vb` | todo | — | — |
| `DataHandler\FrmDataViewer.Designer.vb` | generated | VS designer partial | — |
| `DataHandler\FrmDataViewer.resx` | generated | resource bundle (designer-managed) | — |
| `DataHandler\FrmDataViewer.vb` | todo | — | — |
| `DataHandler\GenericQuery.vb` | todo | — | — |
| `DataHandler\IStreamWrapper.vb` | todo | — | — |
| `DataHandler\JsonHelper.vb` | todo | — | — |
| `DataHandler\OpenXml\Excel.vb` | todo | — | — |
| `DataHandler\OpenXml\OpenXmlSpreadsheet.vb` | todo | — | — |
| `DataHandler\QrCode.vb` | todo | — | — |
| `DataHandler\Selection.vb` | todo | — | — |
| `DataHandler\Toolbox.vb` | todo | — | — |
| `DataHandler\ZeroCode.vb` | todo | — | — |

### ICenterLib / DataServices

**Total: 10** &nbsp; | &nbsp; .vb: 10 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `DataServices\Contracts\IProdexConfiguratorDataService.vb` | todo | — | — |
| `DataServices\Contracts\IProdexDossierDetailDesignDataService.vb` | todo | — | — |
| `DataServices\Contracts\IProdexModelgeneratorDataService.vb` | todo | — | — |
| `DataServices\CrystalReportDataService.vb` | todo | — | — |
| `DataServices\DistributedLockDataService.vb` | todo | — | — |
| `DataServices\ElfsquadDataService.vb` | todo | — | — |
| `DataServices\ProdexConfiguratorDataService.vb` | todo | — | — |
| `DataServices\ProdexDossierDetailDesignDataService.vb` | todo | — | — |
| `DataServices\ProdexModelgeneratorDataService.vb` | todo | — | — |
| `DataServices\ProductDbDataService.vb` | todo | — | — |

### ICenterLib / Debug

**Total: 1** &nbsp; | &nbsp; .vb:  | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Debug\Debug1.vb` | todo | — | — |

### ICenterLib / DistributedLock

**Total: 1** &nbsp; | &nbsp; .vb:  | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `DistributedLock\DistributedLock.vb` | todo | — | — |

### ICenterLib / Elfsquad

**Total: 9** &nbsp; | &nbsp; .vb: 5 | .cs: 0 | .resx: 2

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Elfsquad\ConfigurationMappingMismatchException.vb` | todo | — | — |
| `Elfsquad\ConfigurationRequestType.vb` | todo | — | — |
| `Elfsquad\ModelgeneratorHelper.vb` | todo | — | — |
| `Elfsquad\UCConfigurationManager.Designer.vb` | generated | VS designer partial | — |
| `Elfsquad\UCConfigurationManager.resx` | generated | resource bundle (designer-managed) | — |
| `Elfsquad\UCConfigurationManager.vb` | todo | — | — |
| `Elfsquad\UCModelgenerator.Designer.vb` | generated | VS designer partial | — |
| `Elfsquad\UCModelgenerator.resx` | generated | resource bundle (designer-managed) | — |
| `Elfsquad\UCModelgenerator.vb` | todo | — | — |

### ICenterLib / Enums

**Total: 2** &nbsp; | &nbsp; .vb: 2 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Enums\Application.vb` | todo | — | — |
| `Enums\ISAH.vb` | todo | — | — |

### ICenterLib / GUI

**Total: 3** &nbsp; | &nbsp; .vb: 3 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `GUI\MyMenuItem.vb` | todo | — | — |
| `GUI\MyMenuItemConverter.vb` | todo | — | — |
| `GUI\ToolStripMenuItemHelper.vb` | todo | — | — |

### ICenterLib / Helpers

**Total: 1** &nbsp; | &nbsp; .vb:  | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Helpers\UrlHelper.vb` | todo | — | — |

### ICenterLib / iCenter

**Total: 29** &nbsp; | &nbsp; .vb: 24 | .cs: 0 | .resx: 2

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `iCenter\BillOfOper.vb` | todo | — | — |
| `iCenter\Client.vb` | todo | — | — |
| `iCenter\CtrlImageIdentification.designer.vb` | generated | VS designer partial | — |
| `iCenter\CtrlImageIdentification.resx` | generated | resource bundle (designer-managed) | — |
| `iCenter\CtrlImageIdentification.vb` | todo | — | — |
| `iCenter\CtrlRadButtonIdent.designer.vb` | generated | VS designer partial | — |
| `iCenter\CtrlRadButtonIdent.vb` | todo | — | — |
| `iCenter\DataServices\CoatingLayerThicknessDataService.vb` | todo | — | — |
| `iCenter\DossierContactFavorite.vb` | todo | — | — |
| `iCenter\ExternalReference.vb` | todo | — | — |
| `iCenter\ExternalReferences.vb` | todo | — | — |
| `iCenter\FrmIdentification.designer.vb` | generated | VS designer partial | — |
| `iCenter\FrmIdentification.resx` | generated | resource bundle (designer-managed) | — |
| `iCenter\FrmIdentification.vb` | todo | — | — |
| `iCenter\IPBatch.vb` | todo | — | — |
| `iCenter\IPOrder.vb` | todo | — | — |
| `iCenter\IPPacket.vb` | todo | — | — |
| `iCenter\IPPart.vb` | todo | — | — |
| `iCenter\Part.vb` | todo | — | — |
| `iCenter\ProductionMachineMultiPurpose.vb` | todo | — | — |
| `iCenter\ProductionMachines.vb` | todo | — | — |
| `iCenter\Servicedesk.vb` | todo | — | — |
| `iCenter\SurfaceTreatmentDefinition.vb` | todo | — | — |
| `iCenter\TimeRegistration.vb` | todo | — | — |
| `iCenter\WebClockAssistant\WebClockAssistantAnonymousHandler.vb` | todo | — | — |
| `iCenter\WebClockAssistant\WebClockAssistantGenericHandler.vb` | todo | — | — |
| `iCenter\WebClockAssistant\WebClockAssistantRepository.vb` | todo | — | — |
| `iCenter\WebClockAssistant\WebClockAssistantUserSelectionHandler.vb` | todo | — | — |
| `iCenter\XmlFile.vb` | todo | — | — |

### ICenterLib / ISAH

**Total: 65** &nbsp; | &nbsp; .vb: 63 | .cs: 0 | .resx: 

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `ISAH\BillOfMat.vb` | todo | — | — |
| `ISAH\BillOfOper.vb` | todo | — | — |
| `ISAH\CallRegistration.vb` | todo | — | — |
| `ISAH\Company.vb` | todo | — | — |
| `ISAH\Contact.vb` | todo | — | — |
| `ISAH\Customer.vb` | todo | — | — |
| `ISAH\CustomerRelation.vb` | todo | — | — |
| `ISAH\CustomerSelection.vb` | todo | — | — |
| `ISAH\Database.vb` | todo | — | — |
| `ISAH\DataServices\EmployeeDataService.vb` | todo | — | — |
| `ISAH\DataServices\IsahCustomisingElfsquadDataService.vb` | todo | — | — |
| `ISAH\DataServices\MemoDetailDataService.vb` | todo | — | — |
| `ISAH\DataServices\PartDataService.vb` | todo | — | — |
| `ISAH\DataServices\PartDispatchCollectorDataService.vb` | todo | — | — |
| `ISAH\DataServices\ToolboxDataService.vb` | todo | — | — |
| `ISAH\DataServices\UpdateProdLeadTimeDataService.vb` | todo | — | — |
| `ISAH\DateDimension.vb` | todo | — | — |
| `ISAH\DeliveryLine.vb` | todo | — | — |
| `ISAH\Design.vb` | todo | — | — |
| `ISAH\DossierDetail.vb` | todo | — | — |
| `ISAH\DossierDetailExtra.vb` | todo | — | — |
| `ISAH\DossierDetailExtraDto.vb` | todo | — | — |
| `ISAH\DossierDocFolder.vb` | todo | — | — |
| `ISAH\DossierMain.vb` | todo | — | — |
| `ISAH\Employee.vb` | todo | — | — |
| `ISAH\FrmJConfigParamDesignCode.Designer.vb` | generated | VS designer partial | — |
| `ISAH\FrmJConfigParamDesignCode.resx` | generated | resource bundle (designer-managed) | — |
| `ISAH\FrmJConfigParamDesignCode.vb` | todo | — | — |
| `ISAH\Handlers\UpdateProdLeadTimeHandler.vb` | todo | — | — |
| `ISAH\Helpers\DossierDetailExtraHelper.vb` | todo | — | — |
| `ISAH\Helpers\EncryptionHelper.vb` | todo | — | — |
| `ISAH\Helpers\TextStyling\HtmlPlainTextHelper.vb` | todo | — | — |
| `ISAH\Helpers\TextStyling\IPlainTextHelper.vb` | todo | — | — |
| `ISAH\Helpers\TextStyling\PlainTextHelper.vb` | todo | — | — |
| `ISAH\Helpers\TextStyling\RtfPlainTextHelper.vb` | todo | — | — |
| `ISAH\Icenter2Isah.vb` | todo | — | — |
| `ISAH\IsahFieldML.vb` | todo | — | — |
| `ISAH\JConfigParam.vb` | todo | — | — |
| `ISAH\Language.vb` | todo | — | — |
| `ISAH\MachGrp.vb` | todo | — | — |
| `ISAH\MemoDetailElfsquadConfiguration.vb` | todo | — | — |
| `ISAH\MultiFinance.vb` | todo | — | — |
| `ISAH\Part.vb` | todo | — | — |
| `ISAH\PartDispatch.vb` | todo | — | — |
| `ISAH\PartDispatchCollectorDataService.vb` | todo | — | — |
| `ISAH\PartSelection.vb` | todo | — | — |
| `ISAH\PartVendor.vb` | todo | — | — |
| `ISAH\PBOM.vb` | todo | — | — |
| `ISAH\PBOO.vb` | todo | — | — |
| `ISAH\PBOS.vb` | todo | — | — |
| `ISAH\ProductionHeader.vb` | todo | — | — |
| `ISAH\PurchaseDocumentPartLine.vb` | todo | — | — |
| `ISAH\PurDoc.vb` | todo | — | — |
| `ISAH\Selection.vb` | todo | — | — |
| `ISAH\Setting.vb` | todo | — | — |
| `ISAH\ShopDoc.vb` | todo | — | — |
| `ISAH\ShopDocCollection.vb` | todo | — | — |
| `ISAH\TimeRegCollector.vb` | todo | — | — |
| `ISAH\TimeRegistration.vb` | todo | — | — |
| `ISAH\User.vb` | todo | — | — |
| `ISAH\Vendor.vb` | todo | — | — |
| `ISAH\ViewModels\CustomerAddressViewModel.vb` | todo | — | — |
| `ISAH\ViewModels\PartBasicViewModel.vb` | todo | — | — |
| `ISAH\WeighingFactor.vb` | todo | — | — |
| `ISAH\WorkView.vb` | todo | — | — |

### ICenterLib / JIBA

**Total: 14** &nbsp; | &nbsp; .vb: 14 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `JIBA\AppParameter.vb` | todo | — | — |
| `JIBA\Asset.vb` | todo | — | — |
| `JIBA\Company.vb` | todo | — | — |
| `JIBA\ConfigPart.vb` | todo | — | — |
| `JIBA\CustSat.vb` | todo | — | — |
| `JIBA\Employee.vb` | todo | — | — |
| `JIBA\Encryption.vb` | todo | — | — |
| `JIBA\Enums.vb` | todo | — | — |
| `JIBA\LinkItem.vb` | todo | — | — |
| `JIBA\Log.vb` | todo | — | — |
| `JIBA\Menu.vb` | todo | — | — |
| `JIBA\NavigationGroupItem.vb` | todo | — | — |
| `JIBA\SubMenu.vb` | todo | — | — |
| `JIBA\XmlData.vb` | todo | — | — |

### ICenterLib / JMail

**Total: 6** &nbsp; | &nbsp; .vb: 6 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `JMail\FileHelper.vb` | todo | — | — |
| `JMail\Message.vb` | todo | — | — |
| `JMail\SpecificationReportData.vb` | todo | — | — |
| `JMail\SpecificationReportHandler.vb` | todo | — | — |
| `JMail\StartOptions.vb` | todo | — | — |
| `JMail\StartOptionsHelper.vb` | todo | — | — |

### ICenterLib / LaserWork

**Total: 4** &nbsp; | &nbsp; .vb: 2 | .cs: 0 | .resx: 

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `LaserWork\AppWrapper.vb` | todo | — | — |
| `LaserWork\UCLaserWork.Designer.vb` | generated | VS designer partial | — |
| `LaserWork\UCLaserWork.resx` | generated | resource bundle (designer-managed) | — |
| `LaserWork\UCLaserWork.vb` | todo | — | — |

### ICenterLib / Metabase

**Total: 4** &nbsp; | &nbsp; .vb: 4 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Metabase\ParameterParsing.vb` | todo | — | — |
| `Metabase\ParameterResolver.vb` | todo | — | — |
| `Metabase\QueryHelper.vb` | todo | — | — |
| `Metabase\UrlHelper.vb` | todo | — | — |

### ICenterLib / ModelDefinition

**Total: 3** &nbsp; | &nbsp; .vb: 3 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `ModelDefinition\Dimension.vb` | todo | — | — |
| `ModelDefinition\DimensionCollection.vb` | todo | — | — |
| `ModelDefinition\DimensionType.vb` | todo | — | — |

### ICenterLib / MySystem

**Total: 22** &nbsp; | &nbsp; .vb: 20 | .cs: 0 | .resx: 

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `MySystem\Computer.vb` | todo | — | — |
| `MySystem\Encryption\FrmEncrypt.Designer.vb` | generated | VS designer partial | — |
| `MySystem\Encryption\FrmEncrypt.resx` | generated | resource bundle (designer-managed) | — |
| `MySystem\Encryption\FrmEncrypt.vb` | todo | — | — |
| `MySystem\Encryption\SecurityController.vb` | todo | — | — |
| `MySystem\Environment.vb` | todo | — | — |
| `MySystem\ExceptionList.vb` | todo | — | — |
| `MySystem\FileSystem.vb` | todo | — | — |
| `MySystem\HealthMonitorClient.vb` | todo | — | — |
| `MySystem\HelpHandler.vb` | todo | — | — |
| `MySystem\ICenterObjectNotFoundException.vb` | todo | — | — |
| `MySystem\Math.vb` | todo | — | — |
| `MySystem\MyProcess.vb` | todo | — | — |
| `MySystem\Net\TcpServer.vb` | todo | — | — |
| `MySystem\Network.vb` | todo | — | — |
| `MySystem\OpenWindowGetter.vb` | todo | — | — |
| `MySystem\PowerShellWrapper.vb` | todo | — | — |
| `MySystem\Printer.vb` | todo | — | — |
| `MySystem\Registry.vb` | todo | — | — |
| `MySystem\TerminalServerSessions.vb` | todo | — | — |
| `MySystem\Window.vb` | todo | — | — |
| `MySystem\WindowsUser.vb` | todo | — | — |

### ICenterLib / PCFNet

**Total: 45** &nbsp; | &nbsp; .vb: 39 | .cs: 0 | .resx: 3

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `PCFNet\BOM.vb` | todo | — | — |
| `PCFNet\BOO.vb` | todo | — | — |
| `PCFNet\Calculation.vb` | todo | — | — |
| `PCFNet\ControlDefinition.vb` | todo | — | — |
| `PCFNet\ControlDefinitionComparer.vb` | todo | — | — |
| `PCFNet\ControlMapping\ControlMap.vb` | todo | — | — |
| `PCFNet\ControlMapping\Mapping.vb` | todo | — | — |
| `PCFNet\ControlMapping\MappingCollection.vb` | todo | — | — |
| `PCFNet\ControlMapping\MappingQuery.vb` | todo | — | — |
| `PCFNet\ControlMapping\Parameter.vb` | todo | — | — |
| `PCFNet\ControlMappingDesigner\UCControlMappingDesigner.Designer.vb` | generated | VS designer partial | — |
| `PCFNet\ControlMappingDesigner\UCControlMappingDesigner.resx` | generated | resource bundle (designer-managed) | — |
| `PCFNet\ControlMappingDesigner\UCControlMappingDesigner.vb` | todo | — | — |
| `PCFNet\ControlMappingImport.vb` | todo | — | — |
| `PCFNet\CPart.vb` | todo | — | — |
| `PCFNet\ExcelObject.vb` | todo | — | — |
| `PCFNet\ExcelObjectCalculator.vb` | todo | — | — |
| `PCFNet\FastenerCalculator.vb` | todo | — | — |
| `PCFNet\GenericPart.vb` | todo | — | — |
| `PCFNet\GenericPartTemplate.vb` | todo | — | — |
| `PCFNet\JConfigurator.vb` | todo | — | — |
| `PCFNet\ManualOperation.vb` | todo | — | — |
| `PCFNet\ManualProperty.vb` | todo | — | — |
| `PCFNet\NameMapping.vb` | todo | — | — |
| `PCFNet\NameMappingRule.vb` | todo | — | — |
| `PCFNet\ParametersMapping\AluBasicWallLouver001.vb` | todo | — | — |
| `PCFNet\ParametersMapping\AluDoor001.vb` | todo | — | — |
| `PCFNet\ParametersMapping\AluLouver001.vb` | todo | — | — |
| `PCFNet\ParametersMapping\ParametersMappingBase.vb` | todo | — | — |
| `PCFNet\ParametersMapping\PlankAssembly.vb` | todo | — | — |
| `PCFNet\ParametersMapping\StlDoor001.vb` | todo | — | — |
| `PCFNet\PcfCompiler.vb` | todo | — | — |
| `PCFNet\PcfNetDataSet.vb` | todo | — | — |
| `PCFNet\ProductValidation.vb` | todo | — | — |
| `PCFNet\PropertyDefinition.vb` | todo | — | — |
| `PCFNet\PropertyDefinitionType.vb` | todo | — | — |
| `PCFNet\SmtCalculator.vb` | todo | — | — |
| `PCFNet\TcpClientConfiguration.vb` | todo | — | — |
| `PCFNet\UCInputControls.Designer.vb` | generated | VS designer partial | — |
| `PCFNet\UCInputControls.resx` | generated | resource bundle (designer-managed) | — |
| `PCFNet\UCInputControls.vb` | todo | — | — |
| `PCFNet\UCOption.Designer.vb` | generated | VS designer partial | — |
| `PCFNet\UCOption.resx` | generated | resource bundle (designer-managed) | — |
| `PCFNet\UCOption.vb` | todo | — | — |
| `PCFNet\WAASHandler.vb` | todo | — | — |

### ICenterLib / Prodex

**Total: 6** &nbsp; | &nbsp; .vb: 6 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Prodex\Models\DesignConfiguration.vb` | todo | — | — |
| `Prodex\Models\DesignParameter.vb` | todo | — | — |
| `Prodex\Models\DesignParameters.vb` | todo | — | — |
| `Prodex\Models\DossierDetailDesignCalculationRequest.vb` | todo | — | — |
| `Prodex\Models\DossierDetailDesignUpdateRequest.vb` | todo | — | — |
| `Prodex\ViewModels\DossierDetailDesignCalculationVM.vb` | todo | — | — |

### ICenterLib / ProductDb

**Total: 31** &nbsp; | &nbsp; .vb: 21 | .cs: 0 | .resx: 5

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `ProductDb\CloneProductHandler.vb` | todo | — | — |
| `ProductDb\CommonDb.vb` | todo | — | — |
| `ProductDb\DeclarationOfPerformance.vb` | todo | — | — |
| `ProductDb\ExcelTemplate.vb` | todo | — | — |
| `ProductDb\ExcelWorkBookHelper.vb` | todo | — | — |
| `ProductDb\FrmCloneProduct.Designer.vb` | generated | VS designer partial | — |
| `ProductDb\FrmCloneProduct.resx` | generated | resource bundle (designer-managed) | — |
| `ProductDb\FrmCloneProduct.vb` | todo | — | — |
| `ProductDb\FrmProductDbPriceList.Designer.vb` | generated | VS designer partial | — |
| `ProductDb\FrmProductDbPriceList.resx` | generated | resource bundle (designer-managed) | — |
| `ProductDb\FrmProductDbPriceList.vb` | todo | — | — |
| `ProductDb\FrmProductPricePart.Designer.vb` | generated | VS designer partial | — |
| `ProductDb\FrmProductPricePart.resx` | generated | resource bundle (designer-managed) | — |
| `ProductDb\FrmProductPricePart.vb` | todo | — | — |
| `ProductDb\Language.vb` | todo | — | — |
| `ProductDb\PriceListHelper.vb` | todo | — | — |
| `ProductDb\PricePart.vb` | todo | — | — |
| `ProductDb\Product.vb` | todo | — | — |
| `ProductDb\ProductConfiguration.vb` | todo | — | — |
| `ProductDb\ProductConfigurationDataService.vb` | todo | — | — |
| `ProductDb\ProductConfiguratorMapping.vb` | todo | — | — |
| `ProductDb\ProductFilter.vb` | todo | — | — |
| `ProductDb\ProductFilterGroup.vb` | todo | — | — |
| `ProductDb\ProductGroup.vb` | todo | — | — |
| `ProductDb\ProductPricePart.vb` | todo | — | — |
| `ProductDb\UCPropertyDefinitionEditor.Designer.vb` | generated | VS designer partial | — |
| `ProductDb\UCPropertyDefinitionEditor.resx` | generated | resource bundle (designer-managed) | — |
| `ProductDb\UCPropertyDefinitionEditor.vb` | todo | — | — |
| `ProductDb\UCPropertyDefinitionManager.Designer.vb` | generated | VS designer partial | — |
| `ProductDb\UCPropertyDefinitionManager.resx` | generated | resource bundle (designer-managed) | — |
| `ProductDb\UCPropertyDefinitionManager.vb` | todo | — | — |

### ICenterLib / Production

**Total: 23** &nbsp; | &nbsp; .vb: 19 | .cs: 0 | .resx: 2

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Production\AutoIPOrderSelection.vb` | todo | — | — |
| `Production\BaseProductionItem.vb` | todo | — | — |
| `Production\BOMFilter.vb` | todo | — | — |
| `Production\CEChecklist.vb` | todo | — | — |
| `Production\FGQualityControlPart.vb` | todo | — | — |
| `Production\FrmProdMachineSelector.Designer.vb` | generated | VS designer partial | — |
| `Production\FrmProdMachineSelector.resx` | generated | resource bundle (designer-managed) | — |
| `Production\FrmProdMachineSelector.vb` | todo | — | — |
| `Production\KanbanBin.vb` | todo | — | — |
| `Production\KeyPerformanceIndicator.vb` | todo | — | — |
| `Production\LabelLog.vb` | todo | — | — |
| `Production\ProdChecklist.vb` | todo | — | — |
| `Production\ProdChecklistTag.vb` | todo | — | — |
| `Production\ProductionLine.vb` | todo | — | — |
| `Production\ProductionLog.vb` | todo | — | — |
| `Production\ProductionProfileCutItem.vb` | todo | — | — |
| `Production\ProductionProfileCutItemHandler.vb` | todo | — | — |
| `Production\ProductionProfileCutItemsHandler.vb` | todo | — | — |
| `Production\ProductionRegistrationAnalysisRange.vb` | todo | — | — |
| `Production\SmtBendQueue.vb` | todo | — | — |
| `Production\UCProdLineLeanStatus.Designer.vb` | generated | VS designer partial | — |
| `Production\UCProdLineLeanStatus.resx` | generated | resource bundle (designer-managed) | — |
| `Production\UCProdLineLeanStatus.vb` | todo | — | — |

### ICenterLib / Resources

**Total: 75** &nbsp; | &nbsp; .vb: 0 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Resources\1downarrow1-32.png` | config | image/icon asset | — |
| `Resources\1leftarrow-32.png` | config | image/icon asset | — |
| `Resources\1rightarrow-32.png` | config | image/icon asset | — |
| `Resources\1uparrow-32.png` | config | image/icon asset | — |
| `Resources\2008-09-23 JAZO witte omranding 3cm.jpg` | config | image/icon asset | — |
| `Resources\2dowarrow-32.png` | config | image/icon asset | — |
| `Resources\2leftarrow-32.png` | config | image/icon asset | — |
| `Resources\2rightarrow-32.png` | config | image/icon asset | — |
| `Resources\2uparrow-32.png` | config | image/icon asset | — |
| `Resources\Add Button-24.png` | config | image/icon asset | — |
| `Resources\Add_green_16.png` | config | image/icon asset | — |
| `Resources\Batch_128.png` | config | image/icon asset | — |
| `Resources\Bullet-Black-16.png` | config | image/icon asset | — |
| `Resources\button_cancel-24.png` | config | image/icon asset | — |
| `Resources\Check-16.png` | config | image/icon asset | — |
| `Resources\checkbox_checked.gif` | config | image/icon asset | — |
| `Resources\checkbox_checked_png.png` | config | image/icon asset | — |
| `Resources\checkbox_unchecked.gif` | config | image/icon asset | — |
| `Resources\checkbox_unchecked_png.png` | config | image/icon asset | — |
| `Resources\Copy_32.png` | config | image/icon asset | — |
| `Resources\creo_logo_16.png` | config | image/icon asset | — |
| `Resources\creo_logo_32.jpg` | config | image/icon asset | — |
| `Resources\creo_regen_jz_16.png` | config | image/icon asset | — |
| `Resources\creo_regen_jz_24.png` | config | image/icon asset | — |
| `Resources\creo_with_regen_jz_16.png` | config | image/icon asset | — |
| `Resources\Edit-32.png` | config | image/icon asset | — |
| `Resources\elfsquad-logo-16.png` | config | image/icon asset | — |
| `Resources\elfsquad-logo-large.png` | config | image/icon asset | — |
| `Resources\ErrorX-32.png` | config | image/icon asset | — |
| `Resources\excel_16.png` | config | image/icon asset | — |
| `Resources\excel_24.png` | config | image/icon asset | — |
| `Resources\excel_32.png` | config | image/icon asset | — |
| `Resources\excel_64.png` | config | image/icon asset | — |
| `Resources\fabpartseverity_1.png` | config | image/icon asset | — |
| `Resources\fabpartseverity_2.png` | config | image/icon asset | — |
| `Resources\fabpartstatus_1.png` | config | image/icon asset | — |
| `Resources\fabpartstatus_12.png` | config | image/icon asset | — |
| `Resources\fabpartstatus_13.png` | config | image/icon asset | — |
| `Resources\fabpartstatus_14.png` | config | image/icon asset | — |
| `Resources\fabpartstatus_2.png` | config | image/icon asset | — |
| `Resources\fabpartstatus_20.png` | config | image/icon asset | — |
| `Resources\fabpartstatus_3.png` | config | image/icon asset | — |
| `Resources\fabpartstatus_4.png` | config | image/icon asset | — |
| `Resources\fabpartstatus_5.png` | config | image/icon asset | — |
| `Resources\fabpartstatus_6.png` | config | image/icon asset | — |
| `Resources\fabpartstatus_7.png` | config | image/icon asset | — |
| `Resources\fabpartstatus_70.png` | config | image/icon asset | — |
| `Resources\fabpartstatus_8.png` | config | image/icon asset | — |
| `Resources\Female_64.png` | config | image/icon asset | — |
| `Resources\filesave-16.png` | config | image/icon asset | — |
| `Resources\filesave-32.png` | config | image/icon asset | — |
| `Resources\Gnome-Stock-Person-64.png` | config | image/icon asset | — |
| `Resources\gridview.jpg` | config | image/icon asset | — |
| `Resources\iCenter.ico` | config | image/icon asset | — |
| `Resources\import_24.png` | config | image/icon asset | — |
| `Resources\info_blue_16.png` | config | image/icon asset | — |
| `Resources\info_blue_161.png` | config | image/icon asset | — |
| `Resources\info_blue-32.png` | config | image/icon asset | — |
| `Resources\Interface-builder_16.png` | config | image/icon asset | — |
| `Resources\merge-icon-16x16.png` | config | image/icon asset | — |
| `Resources\merge-icon-32x32.png` | config | image/icon asset | — |
| `Resources\Modelgenerator-creo-icon.png` | config | image/icon asset | — |
| `Resources\Person-Undefined-Male-Light-64.png` | config | image/icon asset | — |
| `Resources\Refresh-32.png` | config | image/icon asset | — |
| `Resources\Resources-32.png` | config | image/icon asset | — |
| `Resources\Setting-16.png` | config | image/icon asset | — |
| `Resources\Setting-64.png` | config | image/icon asset | — |
| `Resources\texture_alu01.bmp` | config | image/icon asset | — |
| `Resources\warning_anim_32.gif` | config | image/icon asset | — |
| `Resources\Warning-32 (1).png` | config | image/icon asset | — |
| `Resources\Warning-64.png` | config | image/icon asset | — |
| `Resources\Windchill11.png` | config | image/icon asset | — |
| `Resources\Zammad_icon_24.png` | config | image/icon asset | — |
| `Resources\Zammad_icon_32.png` | config | image/icon asset | — |
| `Resources\Zammad_icon_64.png` | config | image/icon asset | — |

### ICenterLib / SmartForms

**Total: 1** &nbsp; | &nbsp; .vb:  | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `SmartForms\SmartForm.vb` | todo | — | — |

### ICenterLib / SmtCadCam

**Total: 4** &nbsp; | &nbsp; .vb: 4 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `SmtCadCam\PreProcessorHandler.vb` | todo | — | — |
| `SmtCadCam\SpaceClaimApiHelper.vb` | todo | — | — |
| `SmtCadCam\SpaceClaimServerRequest.vb` | todo | — | — |
| `SmtCadCam\SpaceClaimServerRequestDataService.vb` | todo | — | — |

### ICenterLib / SmtProduction

**Total: 128** &nbsp; | &nbsp; .vb: 126 | .cs: 0 | .resx: 

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `SmtProduction\DataServices\CutSheetOperRegistrationDataService.vb` | todo | — | — |
| `SmtProduction\DataServices\OseonAppContextDataService.vb` | todo | — | — |
| `SmtProduction\DataServices\PartDataService.vb` | todo | — | — |
| `SmtProduction\DataServices\ProductionOrderDataService.vb` | todo | — | — |
| `SmtProduction\DataServices\WorkplaceEmployeeLinkDataService.vb` | todo | — | — |
| `SmtProduction\Entities\CutSheetOperRegistration.vb` | todo | — | — |
| `SmtProduction\Entities\ImportSettings2D.vb` | todo | — | — |
| `SmtProduction\Entities\ImportSettings3D.vb` | todo | — | — |
| `SmtProduction\Entities\ImportSettingsBase.vb` | todo | — | — |
| `SmtProduction\Entities\OseonAppContext.vb` | todo | — | — |
| `SmtProduction\Entities\WorkplaceEmployeeLink.vb` | todo | — | — |
| `SmtProduction\ImportFileValidator.vb` | todo | — | — |
| `SmtProduction\ProgrammingEnvironment.vb` | todo | — | — |
| `SmtProduction\TruTops\Client\Application.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Collections\CadCamDocumentCollection.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Contracts\IAppMerkerDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Contracts\IBasicMaterialDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Contracts\IBendSolutionDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Contracts\ICadCamDocumentDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Contracts\ICutSheetDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Contracts\IDataTableColumnSchemaDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Contracts\IMasterWorkPlanDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Contracts\IPartBendSolutionDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Contracts\IPartDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Contracts\IPartOnTableDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Contracts\IPartStatusDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Contracts\IPdmDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Contracts\IProductionOperationDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Contracts\IProductionOrderDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Contracts\IRawMaterialDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Contracts\ITTNGActionDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Contracts\IWorkplaceDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\DataServices\AppMerkerDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\DataServices\BasicMaterialDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\DataServices\BendSolutionDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\DataServices\CadCamDocumentDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\DataServices\CutSheetDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\DataServices\MasterWorkPlanDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\DataServices\PartBendSolutionDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\DataServices\PartDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\DataServices\PartOnTableDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\DataServices\PartStatusDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\DataServices\PdmDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\DataServices\ProductionOperationDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\DataServices\ProductionOrderDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\DataServices\RawMaterialDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\DataServices\TTNGActionDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\DataServices\WorkplaceDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Entities\AppMerker.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Entities\BasicMaterial.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Entities\CadCamDocument.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Entities\CadCamDocumentType.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Entities\CutSheet.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Entities\MasterWorkPlan.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Entities\Operation.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Entities\Part.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Entities\PartBendSolution.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Entities\PartDisplayStatus.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Entities\PartOnTable.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Entities\PartStatus.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Entities\PartStatusMaster.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Entities\ProductionOperation.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Entities\ProductionOrder.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Entities\RawMaterial.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Entities\TTNGBendToolList.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\Entities\Workplace.vb` | todo | — | — |
| `SmtProduction\TruTops\Oseon\PDM.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\DataServices\ImportLogDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\DataServices\ImportResultDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\DataServices\PPSInterfaceDataService.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Exceptions\PropertyNotSpecifiedException.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjectItems\FeedbackObjectItemBase.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjectItems\FeedbackObjectItemDimensions.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjectItems\FeedbackObjectItemPartOnSheet.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjectItems\FeedbackObjectItemSheet.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjects\FeedbackObjectBase.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjects\FeedbackObjectPDAMessage.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjects\FeedbackObjectProcessedSheetReport.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjects\FeedbackObjectProductionOperation.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjects\FeedbackObjectProductionOrder.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Export\PPSExport\PPSExportBase.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Export\PPSExport\PPSExportConsumptionReport.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Export\PPSExport\PPSExportManufacturedSheet.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Export\PPSExport\PPSExportOperation.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Export\PPSExport\PPSExportPDAMessage.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Export\PPSExport\PPSExportPPSExportManufacturedSheetOper.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Export\PPSExport\PPSExportProductionOrder.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Export\ProductionOrderExportHandler.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Export\ProductionQuantityReport.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\Collections\BendSolutionCollection.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\Collections\DeleteProductionOrderCollection.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\Collections\DeleteProductionOrderCollectionXmlHandler.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\Collections\DocumentCollection.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\Collections\FinishProductionOrderCollection.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\Collections\PartCollection.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\Collections\ProductionOrderCollection.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\BendSolution.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\DeleteProductionOrder.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\Document.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\FinishProductionOrder.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\ImportResult.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\Part.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\PPSImport.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\PPSInterface.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\PPSInterfaceDate.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\ProductionOrder.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\WorkingPlan.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\WorkingStep.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\ProductionOrderImportHandler.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\BendSolutionXmlHandler.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\DeleteProductionOrderXmlHandler.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\DocumentXmlHandler.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\FinishProductionOrderCollectionXmlHandler.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\FinishProductionOrderXmlHandler.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\PartCollectionXmlHandler.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\PartXmlHandler.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\PPSImportXmlHandler.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\ProductionOrderCollectionXmlHandler.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\ProductionOrderXmlHandler.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\WorkingPlanXmlHandler.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\WorkingStepXmlHandler.vb` | todo | — | — |
| `SmtProduction\TruTops\PPSInterface\Utils\TimeConversionHelper.vb` | todo | — | — |
| `SmtProduction\TruTops\Utils\ProductionOrderNumberHelper.vb` | todo | — | — |
| `SmtProduction\TruTops\Utils\TruTopsConvertHandler.vb` | todo | — | — |
| `SmtProduction\UI\CtrlTTNGProcessingErrorInfo.Designer.vb` | generated | VS designer partial | — |
| `SmtProduction\UI\CtrlTTNGProcessingErrorInfo.resx` | generated | resource bundle (designer-managed) | — |
| `SmtProduction\UI\CtrlTTNGProcessingErrorInfo.vb` | todo | — | — |
| `SmtProduction\UI\OseonAppContextSelectionWrapper.vb` | todo | — | — |

### ICenterLib / STEP3D

**Total: 5** &nbsp; | &nbsp; .vb: 5 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `STEP3D\AssySplitter.vb` | todo | — | — |
| `STEP3D\DefinitionAnalyser.vb` | todo | — | — |
| `STEP3D\Model.vb` | todo | — | — |
| `STEP3D\Product.vb` | todo | — | — |
| `STEP3D\Reader.vb` | todo | — | — |

### ICenterLib / Ticketing

**Total: 5** &nbsp; | &nbsp; .vb: 5 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Ticketing\ApiConnector.vb` | todo | — | — |
| `Ticketing\Article.vb` | todo | — | — |
| `Ticketing\Attachment.vb` | todo | — | — |
| `Ticketing\Ticket.vb` | todo | — | — |
| `Ticketing\TicketHandler.vb` | todo | — | — |

### ICenterLib / TimeRegistration

**Total: 2** &nbsp; | &nbsp; .vb: 2 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `TimeRegistration\MovingEmployeeTimeRegistration.vb` | todo | — | — |
| `TimeRegistration\WebClock.vb` | todo | — | — |

### ICenterLib / UserControls

**Total: 34** &nbsp; | &nbsp; .vb: 15 | .cs: 0 | .resx: 8

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `UserControls\DataGridViewFormatter.vb` | todo | — | — |
| `UserControls\FriendlyComboBox.Designer.vb` | generated | VS designer partial | — |
| `UserControls\FriendlyComboBox.resx` | generated | resource bundle (designer-managed) | — |
| `UserControls\FriendlyComboBox.vb` | todo | — | — |
| `UserControls\FrmRemoteDesktop.Designer.vb` | generated | VS designer partial | — |
| `UserControls\FrmRemoteDesktop.resx` | generated | resource bundle (designer-managed) | — |
| `UserControls\FrmRemoteDesktop.vb` | todo | — | — |
| `UserControls\FrmUserSelection.Designer.vb` | generated | VS designer partial | — |
| `UserControls\FrmUserSelection.resx` | generated | resource bundle (designer-managed) | — |
| `UserControls\FrmUserSelection.vb` | todo | — | — |
| `UserControls\FrmWebView.Designer.vb` | generated | VS designer partial | — |
| `UserControls\FrmWebView.resx` | generated | resource bundle (designer-managed) | — |
| `UserControls\FrmWebView.vb` | todo | — | — |
| `UserControls\HintComboBox.Designer.vb` | generated | VS designer partial | — |
| `UserControls\HintComboBox.vb` | todo | — | — |
| `UserControls\ListViewWithReordering.vb` | todo | — | — |
| `UserControls\ProProgramEditor.Designer.vb` | generated | VS designer partial | — |
| `UserControls\ProProgramEditor.vb` | todo | — | — |
| `UserControls\Ticker.vb` | todo | — | — |
| `UserControls\UCDossierDetailDesign.Designer.vb` | generated | VS designer partial | — |
| `UserControls\UCDossierDetailDesign.resx` | generated | resource bundle (designer-managed) | — |
| `UserControls\UCDossierDetailDesign.vb` | todo | — | — |
| `UserControls\UCFastColoredTextBox.Designer.vb` | generated | VS designer partial | — |
| `UserControls\UCFastColoredTextBox.vb` | todo | — | — |
| `UserControls\UCNoAccess.Designer.vb` | generated | VS designer partial | — |
| `UserControls\UCNoAccess.resx` | generated | resource bundle (designer-managed) | — |
| `UserControls\UCNoAccess.vb` | todo | — | — |
| `UserControls\UCTicker.Designer.vb` | generated | VS designer partial | — |
| `UserControls\UCTicker.resx` | generated | resource bundle (designer-managed) | — |
| `UserControls\UCTicker.vb` | todo | — | — |
| `UserControls\UCWebView.Designer.vb` | generated | VS designer partial | — |
| `UserControls\UCWebView.resx` | generated | resource bundle (designer-managed) | — |
| `UserControls\UCWebView.vb` | todo | — | — |
| `UserControls\XWikiForm.vb` | todo | — | — |

### ICenterLib / Zabbix

**Total: 1** &nbsp; | &nbsp; .vb:  | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Zabbix\ZabbixHandler.vb` | todo | — | — |


## Roll-up

_Heuristic snapshot updated whenever new projects come into scope or files transition out of `todo`._

### By project

| Project | Total | Done | Todo | Config | Generated | Dead | Needs-review |
|---------|------:|-----:|-----:|-------:|----------:|-----:|-------------:|
| iCENTER | 1237 | 36 | 496 | 358 | 341 | 0 | 6 |
| TruTopsLib | 65 | 0 | 57 | 6 | 2 | 0 | 0 |
| ICenterLib | 723 | 0 | 571 | 81 | 71 | 0 | 0 |
| **TOTAL** | **2025** | **36** | **1124** | **445** | **414** | **0** | **6** |

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
| Engineering | 21 | 0 | 9 | 0 | 12 | 0 | 0 |
| Forms | 188 | 0 | 64 | 0 | 124 | 0 | 0 |
| IcImporter | 6 | 0 | 4 | 0 | 2 | 0 | 0 |
| Kardex | 4 | 0 | 2 | 0 | 2 | 0 | 0 |
| MarkTool | 5 | 0 | 3 | 0 | 2 | 0 | 0 |
| Modules | 3 | 1 | 2 | 0 | 0 | 0 | 0 |
| PCFNetStudio | 17 | 0 | 7 | 0 | 10 | 0 | 0 |
| Production | 6 | 0 | 6 | 0 | 0 | 0 | 0 |
| Resources | 354 | 0 | 0 | 354 | 0 | 0 | 0 |
| Sales | 3 | 0 | 1 | 0 | 2 | 0 | 0 |
| SmtManufacturing | 78 | 0 | 47 | 0 | 31 | 0 | 0 |
| SolaDataConnector | 2 | 0 | 2 | 0 | 0 | 0 | 0 |
| UniLink | 30 | 0 | 28 | 0 | 2 | 0 | 0 |
| VentDuctConfigurator | 4 | 0 | 2 | 0 | 2 | 0 | 0 |
| WebClock | 13 | 0 | 5 | 0 | 8 | 0 | 0 |
| WorkPreparation | 6 | 0 | 4 | 0 | 2 | 0 | 0 |
| iCENTER **TOTAL** | **1237** | **36** | **496** | **358** | **341** | **0** | **6** |
