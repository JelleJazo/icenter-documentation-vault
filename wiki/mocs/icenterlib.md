---
type: moc
title: "ICenterLib — Map of Content"
status: draft
tags: [moc, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# ICenterLib

## What this hub covers

The **shared library** at `C:\Users\jelle-r\source\repos\JIBA\iCenter And Tools\ICenterLib\ICenterLib\` — 723 files across 35 top-level folders. Compiled into `iCenter.exe` as a project reference (`{9f3078de-2386-4ff8-b66d-a6df9e100953}`). Hosts the actual data-access, integration, and helper plumbing for everything in iCENTER.

> **Note**: the project-reference path in `iCenter.vbproj` / `iCENTER.sln` (`..\..\ICenterLib\...`) **does not resolve to this location**. See [[../needs-review/_index|Q-018]].

## Top-level layout

| Folder | Files | Role |
|--------|------:|------|
| **(root)** | 15 | `Common`, `AppSettings`, `Connections`, `Log`, `PdfTools`, `Images`, `Main`, `app.config`, etc. **Documented in this batch.** |
| `CAD/` | 127 | PTC Creo + Windchill PLM. Sub-folders: `Creo/`, `PLM/`. `#safety-relevant` |
| `CadBatchServer/` | 17 | Server-side support for `iCenter -m cadbatchserver`. |
| `Comparer/` | 1 | Comparer types. |
| `Connections/` | 1 | Additional connection helpers. |
| `CrystalReport/` | 5 | Crystal-Reports glue. |
| `DataHandler/` | 23 | `GenericQuery`, parameterised SQL helpers, `Selection.RowSelectionMode`, `Toolbox`. |
| `DataServices/` | 10 | Per-domain data-service interfaces. |
| `Debug/` | 1 | Debug helpers. |
| `DistributedLock/` | 1 | Distributed lock primitive. |
| `Elfsquad/` | 9 | Elfsquad configurator API client. |
| `Enums/` | 2 | Application enums (`OrderType`, …). |
| `GUI/` | 3 | Shared GUI helpers. |
| `Helpers/` | 1 | Misc helpers. |
| `iCenter/` | 29 | The **iCenter domain classes**: `ProductionMachines`, `IPPart`, `IPBatch`, `IPOrder`, `Client`, `XmlFile`, `Servicedesk`, `BillOfOper`, `ExternalReferences`, etc. Phase 3c priority. |
| `ISAH/` | 65 | The **ISAH ERP wrapper**: `Part`, `Employee`, `BillOfMat`, `DossierMain`, `DossierDetail`, `ProductionHeader`, `Customer`, `MachGrp`, `User`, `TimeRegistration`, `ShopDoc`, `PurDoc`, … `#safety-relevant`. Phase 3c priority. |
| `JIBA/` | 14 | JIBA portal client. |
| `JMail/` | 6 | Email sending. |
| `LaserWork/` | 4 | Laser-cutting integration. |
| `Metabase/` | 4 | Metabase-style metadata. |
| `ModelDefinition/` | 3 | Model-definition entities. |
| `MySystem/` | 22 | Shared system-level helpers: `Network`, `Computer`, `Printer`, `ExceptionList`, `Zip`. |
| `PCFNet/` | 45 | PCFNet sheet-metal calculator integration. Used by iCENTER's `BendMassFactorTable` etc. |
| `Prodex/` | 6 | Prodex API client. |
| `ProductDb/` | 31 | Product-database wrapper: `CommonDb`, `ProductConfigurationDataService`. |
| `Production/` | 23 | Production-side helpers: `BOMMachGrpFilter`, `LabelLog`, `ProductionProfileCutItemsHandler`, … |
| `Resources/` | 75 | Image/icon assets. |
| `SmartForms/` | 1 | SmartForms helper. |
| `SmtCadCam/` | 4 | Sheet-metal CAD/CAM glue. |
| `SmtProduction/` | 128 | **Trumpf / TruTops Oseon plumbing**: `OseonAppContext`, `PartDataService`, `CadCamDocumentDataService`, `PartStatusDataService`, `ImportSettings*`, `CutSheetOperRegistration`, `WorkplaceEmployeeLink`. `#safety-relevant` |
| `STEP3D/` | 5 | STEP file handling. |
| `Ticketing/` | 5 | Ticketing-system integration. |
| `TimeRegistration/` | 2 | Time-registration helpers (separate from `ISAH/TimeRegistration.vb`). |
| `UserControls/` | 34 | Shared WinForms user controls + `DataGridViewFormatter`. |
| `Zabbix/` | 1 | Zabbix-monitoring integration. |

## Documented so far (this batch — foundation only)

### Root files

| File | Documented in |
|------|---------------|
| [[../modules/icenterlib-common|`Common.vb`]] (19 KB) | static helpers, constants, machine-identity, version |
| [[../modules/icenterlib-appsettings|`AppSettings.vb`]] (12 KB) | reader for the iCenter DB `T_ApplicationSettings` settings table |
| [[../modules/icenterlib-connections|`Connections.vb`]] (9 KB) | 15 database + HTTP connection factories. **`#safety-relevant`** — hard-coded creds throughout |
| `Log.vb` (10 KB) | not yet documented (Phase-3 follow-up) |
| `PdfTools.vb` (23 KB) | not yet documented |
| `Images.vb` (7 KB) | image registry — `oICENTER.AppImages` source |
| `UserSetting.vb` (3 KB) | per-user settings persistence |
| `ComputerSetting.vb` (1 KB) | per-machine settings |
| `Main.vb` (5 lines) | trivial — only creates `AppImages = New ICenterLib.Images` |
| `ICenterLib.vbproj` (76 KB) | project file — not deep-dived |
| `app.config` (5 KB) | ICenterLib-side config — not yet documented |

### Business rules surfaced from the root files

- [[../business-rules/icenter-part-code-prefixes|Part-code prefixes IA / IAK / PRN]] (`Common.IPPARTPREFIX/IPPARTCOPYPREFIX/IPPRNPREFIX`)
- [[../business-rules/icenter-status-code-default-range|Default work-view status range 40-49]] (`Common.WorkViewDefaultFromStatusCode/TillStatusCode`)
- [[../business-rules/icenterlib-maintenance-window|Maintenance window 1-4 AM]] (`AppSettings.IsInsideMaintenanceWindow`)
- [[../business-rules/smt-deburr-speed|SMT deburr speed = 0.225 m²/min]] (`AppSettings.SmtDeburrSpeed`)

## Documentation roadmap (per priority)

1. **ISAH/** (65 files) — every Sales/WorkPreparation/Production note already references `oISAH`. Highest-priority subsystem. Sub-MOC needed.
2. **iCenter/** (29 files) — `ClsICenter`, `IPPart`, `IPBatch`, `IPOrder`, `ProductionMachines` (the COM-watcher's `prodMachines` global). Many references from iCENTER.
3. **SmtProduction/** (128 files) — Oseon plumbing; `Modules\Main.vb` eagerly constructs all four `Smt*DataService`s.
4. **CAD/** (127 files) — Creo + PLM; tied to all of CadBatchserver + ProfMillJob.
5. **PCFNet/** (45 files) — sheet-metal calculator.
6. **ProductDb/** (31 files), **Production/** (23 files), **DataHandler/** (23 files), **MySystem/** (22 files).
7. Smaller specialised folders (JMail, JIBA, Elfsquad, Prodex, Ticketing, Zabbix, LaserWork, STEP3D, etc.) — one note each, light coverage.

## Notable findings from the foundation files

1. **All Connection methods carry hard-coded credentials** (Q-107). Database passwords for iCenter, JIBA, Windchill, ProductDb, Kardex, Isah (including `sa`), TruTops Oseon are all literals in `Connections.vb`. The `GetZeroCodeParameters()` even includes a base64-encoded password as a constant (Q-108).
2. **`Common.GetTableData(TableName)`** executes raw `EXEC('SELECT * FROM ' + @TableName)` — parameterised SP-style, but the `TableName` is interpolated into the SQL string at the server. If `TableName` ever comes from untrusted input it's a SQL-injection. Q-109.
3. **`Connections.SqlServerHostName`** is a `Private Const` switch (`UseSql2 = False` → host `JGSDS`; if `True` → `JZSQL02`). Hostname switching requires recompile.
4. **`Connections.UseIsahTestDb`** is a `Public Shared` field — runtime-mutable. The connection methods consult it on every call. Means: setting it once at startup affects every subsequent ISAH connection in the process. Q-110.
5. **`Common.GetComputername`** has special handling for `JZTS*` (terminal-server), `JZRAS*` (RAS), `JZADS*` (ADS) hostnames — these use the `CLIENTNAME` env var (set by the terminal-server session) to identify the *actual* client machine, not the server. iCenter therefore knows it's running in a TS / Citrix session.
6. **`Common.IsSharedWindowsAccount`** returns `True` if `USERNAME = "PVS"` — only PVS triggers the shared-account branch. The same `PVS` user owns the XWiki credentials in `app.config` (Q-015). One account, many roles.
7. **`AppSettings.GetDecryptedSetting`** reads the encrypted setting AND the `CipherKey1` row in `T_ApplicationSettings` — the encryption key is in the same table as the encrypted data. Q-111.
8. **`AppSettings.IsInsideMaintenanceWindow`** uses `My.Settings` (the `app.config` of the *consuming* assembly, not ICenterLib's) — meaning it actually reads iCENTER's `app.config`. Q-112.
9. **`SmtDeburrSpeed = 0.225 / 60`** (m²/sec) — declared `As Double` so evaluated at compile time. Magic constant in `AppSettings.vb` line 45. Process-relevant. Q-113.
10. **`Common.GetSqlInString`** builds a `'a','b','c'` SQL fragment by string concatenation — used in many places to build `IN (...)` clauses. If any of the values contain a `'`, the resulting SQL is malformed. Q-114.

## Open questions (foundation-level)

- **Q-107 (new):** Hard-coded database credentials throughout `Connections.vb` (including `sa` for ISAH). Move to secret store? `#safety-relevant`
- **Q-108 (new):** `GetZeroCodeParameters()` returns hard-coded API key + username + base64-encoded password. `#safety-relevant`
- **Q-109 (new):** `Common.GetTableData(TableName)` uses `EXEC('SELECT * FROM ' + @TableName)` — confirm `TableName` is never untrusted. `#safety-relevant`
- **Q-110 (new):** `Connections.UseIsahTestDb` is process-mutable but documented "For debug purposes only". Confirm production never flips it accidentally. `#safety-relevant`
- **Q-111 (new):** `AppSettings.GetDecryptedSetting` reads the cipher key from the same DB table as the encrypted value. Threat model?
- **Q-112 (new):** `AppSettings.IsInsideMaintenanceWindow` uses `My.Settings` of the consuming assembly. Confirm iCenter's `app.config` has `MaintenanceWindowStart/End` defined.
- **Q-113 (new):** `SmtDeburrSpeed = 0.225 m²/min` is a hard-coded process constant. SME-confirm and document. `#safety-relevant`
- **Q-114 (new):** `Common.GetSqlInString` does string concatenation; vulnerable to `'`-quote breakage. Audit callers. `#safety-relevant`
- **Q-115 (new):** `Common.IsSharedWindowsAccount` recognises only `"PVS"` as shared. Confirm with SME that no other shared accounts exist.

All logged in [[../needs-review/_index]].

## Related

- [[../architecture/project-references]] — explains how ICenterLib is wired into iCenter.exe
- [[../architecture/external-surface]] — many of the connection methods here are how iCenter reaches the external systems mapped there
