---
type: module
title: "OutsourceOperationsHandler + FrmOutsourceOperations"
status: done
module: "iCENTER/WorkPreparation"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\WorkPreparation\\OutsourceOperationsHandler.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\WorkPreparation\\FrmOutsourceOperations.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Outsource Operations — pipeline from werkvoorbereiding to purchase order

## Purpose

Implements the **outsourcing pipeline**: select operations that JAZO wants to send to an external vendor, package their STEP+PDF documents into a zip, create an ISAH purchase order via `PurOrd.CreatePurOrdByExtOperParts`, and drop the zip into the matching `IsahDoc\Purchase\<PurOrdNr>\01\` folder so the buyer-side workflow picks it up. Marks the corresponding ShopDoc as "started" and the iCenter parts as "completed" on success.

The big function is `ProcessDataByVendIdProfileId(VendId, ProfileId)` — ~140 lines orchestrating the whole vendor-batch handoff.

## Public surface (handler)

| Symbol | Kind | Note |
|--------|------|------|
| `Property DTData As DataTable` | property | the outsource-candidate rows (mirrored from `IPBatchCollector.GetProdBillOfOper`) |
| `Property DTDocument As DataTable` | property | per-IPpart document list (path + DocumentType enum) |
| `Property OutputFolderPath As String` | property | where to drop the per-vendor folders |
| `Property ZipOutputInd As Boolean` | property | whether to zip the per-`Reference` folder |
| `Property DelDate As Date = Date.Now` | property | required delivery date stamped into the PurOrd and the CSV |
| `Property PurDocFolderpath As String` | property | resolved `IsahDoc\Purchase\<PurOrdNr>\01\` folder |
| `Property PurDoc As ISAH.PurDoc` | property | the created/updated ISAH purchase document |
| `Enum Status { Initial=10, Invalid=1, Valid=2 }` | enum | per-row status |
| `Enum DocumentType { Drawing=1, Model3D=2 }` | enum | per-document tag |
| `New()` | constructor | trivial |
| `Init()` | sub | builds `DTData` from `IPBatchCollector.GetProdBillOfOper("")` and adds 12 columns; constructs `PublishMonitor`; loads statuses |
| `RemoveRow(Value)` | sub | remove row + its documents |
| `CreateFilesByRow(Value)` | sub | trigger STEP creation per row (`ICenterObject.CreateStp` if missing) |
| `GetOverallStatus()` | function | aggregate Status across all rows (`Valid` only if all rows agree) |
| `Refresh()` | sub | re-evaluate every row's status |
| `UpdateByDetailCode(DR, VendId, ProfileId)` | sub | add new rows for an IPBatch's parts, assign vendor + profile |
| `ProcessData()` | sub | orchestrate processing across all vendors |
| `GetMinEndDate()` | function | earliest end-date across rows |
| `ContainsLazyMachGrp() As Boolean` | function | true if any row uses a "lazy" machine-group (codes from `FrmOutsourceOperations.GetLazyMachGrpFilterIn`) |

Private helpers handle the per-vendor / per-profile inner loops, file copying, zip creation, exchange-file writing, ShopDoc / IPpart marking, and pur-doc folder polling.

## The handoff sequence in `ProcessDataByVendIdProfileId`

For `ProfileId = 1` only (everything else throws `NotImplementedException`):

1. Build `VendOutputFolderPath = <OutputFolderPath>\<VendId>` and `mkdir` if missing.
2. Filter `DTData` to rows for this `(VendId, ProfileId)`. Add columns `StepFilename`, `DrawingFilename`, `LabelFilename`, `Reference`.
3. Look up `DeptCode = ISAH.MachGrp.GetDeptCode(MachGrpCode)` from the first row.
4. For each distinct `ProdHeaderDossierCode`: build an `ExtOperPart` (with `RequiredDate = DelDate` and `PredefinedMemoValue = GetMemoValue(...)`). Collect into `lExtOperPart`.
5. **Hard-coded** `extOperPartCode = "UITBESTEDING01"` (line 311). [[../business-rules/outsource-ext-oper-part-code|Documented as a business rule.]]
6. Create or update the PurOrd: `PurDocCode = myPurOrd.CreatePurOrdByExtOperParts(extOperPartCode, lExtOperPart, VendId, AutoSendEmail:=False, False, ExistingPurDocCode, DelDate)`.
7. `PurDoc = New ISAH.PurDoc(PurDocCode)`; read `PurOrdNr`.
8. For each `ExtOperPart`:
   - Build `Reference = PurOrdNr & "_" & PDPartLineNr`.
   - Build `MyOutputFolderPath = <VendOutputFolderPath>\<Reference>` and `mkdir`.
   - **Filter rows on "lazy" MachGrpCodes** (per CB 2022-11-15, switched from `=` to `IN` against `FrmOutsourceOperations.GetLazyMachGrpFilterIn`).
   - For each lazy-MachGrp row:
     - Set `LabelFilename = "<IPPARTPREFIX><IPpartId>.pdf"` (the part-identification sticker).
     - Set `Reference`.
     - For each row in `DTDocument` matching `IPpartId`: copy `.stp`/`.step` → `StepFilename`, `.pdf` → `DrawingFilename`. `File.Copy` to `MyOutputFolderPath`.
     - Build a `SmtManufacturing.JPLT_PartIdentSticker.CreateSticker(IPPart, Qty)`, print it to `MICROSOFTPRINTTOPDF` printer into `LabelFilename`.
   - **Write `<Reference>.csv`** (`WriteExchangeFile`): 8 columns hard-coded — `PartCode;Version;Qty;Material;SmtThickness;StepFilename;DrawingFilename;StickerFilename`. Q-086.
   - If `ZipOutputInd = True`:
     - Build `<Reference>.zip` from the folder.
     - Resolve `PurDocFolderpath = Functions.GetIsahDocPurchaseFolder(PurOrdNr, "01")`.
     - `WaitForPurDocFolder(PurDocFolderpath)` — polls every 500 ms for 20 s, throws on timeout. Q-087.
     - `File.Move(zipFilepath, PurDocFolderpath\…)`.
   - Else: `NotImplementedException` — the non-zip path is not implemented.
   - Delete the `<Reference>` folder if it still exists (best-effort, exceptions swallowed).
9. For each distinct `IbId` (IPBatch) in this vendor batch:
   - Get the `ShopDoc` via `IPBatch.GetShopDoc`.
   - `ShopDoc.SetShopDocStartedInd(True)` (exceptions added to the per-batch exception list, but don't stop processing).
   - For each IPpart row: `IPpart.UpdateCompleted(True)`; `IPpart.UpdateCompletedPartsInBatch(IPBatch, True)`.

If any exceptions accumulated → throw the `MySystem.ExceptionList`.

## Business rules surfaced here

- [[../business-rules/outsource-ext-oper-part-code|`UITBESTEDING01` ext-operation part code]] — hard-coded; the *only* ext-operation part type currently emitted by iCenter. Q-083.
- **Only `ProfileId = 1` is implemented.** All other profiles throw `NotImplementedException`. Q-084.
- **CSV exchange-file schema** (8 columns) is hard-coded. Vendors that need a different layout → silent breakage. Q-086.
- **`WaitForPurDocFolder` 20-second polling timeout.** ISAH must create the purchase-doc folder within 20 seconds of the PurOrd creation. Q-087.
- **`MICROSOFTPRINTTOPDF`** printer is hard-coded as the sticker output. Means each operator workstation must have the "Microsoft Print to PDF" virtual printer installed and named exactly that. `#needs-review`.
- **Lazy MachGrp filter** comes from `FrmOutsourceOperations.GetLazyMachGrpFilterIn` (the form) — outsourcing rules depend on the *UI form's* class. Q-093.
- **Per-batch ShopDoc + IPpart state changes** (`SetShopDocStartedInd(True)`, `UpdateCompleted(True)`) are side-effects that flip ISAH state. If a downstream write fails, the upstream state changes are not rolled back.

## External systems touched

- [[../external-systems/isah|ISAH]] via `ISAH.PurDoc`, `ISAH.MachGrp.GetDeptCode`, `ICenterLib.ICenter.IPBatch`, `IPPart`, `ShopDoc`, `Functions.GetIsahDocPurchaseFolder`.
- [[../external-systems/icenter-db|iCenter DB]] via `ICenterObject.GetIcenterObject`, `ICenterObject.GetFirstStpDoc`, `GetFirstPdfDoc`, `CreateStp`.
- DFS share for IsahDoc and per-vendor output folders.
- [[../external-systems/dymo-label|Dymo / PDF printer]] indirectly via `JPLT_PartIdentSticker.SetPrinterName(MICROSOFTPRINTTOPDF)`.

## Surprises

1. **`PurDocFolderpath` is `Nothing` until the first vendor processed and zipped.** Callers reading it before `ProcessData()` get `Nothing`.
2. **`SetShopDocFinInd` was once called** but is now commented (line 405) with `'CB 2022-10-13: is dit nodig?` ("is this needed?"). Whether marking a ShopDoc as *finished* still happens elsewhere is unknown. Q-094.
3. **`AutoSendEmail = False`** is hard-coded into the `CreatePurOrdByExtOperParts` call (line 310). No vendor receives auto-email from iCenter today.
4. **`ProcessDataByVendIdProfileId` exception aggregation** — exceptions from `SetShopDocStartedInd` are collected but exceptions from `UpdateCompleted` are not. Asymmetric error handling.
5. The whole flow assumes `OutputFolderPath` is set before `ProcessData()` runs. There's no null-check / default.
6. **Two distinct surface-treatment column updates** (`SubstituteInSurfTreatmentPart` vs `SubstituteInSurfTreatmentOper`) in the sibling [[workprep-operation-substitution|OperationSubstitutionHandler]] have **swapped iteration sources**: `*Part` iterates `dtSurfTreatmentOper` and `*Oper` iterates `dtSurfTreatmentPart`. Looks like a bug. Reported under that note as Q-095.

## Open questions

- **Q-083, Q-084, Q-086, Q-087** (from MOC).
- **Q-093 (new):** `FrmOutsourceOperations.GetLazyMachGrpFilterIn` — what determines the list, and is it user-editable or hard-coded inside the form?
- **Q-094 (new):** `SetShopDocFinInd(True)` is commented out (line 405). Is the ShopDoc ever marked finished by iCenter, or has that moved elsewhere?
- **Q-096 (new):** `WriteExchangeFile` writes UTF-8/ANSI? `File.WriteAllText(Filepath, SB.ToString)` uses the default encoding (Windows-1252 on a Dutch box). Vendors expecting UTF-8 → mojibake on special characters.

Logged in [[../needs-review/_index]].

## Coverage

`_coverage.md`:
- `WorkPreparation\OutsourceOperationsHandler.vb` → `done`
- `WorkPreparation\FrmOutsourceOperations.vb` → `done` (referenced; per-method UI not deep-dived)
