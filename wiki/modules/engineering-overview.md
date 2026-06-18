---
type: module
title: "Engineering/ — overview of forms and ModelCopies"
status: draft
module: "iCENTER/Engineering"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Engineering\\FrmDrwCheck.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Engineering\\FrmOrdersAsBuilt.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Engineering\\frmEngGeneratedProductOverview.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Engineering\\frmSelectAnnotGenericStatus.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Engineering\\frmGenericStatus.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Engineering\\FrmDesignCodeTool.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Engineering\\ModelCopies\\CopyLocalizer.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Engineering\\ModelCopies\\GenericSmtPartFinder.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Engineering\\ModelCopies\\UitsparingVoorplaatMeerpslAlu.vb"
last-reviewed: 2026-06-18
tags: [module, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `Engineering/` — forms + ModelCopies overview

## Purpose

The Engineering folder hosts **engineering-side UI forms** plus a small `ModelCopies/` subsystem for finding parts that match certain patterns. None of the UI forms are dialogs spawned by other code; they're top-level windows opened from FrmMain's menu. Two cross-cutting helpers (`FrmDrwCheck` and `FrmOrdersAsBuilt`) are central to engineering workflow.

## File-by-file

### `FrmDrwCheck.vb` (31 KB) — drawing-check / as-built / generic-check viewer

Three modes via `Enum ShowMode { DrwCheck, AsBuilt, GenericCheck }`. Constructor takes a mode and loads `oICENTER.GetCADinStock(False)` into a `dtModelsInStock` field.

Embeds a **PDF-XChange ActiveX viewer** (`AxPDFXCviewAxLib.AxCoPDFXCview`) into `SplitContainer1.Panel2`. Configures it on load:
- `SetDevInfo(PDFXLicensekey, PDFXLicensecode)` — activate the license to suppress watermark.
- `SetProperty("JavaScript.Enable", "false")` — disable JS (defensive).
- `SetProperty("International.LocaleID", 1043)` — Dutch locale.
- `SetProperty("Identity.Name", MyMainForm.lblUserFullName.Text, 0)` — identity for PDF metadata.

Keyboard navigation: arrow keys page through the grid. Uses `MyMainForm.OpenPdfAsStreamInPdfXChange(viewer, filepath)` to load each PDF.

**`#needs-review`**: the PDF-XChange license key + code come from `My.Settings.Properties("PDFXLicensekey").DefaultValue` (and `code`) — embedded in `app.config` as plaintext (also flagged as Q-016 earlier).

### `FrmOrdersAsBuilt.vb` (13 KB) — assign engineer to order

Lists engineering employees from ISAH (`oISAH.GetAvailableEmpsPlanDossierDetail(DeptCodeFilter, CompanyCode)`) filtered by `PlanDosDetailDeptCodes` setting. Allows assigning an engineer to a dossier-detail. First entry is hard-coded `"0000 Niemand"` ("nobody") as a "no assignment" sentinel.

Uses `ImageList1` with four icons (`NEW`, `ALL`, `MALE`, `OBSOLETE`) — the OBSOLETE icon is a composite (Person + Stop sign) for retired employees.

References `ClsActiveDirectory` (commented-out paths suggest an earlier version pulled engineers from AD groups; now reads from ISAH instead).

### `frmEngGeneratedProductOverview.vb` (8 KB) — Excel-driven product catalogue

Reads `My.Settings.Properties("GeneratedProductFilepath").DefaultValue` (= `\\jazo.local\dfs\DATA\Engineering\3D Pui Generator\Overzicht gemaakte puien.xlsm` per `app.config`). Imports the Excel via `ImportExcelToDatatable` into a `DataGridViewAutoFilter`-bound grid. Title bar shows the source path.

`#needs-review` — coupling to a specific filename on the shared drive. Renaming or moving the Excel breaks this view silently.

### `frmGenericStatus.vb` (3 KB) — generic-PDF status tree

Walks `MyMainForm.TreeView.Nodes` recursively, picks checked nodes whose ICenterObject has `DocumentType = MBOM_OBJECT` and a non-empty `GenName`, and builds a parallel TreeView showing **all models that share the same `GenName`** (via `oPDFCOMMENTLINES.GetModelnamesByGeneric(GenName)`). Lets engineers compare same-generic parts across orders.

Has an `oPDFXViewer` field but the viewer-activation line is commented out.

### `frmSelectAnnotGenericStatus.vb` (4 KB) — annotation-status selector

Phase-3 follow-up — not deep-read in this batch. Likely a selector dialog before `frmGenericStatus`.

### `FrmDesignCodeTool.vb` (3 KB) — request new drawing numbers

Embedded **WebView2** pointing at `https://tekeningnummers.jazo.com` (hard-coded `WebsiteUrl` const). On load:
- Injects the order number into `document.getElementById("CphMainContent_TxtSearchValue").value`.
- Clicks the search button via `document.getElementsByName("ctl00$CphMainContent$BtnSearch")[0].click()`.

On close, retrieves whatever value the user left in the textbox via `ExecuteScript("...value;")` — exposed via `GetUserInputOrdNr` property.

**Brittle to any HTML change on tekeningnummers.jazo.com** (button name, textbox id). Q-090.

The form also branches the tip text on `OrdNr.Substring(2, 1) = "0"`: position 2 of the order number is `"0"` → it's a quote number ("offertenummer"); otherwise a regular order ("ordernummer"). Q-104 — confirm this convention with SME.

### `ModelCopies/` — find-copies jobs

A small abstract-factory pattern for "search the model file server for parts matching some criteria":

- **`CopyLocalizer`** (abstract base) — exposes `MustOverride search()`, `Description`, `Name`. Helpers: `PartCount`, `ErrorCount`, `GetResultAsSearchString` (`;`-joined model names). Static `GetAvailableSearches()` returns the registered jobs (currently only `GenericSmtPartFinder`; `UitsparingVoorplaatMeerpslAlu` is **commented out** with note _"Orders die niet in iCenter worden hiermee overgeslagen, dus niet toepassen"_ — Q-089). Default click-handler asks for confirmation, runs the search, then offers to drop the result into `MyMainForm.Search(...)`.
- **`GenericSmtPartFinder`** — iterates `Directory.GetDirectories(XML_Root_SERVER, "gn*")` for every `gn*.xml` file; resolves the ICenterObject; if `SmtThickness > 0`, adds to results. Temporarily flips `BatchServerMode = True` during the search (Q-088).
- **`UitsparingVoorplaatMeerpslAlu`** — same shape but iterates `"ad*"` directories with regex `AD\d{5}-P[0-9]{1,3}-D[1-2]-01-90[16].xml\z`, matches models whose `GenName ∈ {"GN10188-P1-D1-01-901", "GN10188-P1-D1-01-906"}`. Currently *registered-out* (Q-089) — `#needs-review` `#dead-code`.

The `BatchServerMode = True` flip during search likely **suppresses side effects** (e.g. UI prompts, error message boxes) that would otherwise fire per file. Q-088 connection: this is one of the unknown places `BatchServerMode` gets manipulated.

## Open questions

- **Q-088** (from prior batches): why is `BatchServerMode` temporarily set to `True` during ModelCopies searches?
- **Q-089** (from prior batches): is `UitsparingVoorplaatMeerpslAlu` truly dead code?
- **Q-090** (from MOC): `FrmDesignCodeTool` brittle JavaScript injection — document the contract with tekeningnummers.jazo.com.
- **Q-104 (new):** order-number convention — position 2 = `"0"` means quote, otherwise order. Confirm with SME. `#needs-review`
- **Q-105 (new):** `frmGenericStatus`'s commented-out PDF-XChange viewer activation — was it removed deliberately or did someone forget to re-enable? `#needs-review`

Logged in [[../needs-review/_index]].

## Coverage

`_coverage.md`:
- `Engineering\FrmDrwCheck.vb` → `done` (overview level)
- `Engineering\FrmOrdersAsBuilt.vb` → `done` (overview level)
- `Engineering\frmEngGeneratedProductOverview.vb` → `done`
- `Engineering\frmSelectAnnotGenericStatus.vb` → `needs-review` (deferred)
- `Engineering\frmGenericStatus.vb` → `done`
- `Engineering\FrmDesignCodeTool.vb` → `done`
- `Engineering\ModelCopies\CopyLocalizer.vb` → `done`
- `Engineering\ModelCopies\GenericSmtPartFinder.vb` → `done`
- `Engineering\ModelCopies\UitsparingVoorplaatMeerpslAlu.vb` → `done` (`#dead-code` tag)
