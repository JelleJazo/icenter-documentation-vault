---
type: architecture
title: "Global state — Module Main"
status: draft
module: "iCENTER/Modules"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Modules\\Main.vb"
last-reviewed: ""
tags: [architecture, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Global state — `Module Main`

## What this view shows

iCenter has no DI container. Process-wide state is held in **`Modules\Main.vb`** as `Public` (mostly mutable) fields on a VB.NET `Module`. Every form, batch job, and class can read and write any of these by bare name. Understanding the dependency graph means understanding this module.

> **Source:** `C:\DevOps\iCenter\iCenter\iCENTER\Modules\Main.vb`, ~266 lines, declares ~80 public symbols at module scope.

## Categories

### 1. External-system clients (singletons, eagerly constructed)

| Symbol | Type | What it talks to |
|--------|------|------------------|
| `oICENTER` | `ClsICenter` | The iCenter database (own DB, separate from ISAH) |
| `oISAH` | `ClsISAH` | ISAH ERP database / API |
| `oJIBA` | `ClsJIBA` | JIBA portal (`jiba.jazo.com`) |
| `oProductDb` | `ProductDb.CommonDb` | Product/configuration DB |
| `SmtPartDataService` | `IPartDataService` | TruTops Oseon (sheet metal) parts |
| `SmtCadCamDocumentDataService` | `ICadCamDocumentDataService` | TruTops Oseon CAD/CAM docs |
| `SmtPartStatusDataService` | `IPartStatusDataService` | TruTops Oseon part status |
| `OseonAppContextDataService` | `OseonAppContextDataService` | TruTops Oseon app context |
| `IsahCustomisingElfsquadDataService` | (ISAH+Elfsquad bridge) | Elfsquad configurator ↔ ISAH |
| `ProductConfigurationDataService` | `ProductDb.ProductConfigurationDataService` | Wraps the above |
| `SolaDataConnectorAppHandler` | `SolaDataConnector.AppHandler` | Sola data connector |
| `TimeReg` | `ISAH.TimeRegistration` | ISAH time registration |
| `iCenter2DataBaseConnection` | `DataMigration.DataBaseConnection` | Connection to "iCenter2NewestDataModel" — **note: this references a *separate* iCenter2 database**, confirming the iCenter2 codebase/data is out-of-scope but the DB is still queried for migration purposes. |

### 2. User / session globals

- `MyMainForm As FrmMain` — the main UI form (singleton).
- `MyMode` (local to `Sub Main`) — not stored, but informs the `BatchServerMode` / `ComWatcherMode` globals.
- `BatchServerMode`, `ComWatcherMode As Boolean` — flipped inside `FrmMain` to request post-main behavior. See [[runtime-modes]].
- `WCUser As ICenterLib.CAD.PLM.User` — current Windchill PLM user.
- `InitialUserEmployee As ISAH.Employee` — the user mapped to an ISAH employee record.
- `iClientMachineGroup`, `iClientMachineType As FrmWorkChange.ClientType` — machine-group/type the client PC is associated with.
- `ProdMachineId`, `ProdMachineIcenterOperId As Integer` — current production machine selection.
- `ProductionMachineMultiPurpose As Boolean` — flag.
- `IAPartDesktopRecorder As DesktopRecorder.DesktopRecorder` — desktop screen recorder (?). **#needs-review** — what gets recorded?

### 3. Hard-coded constants that act as business rules

These deserve dedicated [[../business-rules/_index|business-rule]] notes during Phase 4. Cataloguing here as a hand-off list:

| Symbol | Value | Likely meaning |
|--------|-------|----------------|
| `UseSbzCalculatedDuration` | `True` | Use SBZ (Elumatec) machine-calculated cycle duration rather than ISAH estimate |
| `JIBA_ADMIN_EMPID` | `"0798"` | Hard-coded admin employee ID |
| `SmtMachGrpCodesOverrideGenSetupTime` | `{"P44", "P46", "P03"}` | Sheet-metal machine groups that override generic setup time |
| `KickOffOperations` | `{"A80", "S80", "Y80", "K80"}` | Operations treated as "kick-off" (start-of-routing) |
| `OfficeClockDeptCodes` | `{"JENG", "JWI", "JADM", "JRD", "FENG"}` | Departments where office-clock applies |
| `SalesDeptCodes` | `{"JVMV"}` | Sales department code |
| `allowSpecialSmtMaterial` | `False` | Toggle for special sheet-metal materials |
| `AutoUpdatePartCodeFromIsah` | `"1"` | Stringly-typed boolean; controls auto-update of part codes from ISAH |
| `BoostPartViewerEmpHasAccess` | `False` | Boost PPS part-viewer ACL — default-deny, set later |
| `BoostPartViewerUserHasAccess` | `False` | Same |
| `BoostOverrideStatusEmpHasAccess` | `False` | Boost PPS override-status ACL |
| `OfficeClockDeptCodes`, `SalesDeptCodes` | (above) | |
| `FlowGrillDesignCodePrefixes` | `GetFlowGrillDesignCodePrefixes()` | Computed at module init — prefixes that mark flow-grill products |

Each of these is `#business-rule` and most are also `#safety-relevant` or workflow-gating (kick-off operations, dept codes, machine-group overrides).

### 4. Hard-coded paths and credentials (security observations)

| Symbol | Value | Notes |
|--------|-------|-------|
| `sPDF_R_Root` | `\\jazo.local\dfs\TEKDB\TEKENINGENDB\PDF_PROD_DIR\` | DFS share root for production-PDF drawings |
| `sFDF_Root` | `\\jazo.local\dfs\TEKDB\TEKENINGENDB\FDF_DIR\` | FDF annotations |
| `sDXF_Root` | `\\jazo.local\dfs\TEKDB\TEKENINGENDB\DXF_DIR\` | DXF root |
| `sXML_Root_SeImporter` | `C:\temp\SolidEdgeImporter\XML_DIR\` | Local temp; not server share |
| `sPDFXChange` | `C:\Program Files\Tracker Software\PDF Viewer\PDFXCview.exe` | Hard-coded executable path |
| `sGSWin32C` / `sGSPrint64` | `\\jazo.local\dfs\applications\iCenter\Resources\GhostScript\...` | GhostScript shipped from DFS |
| `PdfStampDir` | `\\jazo.local\dfs\applications\iCenter\Resources\Stamps\` | PDF stamp directory |
| `sMijnPDFdir`, `sMijnDXFdir` | `M:\Mijn PDFs\`, `M:\Mijn DXFs\` | Per-user mapped drive |
| **`sPDFOwnerPassword`** | hard-coded literal | **Secret in source.** `#needs-review` |
| `PurchaseFromEmailAddress` | `inkoop@jazo.com` | Sender for outbound purchase emails |

### 5. Caches and UI handles

- `ICenterObjects As New Dictionary(Of String, ICenterObject)` — generic per-process object cache.
- `sTekeningen As ArrayList` — cached drawing list (legacy `ArrayList`).
- `AppImages As New ICenterLib.Images` — central icon registry.
- `WarningBmp / ErrorBmp / StopBmp / LengthBmp / GeoErrorBmp / GeoOpenContourBmp / ProfMillErrorBmp / EmptyOperBmp / NoOseonPartBmp As Byte()` — pre-loaded inline image bytes.
- `oFontDefault`, `oFontItalic As Font` — process-wide GDI+ font handles.
- `BendMassFactorTableCached As New DataView(PCFNet.SmtCalculator.BendMassFactorTable)` — sheet-metal bend-mass factor view.

## Why this matters

- Any module note that says "we pass `oISAH` to method X" is **wrong** — it's not passed, it's referenced by bare name from the global module. Renaming `oISAH` would break every file in the project.
- Singletons are constructed at field-init time (before `Sub Main` runs). If a constructor throws (e.g., DB unreachable, ISAH down), the process dies before `Main` can log it — see [[entry-points]] note about the absent application-level exception handler.
- The `iCenter2DataBaseConnection` reference is the **only** code link to the out-of-scope `iCenter2` codebase: a separate database connection labelled `"iCenter2NewestDataModel"`. `#needs-review` whether this DB is still live or vestigial.

## Open questions

- **Q-007:** Replace `sPDFOwnerPassword` literal with a secret store? Confirm it actually controls PDF protection and is not vestigial. `#safety-relevant` to documents legally signed.
- **Q-008:** Confirm `iCenter2NewestDataModel` DB is still hit at runtime. If yes, document its schema scope.
- **Q-009:** Confirm the hard-coded constant lists (kick-off ops, lean-mode machine groups, sales dept codes) match current factory configuration. These are silent business rules.

## Related

- [[_index]]
- [[entry-points]]
- [[../business-rules/_index|business-rules/]] (Phase 4 will spawn one note per constant)
- [[../needs-review/_index]]
