---
type: moc
title: "Business Rules — Index"
status: draft
tags: [moc, business-rule]
created: 2026-06-18
updated: 2026-06-18
---

# Business Rules

Every rule affecting a factory or business process. Surfaced in plain language; located in code by path + symbol; flagged for SME review.

> **Critical:** rules touching physical processes, setpoints, interlocks, or safety MUST carry `#safety-relevant` and `#needs-review` until SME confirms.

## Pre-Phase-4 watchlist

Surfaced during the Phase 2 architecture pass. Each is a candidate for a dedicated [[../_templates/business-rule|business-rule]] note. Several are hard-coded literals in `Modules\Main.vb` or `app.config`; some are threshold values that smell process-relevant.

### From `Modules\Main.vb` (constants that gate behavior)

- `SmtMachGrpCodesOverrideGenSetupTime = {"P44", "P46", "P03"}` — sheet-metal machine groups that bypass the generic setup-time estimate.
- `KickOffOperations = {"A80", "S80", "Y80", "K80"}` — operations treated as "kick-off". `#needs-review`
- `OfficeClockDeptCodes = {"JENG", "JWI", "JADM", "JRD", "FENG"}` — dept-code allowlist for the office-clock feature.
- `SalesDeptCodes = {"JVMV"}` — single sales-dept code.
- `JIBA_ADMIN_EMPID = "0798"` — hard-coded admin employee.
- `UseSbzCalculatedDuration = True` — prefer SBZ-calculated duration over ISAH estimate. `#safety-relevant` (drives scheduling)
- `allowSpecialSmtMaterial = False` — special-material toggle.
- `FrmMain.leanMachGrpCodes = {"A80","F32","F33","P44","S80","EBTV","S17","Y80","P46"}` — machine groups visible in lean-mode.

### From `app.config` (thresholds / setpoints)

- `EluMaxStepDepthSTL = 1.6` — Elumatec maximum step depth (steel). `#safety-relevant` `#needs-review`
- `EluMaxStepDepthALU = 6` — Elumatec maximum step depth (aluminum). `#safety-relevant` `#needs-review`
- `EluLargeRectangleMinLength = 260`, `EluLargeRectangleMinWidth = 20` — large-rectangle classifier thresholds.
- `SmtMaxValueSmallestOutline = 1480`, `SmtMaxValueOutline = 2980` — sheet-metal outline limits. `#safety-relevant` (machine-bed limits?)
- `CoatingPickTimeWarning = 16` — coating-pick warning threshold (hours?). `#needs-review`
- `ProfMillMachGrps = "A48;S48;"`, `SawListMachGrps = "A01;"`, `ShowAdvSawControl = "A01;"` — machine-group classification.
- `FrmGetCoatingNextJob_TimerInterval = 25210` — coating job poll interval (ms).
- `BoostPPSImportTimeout = 1800` — Boost PPS import timeout (s).
- `DosDetailExtraProgCode = 1000000014` — magic program code for "extra detail" dossier rows.
- `FGQCPartCode = "QLTYCNTRL_ALU"` — quality-control part code (FG = finished good?).
- `MachGrpSortingPartCode = "BEWERKINGSVOLG4"` — magic part code that drives machine-group sort order. Dutch token; `BEWERKING` = "processing/operation".
- `IPPartNotReleasedModelname = "TL10193"` — placeholder model name for unreleased parts.
- `alwaysDownloadProductViewFile = "1"`, `AutoCheckActivePdfIsUpToDate = "1"`, `ShowVisualStyles = "1"` — stringly-typed feature flags.

## Pages

_(Each watchlist row becomes a `business-rules/<slug>.md` page during Phase 4. Use the [[../_templates/business-rule|template]] and link back here.)_

### Written so far

- [[elu-max-step-depth]] — `EluMaxStepDepth*` per-material step-depth limits. **`#safety-relevant`**
- [[elu-large-rectangle-classification]] — large-rectangle → broach classifier. **`#safety-relevant`**

### By severity (will be filled by Phase 4 / lint pass)
- `#safety-relevant` — [[elu-max-step-depth]], [[elu-large-rectangle-classification]] _(plus the watchlist above)_
- `#needs-review` — all watchlist rows + the two written rules above
