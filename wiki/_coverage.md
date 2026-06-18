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

_One table per top-level sub-folder of `iCENTER\`. Files at the root of `iCENTER\` are listed under `(root)`. The Phase 1 pass populates `status` heuristically: `generated` for `*.Designer.vb` and `*.resx`; `config` for assets, project metadata, and signing keys; `todo` for everything else (mostly `.vb` source). Heuristics get verified during Phase 3._

**Inventory snapshot:** 1237 total files / 707 `.vb` / 169 `.resx` / 354 image assets / 7 misc. Source: `Get-ChildItem -Recurse` on 2026-06-18, excluding `bin`, `obj`, `.vs`, `packages`, `My Project`, `Web References`.

---

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
| `Elumatec\AufSerializer\EluXmlJob.vb` | todo | — | — |
| `Elumatec\AufSerializer\EluXmlJobItem.vb` | todo | — | — |
| `Elumatec\AufSerializer\EluXmlJobSubItem.vb` | todo | — | — |
| `Elumatec\AufSerializer\EluXmlProgram.vb` | todo | — | — |
| `Elumatec\AufSerializer\EluXmlProgramDetail.vb` | todo | — | — |
| `Elumatec\AufSerializer\Kontur.vb` | todo | — | — |
| `Elumatec\AufSerializer\NcProgram.vb` | todo | — | — |
| `Elumatec\AufSerializer\NcProgramAuf.vb` | todo | — | — |
| `Elumatec\AufSerializer\NcProgramEluXml.vb` | todo | — | — |
| `Elumatec\AufSerializer\Programm.vb` | todo | — | — |
| `Elumatec\AufSerializer\ZeileAuftrag.vb` | todo | — | — |
| `Elumatec\AufSerializer\ZeileKontur.vb` | todo | — | — |
| `Elumatec\AufSerializer\ZeileProgramm.vb` | todo | — | — |
| `Elumatec\AufSerializer\ZeileTTab.vb` | todo | — | — |
| `Elumatec\AutoProfMillProgApproval.vb` | todo | — | — |
| `Elumatec\ClsComWatcher.vb` | todo | — | — |
| `Elumatec\ClsDgxShoppingList.vb` | todo | — | — |
| `Elumatec\ClsDgxStickerPrinter.vb` | todo | — | — |
| `Elumatec\ClsEluLanguage.vb` | todo | — | — |
| `Elumatec\ClsSawList.vb` | todo | — | — |
| `Elumatec\ControlProfSaw.Designer.vb` | generated | VS Forms designer partial | — |
| `Elumatec\ControlProfSaw.resx` | generated | resource bundle (designer-managed) | — |
| `Elumatec\ControlProfSaw.vb` | todo | — | — |
| `Elumatec\CRs232.vb` | todo | — | — |
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
| `Elumatec\EluCadApp.vb` | todo | — | — |
| `Elumatec\EluCadSetting.vb` | todo | — | — |
| `Elumatec\FileFormat.vb` | todo | — | — |
| `Elumatec\FrmComWatcher.Designer.vb` | generated | VS Forms designer partial | — |
| `Elumatec\FrmComWatcher.resx` | generated | resource bundle (designer-managed) | — |
| `Elumatec\FrmComWatcher.vb` | todo | — | — |
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
| `Elumatec\Machine\Sbz140Alu.vb` | todo | — | — |
| `Elumatec\Machine\Sbz140Rvs.vb` | todo | — | — |
| `Elumatec\Machine\Sbz140Stl.vb` | todo | — | — |
| `Elumatec\Machine\Sbz141Alu.vb` | todo | — | — |
| `Elumatec\Machine\Sbz14x.vb` | todo | — | — |
| `Elumatec\MacroDatabase.vb` | todo | — | — |
| `Elumatec\NcStructure\Bar.vb` | todo | — | — |
| `Elumatec\NcStructure\Cut.vb` | todo | — | — |
| `Elumatec\NcStructure\EluCadFile.vb` | todo | — | — |
| `Elumatec\NcStructure\Job.vb` | todo | — | — |
| `Elumatec\NcStructure\Plane.vb` | todo | — | — |
| `Elumatec\NcStructure\PlaneCollection.vb` | todo | — | — |
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
| `Elumatec\ProfMillConverter.vb` | todo | — | — |
| `Elumatec\ProfMillJob.vb` | todo | — | — |
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
| `Elumatec\Works\Replacements\AluGeneral.vb` | todo | — | — |
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
| `Elumatec\Works\Replacements\Flowdrill.vb` | todo | — | — |
| `Elumatec\Works\Replacements\IDoorPlankCalibrationMessage.vb` | todo | — | — |
| `Elumatec\Works\Replacements\LargeRectangle.vb` | todo | — | — |
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
| `Elumatec\Works\Replacements\WorksReplacement.vb` | todo | — | — |
| `Elumatec\Works\Replacements\WorksTranslation.vb` | todo | — | — |
| `Elumatec\Works\Sawcut.vb` | todo | — | — |
| `Elumatec\Works\SlottedHole.vb` | todo | — | — |
| `Elumatec\Works\Work.vb` | todo | — | — |

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
## Roll-up

_Heuristic snapshot from the Phase 1 pass. Refined by [[meta/conventions|the lint pass]] as files transition out of `todo`._

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
| Elumatec | 157 | 0 | 123 | 0 | 34 | 0 | 0 |
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
| **TOTAL** | **1237** | **5** | **532** | **358** | **341** | **0** | **1** |
