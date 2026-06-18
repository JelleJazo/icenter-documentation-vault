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
| `Batchserver\BatchserverToolkit.vb` | todo | â€” | â€” |
| `Batchserver\FrmBatchServer.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Batchserver\FrmBatchServer.resx` | generated | resource bundle (designer-managed) | â€” |
| `Batchserver\FrmBatchServer.vb` | todo | â€” | â€” |

### CAD

**Total: 3** &nbsp; | &nbsp; .vb: 2 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `CAD\PLM\FrmPDMLinkOrdTypeContext.Designer.vb` | generated | VS Forms designer partial | â€” |
| `CAD\PLM\FrmPDMLinkOrdTypeContext.resx` | generated | resource bundle (designer-managed) | â€” |
| `CAD\PLM\FrmPDMLinkOrdTypeContext.vb` | todo | â€” | â€” |

### CadBatchserver

**Total: 29** &nbsp; | &nbsp; .vb: 27 | .resx: 2 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `CadBatchserver\BatchServerWatch.vb` | todo | â€” | â€” |
| `CadBatchserver\CadBatchserverTools.vb` | todo | â€” | â€” |
| `CadBatchserver\FormRunJob.Designer.vb` | generated | VS Forms designer partial | â€” |
| `CadBatchserver\FormRunJob.resx` | generated | resource bundle (designer-managed) | â€” |
| `CadBatchserver\FormRunJob.vb` | todo | â€” | â€” |
| `CadBatchserver\FrmCadBatchServer.Designer.vb` | generated | VS Forms designer partial | â€” |
| `CadBatchserver\FrmCadBatchServer.resx` | generated | resource bundle (designer-managed) | â€” |
| `CadBatchserver\FrmCadBatchServer.vb` | todo | â€” | â€” |
| `CadBatchserver\Job.vb` | todo | â€” | â€” |
| `CadBatchserver\JobArchive.vb` | todo | â€” | â€” |
| `CadBatchserver\JobAutoManufacturing.vb` | todo | â€” | â€” |
| `CadBatchserver\JobCreateProdOrd.vb` | todo | â€” | â€” |
| `CadBatchserver\JobCreatePurOrdDocs.vb` | todo | â€” | â€” |
| `CadBatchserver\JobEngOrdFinNotification.vb` | todo | â€” | â€” |
| `CadBatchserver\JobGenerateAndReleaseModel.vb` | todo | â€” | â€” |
| `CadBatchserver\JobGenericModelBackup.vb` | todo | â€” | â€” |
| `CadBatchserver\JobGeo2Dxf.vb` | todo | â€” | â€” |
| `CadBatchserver\JobModelGeneratorCreo.vb` | todo | â€” | â€” |
| `CadBatchserver\JobPartDispatch.vb` | todo | â€” | â€” |
| `CadBatchserver\JobProductionRegistration.vb` | todo | â€” | â€” |
| `CadBatchserver\JobPublishCreo.vb` | todo | â€” | â€” |
| `CadBatchserver\JobRebootMonitor.vb` | todo | â€” | â€” |
| `CadBatchserver\JobSendEmail.vb` | todo | â€” | â€” |
| `CadBatchserver\JobSmtOperSync.vb` | todo | â€” | â€” |
| `CadBatchserver\JobSmtPartDispatch.vb` | todo | â€” | â€” |
| `CadBatchserver\Modelgenerator\ModelgeneratorTask.vb` | todo | â€” | â€” |
| `CadBatchserver\Publisher\Trailfile.vb` | todo | â€” | â€” |
| `CadBatchserver\PublishWatchDirProcessor.vb` | todo | â€” | â€” |
| `CadBatchserver\ServerReboot.vb` | todo | â€” | â€” |

### CAM

**Total: 4** &nbsp; | &nbsp; .vb: 4 | .resx: 0 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `CAM\ManufMachine.vb` | todo | â€” | â€” |
| `CAM\ManufPart.vb` | todo | â€” | â€” |
| `CAM\ManufPartItem.vb` | todo | â€” | â€” |
| `CAM\ManufPartMember.vb` | todo | â€” | â€” |

### Classes

**Total: 111** &nbsp; | &nbsp; .vb: 101 | .resx: 10 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Classes\ApplicationLog.vb` | todo | â€” | â€” |
| `Classes\AutoDocAttach.vb` | todo | â€” | â€” |
| `Classes\AxControl.vb` | todo | â€” | â€” |
| `Classes\BriefcaseCollection.vb` | todo | â€” | â€” |
| `Classes\BriefcaseItem.vb` | todo | â€” | â€” |
| `Classes\ClsEmailEBTV.vb` | todo | â€” | â€” |
| `Classes\ClsPdfCommentLines.vb` | todo | â€” | â€” |
| `Classes\ClsProdObjects.vb` | todo | â€” | â€” |
| `Classes\ClsRecOrdNr.vb` | todo | â€” | â€” |
| `Classes\ClsXMLfile.vb` | todo | â€” | â€” |
| `Classes\Coating\Coating.vb` | todo | â€” | â€” |
| `Classes\Coating\CoatingPickLabel_v3.vb` | todo | â€” | â€” |
| `Classes\Coating\ControlCoating.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Classes\Coating\ControlCoating.resx` | generated | resource bundle (designer-managed) | â€” |
| `Classes\Coating\ControlCoating.vb` | todo | â€” | â€” |
| `Classes\Coating\ControlCoatingGrouped.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Classes\Coating\ControlCoatingGrouped.resx` | generated | resource bundle (designer-managed) | â€” |
| `Classes\Coating\ControlCoatingGrouped.vb` | todo | â€” | â€” |
| `Classes\Coating\CtrlCoatingPickLocation.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Classes\Coating\CtrlCoatingPickLocation.resx` | generated | resource bundle (designer-managed) | â€” |
| `Classes\Coating\CtrlCoatingPickLocation.vb` | todo | â€” | â€” |
| `Classes\Coating\DataGridViewProgressColumn.vb` | todo | â€” | â€” |
| `Classes\Coating\DebugLog.vb` | todo | â€” | â€” |
| `Classes\Coating\EbtvSticker.vb` | todo | â€” | â€” |
| `Classes\Coating\FrmAddLayerThickness.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Classes\Coating\FrmAddLayerThickness.resx` | generated | resource bundle (designer-managed) | â€” |
| `Classes\Coating\FrmAddLayerThickness.vb` | todo | â€” | â€” |
| `Classes\Coating\FrmCoatingLayerThickness.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Classes\Coating\FrmCoatingLayerThickness.resx` | generated | resource bundle (designer-managed) | â€” |
| `Classes\Coating\FrmCoatingLayerThickness.vb` | todo | â€” | â€” |
| `Classes\Coating\FrmCoatingPick.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Classes\Coating\FrmCoatingPick.resx` | generated | resource bundle (designer-managed) | â€” |
| `Classes\Coating\FrmCoatingPick.vb` | todo | â€” | â€” |
| `Classes\Coating\FrmGetCoatingMaterials.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Classes\Coating\FrmGetCoatingMaterials.resx` | generated | resource bundle (designer-managed) | â€” |
| `Classes\Coating\FrmGetCoatingMaterials.vb` | todo | â€” | â€” |
| `Classes\Coating\FrmGetCoatingNextJob.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Classes\Coating\FrmGetCoatingNextJob.resx` | generated | resource bundle (designer-managed) | â€” |
| `Classes\Coating\FrmGetCoatingNextJob.vb` | todo | â€” | â€” |
| `Classes\Coating\frmKardexJobIncomplete.designer.vb` | generated | VS Forms designer partial | â€” |
| `Classes\Coating\frmKardexJobIncomplete.resx` | generated | resource bundle (designer-managed) | â€” |
| `Classes\Coating\frmKardexJobIncomplete.vb` | todo | â€” | â€” |
| `Classes\Coating\frmScanNext.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Classes\Coating\frmScanNext.resx` | generated | resource bundle (designer-managed) | â€” |
| `Classes\Coating\frmScanNext.vb` | todo | â€” | â€” |
| `Classes\Coating\IControlCoating.vb` | todo | â€” | â€” |
| `Classes\Connectivity\CalcExcelExport.vb` | todo | â€” | â€” |
| `Classes\Connectivity\ClsActiveDirectory.vb` | todo | â€” | â€” |
| `Classes\Connectivity\ClsICenter.vb` | todo | â€” | â€” |
| `Classes\Connectivity\ClsISAH.vb` | todo | â€” | â€” |
| `Classes\Connectivity\ClsJIBA.vb` | todo | â€” | â€” |
| `Classes\Connectivity\DeviceInfo.vb` | todo | â€” | â€” |
| `Classes\Connectivity\LabelWriter.vb` | todo | â€” | â€” |
| `Classes\Connectivity\NetworkPrinter.vb` | todo | â€” | â€” |
| `Classes\DocElementConverter.vb` | todo | â€” | â€” |
| `Classes\DossierDetail.vb` | todo | â€” | â€” |
| `Classes\DossierDocFolder.vb` | todo | â€” | â€” |
| `Classes\EngineeringOrders.vb` | todo | â€” | â€” |
| `Classes\ExtOperPart.vb` | todo | â€” | â€” |
| `Classes\ICenterDoc.vb` | todo | â€” | â€” |
| `Classes\ICenterObject.vb` | todo | â€” | â€” |
| `Classes\ICenterPartBasicXML.vb` | todo | â€” | â€” |
| `Classes\InvtOrd.vb` | todo | â€” | â€” |
| `Classes\IsahDoc.vb` | todo | â€” | â€” |
| `Classes\MachGrp.vb` | todo | â€” | â€” |
| `Classes\MachineFilter.vb` | todo | â€” | â€” |
| `Classes\ModelTree.vb` | todo | â€” | â€” |
| `Classes\OrdRefNrStatusUpdate.vb` | todo | â€” | â€” |
| `Classes\PartCalculation.vb` | todo | â€” | â€” |
| `Classes\PartOptimisationExcelExport.vb` | todo | â€” | â€” |
| `Classes\PreSelectMachGrpCodes\PreSelectMachGrpCode.vb` | todo | â€” | â€” |
| `Classes\PreSelectMachGrpCodes\PreSelectMachGrpCodeOrdType.vb` | todo | â€” | â€” |
| `Classes\PreSelectMachGrpCodes\PreSelectMachGrpCodeOrdTypeHandler.vb` | todo | â€” | â€” |
| `Classes\PreSelectMachGrpCodes\PreSelectMachGrpCodes.vb` | todo | â€” | â€” |
| `Classes\PreSelectMachGrpCodes\PreSelectMachGrpCodesHandler.vb` | todo | â€” | â€” |
| `Classes\ProcessWatch.vb` | todo | â€” | â€” |
| `Classes\Production\ClsIPbatch.vb` | todo | â€” | â€” |
| `Classes\Production\ClsIPorder.vb` | todo | â€” | â€” |
| `Classes\Production\ClsIPpacket.vb` | todo | â€” | â€” |
| `Classes\Production\ClsIPpart.vb` | todo | â€” | â€” |
| `Classes\Production\Icenter2IsahJob.vb` | todo | â€” | â€” |
| `Classes\Production\LeanJob.vb` | todo | â€” | â€” |
| `Classes\Production\LeanWorkTime.vb` | todo | â€” | â€” |
| `Classes\Production\MachGrpPerformance.vb` | todo | â€” | â€” |
| `Classes\Production\PartProgress.vb` | todo | â€” | â€” |
| `Classes\Production\ProdPlanView.vb` | todo | â€” | â€” |
| `Classes\Production\ProdStatus.vb` | todo | â€” | â€” |
| `Classes\Production\ProductionRegistration.vb` | todo | â€” | â€” |
| `Classes\Production\WorkView.vb` | todo | â€” | â€” |
| `Classes\PurOrd.vb` | todo | â€” | â€” |
| `Classes\RecurrenceCheck.vb` | todo | â€” | â€” |
| `Classes\StandardPartUpdater.vb` | todo | â€” | â€” |
| `Classes\StickersAndLabels\AltecLabel.vb` | todo | â€” | â€” |
| `Classes\StickersAndLabels\DymoLabelTest.vb` | todo | â€” | â€” |
| `Classes\StickersAndLabels\FG_ProductLabel.vb` | todo | â€” | â€” |
| `Classes\StickersAndLabels\InvtPartSticker.vb` | todo | â€” | â€” |
| `Classes\StickersAndLabels\KanbanBinLabel.vb` | todo | â€” | â€” |
| `Classes\StickersAndLabels\LeanBatchSticker.vb` | todo | â€” | â€” |
| `Classes\StickersAndLabels\LeanSticker_v1.vb` | todo | â€” | â€” |
| `Classes\StickersAndLabels\LeanStickerCopy.vb` | todo | â€” | â€” |
| `Classes\StickersAndLabels\OrdRefLabel.vb` | todo | â€” | â€” |
| `Classes\StickersAndLabels\PartIdent.vb` | todo | â€” | â€” |
| `Classes\StickersAndLabels\PartIdentProfMill.vb` | todo | â€” | â€” |
| `Classes\Toolbox\ApplicationHelper.vb` | todo | â€” | â€” |
| `Classes\Toolbox\ClipboardHelper.vb` | todo | â€” | â€” |
| `Classes\Toolbox\HelpHandler.vb` | todo | â€” | â€” |
| `Classes\Toolbox\JzWindow.vb` | todo | â€” | â€” |
| `Classes\Toolbox\RtfBuilder.vb` | todo | â€” | â€” |
| `Classes\Toolbox\SessionHelper.vb` | todo | â€” | â€” |
| `Classes\Toolbox\WindowHandler.vb` | todo | â€” | â€” |
| `Classes\Toolbox\Zip.vb` | todo | â€” | â€” |

### Comparers

**Total: 4** &nbsp; | &nbsp; .vb: 4 | .resx: 0 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Comparers\BewerkingsGroepComparer.vb` | todo | â€” | â€” |
| `Comparers\DefaultNodeSorter.vb` | todo | â€” | â€” |
| `Comparers\DPrintDocumentComparer.vb` | todo | â€” | â€” |
| `Comparers\JAZOTreeViewNodeSorter.vb` | todo | â€” | â€” |

### Controls

**Total: 114** &nbsp; | &nbsp; .vb: 78 | .resx: 36 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Controls\BomControl.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\BomControl.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\BomControl.vb` | todo | â€” | â€” |
| `Controls\ButtonExtended.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\ButtonExtended.vb` | todo | â€” | â€” |
| `Controls\CheckBoxExtended.vb` | todo | â€” | â€” |
| `Controls\CompBriefcaseButton.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CompBriefcaseButton.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CompBriefcaseButton.vb` | todo | â€” | â€” |
| `Controls\CreoViewControl.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CreoViewControl.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CreoViewControl.vb` | todo | â€” | â€” |
| `Controls\CtrlCeChecklistViewer.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlCeChecklistViewer.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlCeChecklistViewer.vb` | todo | â€” | â€” |
| `Controls\CtrlDocumentReplace.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlDocumentReplace.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlDocumentReplace.vb` | todo | â€” | â€” |
| `Controls\CtrlDosDesignCodes.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlDosDesignCodes.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlDosDesignCodes.vb` | todo | â€” | â€” |
| `Controls\CtrlDossierDetailProdDosCompare.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlDossierDetailProdDosCompare.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlDossierDetailProdDosCompare.vb` | todo | â€” | â€” |
| `Controls\CtrlDossierDetailProperties.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlDossierDetailProperties.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlDossierDetailProperties.vb` | todo | â€” | â€” |
| `Controls\CtrlDossierDetailText.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlDossierDetailText.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlDossierDetailText.vb` | todo | â€” | â€” |
| `Controls\CtrlEditOperations.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlEditOperations.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlEditOperations.vb` | todo | â€” | â€” |
| `Controls\CtrlExplorer.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlExplorer.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlExplorer.vb` | todo | â€” | â€” |
| `Controls\CtrlFgProductLabel.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlFgProductLabel.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlFgProductLabel.vb` | todo | â€” | â€” |
| `Controls\CtrlIpInfo.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlIpInfo.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlIpInfo.vb` | todo | â€” | â€” |
| `Controls\CtrlIsahContacts.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlIsahContacts.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlIsahContacts.vb` | todo | â€” | â€” |
| `Controls\CtrlModelgenerator.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlModelgenerator.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlModelgenerator.vb` | todo | â€” | â€” |
| `Controls\CtrlModelgenerators.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlModelgenerators.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlModelgenerators.vb` | todo | â€” | â€” |
| `Controls\CtrlOfficeClockFavorites.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlOfficeClockFavorites.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlOfficeClockFavorites.vb` | todo | â€” | â€” |
| `Controls\CtrlOfficeClockStatus.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlOfficeClockStatus.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlOfficeClockStatus.vb` | todo | â€” | â€” |
| `Controls\CtrlPhoneNr.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlPhoneNr.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlPhoneNr.vb` | todo | â€” | â€” |
| `Controls\CtrlProdChecklistViewer.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlProdChecklistViewer.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlProdChecklistViewer.vb` | todo | â€” | â€” |
| `Controls\CtrlProdRoutingDetail.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlProdRoutingDetail.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlProdRoutingDetail.vb` | todo | â€” | â€” |
| `Controls\CtrlProductConfiguration.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlProductConfiguration.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlProductConfiguration.vb` | todo | â€” | â€” |
| `Controls\CtrlSalesFavorites.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlSalesFavorites.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlSalesFavorites.vb` | todo | â€” | â€” |
| `Controls\CtrlTreeNodeDetails.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlTreeNodeDetails.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlTreeNodeDetails.vb` | todo | â€” | â€” |
| `Controls\CtrlTreeNodeProperties.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlTreeNodeProperties.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlTreeNodeProperties.vb` | todo | â€” | â€” |
| `Controls\CtrlWebClockAssistant.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\CtrlWebClockAssistant.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\CtrlWebClockAssistant.vb` | todo | â€” | â€” |
| `Controls\DxfView\DxfViewControl.designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\DxfView\DxfViewControl.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\DxfView\DxfViewControl.vb` | todo | â€” | â€” |
| `Controls\DxfView\DxfViewDisplayControl.designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\DxfView\DxfViewDisplayControl.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\DxfView\DxfViewDisplayControl.vb` | todo | â€” | â€” |
| `Controls\DxfView\PolygonWireframeGraphicsFactory.vb` | todo | â€” | â€” |
| `Controls\ExtendedDateTimePicker.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\ExtendedDateTimePicker.vb` | todo | â€” | â€” |
| `Controls\Isah\CtrlDosDetailExtra.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\Isah\CtrlDosDetailExtra.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\Isah\CtrlDosDetailExtra.vb` | todo | â€” | â€” |
| `Controls\Isah\DossierDetailExtraAuthorizationHelper.vb` | todo | â€” | â€” |
| `Controls\Isah\FrmDosDetailExtra.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\Isah\FrmDosDetailExtra.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\Isah\FrmDosDetailExtra.vb` | todo | â€” | â€” |
| `Controls\Isah\ICtrlDosDetailExtra.vb` | todo | â€” | â€” |
| `Controls\ModelOpers\ControlModelOper.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\ModelOpers\ControlModelOper.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\ModelOpers\ControlModelOper.vb` | todo | â€” | â€” |
| `Controls\ModelOpers\ControlModelOpers.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\ModelOpers\ControlModelOpers.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\ModelOpers\ControlModelOpers.vb` | todo | â€” | â€” |
| `Controls\MSVistaPBar.vb` | todo | â€” | â€” |
| `Controls\RichTextBoxEditor.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\RichTextBoxEditor.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\RichTextBoxEditor.vb` | todo | â€” | â€” |
| `Controls\TabPageExtended.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\TabPageExtended.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\TabPageExtended.vb` | todo | â€” | â€” |
| `Controls\UCKanbanPart.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Controls\UCKanbanPart.resx` | generated | resource bundle (designer-managed) | â€” |
| `Controls\UCKanbanPart.vb` | todo | â€” | â€” |

### DataMigration

**Total: 50** &nbsp; | &nbsp; .vb: 50 | .resx: 0 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `DataMigration\BasicDataHandler.vb` | todo | â€” | â€” |
| `DataMigration\BasicMigrationHandler.vb` | todo | â€” | â€” |
| `DataMigration\CacheHandler.vb` | todo | â€” | â€” |
| `DataMigration\ContextMenuHelper.vb` | todo | â€” | â€” |
| `DataMigration\DataBaseConnection.vb` | todo | â€” | â€” |
| `DataMigration\DataBaseConnectionHandler.vb` | todo | â€” | â€” |
| `DataMigration\DataHandlerToolbox.vb` | todo | â€” | â€” |
| `DataMigration\DossierItemMigrationHandler.vb` | todo | â€” | â€” |
| `DataMigration\DossierMigrationHandler.vb` | todo | â€” | â€” |
| `DataMigration\Entities\AuditableEntity.vb` | todo | â€” | â€” |
| `DataMigration\Entities\BillOfMaterialItem.vb` | todo | â€” | â€” |
| `DataMigration\Entities\BillOfOperation.vb` | todo | â€” | â€” |
| `DataMigration\Entities\BillOfOperationItem.vb` | todo | â€” | â€” |
| `DataMigration\Entities\Document.vb` | todo | â€” | â€” |
| `DataMigration\Entities\Dossier.vb` | todo | â€” | â€” |
| `DataMigration\Entities\DossierItem.vb` | todo | â€” | â€” |
| `DataMigration\Entities\IEntity.vb` | todo | â€” | â€” |
| `DataMigration\Entities\Material.vb` | todo | â€” | â€” |
| `DataMigration\Entities\Operation.vb` | todo | â€” | â€” |
| `DataMigration\Entities\Part.vb` | todo | â€” | â€” |
| `DataMigration\Entities\ProductionDossier.vb` | todo | â€” | â€” |
| `DataMigration\Entities\Project.vb` | todo | â€” | â€” |
| `DataMigration\Entities\User.vb` | todo | â€” | â€” |
| `DataMigration\Features\BillOfMaterialItemFactory.vb` | todo | â€” | â€” |
| `DataMigration\Features\BillOfMaterialItemHandler.vb` | todo | â€” | â€” |
| `DataMigration\Features\BillOfOperationItemFactory.vb` | todo | â€” | â€” |
| `DataMigration\Features\BillOfOperationItemHandler.vb` | todo | â€” | â€” |
| `DataMigration\Features\DocumentFactory.vb` | todo | â€” | â€” |
| `DataMigration\Features\DocumentHandler.vb` | todo | â€” | â€” |
| `DataMigration\Features\DossierFactory.vb` | todo | â€” | â€” |
| `DataMigration\Features\DossierHandler.vb` | todo | â€” | â€” |
| `DataMigration\Features\DossierItemFactory.vb` | todo | â€” | â€” |
| `DataMigration\Features\DossierItemHandler.vb` | todo | â€” | â€” |
| `DataMigration\Features\MaterialFactory.vb` | todo | â€” | â€” |
| `DataMigration\Features\MaterialHandler.vb` | todo | â€” | â€” |
| `DataMigration\Features\OperationFactory.vb` | todo | â€” | â€” |
| `DataMigration\Features\OperationHandler.vb` | todo | â€” | â€” |
| `DataMigration\Features\PartFactory.vb` | todo | â€” | â€” |
| `DataMigration\Features\PartHandler.vb` | todo | â€” | â€” |
| `DataMigration\Features\ProductionDossierFactory.vb` | todo | â€” | â€” |
| `DataMigration\Features\ProductionDossierHandler.vb` | todo | â€” | â€” |
| `DataMigration\Features\ProjectFactory.vb` | todo | â€” | â€” |
| `DataMigration\Features\ProjectHandler.vb` | todo | â€” | â€” |
| `DataMigration\Features\UserFactory.vb` | todo | â€” | â€” |
| `DataMigration\Features\UserHandler.vb` | todo | â€” | â€” |
| `DataMigration\IMigrationHandler.vb` | todo | â€” | â€” |
| `DataMigration\MockDataHandler.vb` | todo | â€” | â€” |
| `DataMigration\PartMigrationHandler.vb` | todo | â€” | â€” |
| `DataMigration\ProductionDossierMigrationHandler.vb` | todo | â€” | â€” |
| `DataMigration\UserMigrationHandler.vb` | todo | â€” | â€” |

### DesignComments

**Total: 10** &nbsp; | &nbsp; .vb: 7 | .resx: 3 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `DesignComments\ClsDesignComments.vb` | todo | â€” | â€” |
| `DesignComments\dgCommentLines.designer.vb` | generated | VS Forms designer partial | â€” |
| `DesignComments\dgCommentLines.resx` | generated | resource bundle (designer-managed) | â€” |
| `DesignComments\dgCommentLines.vb` | todo | â€” | â€” |
| `DesignComments\frmCommentLine.designer.vb` | generated | VS Forms designer partial | â€” |
| `DesignComments\frmCommentLine.resx` | generated | resource bundle (designer-managed) | â€” |
| `DesignComments\frmCommentLine.vb` | todo | â€” | â€” |
| `DesignComments\FrmDesigncomments.designer.vb` | generated | VS Forms designer partial | â€” |
| `DesignComments\FrmDesigncomments.resx` | generated | resource bundle (designer-managed) | â€” |
| `DesignComments\FrmDesigncomments.vb` | todo | â€” | â€” |

### Elumatec

**Total: 157** &nbsp; | &nbsp; .vb: 140 | .resx: 17 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Elumatec\AppVersion.vb` | todo | â€” | â€” |
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
| `Elumatec\AutoProfMillProgApproval.vb` | todo | â€” | â€” |
| `Elumatec\ClsComWatcher.vb` | done | [[modules/elumatec-com-watcher]] | 2026-06-18 |
| `Elumatec\ClsDgxShoppingList.vb` | todo | â€” | â€” |
| `Elumatec\ClsDgxStickerPrinter.vb` | todo | â€” | â€” |
| `Elumatec\ClsEluLanguage.vb` | todo | â€” | â€” |
| `Elumatec\ClsSawList.vb` | todo | â€” | â€” |
| `Elumatec\ControlProfSaw.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\ControlProfSaw.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\ControlProfSaw.vb` | todo | â€” | â€” |
| `Elumatec\CRs232.vb` | done | third-party (Corrado Cavalli Â©2003); [[modules/elumatec-com-watcher]] | 2026-06-18 |
| `Elumatec\CtrlEluOpenGLViewer.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\CtrlEluOpenGLViewer.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\CtrlEluOpenGLViewer.vb` | todo | â€” | â€” |
| `Elumatec\CtrlProfMillCam.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\CtrlProfMillCam.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\CtrlProfMillCam.vb` | todo | â€” | â€” |
| `Elumatec\CtrlProfMillElu.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\CtrlProfMillElu.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\CtrlProfMillElu.vb` | todo | â€” | â€” |
| `Elumatec\CutFactory.vb` | todo | â€” | â€” |
| `Elumatec\CycleTime.vb` | todo | â€” | â€” |
| `Elumatec\Database\Fixture.vb` | todo | â€” | â€” |
| `Elumatec\Database\FixtureCollection.vb` | todo | â€” | â€” |
| `Elumatec\Database\Offset.vb` | todo | â€” | â€” |
| `Elumatec\Database\OffsetCollection.vb` | todo | â€” | â€” |
| `Elumatec\Database\OffsetFile.vb` | todo | â€” | â€” |
| `Elumatec\Database\Offsets.vb` | todo | â€” | â€” |
| `Elumatec\Database\Profile.vb` | todo | â€” | â€” |
| `Elumatec\Database\ProfileExportHandler.vb` | todo | â€” | â€” |
| `Elumatec\Database\ProfileMachineSetting.vb` | todo | â€” | â€” |
| `Elumatec\Database\ToolDbSimplified.vb` | todo | â€” | â€” |
| `Elumatec\DXF\EluDxf.vb` | todo | â€” | â€” |
| `Elumatec\DXF\EluDxfEntity.vb` | todo | â€” | â€” |
| `Elumatec\DXF\EluDxfPolyline.vb` | todo | â€” | â€” |
| `Elumatec\DXF\EluDxfVertex.vb` | todo | â€” | â€” |
| `Elumatec\EluCadApp.vb` | needs-review | overview only in [[modules/elumatec-cad-app]]; per-cluster sub-notes pending | â€” |
| `Elumatec\EluCadSetting.vb` | todo | â€” | â€” |
| `Elumatec\FileFormat.vb` | todo | â€” | â€” |
| `Elumatec\FrmComWatcher.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\FrmComWatcher.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\FrmComWatcher.vb` | done | [[modules/elumatec-com-watcher]] | 2026-06-18 |
| `Elumatec\frmDgxStack.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\frmDgxStack.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\frmDgxStack.vb` | todo | â€” | â€” |
| `Elumatec\FrmEluMissingProfile.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\FrmEluMissingProfile.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\FrmEluMissingProfile.vb` | todo | â€” | â€” |
| `Elumatec\frmExportDgx.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\frmExportDgx.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\frmExportDgx.vb` | todo | â€” | â€” |
| `Elumatec\frmManualProfile.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\frmManualProfile.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\frmManualProfile.vb` | todo | â€” | â€” |
| `Elumatec\FrmNcxErrors.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\FrmNcxErrors.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\FrmNcxErrors.vb` | todo | â€” | â€” |
| `Elumatec\FrmProfileView.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\FrmProfileView.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\FrmProfileView.vb` | todo | â€” | â€” |
| `Elumatec\frmSawQtyDone.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\frmSawQtyDone.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\frmSawQtyDone.vb` | todo | â€” | â€” |
| `Elumatec\frmSawQtyDoneExt.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\frmSawQtyDoneExt.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\frmSawQtyDoneExt.vb` | todo | â€” | â€” |
| `Elumatec\LicenseHelper.vb` | todo | â€” | â€” |
| `Elumatec\LicenseManagementCenter.vb` | todo | â€” | â€” |
| `Elumatec\Machine\Sbz140Alu.vb` | done | [[modules/elumatec-machine-base]] | 2026-06-18 |
| `Elumatec\Machine\Sbz140Rvs.vb` | needs-review | [[modules/elumatec-machine-base]] (values not transcribed; Q-027) | â€” |
| `Elumatec\Machine\Sbz140Stl.vb` | needs-review | [[modules/elumatec-machine-base]] (values not transcribed; Q-027) | â€” |
| `Elumatec\Machine\Sbz141Alu.vb` | needs-review | [[modules/elumatec-machine-base]] (values not transcribed) | â€” |
| `Elumatec\Machine\Sbz14x.vb` | done | [[modules/elumatec-machine-base]] | 2026-06-18 |
| `Elumatec\MacroDatabase.vb` | todo | â€” | â€” |
| `Elumatec\NcStructure\Bar.vb` | done | [[modules/elumatec-ncstructure-hierarchy]] | 2026-06-18 |
| `Elumatec\NcStructure\Cut.vb` | done | [[modules/elumatec-ncstructure-hierarchy]] | 2026-06-18 |
| `Elumatec\NcStructure\EluCadFile.vb` | done | [[modules/elumatec-elucadfile]] | 2026-06-18 |
| `Elumatec\NcStructure\Job.vb` | done | [[modules/elumatec-ncstructure-hierarchy]] | 2026-06-18 |
| `Elumatec\NcStructure\Plane.vb` | done | [[modules/elumatec-ncstructure-hierarchy]] | 2026-06-18 |
| `Elumatec\NcStructure\PlaneCollection.vb` | done | [[modules/elumatec-ncstructure-hierarchy]] | 2026-06-18 |
| `Elumatec\NcVersionHandler.vb` | todo | â€” | â€” |
| `Elumatec\NcwExportProfile.vb` | todo | â€” | â€” |
| `Elumatec\NcwViewer.vb` | todo | â€” | â€” |
| `Elumatec\NcxContainer.vb` | todo | â€” | â€” |
| `Elumatec\NumberLib.vb` | todo | â€” | â€” |
| `Elumatec\Optimizer\CutOptimizer.vb` | todo | â€” | â€” |
| `Elumatec\Optimizer\frmSawJobOptimizer.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\Optimizer\frmSawJobOptimizer.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\Optimizer\frmSawJobOptimizer.vb` | todo | â€” | â€” |
| `Elumatec\ProfileMatcher.vb` | todo | â€” | â€” |
| `Elumatec\ProfMillConverter.vb` | done | [[modules/elumatec-profmill-converter]] | 2026-06-18 |
| `Elumatec\ProfMillJob.vb` | done | [[modules/elumatec-profmill-job]] | 2026-06-18 |
| `Elumatec\ReferenceDxf.vb` | todo | â€” | â€” |
| `Elumatec\SawListReport.vb` | todo | â€” | â€” |
| `Elumatec\SectionCutOffBox.vb` | todo | â€” | â€” |
| `Elumatec\UCDgxWorksheet.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\UCDgxWorksheet.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\UCDgxWorksheet.vb` | todo | â€” | â€” |
| `Elumatec\Workpiece.vb` | todo | â€” | â€” |
| `Elumatec\Works\Circle.vb` | todo | â€” | â€” |
| `Elumatec\Works\Deburr.vb` | todo | â€” | â€” |
| `Elumatec\Works\Drill.vb` | todo | â€” | â€” |
| `Elumatec\Works\DxfFreeForm.vb` | todo | â€” | â€” |
| `Elumatec\Works\FreeForm.vb` | todo | â€” | â€” |
| `Elumatec\Works\FreeFormPoint.vb` | todo | â€” | â€” |
| `Elumatec\Works\FrmTestDrwProfile.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\Works\FrmTestDrwProfile.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\Works\FrmTestDrwProfile.vb` | todo | â€” | â€” |
| `Elumatec\Works\Group.vb` | todo | â€” | â€” |
| `Elumatec\Works\Line.vb` | todo | â€” | â€” |
| `Elumatec\Works\Macro.vb` | todo | â€” | â€” |
| `Elumatec\Works\Rectangle.vb` | todo | â€” | â€” |
| `Elumatec\Works\Replacements\AluGeneral.vb` | needs-review | [[modules/elumatec-replacement-alu-general]] (overview only; per-branch notes pending) | â€” |
| `Elumatec\Works\Replacements\AluHinge.vb` | todo | â€” | â€” |
| `Elumatec\Works\Replacements\AluHUPO.vb` | todo | â€” | â€” |
| `Elumatec\Works\Replacements\AluSinglePnotch.vb` | todo | â€” | â€” |
| `Elumatec\Works\Replacements\AluSRkom.vb` | todo | â€” | â€” |
| `Elumatec\Works\Replacements\DoorPlankCalibration.vb` | todo | â€” | â€” |
| `Elumatec\Works\Replacements\DoorPlankCalibrationMessageCutOff.vb` | todo | â€” | â€” |
| `Elumatec\Works\Replacements\DoublePnotch.vb` | todo | â€” | â€” |
| `Elumatec\Works\Replacements\ExtraLength.vb` | todo | â€” | â€” |
| `Elumatec\Works\Replacements\ExtraLengthMacro.vb` | todo | â€” | â€” |
| `Elumatec\Works\Replacements\ExtraLengthMacroFactory.vb` | todo | â€” | â€” |
| `Elumatec\Works\Replacements\Flowdrill.vb` | done | [[modules/elumatec-replacement-flowdrill]] | 2026-06-18 |
| `Elumatec\Works\Replacements\IDoorPlankCalibrationMessage.vb` | todo | â€” | â€” |
| `Elumatec\Works\Replacements\LargeRectangle.vb` | done | [[modules/elumatec-replacement-large-rectangle]] | 2026-06-18 |
| `Elumatec\Works\Replacements\OpdekH.vb` | todo | â€” | â€” |
| `Elumatec\Works\Replacements\RDHS27Notch.vb` | todo | â€” | â€” |
| `Elumatec\Works\Replacements\RuntimeManipulation.vb` | todo | â€” | â€” |
| `Elumatec\Works\Replacements\RuntimeManipulationInstruction.vb` | todo | â€” | â€” |
| `Elumatec\Works\Replacements\RuntimeManipulations.vb` | todo | â€” | â€” |
| `Elumatec\Works\Replacements\StlDoublePnotch.vb` | todo | â€” | â€” |
| `Elumatec\Works\Replacements\StlFlowDrill.vb` | todo | â€” | â€” |
| `Elumatec\Works\Replacements\StlGeneral.vb` | todo | â€” | â€” |
| `Elumatec\Works\Replacements\StlHinge.vb` | todo | â€” | â€” |
| `Elumatec\Works\Replacements\UCDoorPlankCalibration.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Elumatec\Works\Replacements\UCDoorPlankCalibration.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elumatec\Works\Replacements\UCDoorPlankCalibration.vb` | todo | â€” | â€” |
| `Elumatec\Works\Replacements\WorksReplacement.vb` | done | [[modules/elumatec-works-replacement-base]] | 2026-06-18 |
| `Elumatec\Works\Replacements\WorksTranslation.vb` | todo | â€” | â€” |
| `Elumatec\Works\Sawcut.vb` | todo | â€” | â€” |
| `Elumatec\Works\SlottedHole.vb` | todo | â€” | â€” |
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
| `Forms\FrmProdChecklistViewer.vb` | todo | â€” | â€” |
| `Forms\FrmProdObjects.designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\FrmProdObjects.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\FrmProdObjects.vb` | todo | â€” | â€” |
| `Forms\FrmProdObjectsSelectCoating.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\FrmProdObjectsSelectCoating.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\FrmProdObjectsSelectCoating.vb` | todo | â€” | â€” |
| `Forms\ISAH\FrmPartBrowse.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\ISAH\FrmPartBrowse.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\ISAH\FrmPartBrowse.vb` | todo | â€” | â€” |
| `Forms\Management\ClsApplRevisions.vb` | todo | â€” | â€” |
| `Forms\Management\frmAddLeanProdTraject.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmAddLeanProdTraject.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmAddLeanProdTraject.vb` | todo | â€” | â€” |
| `Forms\Management\frmAdminActivePdfMarkups.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmAdminActivePdfMarkups.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmAdminActivePdfMarkups.vb` | todo | â€” | â€” |
| `Forms\Management\FrmApplNewRelease.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\FrmApplNewRelease.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\FrmApplNewRelease.vb` | todo | â€” | â€” |
| `Forms\Management\FrmApplRevisions.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\FrmApplRevisions.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\FrmApplRevisions.vb` | todo | â€” | â€” |
| `Forms\Management\frmCoatManagement.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmCoatManagement.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmCoatManagement.vb` | todo | â€” | â€” |
| `Forms\Management\frmCoatManagementAdd.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmCoatManagementAdd.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmCoatManagementAdd.vb` | todo | â€” | â€” |
| `Forms\Management\frmColors.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmColors.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmColors.vb` | todo | â€” | â€” |
| `Forms\Management\frmConvertDocElement.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmConvertDocElement.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmConvertDocElement.vb` | todo | â€” | â€” |
| `Forms\Management\frmEditCadBatchserverRules.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmEditCadBatchserverRules.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmEditCadBatchserverRules.vb` | todo | â€” | â€” |
| `Forms\Management\frmEditCapacityTickets.designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmEditCapacityTickets.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmEditCapacityTickets.vb` | todo | â€” | â€” |
| `Forms\Management\frmEditCoatingDefPrimer.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmEditCoatingDefPrimer.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmEditCoatingDefPrimer.vb` | todo | â€” | â€” |
| `Forms\Management\frmEditKanbanBin.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmEditKanbanBin.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmEditKanbanBin.vb` | todo | â€” | â€” |
| `Forms\Management\FrmEditLeanMachines.designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\FrmEditLeanMachines.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\FrmEditLeanMachines.vb` | todo | â€” | â€” |
| `Forms\Management\frmEditMachGrps.designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmEditMachGrps.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmEditMachGrps.vb` | todo | â€” | â€” |
| `Forms\Management\FrmEditTable.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\FrmEditTable.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\FrmEditTable.vb` | todo | â€” | â€” |
| `Forms\Management\frmExtractIconFromFile.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmExtractIconFromFile.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmExtractIconFromFile.vb` | todo | â€” | â€” |
| `Forms\Management\frmFeedbackUpdate.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmFeedbackUpdate.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmFeedbackUpdate.vb` | todo | â€” | â€” |
| `Forms\Management\frmGenericsAdmin.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\frmGenericsAdmin.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\frmGenericsAdmin.vb` | todo | â€” | â€” |
| `Forms\Management\FrmHelp.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\FrmHelp.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\FrmHelp.vb` | todo | â€” | â€” |
| `Forms\Management\FrmUpdateProdTrackInWorkView.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Management\FrmUpdateProdTrackInWorkView.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Management\FrmUpdateProdTrackInWorkView.vb` | todo | â€” | â€” |
| `Forms\ShopProcess\FrmDgxPickJob.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\ShopProcess\FrmDgxPickJob.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\ShopProcess\FrmDgxPickJob.vb` | todo | â€” | â€” |
| `Forms\ShopProcess\FrmLeanAdmin.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\ShopProcess\FrmLeanAdmin.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\ShopProcess\FrmLeanAdmin.vb` | todo | â€” | â€” |
| `Forms\ShopProcess\FrmLeanDashboard.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\ShopProcess\FrmLeanDashboard.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\ShopProcess\FrmLeanDashboard.vb` | todo | â€” | â€” |
| `Forms\ShopProcess\FrmLeanFlowChart.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\ShopProcess\FrmLeanFlowChart.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\ShopProcess\FrmLeanFlowChart.vb` | todo | â€” | â€” |
| `Forms\ShopProcess\FrmLeanStatus.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\ShopProcess\FrmLeanStatus.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\ShopProcess\FrmLeanStatus.vb` | todo | â€” | â€” |
| `Forms\ShopProcess\FrmPartDispatchCollector.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\ShopProcess\FrmPartDispatchCollector.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\ShopProcess\FrmPartDispatchCollector.vb` | todo | â€” | â€” |
| `Forms\ShopProcess\FrmPartPick.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\ShopProcess\FrmPartPick.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\ShopProcess\FrmPartPick.vb` | todo | â€” | â€” |
| `Forms\ShopProcess\FrmWorkChange.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\ShopProcess\FrmWorkChange.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\ShopProcess\FrmWorkChange.vb` | todo | â€” | â€” |
| `Forms\ShopProcess\FrmWorkViewWithStatus.designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\ShopProcess\FrmWorkViewWithStatus.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\ShopProcess\FrmWorkViewWithStatus.vb` | todo | â€” | â€” |
| `Forms\ShopProcess\FrmWorkViewWithStatusJCOA.designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\ShopProcess\FrmWorkViewWithStatusJCOA.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\ShopProcess\FrmWorkViewWithStatusJCOA.vb` | todo | â€” | â€” |
| `Forms\Template\AboutBox1.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Template\AboutBox1.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Template\AboutBox1.vb` | todo | â€” | â€” |
| `Forms\Template\FrmBulkRename.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Template\FrmBulkRename.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Template\FrmBulkRename.vb` | todo | â€” | â€” |
| `Forms\Template\FrmFadingMsgBox.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Template\FrmFadingMsgBox.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Template\FrmFadingMsgBox.vb` | todo | â€” | â€” |
| `Forms\Template\frmHappyNewYear.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Template\frmHappyNewYear.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Template\frmHappyNewYear.vb` | todo | â€” | â€” |
| `Forms\Template\FrmPrintPdf.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Template\FrmPrintPdf.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Template\FrmPrintPdf.vb` | todo | â€” | â€” |
| `Forms\Template\FrmTextBox.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Template\FrmTextBox.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Template\FrmTextBox.vb` | todo | â€” | â€” |
| `Forms\Template\LoginForm1.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Template\LoginForm1.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Template\LoginForm1.vb` | todo | â€” | â€” |
| `Forms\Template\SelectFromListDialog.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Template\SelectFromListDialog.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Template\SelectFromListDialog.vb` | todo | â€” | â€” |
| `Forms\Test3DSpace_Form1.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Test3DSpace_Form1.vb` | todo | â€” | â€” |
| `Forms\Test3DSpace_Form2.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Test3DSpace_Form2.vb` | todo | â€” | â€” |
| `Forms\Toolbox\FrmAddICenterDoc.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmAddICenterDoc.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmAddICenterDoc.vb` | todo | â€” | â€” |
| `Forms\Toolbox\FrmBatchProdOrd.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmBatchProdOrd.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmBatchProdOrd.vb` | todo | â€” | â€” |
| `Forms\Toolbox\FrmBomMember.designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmBomMember.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmBomMember.vb` | todo | â€” | â€” |
| `Forms\Toolbox\FrmCadLicensesStatus.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmCadLicensesStatus.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmCadLicensesStatus.vb` | todo | â€” | â€” |
| `Forms\Toolbox\FrmComPortScanner.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmComPortScanner.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmComPortScanner.vb` | todo | â€” | â€” |
| `Forms\Toolbox\FrmContact.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmContact.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmContact.vb` | todo | â€” | â€” |
| `Forms\Toolbox\FrmCrystalReport.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmCrystalReport.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmCrystalReport.vb` | todo | â€” | â€” |
| `Forms\Toolbox\FrmDossierFromPurOrd.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmDossierFromPurOrd.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmDossierFromPurOrd.vb` | todo | â€” | â€” |
| `Forms\Toolbox\frmGetValidPartcode.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\frmGetValidPartcode.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\frmGetValidPartcode.vb` | todo | â€” | â€” |
| `Forms\Toolbox\frmJumpToOrdNr.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\frmJumpToOrdNr.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\frmJumpToOrdNr.vb` | todo | â€” | â€” |
| `Forms\Toolbox\FrmLinkIsahToIPO.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmLinkIsahToIPO.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmLinkIsahToIPO.vb` | todo | â€” | â€” |
| `Forms\Toolbox\FrmModelGenerator.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmModelGenerator.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmModelGenerator.vb` | todo | â€” | â€” |
| `Forms\Toolbox\FrmNewDossierPos.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmNewDossierPos.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmNewDossierPos.vb` | todo | â€” | â€” |
| `Forms\Toolbox\FrmNewICenterObject.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmNewICenterObject.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmNewICenterObject.vb` | todo | â€” | â€” |
| `Forms\Toolbox\FrmPartCalculationUpdate.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmPartCalculationUpdate.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmPartCalculationUpdate.vb` | todo | â€” | â€” |
| `Forms\Toolbox\FrmPosGenerator.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmPosGenerator.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmPosGenerator.vb` | todo | â€” | â€” |
| `Forms\Toolbox\FrmPrintCeSticker.designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\FrmPrintCeSticker.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\FrmPrintCeSticker.vb` | todo | â€” | â€” |
| `Forms\Toolbox\frmQuickview.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\frmQuickview.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\frmQuickview.vb` | todo | â€” | â€” |
| `Forms\Toolbox\frmSelectProject.Designer.vb` | generated | VS Forms designer partial | â€” |
| `Forms\Toolbox\frmSelectProject.resx` | generated | resource bundle (designer-managed) | â€” |
| `Forms\Toolbox\frmSelectProject.vb` | todo | â€” | â€” |

### IcImporter

**Total: 6** &nbsp; | &nbsp; .vb: 5 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `IcImporter\FrmIcImport.Designer.vb` | generated | VS Forms designer partial | â€” |
| `IcImporter\FrmIcImport.resx` | generated | resource bundle (designer-managed) | â€” |
| `IcImporter\FrmIcImport.vb` | todo | â€” | â€” |
| `IcImporter\ICenterPart.vb` | todo | â€” | â€” |
| `IcImporter\IcImportHandler.vb` | todo | â€” | â€” |
| `IcImporter\IcImportStatus.vb` | todo | â€” | â€” |

### Kardex

**Total: 4** &nbsp; | &nbsp; .vb: 3 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Kardex\FrmKardexInterface.designer.vb` | generated | VS Forms designer partial | â€” |
| `Kardex\FrmKardexInterface.resx` | generated | resource bundle (designer-managed) | â€” |
| `Kardex\FrmKardexInterface.vb` | todo | â€” | â€” |
| `Kardex\KardexProcessor.vb` | todo | â€” | â€” |

### MarkTool

**Total: 5** &nbsp; | &nbsp; .vb: 4 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `MarkTool\ClsComPort.vb` | todo | â€” | â€” |
| `MarkTool\ClsMarkToolDb.vb` | todo | â€” | â€” |
| `MarkTool\frmMarkTool.Designer.vb` | generated | VS Forms designer partial | â€” |
| `MarkTool\frmMarkTool.resx` | generated | resource bundle (designer-managed) | â€” |
| `MarkTool\frmMarkTool.vb` | todo | â€” | â€” |

### Modules

**Total: 3** &nbsp; | &nbsp; .vb: 3 | .resx: 0 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Modules\Functions.vb` | todo | (referenced from [[modules/main-module]] for `GetApplicationArguments`, `UpdateIcenter`, `GetXMLWriteAccess`, `InitProfMillMachGrps`, etc. â€” Phase 3) | â€” |
| `Modules\MailMessageExt.vb` | todo | â€” | â€” |
| `Modules\Main.vb` | done | [[modules/main-module]] | 2026-06-18 |

### PCFNetStudio

**Total: 17** &nbsp; | &nbsp; .vb: 12 | .resx: 5 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `PCFNetStudio\CPartBuilder.vb` | todo | â€” | â€” |
| `PCFNetStudio\FrmCodeConverter.Designer.vb` | generated | VS Forms designer partial | â€” |
| `PCFNetStudio\FrmCodeConverter.resx` | generated | resource bundle (designer-managed) | â€” |
| `PCFNetStudio\FrmCodeConverter.vb` | todo | â€” | â€” |
| `PCFNetStudio\FrmProductComparer.Designer.vb` | generated | VS Forms designer partial | â€” |
| `PCFNetStudio\FrmProductComparer.resx` | generated | resource bundle (designer-managed) | â€” |
| `PCFNetStudio\FrmProductComparer.vb` | todo | â€” | â€” |
| `PCFNetStudio\PartEditor.Designer.vb` | generated | VS Forms designer partial | â€” |
| `PCFNetStudio\PartEditor.resx` | generated | resource bundle (designer-managed) | â€” |
| `PCFNetStudio\PartEditor.vb` | todo | â€” | â€” |
| `PCFNetStudio\ProductionSet.vb` | todo | â€” | â€” |
| `PCFNetStudio\UCCalculation.Designer.vb` | generated | VS Forms designer partial | â€” |
| `PCFNetStudio\UCCalculation.resx` | generated | resource bundle (designer-managed) | â€” |
| `PCFNetStudio\UCCalculation.vb` | todo | â€” | â€” |
| `PCFNetStudio\UCProductComparer.Designer.vb` | generated | VS Forms designer partial | â€” |
| `PCFNetStudio\UCProductComparer.resx` | generated | resource bundle (designer-managed) | â€” |
| `PCFNetStudio\UCProductComparer.vb` | todo | â€” | â€” |

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
| `SmtManufacturing\BendNote.vb` | todo | â€” | â€” |
| `SmtManufacturing\BendPart.vb` | todo | â€” | â€” |
| `SmtManufacturing\BendTool.vb` | todo | â€” | â€” |
| `SmtManufacturing\BendToolGroup.vb` | todo | â€” | â€” |
| `SmtManufacturing\BendToolStation.vb` | todo | â€” | â€” |
| `SmtManufacturing\BncInterpreter\BNC.vb` | todo | â€” | â€” |
| `SmtManufacturing\BoostMigrator.vb` | todo | â€” | â€” |
| `SmtManufacturing\BoostPartViewerControl.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\BoostPartViewerControl.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\BoostPartViewerControl.vb` | todo | â€” | â€” |
| `SmtManufacturing\ContourCheck.vb` | todo | â€” | â€” |
| `SmtManufacturing\ControlSmtCut.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\ControlSmtCut.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\ControlSmtCut.vb` | todo | â€” | â€” |
| `SmtManufacturing\CtrlBendBoost.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\CtrlBendBoost.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\CtrlBendBoost.vb` | todo | â€” | â€” |
| `SmtManufacturing\CtrlBendCalcDetails.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\CtrlBendCalcDetails.vb` | todo | â€” | â€” |
| `SmtManufacturing\CtrlMachinePartBendSolutions.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\CtrlMachinePartBendSolutions.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\CtrlMachinePartBendSolutions.vb` | todo | â€” | â€” |
| `SmtManufacturing\CtrlPartBendSolution.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\CtrlPartBendSolution.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\CtrlPartBendSolution.vb` | todo | â€” | â€” |
| `SmtManufacturing\CutSheetLabelPrintHandler.vb` | todo | â€” | â€” |
| `SmtManufacturing\DxfContour.vb` | todo | â€” | â€” |
| `SmtManufacturing\FileMerger.vb` | todo | â€” | â€” |
| `SmtManufacturing\FlatPatternConverter.vb` | todo | â€” | â€” |
| `SmtManufacturing\FrmBendLicense.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\FrmBendLicense.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\FrmBendLicense.vb` | todo | â€” | â€” |
| `SmtManufacturing\FrmCalcCycleTimeManagement.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\FrmCalcCycleTimeManagement.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\FrmCalcCycleTimeManagement.vb` | todo | â€” | â€” |
| `SmtManufacturing\FrmNestedSheet.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\FrmNestedSheet.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\FrmNestedSheet.vb` | todo | â€” | â€” |
| `SmtManufacturing\FrmPartIdentifier.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\FrmPartIdentifier.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\FrmPartIdentifier.vb` | todo | â€” | â€” |
| `SmtManufacturing\FrmSmtMaterialManagement.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\FrmSmtMaterialManagement.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\FrmSmtMaterialManagement.vb` | todo | â€” | â€” |
| `SmtManufacturing\frmTopsLicenses.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\frmTopsLicenses.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\frmTopsLicenses.vb` | todo | â€” | â€” |
| `SmtManufacturing\GeoViewerControl.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\GeoViewerControl.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\GeoViewerControl.vb` | todo | â€” | â€” |
| `SmtManufacturing\ISmtFileViewer.vb` | todo | â€” | â€” |
| `SmtManufacturing\Job.vb` | todo | â€” | â€” |
| `SmtManufacturing\JPLT_DistrSticker.vb` | todo | â€” | â€” |
| `SmtManufacturing\JPLT_PartIdentSticker.vb` | todo | â€” | â€” |
| `SmtManufacturing\JPLT_SheetIdentSticker.vb` | todo | â€” | â€” |
| `SmtManufacturing\LaserCalc.vb` | todo | â€” | â€” |
| `SmtManufacturing\NestedSheet.vb` | todo | â€” | â€” |
| `SmtManufacturing\NestPart.vb` | todo | â€” | â€” |
| `SmtManufacturing\Oid.vb` | todo | â€” | â€” |
| `SmtManufacturing\Part.vb` | todo | â€” | â€” |
| `SmtManufacturing\PreProcessorTest.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\PreProcessorTest.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\PreProcessorTest.vb` | todo | â€” | â€” |
| `SmtManufacturing\ProductionOrderCleanupHandler.vb` | todo | â€” | â€” |
| `SmtManufacturing\Smt3DImportExclusionHelper.vb` | todo | â€” | â€” |
| `SmtManufacturing\SmtFileViewerFactory.vb` | todo | â€” | â€” |
| `SmtManufacturing\SmtPartViewer.Designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\SmtPartViewer.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\SmtPartViewer.vb` | todo | â€” | â€” |
| `SmtManufacturing\TafInterpreter\Part.vb` | todo | â€” | â€” |
| `SmtManufacturing\TafInterpreter\PartInstance.vb` | todo | â€” | â€” |
| `SmtManufacturing\TafInterpreter\TAF.vb` | todo | â€” | â€” |
| `SmtManufacturing\TmtInterpreter\TMT.vb` | todo | â€” | â€” |
| `SmtManufacturing\TruTops.vb` | todo | â€” | â€” |
| `SmtManufacturing\UCTAFViewer.designer.vb` | generated | VS Forms designer partial | â€” |
| `SmtManufacturing\UCTAFViewer.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtManufacturing\UCTAFViewer.vb` | todo | â€” | â€” |
| `SmtManufacturing\WorkViewBoost.vb` | todo | â€” | â€” |

### SolaDataConnector

**Total: 2** &nbsp; | &nbsp; .vb: 2 | .resx: 0 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `SolaDataConnector\AppHandler.vb` | todo | â€” | â€” |
| `SolaDataConnector\AppWindowHelper.vb` | todo | â€” | â€” |

### UniLink

**Total: 30** &nbsp; | &nbsp; .vb: 29 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `UniLink\ApplicationHandler.vb` | todo | â€” | â€” |
| `UniLink\ApplicationService.vb` | todo | â€” | â€” |
| `UniLink\CSVImportHandler.vb` | todo | â€” | â€” |
| `UniLink\Entities\List.vb` | todo | â€” | â€” |
| `UniLink\Entities\Profile.vb` | todo | â€” | â€” |
| `UniLink\ExceptionHandlers\ProfileSeriesExceptionHandler.vb` | todo | â€” | â€” |
| `UniLink\ExceptionHandlers\SourceFilesExceptionHandler.vb` | todo | â€” | â€” |
| `UniLink\Exceptions\ProfileSeriesException.vb` | todo | â€” | â€” |
| `UniLink\Exceptions\SourceFilesException.vb` | todo | â€” | â€” |
| `UniLink\ExportConverter.vb` | todo | â€” | â€” |
| `UniLink\ExportHelper.vb` | todo | â€” | â€” |
| `UniLink\ExportInstructions.vb` | todo | â€” | â€” |
| `UniLink\Factories\ListFactory.vb` | todo | â€” | â€” |
| `UniLink\ListDataService.vb` | todo | â€” | â€” |
| `UniLink\Machine.vb` | todo | â€” | â€” |
| `UniLink\MachineHelper.vb` | todo | â€” | â€” |
| `UniLink\MecalAriel4Export.vb` | todo | â€” | â€” |
| `UniLink\MultiStepReader\CSVHandler.vb` | todo | â€” | â€” |
| `UniLink\MultiStepReader\LogDetailContent.vb` | todo | â€” | â€” |
| `UniLink\MultiStepReader\MasterLog.vb` | todo | â€” | â€” |
| `UniLink\MultiStepReader\MasterLogDetail.vb` | todo | â€” | â€” |
| `UniLink\MultiStepReader\MasterLogDetailFactory.vb` | todo | â€” | â€” |
| `UniLink\MultiStepReader\MasterLogDetailFile.vb` | todo | â€” | â€” |
| `UniLink\MultiStepReader\MasterLogDetailOptimiser.vb` | todo | â€” | â€” |
| `UniLink\MultiStepReader\MasterLogFactory.vb` | todo | â€” | â€” |
| `UniLink\MultiStepReader\MasterLogService.vb` | todo | â€” | â€” |
| `UniLink\Settings.vb` | todo | â€” | â€” |
| `UniLink\UI\TabControl.Designer.vb` | generated | VS Forms designer partial | â€” |
| `UniLink\UI\TabControl.resx` | generated | resource bundle (designer-managed) | â€” |
| `UniLink\UI\TabControl.vb` | todo | â€” | â€” |

### VentDuctConfigurator

**Total: 4** &nbsp; | &nbsp; .vb: 3 | .resx:  | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `VentDuctConfigurator\FrmVentDuctConfigurator.Designer.vb` | generated | VS Forms designer partial | â€” |
| `VentDuctConfigurator\FrmVentDuctConfigurator.resx` | generated | resource bundle (designer-managed) | â€” |
| `VentDuctConfigurator\FrmVentDuctConfigurator.vb` | todo | â€” | â€” |
| `VentDuctConfigurator\VentDuct.vb` | todo | â€” | â€” |

### WebClock

**Total: 13** &nbsp; | &nbsp; .vb: 9 | .resx: 4 | assets: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `WebClock\FrmChangedTimeReg.Designer.vb` | generated | VS Forms designer partial | â€” |
| `WebClock\FrmChangedTimeReg.resx` | generated | resource bundle (designer-managed) | â€” |
| `WebClock\FrmChangedTimeReg.vb` | todo | â€” | â€” |
| `WebClock\FrmCurrentTimeReg.Designer.vb` | generated | VS Forms designer partial | â€” |
| `WebClock\FrmCurrentTimeReg.resx` | generated | resource bundle (designer-managed) | â€” |
| `WebClock\FrmCurrentTimeReg.vb` | todo | â€” | â€” |
| `WebClock\FrmOfficeClockDetailLines.Designer.vb` | generated | VS Forms designer partial | â€” |
| `WebClock\FrmOfficeClockDetailLines.resx` | generated | resource bundle (designer-managed) | â€” |
| `WebClock\FrmOfficeClockDetailLines.vb` | todo | â€” | â€” |
| `WebClock\FrmTimeRegistration.designer.vb` | generated | VS Forms designer partial | â€” |
| `WebClock\FrmTimeRegistration.resx` | generated | resource bundle (designer-managed) | â€” |
| `WebClock\FrmTimeRegistration.vb` | todo | â€” | â€” |
| `WebClock\WebClock.vb` | todo | â€” | â€” |

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
| `FileReaderHelper.cs` | todo | â€” | â€” |
| `FileReaderHelper.vb` | todo | â€” | â€” |
| `JAZO Zevenaar bv.snk` | config | signing key | â€” |
| `LayerConverter.cs` | todo | â€” | â€” |
| `LayerConverter.vb` | todo | â€” | â€” |
| `MigrationFix.cs` | todo | â€” | â€” |
| `MigrationFix.vb` | todo | â€” | â€” |
| `PMILabel.cs` | todo | â€” | â€” |
| `PMILabelCollection.cs` | todo | â€” | â€” |
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
| `GeoInterpreter\FlatGeometry.cs` | todo | â€” | â€” |
| `GeoInterpreter\FlatGeometry.vb` | todo | â€” | â€” |
| `GeoInterpreter\FlatGeometryDxfExport.cs` | todo | â€” | â€” |
| `GeoInterpreter\FlatGeometryDxfExport.vb` | todo | â€” | â€” |
| `GeoInterpreter\FlatGeometryGeoExport.cs` | todo | â€” | â€” |
| `GeoInterpreter\FlatGeometryGeoExport.vb` | todo | â€” | â€” |
| `GeoInterpreter\FlatGeometryReader.cs` | todo | â€” | â€” |
| `GeoInterpreter\FlatGeometryReader.vb` | todo | â€” | â€” |
| `GeoInterpreter\IFlatGeometryExport.cs` | todo | â€” | â€” |
| `GeoInterpreter\IFlatGeometryExport.vb` | todo | â€” | â€” |

### TruTopsLib / PMI

**Total: 5** &nbsp; | &nbsp; .vb: 5 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `PMI\IPmiLabel.vb` | todo | â€” | â€” |
| `PMI\PMILabel.vb` | todo | â€” | â€” |
| `PMI\PMILabelCollection.vb` | todo | â€” | â€” |
| `PMI\PmiLabelCountersink.vb` | todo | â€” | â€” |
| `PMI\PmiLabelThreadNote.vb` | todo | â€” | â€” |

### TruTopsLib / TopsFile

**Total: 34** &nbsp; | &nbsp; .vb: 17 | .cs: 17 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `TopsFile\BendLine.cs` | todo | â€” | â€” |
| `TopsFile\BendLine.vb` | todo | â€” | â€” |
| `TopsFile\Body.cs` | todo | â€” | â€” |
| `TopsFile\Body.vb` | todo | â€” | â€” |
| `TopsFile\Bounds.cs` | todo | â€” | â€” |
| `TopsFile\Bounds.vb` | todo | â€” | â€” |
| `TopsFile\Contour.cs` | todo | â€” | â€” |
| `TopsFile\Contour.vb` | todo | â€” | â€” |
| `TopsFile\DataTypeHandler.cs` | todo | â€” | â€” |
| `TopsFile\DataTypeHandler.vb` | todo | â€” | â€” |
| `TopsFile\Parameters.cs` | todo | â€” | â€” |
| `TopsFile\Parameters.vb` | todo | â€” | â€” |
| `TopsFile\Point.cs` | todo | â€” | â€” |
| `TopsFile\Point.vb` | todo | â€” | â€” |
| `TopsFile\PointCollection.cs` | todo | â€” | â€” |
| `TopsFile\PointCollection.vb` | todo | â€” | â€” |
| `TopsFile\Properties.cs` | todo | â€” | â€” |
| `TopsFile\Properties.vb` | todo | â€” | â€” |
| `TopsFile\SubContour\Arc.cs` | todo | â€” | â€” |
| `TopsFile\SubContour\Arc.vb` | todo | â€” | â€” |
| `TopsFile\SubContour\Circle.cs` | todo | â€” | â€” |
| `TopsFile\SubContour\Circle.vb` | todo | â€” | â€” |
| `TopsFile\SubContour\Fillet.cs` | todo | â€” | â€” |
| `TopsFile\SubContour\Fillet.vb` | todo | â€” | â€” |
| `TopsFile\SubContour\Line.cs` | todo | â€” | â€” |
| `TopsFile\SubContour\Line.vb` | todo | â€” | â€” |
| `TopsFile\SubContour\SubContour.cs` | todo | â€” | â€” |
| `TopsFile\SubContour\SubContour.vb` | todo | â€” | â€” |
| `TopsFile\SubContour\Text.cs` | todo | â€” | â€” |
| `TopsFile\SubContour\Text.vb` | todo | â€” | â€” |
| `TopsFile\TextCollection.cs` | todo | â€” | â€” |
| `TopsFile\TextCollection.vb` | todo | â€” | â€” |
| `TopsFile\TTInfo.cs` | todo | â€” | â€” |
| `TopsFile\TTInfo.vb` | todo | â€” | â€” |



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
| `ComputerSetting.vb` | todo | â€” | â€” |
| `Connections.vb` | done | [[modules/icenterlib-connections]] | 2026-06-18 |
| `ICenterLib.vbproj` | config | config / project metadata | â€” |
| `ICenterLib.vbproj.user` | config | config / project metadata | â€” |
| `ICenterLib.vbproj.vspscc` | config | config / project metadata | â€” |
| `Images.vb` | todo | â€” | â€” |
| `JAZO Zevenaar bv.snk` | config | signing key | â€” |
| `Log.vb` | todo | â€” | â€” |
| `Main.vb` | todo | â€” | â€” |
| `packages.config` | config | config / project metadata | â€” |
| `PdfTools.vb` | todo | â€” | â€” |
| `UserSetting.vb` | todo | â€” | â€” |

### ICenterLib / CAD

**Total: 127** &nbsp; | &nbsp; .vb: 116 | .cs: 0 | .resx: 5

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `CAD\Creo\AppManager.vb` | todo | â€” | â€” |
| `CAD\Creo\AppVersion.vb` | todo | â€” | â€” |
| `CAD\Creo\CadApp.vb` | todo | â€” | â€” |
| `CAD\Creo\CadAppVersion.vb` | todo | â€” | â€” |
| `CAD\Creo\CtrlAppManager.Designer.vb` | generated | VS designer partial | â€” |
| `CAD\Creo\CtrlAppManager.resx` | generated | resource bundle (designer-managed) | â€” |
| `CAD\Creo\CtrlAppManager.vb` | todo | â€” | â€” |
| `CAD\Creo\Dimension.vb` | todo | â€” | â€” |
| `CAD\Creo\Environment.vb` | todo | â€” | â€” |
| `CAD\Creo\Feature.vb` | todo | â€” | â€” |
| `CAD\Creo\Features.vb` | todo | â€” | â€” |
| `CAD\Creo\FrmAppManager.Designer.vb` | generated | VS designer partial | â€” |
| `CAD\Creo\FrmAppManager.resx` | generated | resource bundle (designer-managed) | â€” |
| `CAD\Creo\FrmAppManager.vb` | todo | â€” | â€” |
| `CAD\Creo\FrmCreoLicense.Designer.vb` | generated | VS designer partial | â€” |
| `CAD\Creo\FrmCreoLicense.resx` | generated | resource bundle (designer-managed) | â€” |
| `CAD\Creo\FrmCreoLicense.vb` | todo | â€” | â€” |
| `CAD\Creo\License\License.vb` | todo | â€” | â€” |
| `CAD\Creo\License\LicenseResource.vb` | todo | â€” | â€” |
| `CAD\Creo\License\LicenseResourceHandler.vb` | todo | â€” | â€” |
| `CAD\Creo\License\LicenseSelectionHandler.vb` | todo | â€” | â€” |
| `CAD\Creo\Material.vb` | todo | â€” | â€” |
| `CAD\Creo\Materials.vb` | todo | â€” | â€” |
| `CAD\Creo\ModelInformation.vb` | todo | â€” | â€” |
| `CAD\Creo\ModelItem.vb` | todo | â€” | â€” |
| `CAD\Creo\Parameter.vb` | todo | â€” | â€” |
| `CAD\Creo\ParameterCollection.vb` | todo | â€” | â€” |
| `CAD\Creo\ParamValue\ParamValue.vb` | todo | â€” | â€” |
| `CAD\Creo\ParamValue\ParamValueBoolean.vb` | todo | â€” | â€” |
| `CAD\Creo\ParamValue\ParamValueDouble.vb` | todo | â€” | â€” |
| `CAD\Creo\ParamValue\ParamValueInteger.vb` | todo | â€” | â€” |
| `CAD\Creo\ParamValue\ParamValueString.vb` | todo | â€” | â€” |
| `CAD\Creo\PlmAppVersion.vb` | todo | â€” | â€” |
| `CAD\Creo\ProProgram\Design.vb` | todo | â€” | â€” |
| `CAD\Creo\ProProgram\ExecuteStatement.vb` | todo | â€” | â€” |
| `CAD\Creo\ProProgram\Functions.vb` | todo | â€” | â€” |
| `CAD\Creo\ProProgram\Input.vb` | todo | â€” | â€” |
| `CAD\Creo\ProProgram\VBCodeConverter.vb` | todo | â€” | â€” |
| `CAD\Creo\RegenerationInput.vb` | todo | â€” | â€” |
| `CAD\Creo\StartupFile.vb` | todo | â€” | â€” |
| `CAD\Creo\Toolbox.vb` | todo | â€” | â€” |
| `CAD\Creo\Tools\StpAssySplitter.vb` | todo | â€” | â€” |
| `CAD\Creo\Trailfile.vb` | todo | â€” | â€” |
| `CAD\CreoView\Application.vb` | todo | â€” | â€” |
| `CAD\CreoView\Configuration.vb` | todo | â€” | â€” |
| `CAD\CreoView\Converter.vb` | todo | â€” | â€” |
| `CAD\DXF\SvgConverter\DxfHelper.vb` | todo | â€” | â€” |
| `CAD\DXF\SvgConverter\DxfToSvgConverter.vb` | todo | â€” | â€” |
| `CAD\DXF\SvgConverter\Export.vb` | todo | â€” | â€” |
| `CAD\DXF\SvgConverter\Modifier.vb` | todo | â€” | â€” |
| `CAD\Geometry\BoundingBox.vb` | todo | â€” | â€” |
| `CAD\Geometry\BoundingBoxFactory.vb` | todo | â€” | â€” |
| `CAD\Geometry\Dxf3DProfileMill.vb` | todo | â€” | â€” |
| `CAD\Geometry\DxfEntityColor.vb` | todo | â€” | â€” |
| `CAD\Geometry\Earcut.vb` | todo | â€” | â€” |
| `CAD\Geometry\Earcut_CSharp.vb` | todo | â€” | â€” |
| `CAD\Geometry\GraphicsPathHelper.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\Configuration.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\Definition.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\DefinitionCollection.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\Exceptions\GenericObjectNotFoundException.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\GenericModelBackupConfiguration.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\ModelGeneratorOption.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\ModelgeneratorRequests\DesignDuplicationConfiguration.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\ModelgeneratorRequests\ElfsquadModelgeneratorRequest.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\ModelgeneratorRequests\IrisModelgeneratorInstructions.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\ModelgeneratorRequests\IrisModelgeneratorTaskResult.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\Operations\Checkin.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\Operations\CreateWS.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\Operations\DeleteWS.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\Operations\Download.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\Operations\EraseUndisplayedModels.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\Operations\ExportDocumentOperation.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\Operations\JZCheckoutFolders.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\Operations\JZExportByNumber.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\Operations\JZImportByNumber.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\Operations\JZRenameObjectNoServer.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\Operations\OpenInProE.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\Operations\Operation.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\Operations\OperationCollection.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\Operations\RegenReadPar.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\Operations\Register.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\Operations\Rename.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\Operations\Save.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\Operations\SetWorkingDirectory.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\Operations\Unregister.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\RenameRule.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\RenameRuleCollection.vb` | todo | â€” | â€” |
| `CAD\Modelgenerator\TriggerFile.vb` | todo | â€” | â€” |
| `CAD\OpenGL\CtrlOpenGLViewer.Designer.vb` | generated | VS designer partial | â€” |
| `CAD\OpenGL\CtrlOpenGLViewer.vb` | todo | â€” | â€” |
| `CAD\OpenGL\GLUtil.vb` | todo | â€” | â€” |
| `CAD\PLM\Archive.vb` | todo | â€” | â€” |
| `CAD\PLM\AutoPromotionRequest.vb` | todo | â€” | â€” |
| `CAD\PLM\AutoPromotionRequestHandler.vb` | todo | â€” | â€” |
| `CAD\PLM\AutoPromotionRequestParameters.vb` | todo | â€” | â€” |
| `CAD\PLM\Container.vb` | todo | â€” | â€” |
| `CAD\PLM\EPMDocument.vb` | todo | â€” | â€” |
| `CAD\PLM\FileServer.vb` | todo | â€” | â€” |
| `CAD\PLM\Folder.vb` | todo | â€” | â€” |
| `CAD\PLM\FrmAutomaticPromotionRequestInput.Designer.vb` | generated | VS designer partial | â€” |
| `CAD\PLM\FrmAutomaticPromotionRequestInput.resx` | generated | resource bundle (designer-managed) | â€” |
| `CAD\PLM\FrmAutomaticPromotionRequestInput.vb` | todo | â€” | â€” |
| `CAD\PLM\FrmVaultFolderChart.Designer.vb` | generated | VS designer partial | â€” |
| `CAD\PLM\FrmVaultFolderChart.resx` | generated | resource bundle (designer-managed) | â€” |
| `CAD\PLM\FrmVaultFolderChart.vb` | todo | â€” | â€” |
| `CAD\PLM\InfoEngineParameter.vb` | todo | â€” | â€” |
| `CAD\PLM\InfoEngineParameterCollection.vb` | todo | â€” | â€” |
| `CAD\PLM\LifeCycleState.vb` | todo | â€” | â€” |
| `CAD\PLM\Product.vb` | todo | â€” | â€” |
| `CAD\PLM\PromotionNotice.vb` | todo | â€” | â€” |
| `CAD\PLM\ServerManagement.vb` | todo | â€” | â€” |
| `CAD\PLM\Task\AddVaultFolderDetailsLogEntry.vb` | todo | â€” | â€” |
| `CAD\PLM\Task\GenericModelBackup.vb` | todo | â€” | â€” |
| `CAD\PLM\Task\PurgePromotionNotices.vb` | todo | â€” | â€” |
| `CAD\PLM\Task\Task.vb` | todo | â€” | â€” |
| `CAD\PLM\Toolbox.vb` | todo | â€” | â€” |
| `CAD\PLM\User.vb` | todo | â€” | â€” |
| `CAD\PLM\VaultCleanupAuditLogs.vb` | todo | â€” | â€” |
| `CAD\PLM\Version.vb` | todo | â€” | â€” |
| `CAD\PLM\WCObject.vb` | todo | â€” | â€” |
| `CAD\Publisher\Common.vb` | todo | â€” | â€” |
| `CAD\SolidEdge\IfcExport.vb` | todo | â€” | â€” |
| `CAD\SolidEdge\Importer.vb` | todo | â€” | â€” |
| `CAD\SolidEdge\Settings.vb` | todo | â€” | â€” |
| `CAD\SolidEdge\StepExport.vb` | todo | â€” | â€” |
| `CAD\SolidEdge\Toolkit.vb` | todo | â€” | â€” |

### ICenterLib / CadBatchServer

**Total: 17** &nbsp; | &nbsp; .vb: 17 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `CadBatchServer\CadBatchserverDataService.vb` | todo | â€” | â€” |
| `CadBatchServer\CadBatchserverStatus.vb` | todo | â€” | â€” |
| `CadBatchServer\CadBatchserverStatusCollection.vb` | todo | â€” | â€” |
| `CadBatchServer\CadBatchserverStatusDataService.vb` | todo | â€” | â€” |
| `CadBatchServer\DistributedLockCreoPublish.vb` | todo | â€” | â€” |
| `CadBatchServer\JobAlreadyExistsException.vb` | todo | â€” | â€” |
| `CadBatchServer\JobDataService.vb` | todo | â€” | â€” |
| `CadBatchServer\JobParameters.vb` | todo | â€” | â€” |
| `CadBatchServer\JobToolbox.vb` | todo | â€” | â€” |
| `CadBatchServer\Modelgenerator\CadInputParameters.vb` | todo | â€” | â€” |
| `CadBatchServer\Modelgenerator\DuplicateInstruction.vb` | todo | â€” | â€” |
| `CadBatchServer\Modelgenerator\ModelgeneratorInstructions.vb` | todo | â€” | â€” |
| `CadBatchServer\Modelgenerator\ModelgeneratorTask.vb` | todo | â€” | â€” |
| `CadBatchServer\ModelgeneratorDataService.vb` | todo | â€” | â€” |
| `CadBatchServer\PublishJobInstructions.vb` | todo | â€” | â€” |
| `CadBatchServer\PublishMonitor.vb` | todo | â€” | â€” |
| `CadBatchServer\PublishWatchDirProcessor.vb` | todo | â€” | â€” |

### ICenterLib / Comparer

**Total: 1** &nbsp; | &nbsp; .vb:  | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Comparer\DateComparer.vb` | todo | â€” | â€” |

### ICenterLib / Connections

**Total: 1** &nbsp; | &nbsp; .vb:  | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Connections\WebClients\WindchillWebClientProvider.vb` | todo | â€” | â€” |

### ICenterLib / CrystalReport

**Total: 5** &nbsp; | &nbsp; .vb: 5 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `CrystalReport\ExportInstruction.vb` | todo | â€” | â€” |
| `CrystalReport\ExportRequest.vb` | todo | â€” | â€” |
| `CrystalReport\ExportResult.vb` | todo | â€” | â€” |
| `CrystalReport\PrintingInstruction.vb` | todo | â€” | â€” |
| `CrystalReport\ReportParameter.vb` | todo | â€” | â€” |

### ICenterLib / DataHandler

**Total: 23** &nbsp; | &nbsp; .vb: 19 | .cs: 0 | .resx: 2

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `DataHandler\BetterDataGridView.vb` | todo | â€” | â€” |
| `DataHandler\DataSetComparer.vb` | todo | â€” | â€” |
| `DataHandler\DataSetCompareResult.vb` | todo | â€” | â€” |
| `DataHandler\DataSetCompareTolerance.vb` | todo | â€” | â€” |
| `DataHandler\DataTableColumnSchema.vb` | todo | â€” | â€” |
| `DataHandler\DataTableColumnSchemaHandler.vb` | todo | â€” | â€” |
| `DataHandler\DataTableColumnSchemaRecord.vb` | todo | â€” | â€” |
| `DataHandler\ExportExcel.vb` | todo | â€” | â€” |
| `DataHandler\FrmDataGridView.Designer.vb` | generated | VS designer partial | â€” |
| `DataHandler\FrmDataGridView.resx` | generated | resource bundle (designer-managed) | â€” |
| `DataHandler\FrmDataGridView.vb` | todo | â€” | â€” |
| `DataHandler\FrmDataViewer.Designer.vb` | generated | VS designer partial | â€” |
| `DataHandler\FrmDataViewer.resx` | generated | resource bundle (designer-managed) | â€” |
| `DataHandler\FrmDataViewer.vb` | todo | â€” | â€” |
| `DataHandler\GenericQuery.vb` | todo | â€” | â€” |
| `DataHandler\IStreamWrapper.vb` | todo | â€” | â€” |
| `DataHandler\JsonHelper.vb` | todo | â€” | â€” |
| `DataHandler\OpenXml\Excel.vb` | todo | â€” | â€” |
| `DataHandler\OpenXml\OpenXmlSpreadsheet.vb` | todo | â€” | â€” |
| `DataHandler\QrCode.vb` | todo | â€” | â€” |
| `DataHandler\Selection.vb` | todo | â€” | â€” |
| `DataHandler\Toolbox.vb` | todo | â€” | â€” |
| `DataHandler\ZeroCode.vb` | todo | â€” | â€” |

### ICenterLib / DataServices

**Total: 10** &nbsp; | &nbsp; .vb: 10 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `DataServices\Contracts\IProdexConfiguratorDataService.vb` | todo | â€” | â€” |
| `DataServices\Contracts\IProdexDossierDetailDesignDataService.vb` | todo | â€” | â€” |
| `DataServices\Contracts\IProdexModelgeneratorDataService.vb` | todo | â€” | â€” |
| `DataServices\CrystalReportDataService.vb` | todo | â€” | â€” |
| `DataServices\DistributedLockDataService.vb` | todo | â€” | â€” |
| `DataServices\ElfsquadDataService.vb` | todo | â€” | â€” |
| `DataServices\ProdexConfiguratorDataService.vb` | todo | â€” | â€” |
| `DataServices\ProdexDossierDetailDesignDataService.vb` | todo | â€” | â€” |
| `DataServices\ProdexModelgeneratorDataService.vb` | todo | â€” | â€” |
| `DataServices\ProductDbDataService.vb` | todo | â€” | â€” |

### ICenterLib / Debug

**Total: 1** &nbsp; | &nbsp; .vb:  | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Debug\Debug1.vb` | todo | â€” | â€” |

### ICenterLib / DistributedLock

**Total: 1** &nbsp; | &nbsp; .vb:  | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `DistributedLock\DistributedLock.vb` | todo | â€” | â€” |

### ICenterLib / Elfsquad

**Total: 9** &nbsp; | &nbsp; .vb: 5 | .cs: 0 | .resx: 2

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Elfsquad\ConfigurationMappingMismatchException.vb` | todo | â€” | â€” |
| `Elfsquad\ConfigurationRequestType.vb` | todo | â€” | â€” |
| `Elfsquad\ModelgeneratorHelper.vb` | todo | â€” | â€” |
| `Elfsquad\UCConfigurationManager.Designer.vb` | generated | VS designer partial | â€” |
| `Elfsquad\UCConfigurationManager.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elfsquad\UCConfigurationManager.vb` | todo | â€” | â€” |
| `Elfsquad\UCModelgenerator.Designer.vb` | generated | VS designer partial | â€” |
| `Elfsquad\UCModelgenerator.resx` | generated | resource bundle (designer-managed) | â€” |
| `Elfsquad\UCModelgenerator.vb` | todo | â€” | â€” |

### ICenterLib / Enums

**Total: 2** &nbsp; | &nbsp; .vb: 2 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Enums\Application.vb` | todo | â€” | â€” |
| `Enums\ISAH.vb` | todo | â€” | â€” |

### ICenterLib / GUI

**Total: 3** &nbsp; | &nbsp; .vb: 3 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `GUI\MyMenuItem.vb` | todo | â€” | â€” |
| `GUI\MyMenuItemConverter.vb` | todo | â€” | â€” |
| `GUI\ToolStripMenuItemHelper.vb` | todo | â€” | â€” |

### ICenterLib / Helpers

**Total: 1** &nbsp; | &nbsp; .vb:  | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Helpers\UrlHelper.vb` | todo | â€” | â€” |

### ICenterLib / iCenter

**Total: 29** &nbsp; | &nbsp; .vb: 24 | .cs: 0 | .resx: 2

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `iCenter\BillOfOper.vb` | todo | â€” | â€” |
| `iCenter\Client.vb` | todo | â€” | â€” |
| `iCenter\CtrlImageIdentification.designer.vb` | generated | VS designer partial | â€” |
| `iCenter\CtrlImageIdentification.resx` | generated | resource bundle (designer-managed) | â€” |
| `iCenter\CtrlImageIdentification.vb` | todo | â€” | â€” |
| `iCenter\CtrlRadButtonIdent.designer.vb` | generated | VS designer partial | â€” |
| `iCenter\CtrlRadButtonIdent.vb` | todo | â€” | â€” |
| `iCenter\DataServices\CoatingLayerThicknessDataService.vb` | todo | â€” | â€” |
| `iCenter\DossierContactFavorite.vb` | todo | â€” | â€” |
| `iCenter\ExternalReference.vb` | todo | â€” | â€” |
| `iCenter\ExternalReferences.vb` | todo | â€” | â€” |
| `iCenter\FrmIdentification.designer.vb` | generated | VS designer partial | â€” |
| `iCenter\FrmIdentification.resx` | generated | resource bundle (designer-managed) | â€” |
| `iCenter\FrmIdentification.vb` | todo | â€” | â€” |
| `iCenter\IPBatch.vb` | todo | â€” | â€” |
| `iCenter\IPOrder.vb` | todo | â€” | â€” |
| `iCenter\IPPacket.vb` | todo | â€” | â€” |
| `iCenter\IPPart.vb` | todo | â€” | â€” |
| `iCenter\Part.vb` | todo | â€” | â€” |
| `iCenter\ProductionMachineMultiPurpose.vb` | todo | â€” | â€” |
| `iCenter\ProductionMachines.vb` | todo | â€” | â€” |
| `iCenter\Servicedesk.vb` | todo | â€” | â€” |
| `iCenter\SurfaceTreatmentDefinition.vb` | todo | â€” | â€” |
| `iCenter\TimeRegistration.vb` | todo | â€” | â€” |
| `iCenter\WebClockAssistant\WebClockAssistantAnonymousHandler.vb` | todo | â€” | â€” |
| `iCenter\WebClockAssistant\WebClockAssistantGenericHandler.vb` | todo | â€” | â€” |
| `iCenter\WebClockAssistant\WebClockAssistantRepository.vb` | todo | â€” | â€” |
| `iCenter\WebClockAssistant\WebClockAssistantUserSelectionHandler.vb` | todo | â€” | â€” |
| `iCenter\XmlFile.vb` | todo | â€” | â€” |

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
| `ISAH\DataServices\EmployeeDataService.vb` | todo | â€” | â€” |
| `ISAH\DataServices\IsahCustomisingElfsquadDataService.vb` | todo | â€” | â€” |
| `ISAH\DataServices\MemoDetailDataService.vb` | todo | â€” | â€” |
| `ISAH\DataServices\PartDataService.vb` | done | [[modules/isah-part-and-dispatch]] | 2026-06-18 |
| `ISAH\DataServices\PartDispatchCollectorDataService.vb` | done | [[modules/isah-part-and-dispatch]] | 2026-06-18 |
| `ISAH\DataServices\ToolboxDataService.vb` | todo | â€” | â€” |
| `ISAH\DataServices\UpdateProdLeadTimeDataService.vb` | todo | â€” | â€” |
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
| `ISAH\Handlers\UpdateProdLeadTimeHandler.vb` | todo | â€” | â€” |
| `ISAH\Helpers\DossierDetailExtraHelper.vb` | done | [[modules/isah-dossier]] | 2026-06-18 |
| `ISAH\Helpers\EncryptionHelper.vb` | todo | â€” | â€” |
| `ISAH\Helpers\TextStyling\HtmlPlainTextHelper.vb` | todo | â€” | â€” |
| `ISAH\Helpers\TextStyling\IPlainTextHelper.vb` | todo | â€” | â€” |
| `ISAH\Helpers\TextStyling\PlainTextHelper.vb` | todo | â€” | â€” |
| `ISAH\Helpers\TextStyling\RtfPlainTextHelper.vb` | todo | â€” | â€” |
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
| `ISAH\ViewModels\CustomerAddressViewModel.vb` | todo | â€” | â€” |
| `ISAH\ViewModels\PartBasicViewModel.vb` | todo | â€” | â€” |
| `ISAH\WeighingFactor.vb` | done | [[modules/isah-leaves]] | 2026-06-18 |
| `ISAH\WorkView.vb` | done | [[modules/isah-leaves]] | 2026-06-18 |

### ICenterLib / JIBA

**Total: 14** &nbsp; | &nbsp; .vb: 14 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `JIBA\AppParameter.vb` | todo | â€” | â€” |
| `JIBA\Asset.vb` | todo | â€” | â€” |
| `JIBA\Company.vb` | todo | â€” | â€” |
| `JIBA\ConfigPart.vb` | todo | â€” | â€” |
| `JIBA\CustSat.vb` | todo | â€” | â€” |
| `JIBA\Employee.vb` | todo | â€” | â€” |
| `JIBA\Encryption.vb` | todo | â€” | â€” |
| `JIBA\Enums.vb` | todo | â€” | â€” |
| `JIBA\LinkItem.vb` | todo | â€” | â€” |
| `JIBA\Log.vb` | todo | â€” | â€” |
| `JIBA\Menu.vb` | todo | â€” | â€” |
| `JIBA\NavigationGroupItem.vb` | todo | â€” | â€” |
| `JIBA\SubMenu.vb` | todo | â€” | â€” |
| `JIBA\XmlData.vb` | todo | â€” | â€” |

### ICenterLib / JMail

**Total: 6** &nbsp; | &nbsp; .vb: 6 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `JMail\FileHelper.vb` | todo | â€” | â€” |
| `JMail\Message.vb` | todo | â€” | â€” |
| `JMail\SpecificationReportData.vb` | todo | â€” | â€” |
| `JMail\SpecificationReportHandler.vb` | todo | â€” | â€” |
| `JMail\StartOptions.vb` | todo | â€” | â€” |
| `JMail\StartOptionsHelper.vb` | todo | â€” | â€” |

### ICenterLib / LaserWork

**Total: 4** &nbsp; | &nbsp; .vb: 2 | .cs: 0 | .resx: 

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `LaserWork\AppWrapper.vb` | todo | â€” | â€” |
| `LaserWork\UCLaserWork.Designer.vb` | generated | VS designer partial | â€” |
| `LaserWork\UCLaserWork.resx` | generated | resource bundle (designer-managed) | â€” |
| `LaserWork\UCLaserWork.vb` | todo | â€” | â€” |

### ICenterLib / Metabase

**Total: 4** &nbsp; | &nbsp; .vb: 4 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Metabase\ParameterParsing.vb` | todo | â€” | â€” |
| `Metabase\ParameterResolver.vb` | todo | â€” | â€” |
| `Metabase\QueryHelper.vb` | todo | â€” | â€” |
| `Metabase\UrlHelper.vb` | todo | â€” | â€” |

### ICenterLib / ModelDefinition

**Total: 3** &nbsp; | &nbsp; .vb: 3 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `ModelDefinition\Dimension.vb` | todo | â€” | â€” |
| `ModelDefinition\DimensionCollection.vb` | todo | â€” | â€” |
| `ModelDefinition\DimensionType.vb` | todo | â€” | â€” |

### ICenterLib / MySystem

**Total: 22** &nbsp; | &nbsp; .vb: 20 | .cs: 0 | .resx: 

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `MySystem\Computer.vb` | todo | â€” | â€” |
| `MySystem\Encryption\FrmEncrypt.Designer.vb` | generated | VS designer partial | â€” |
| `MySystem\Encryption\FrmEncrypt.resx` | generated | resource bundle (designer-managed) | â€” |
| `MySystem\Encryption\FrmEncrypt.vb` | todo | â€” | â€” |
| `MySystem\Encryption\SecurityController.vb` | todo | â€” | â€” |
| `MySystem\Environment.vb` | todo | â€” | â€” |
| `MySystem\ExceptionList.vb` | todo | â€” | â€” |
| `MySystem\FileSystem.vb` | todo | â€” | â€” |
| `MySystem\HealthMonitorClient.vb` | todo | â€” | â€” |
| `MySystem\HelpHandler.vb` | todo | â€” | â€” |
| `MySystem\ICenterObjectNotFoundException.vb` | todo | â€” | â€” |
| `MySystem\Math.vb` | todo | â€” | â€” |
| `MySystem\MyProcess.vb` | todo | â€” | â€” |
| `MySystem\Net\TcpServer.vb` | todo | â€” | â€” |
| `MySystem\Network.vb` | todo | â€” | â€” |
| `MySystem\OpenWindowGetter.vb` | todo | â€” | â€” |
| `MySystem\PowerShellWrapper.vb` | todo | â€” | â€” |
| `MySystem\Printer.vb` | todo | â€” | â€” |
| `MySystem\Registry.vb` | todo | â€” | â€” |
| `MySystem\TerminalServerSessions.vb` | todo | â€” | â€” |
| `MySystem\Window.vb` | todo | â€” | â€” |
| `MySystem\WindowsUser.vb` | todo | â€” | â€” |

### ICenterLib / PCFNet

**Total: 45** &nbsp; | &nbsp; .vb: 39 | .cs: 0 | .resx: 3

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `PCFNet\BOM.vb` | todo | â€” | â€” |
| `PCFNet\BOO.vb` | todo | â€” | â€” |
| `PCFNet\Calculation.vb` | todo | â€” | â€” |
| `PCFNet\ControlDefinition.vb` | todo | â€” | â€” |
| `PCFNet\ControlDefinitionComparer.vb` | todo | â€” | â€” |
| `PCFNet\ControlMapping\ControlMap.vb` | todo | â€” | â€” |
| `PCFNet\ControlMapping\Mapping.vb` | todo | â€” | â€” |
| `PCFNet\ControlMapping\MappingCollection.vb` | todo | â€” | â€” |
| `PCFNet\ControlMapping\MappingQuery.vb` | todo | â€” | â€” |
| `PCFNet\ControlMapping\Parameter.vb` | todo | â€” | â€” |
| `PCFNet\ControlMappingDesigner\UCControlMappingDesigner.Designer.vb` | generated | VS designer partial | â€” |
| `PCFNet\ControlMappingDesigner\UCControlMappingDesigner.resx` | generated | resource bundle (designer-managed) | â€” |
| `PCFNet\ControlMappingDesigner\UCControlMappingDesigner.vb` | todo | â€” | â€” |
| `PCFNet\ControlMappingImport.vb` | todo | â€” | â€” |
| `PCFNet\CPart.vb` | todo | â€” | â€” |
| `PCFNet\ExcelObject.vb` | todo | â€” | â€” |
| `PCFNet\ExcelObjectCalculator.vb` | todo | â€” | â€” |
| `PCFNet\FastenerCalculator.vb` | todo | â€” | â€” |
| `PCFNet\GenericPart.vb` | todo | â€” | â€” |
| `PCFNet\GenericPartTemplate.vb` | todo | â€” | â€” |
| `PCFNet\JConfigurator.vb` | todo | â€” | â€” |
| `PCFNet\ManualOperation.vb` | todo | â€” | â€” |
| `PCFNet\ManualProperty.vb` | todo | â€” | â€” |
| `PCFNet\NameMapping.vb` | todo | â€” | â€” |
| `PCFNet\NameMappingRule.vb` | todo | â€” | â€” |
| `PCFNet\ParametersMapping\AluBasicWallLouver001.vb` | todo | â€” | â€” |
| `PCFNet\ParametersMapping\AluDoor001.vb` | todo | â€” | â€” |
| `PCFNet\ParametersMapping\AluLouver001.vb` | todo | â€” | â€” |
| `PCFNet\ParametersMapping\ParametersMappingBase.vb` | todo | â€” | â€” |
| `PCFNet\ParametersMapping\PlankAssembly.vb` | todo | â€” | â€” |
| `PCFNet\ParametersMapping\StlDoor001.vb` | todo | â€” | â€” |
| `PCFNet\PcfCompiler.vb` | todo | â€” | â€” |
| `PCFNet\PcfNetDataSet.vb` | todo | â€” | â€” |
| `PCFNet\ProductValidation.vb` | todo | â€” | â€” |
| `PCFNet\PropertyDefinition.vb` | todo | â€” | â€” |
| `PCFNet\PropertyDefinitionType.vb` | todo | â€” | â€” |
| `PCFNet\SmtCalculator.vb` | todo | â€” | â€” |
| `PCFNet\TcpClientConfiguration.vb` | todo | â€” | â€” |
| `PCFNet\UCInputControls.Designer.vb` | generated | VS designer partial | â€” |
| `PCFNet\UCInputControls.resx` | generated | resource bundle (designer-managed) | â€” |
| `PCFNet\UCInputControls.vb` | todo | â€” | â€” |
| `PCFNet\UCOption.Designer.vb` | generated | VS designer partial | â€” |
| `PCFNet\UCOption.resx` | generated | resource bundle (designer-managed) | â€” |
| `PCFNet\UCOption.vb` | todo | â€” | â€” |
| `PCFNet\WAASHandler.vb` | todo | â€” | â€” |

### ICenterLib / Prodex

**Total: 6** &nbsp; | &nbsp; .vb: 6 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Prodex\Models\DesignConfiguration.vb` | todo | â€” | â€” |
| `Prodex\Models\DesignParameter.vb` | todo | â€” | â€” |
| `Prodex\Models\DesignParameters.vb` | todo | â€” | â€” |
| `Prodex\Models\DossierDetailDesignCalculationRequest.vb` | todo | â€” | â€” |
| `Prodex\Models\DossierDetailDesignUpdateRequest.vb` | todo | â€” | â€” |
| `Prodex\ViewModels\DossierDetailDesignCalculationVM.vb` | todo | â€” | â€” |

### ICenterLib / ProductDb

**Total: 31** &nbsp; | &nbsp; .vb: 21 | .cs: 0 | .resx: 5

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `ProductDb\CloneProductHandler.vb` | todo | â€” | â€” |
| `ProductDb\CommonDb.vb` | todo | â€” | â€” |
| `ProductDb\DeclarationOfPerformance.vb` | todo | â€” | â€” |
| `ProductDb\ExcelTemplate.vb` | todo | â€” | â€” |
| `ProductDb\ExcelWorkBookHelper.vb` | todo | â€” | â€” |
| `ProductDb\FrmCloneProduct.Designer.vb` | generated | VS designer partial | â€” |
| `ProductDb\FrmCloneProduct.resx` | generated | resource bundle (designer-managed) | â€” |
| `ProductDb\FrmCloneProduct.vb` | todo | â€” | â€” |
| `ProductDb\FrmProductDbPriceList.Designer.vb` | generated | VS designer partial | â€” |
| `ProductDb\FrmProductDbPriceList.resx` | generated | resource bundle (designer-managed) | â€” |
| `ProductDb\FrmProductDbPriceList.vb` | todo | â€” | â€” |
| `ProductDb\FrmProductPricePart.Designer.vb` | generated | VS designer partial | â€” |
| `ProductDb\FrmProductPricePart.resx` | generated | resource bundle (designer-managed) | â€” |
| `ProductDb\FrmProductPricePart.vb` | todo | â€” | â€” |
| `ProductDb\Language.vb` | todo | â€” | â€” |
| `ProductDb\PriceListHelper.vb` | todo | â€” | â€” |
| `ProductDb\PricePart.vb` | todo | â€” | â€” |
| `ProductDb\Product.vb` | todo | â€” | â€” |
| `ProductDb\ProductConfiguration.vb` | todo | â€” | â€” |
| `ProductDb\ProductConfigurationDataService.vb` | todo | â€” | â€” |
| `ProductDb\ProductConfiguratorMapping.vb` | todo | â€” | â€” |
| `ProductDb\ProductFilter.vb` | todo | â€” | â€” |
| `ProductDb\ProductFilterGroup.vb` | todo | â€” | â€” |
| `ProductDb\ProductGroup.vb` | todo | â€” | â€” |
| `ProductDb\ProductPricePart.vb` | todo | â€” | â€” |
| `ProductDb\UCPropertyDefinitionEditor.Designer.vb` | generated | VS designer partial | â€” |
| `ProductDb\UCPropertyDefinitionEditor.resx` | generated | resource bundle (designer-managed) | â€” |
| `ProductDb\UCPropertyDefinitionEditor.vb` | todo | â€” | â€” |
| `ProductDb\UCPropertyDefinitionManager.Designer.vb` | generated | VS designer partial | â€” |
| `ProductDb\UCPropertyDefinitionManager.resx` | generated | resource bundle (designer-managed) | â€” |
| `ProductDb\UCPropertyDefinitionManager.vb` | todo | â€” | â€” |

### ICenterLib / Production

**Total: 23** &nbsp; | &nbsp; .vb: 19 | .cs: 0 | .resx: 2

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Production\AutoIPOrderSelection.vb` | todo | â€” | â€” |
| `Production\BaseProductionItem.vb` | todo | â€” | â€” |
| `Production\BOMFilter.vb` | todo | â€” | â€” |
| `Production\CEChecklist.vb` | todo | â€” | â€” |
| `Production\FGQualityControlPart.vb` | todo | â€” | â€” |
| `Production\FrmProdMachineSelector.Designer.vb` | generated | VS designer partial | â€” |
| `Production\FrmProdMachineSelector.resx` | generated | resource bundle (designer-managed) | â€” |
| `Production\FrmProdMachineSelector.vb` | todo | â€” | â€” |
| `Production\KanbanBin.vb` | todo | â€” | â€” |
| `Production\KeyPerformanceIndicator.vb` | todo | â€” | â€” |
| `Production\LabelLog.vb` | todo | â€” | â€” |
| `Production\ProdChecklist.vb` | todo | â€” | â€” |
| `Production\ProdChecklistTag.vb` | todo | â€” | â€” |
| `Production\ProductionLine.vb` | todo | â€” | â€” |
| `Production\ProductionLog.vb` | todo | â€” | â€” |
| `Production\ProductionProfileCutItem.vb` | todo | â€” | â€” |
| `Production\ProductionProfileCutItemHandler.vb` | todo | â€” | â€” |
| `Production\ProductionProfileCutItemsHandler.vb` | done | [[modules/production-profile-cut-items]] | 2026-06-18 |
| `Production\ProductionRegistrationAnalysisRange.vb` | todo | â€” | â€” |
| `Production\SmtBendQueue.vb` | todo | â€” | â€” |
| `Production\UCProdLineLeanStatus.Designer.vb` | generated | VS designer partial | â€” |
| `Production\UCProdLineLeanStatus.resx` | generated | resource bundle (designer-managed) | â€” |
| `Production\UCProdLineLeanStatus.vb` | todo | â€” | â€” |

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
| `SmartForms\SmartForm.vb` | todo | â€” | â€” |

### ICenterLib / SmtCadCam

**Total: 4** &nbsp; | &nbsp; .vb: 4 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `SmtCadCam\PreProcessorHandler.vb` | todo | â€” | â€” |
| `SmtCadCam\SpaceClaimApiHelper.vb` | todo | â€” | â€” |
| `SmtCadCam\SpaceClaimServerRequest.vb` | todo | â€” | â€” |
| `SmtCadCam\SpaceClaimServerRequestDataService.vb` | todo | â€” | â€” |

### ICenterLib / SmtProduction

**Total: 128** &nbsp; | &nbsp; .vb: 126 | .cs: 0 | .resx: 

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `SmtProduction\DataServices\CutSheetOperRegistrationDataService.vb` | todo | â€” | â€” |
| `SmtProduction\DataServices\OseonAppContextDataService.vb` | todo | â€” | â€” |
| `SmtProduction\DataServices\PartDataService.vb` | todo | â€” | â€” |
| `SmtProduction\DataServices\ProductionOrderDataService.vb` | todo | â€” | â€” |
| `SmtProduction\DataServices\WorkplaceEmployeeLinkDataService.vb` | todo | â€” | â€” |
| `SmtProduction\Entities\CutSheetOperRegistration.vb` | todo | â€” | â€” |
| `SmtProduction\Entities\ImportSettings2D.vb` | todo | â€” | â€” |
| `SmtProduction\Entities\ImportSettings3D.vb` | todo | â€” | â€” |
| `SmtProduction\Entities\ImportSettingsBase.vb` | todo | â€” | â€” |
| `SmtProduction\Entities\OseonAppContext.vb` | todo | â€” | â€” |
| `SmtProduction\Entities\WorkplaceEmployeeLink.vb` | todo | â€” | â€” |
| `SmtProduction\ImportFileValidator.vb` | todo | â€” | â€” |
| `SmtProduction\ProgrammingEnvironment.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Client\Application.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Collections\CadCamDocumentCollection.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Contracts\IAppMerkerDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Contracts\IBasicMaterialDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Contracts\IBendSolutionDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Contracts\ICadCamDocumentDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Contracts\ICutSheetDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Contracts\IDataTableColumnSchemaDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Contracts\IMasterWorkPlanDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Contracts\IPartBendSolutionDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Contracts\IPartDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Contracts\IPartOnTableDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Contracts\IPartStatusDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Contracts\IPdmDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Contracts\IProductionOperationDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Contracts\IProductionOrderDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Contracts\IRawMaterialDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Contracts\ITTNGActionDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Contracts\IWorkplaceDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\DataServices\AppMerkerDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\DataServices\BasicMaterialDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\DataServices\BendSolutionDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\DataServices\CadCamDocumentDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\DataServices\CutSheetDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\DataServices\MasterWorkPlanDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\DataServices\PartBendSolutionDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\DataServices\PartDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\DataServices\PartOnTableDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\DataServices\PartStatusDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\DataServices\PdmDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\DataServices\ProductionOperationDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\DataServices\ProductionOrderDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\DataServices\RawMaterialDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\DataServices\TTNGActionDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\DataServices\WorkplaceDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Entities\AppMerker.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Entities\BasicMaterial.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Entities\CadCamDocument.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Entities\CadCamDocumentType.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Entities\CutSheet.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Entities\MasterWorkPlan.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Entities\Operation.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Entities\Part.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Entities\PartBendSolution.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Entities\PartDisplayStatus.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Entities\PartOnTable.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Entities\PartStatus.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Entities\PartStatusMaster.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Entities\ProductionOperation.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Entities\ProductionOrder.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Entities\RawMaterial.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Entities\TTNGBendToolList.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\Entities\Workplace.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Oseon\PDM.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\DataServices\ImportLogDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\DataServices\ImportResultDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\DataServices\PPSInterfaceDataService.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Exceptions\PropertyNotSpecifiedException.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjectItems\FeedbackObjectItemBase.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjectItems\FeedbackObjectItemDimensions.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjectItems\FeedbackObjectItemPartOnSheet.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjectItems\FeedbackObjectItemSheet.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjects\FeedbackObjectBase.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjects\FeedbackObjectPDAMessage.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjects\FeedbackObjectProcessedSheetReport.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjects\FeedbackObjectProductionOperation.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Export\FeedbackObjects\FeedbackObjectProductionOrder.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Export\PPSExport\PPSExportBase.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Export\PPSExport\PPSExportConsumptionReport.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Export\PPSExport\PPSExportManufacturedSheet.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Export\PPSExport\PPSExportOperation.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Export\PPSExport\PPSExportPDAMessage.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Export\PPSExport\PPSExportPPSExportManufacturedSheetOper.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Export\PPSExport\PPSExportProductionOrder.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Export\ProductionOrderExportHandler.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Export\ProductionQuantityReport.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\Collections\BendSolutionCollection.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\Collections\DeleteProductionOrderCollection.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\Collections\DeleteProductionOrderCollectionXmlHandler.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\Collections\DocumentCollection.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\Collections\FinishProductionOrderCollection.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\Collections\PartCollection.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\Collections\ProductionOrderCollection.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\BendSolution.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\DeleteProductionOrder.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\Document.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\FinishProductionOrder.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\ImportResult.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\Part.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\PPSImport.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\PPSInterface.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\PPSInterfaceDate.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\ProductionOrder.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\WorkingPlan.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\Entities\WorkingStep.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\ProductionOrderImportHandler.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\BendSolutionXmlHandler.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\DeleteProductionOrderXmlHandler.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\DocumentXmlHandler.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\FinishProductionOrderCollectionXmlHandler.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\FinishProductionOrderXmlHandler.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\PartCollectionXmlHandler.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\PartXmlHandler.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\PPSImportXmlHandler.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\ProductionOrderCollectionXmlHandler.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\ProductionOrderXmlHandler.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\WorkingPlanXmlHandler.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Import\XmlHandlers\WorkingStepXmlHandler.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\PPSInterface\Utils\TimeConversionHelper.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Utils\ProductionOrderNumberHelper.vb` | todo | â€” | â€” |
| `SmtProduction\TruTops\Utils\TruTopsConvertHandler.vb` | todo | â€” | â€” |
| `SmtProduction\UI\CtrlTTNGProcessingErrorInfo.Designer.vb` | generated | VS designer partial | â€” |
| `SmtProduction\UI\CtrlTTNGProcessingErrorInfo.resx` | generated | resource bundle (designer-managed) | â€” |
| `SmtProduction\UI\CtrlTTNGProcessingErrorInfo.vb` | todo | â€” | â€” |
| `SmtProduction\UI\OseonAppContextSelectionWrapper.vb` | todo | â€” | â€” |

### ICenterLib / STEP3D

**Total: 5** &nbsp; | &nbsp; .vb: 5 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `STEP3D\AssySplitter.vb` | todo | â€” | â€” |
| `STEP3D\DefinitionAnalyser.vb` | todo | â€” | â€” |
| `STEP3D\Model.vb` | todo | â€” | â€” |
| `STEP3D\Product.vb` | todo | â€” | â€” |
| `STEP3D\Reader.vb` | todo | â€” | â€” |

### ICenterLib / Ticketing

**Total: 5** &nbsp; | &nbsp; .vb: 5 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Ticketing\ApiConnector.vb` | todo | â€” | â€” |
| `Ticketing\Article.vb` | todo | â€” | â€” |
| `Ticketing\Attachment.vb` | todo | â€” | â€” |
| `Ticketing\Ticket.vb` | todo | â€” | â€” |
| `Ticketing\TicketHandler.vb` | todo | â€” | â€” |

### ICenterLib / TimeRegistration

**Total: 2** &nbsp; | &nbsp; .vb: 2 | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `TimeRegistration\MovingEmployeeTimeRegistration.vb` | todo | â€” | â€” |
| `TimeRegistration\WebClock.vb` | todo | â€” | â€” |

### ICenterLib / UserControls

**Total: 34** &nbsp; | &nbsp; .vb: 15 | .cs: 0 | .resx: 8

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `UserControls\DataGridViewFormatter.vb` | todo | â€” | â€” |
| `UserControls\FriendlyComboBox.Designer.vb` | generated | VS designer partial | â€” |
| `UserControls\FriendlyComboBox.resx` | generated | resource bundle (designer-managed) | â€” |
| `UserControls\FriendlyComboBox.vb` | todo | â€” | â€” |
| `UserControls\FrmRemoteDesktop.Designer.vb` | generated | VS designer partial | â€” |
| `UserControls\FrmRemoteDesktop.resx` | generated | resource bundle (designer-managed) | â€” |
| `UserControls\FrmRemoteDesktop.vb` | todo | â€” | â€” |
| `UserControls\FrmUserSelection.Designer.vb` | generated | VS designer partial | â€” |
| `UserControls\FrmUserSelection.resx` | generated | resource bundle (designer-managed) | â€” |
| `UserControls\FrmUserSelection.vb` | todo | â€” | â€” |
| `UserControls\FrmWebView.Designer.vb` | generated | VS designer partial | â€” |
| `UserControls\FrmWebView.resx` | generated | resource bundle (designer-managed) | â€” |
| `UserControls\FrmWebView.vb` | todo | â€” | â€” |
| `UserControls\HintComboBox.Designer.vb` | generated | VS designer partial | â€” |
| `UserControls\HintComboBox.vb` | todo | â€” | â€” |
| `UserControls\ListViewWithReordering.vb` | todo | â€” | â€” |
| `UserControls\ProProgramEditor.Designer.vb` | generated | VS designer partial | â€” |
| `UserControls\ProProgramEditor.vb` | todo | â€” | â€” |
| `UserControls\Ticker.vb` | todo | â€” | â€” |
| `UserControls\UCDossierDetailDesign.Designer.vb` | generated | VS designer partial | â€” |
| `UserControls\UCDossierDetailDesign.resx` | generated | resource bundle (designer-managed) | â€” |
| `UserControls\UCDossierDetailDesign.vb` | todo | â€” | â€” |
| `UserControls\UCFastColoredTextBox.Designer.vb` | generated | VS designer partial | â€” |
| `UserControls\UCFastColoredTextBox.vb` | todo | â€” | â€” |
| `UserControls\UCNoAccess.Designer.vb` | generated | VS designer partial | â€” |
| `UserControls\UCNoAccess.resx` | generated | resource bundle (designer-managed) | â€” |
| `UserControls\UCNoAccess.vb` | todo | â€” | â€” |
| `UserControls\UCTicker.Designer.vb` | generated | VS designer partial | â€” |
| `UserControls\UCTicker.resx` | generated | resource bundle (designer-managed) | â€” |
| `UserControls\UCTicker.vb` | todo | â€” | â€” |
| `UserControls\UCWebView.Designer.vb` | generated | VS designer partial | â€” |
| `UserControls\UCWebView.resx` | generated | resource bundle (designer-managed) | â€” |
| `UserControls\UCWebView.vb` | todo | â€” | â€” |
| `UserControls\XWikiForm.vb` | todo | â€” | â€” |

### ICenterLib / Zabbix

**Total: 1** &nbsp; | &nbsp; .vb:  | .cs: 0 | .resx: 0

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| `Zabbix\ZabbixHandler.vb` | todo | â€” | â€” |


## Roll-up

_Heuristic snapshot updated whenever new projects come into scope or files transition out of `todo`._

### By project

| Project | Total | Done | Todo | Config | Generated | Dead | Needs-review |
|---------|------:|-----:|-----:|-------:|----------:|-----:|-------------:|
| iCENTER | 1237 | 56 | 475 | 358 | 341 | 0 | 7 |
| TruTopsLib | 65 | 0 | 57 | 6 | 2 | 0 | 0 |
| ICenterLib | 723 | 53 | 516 | 81 | 71 | 0 | 0 |
| **TOTAL** | **2025** | **109** | **1048** | **445** | **414** | **0** | **7** |

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
