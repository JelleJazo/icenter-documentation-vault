---
type: module
title: "Modules\\Main.vb — process entry & globals"
status: done
module: "iCENTER/Modules"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Modules\\Main.vb"
last-reviewed: 2026-06-18
tags: [module, entry-point, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `Modules\Main.vb` — process entry & globals

## Purpose

Hosts `Sub Main()` (the actual `<StartupObject>` of `iCenter.exe`) plus the ~80 `Public` fields and constants that make up iCenter's de-facto dependency-injection container. Roughly 266 lines.

## Public surface

| Symbol | Kind | Used by |
|--------|------|---------|
| `Public Module Main` | module | (everywhere; visible by bare name) |
| `Sub Main()` (line 198) | sub | runtime; configured as `<StartupObject>` in `iCenter.vbproj` |
| `Sub Init()` (line 149) | sub | `Sub Main` (line 205) |
| `Public Sub SetOseonDataServices()` (line 171) | sub | `Init`; potentially also when Oseon context changes |
| `Public Function ApplType` (enum) | enum | application-type discriminator |
| ~80 `Public` fields (singletons, caches, constants) | fields | every other `.vb` file |

Detailed inventory of the globals in [[../architecture/global-state]].

## Behavior in plain language

When `iCenter.exe` starts, the runtime constructs the `Main` module (which eagerly initialises all `Public` fields — see "surprises" below) and then calls `Sub Main()`. `Sub Main` calls `Init()` to load CAD/ISAH/Oseon/Elfsquad state, then reads the `-m <mode>` command-line argument to decide which top form to open:

- `-m cadbatchserver` → `CadBatchserver.FrmCadBatchServer`
- `-m updateicenter` → `Functions.UpdateIcenter()` (in `ICenterLib`)
- *(default)* → `FrmMain`

After the chosen form returns, if `FrmMain` flipped one of the post-main globals, a secondary form is opened (`Batchserver.FrmBatchServer` or `Elumatec.FrmComWatcher`). The whole body is wrapped in a `Try…Catch` that logs uncaught exceptions to `oApplicationLog`.

For the interactive path, `Sub Main` also creates a per-PID temp folder (`%TEMP%\iCenter_PID<n>`) and rewrites the process `TMP`/`TEMP` env vars before showing `FrmMain` (lines 235, 255). Any child process spawned from iCenter inherits the redirected `TEMP`.

## Business rules surfaced here

Hard-coded constants at module scope that act as business rules. These deserve dedicated [[../business-rules/_index|business-rule]] notes during Phase 4:

- `SmtMachGrpCodesOverrideGenSetupTime = {"P44","P46","P03"}`
- `KickOffOperations = {"A80","S80","Y80","K80"}`
- `OfficeClockDeptCodes = {"JENG","JWI","JADM","JRD","FENG"}`
- `SalesDeptCodes = {"JVMV"}`
- `JIBA_ADMIN_EMPID = "0798"`
- `UseSbzCalculatedDuration = True`
- `allowSpecialSmtMaterial = False`

See [[../business-rules/_index]] watchlist.

## External systems touched

`Init()` reaches:

- [[../external-systems/icenter-db|iCenter DB]] via `oICENTER.GetGenericNames` / `oICENTER.GetAllGenericMachGrpLines()`
- [[../external-systems/isah|ISAH]] via `RefreshIsahSortOrder` and the eagerly-constructed `oISAH` field
- [[../external-systems/trutops-oseon|TruTops Oseon]] via `OseonAppContextDataService` and the per-context Part/Doc/Status services
- [[../external-systems/elfsquad|Elfsquad]] via `IsahCustomisingElfsquadDataService`
- [[../external-systems/creo|PTC Creo]] via `ICenterLib.CAD.Creo.Environment.DefaultCadAppVersion`

Singletons created at field-init time additionally include `oJIBA`, `oProductDb`, `prodMachines`, `SolaDataConnectorAppHandler`, `TimeReg` (ISAH time reg), and the iCenter2-DB connection placeholder.

## Domain concepts

- [[../domain-concepts/_index|iCenter, iCenter2, MachGrp, Bewerking, KickOff, ProfMill, SBZ, DGX, SMT, Oseon, JIBA]] (and many more — every constant defines a concept).

## Open questions

- **Q-002** — who launches `-m cadbatchserver` in production? See [[../architecture/entry-points]].
- **Q-007** — `sPDFOwnerPassword` is a hard-coded literal here. Replace with secret store? `#safety-relevant`
- **Q-008** — is `iCenter2NewestDataModel` DB still live? `iCenter2DataBaseConnection` declared but commented `As Nothing` at line 124.
- **Q-009** — confirm the constant lists (kick-off ops, lean machine groups, dept codes) still match factory config.

All in [[../needs-review/_index]].

## Surprises worth knowing

1. **Field initializers do real work.** `Public sPDFXChangeEXE As String = SelectPDFXChangeVersion()` (line 30), `Public BendMassFactorTableCached As New DataView(PCFNet.SmtCalculator.BendMassFactorTable)` (line 58), `Public WarningBmp As Byte() = Functions.GetWarningBMP()` (line 98) — all run **before** `Sub Main`. If any of these throw, the process dies before `Sub Main`'s `Try…Catch` is reachable.
2. **`oApplicationLog` is itself one of these eagerly-constructed singletons** (line 63). If logging itself fails to initialise, the inner `Catch ex2 As Exception` in `Sub Main` (line 260) silently swallows the error.
3. The `JIBA_ADMIN_EMPID` constant is the only obvious admin override in the entry code — most other privilege flags (`IamICenterAdmin`, `IamCadManager`, etc.) live on `FrmMain` and are computed during `FrmMain.New`.
4. `Public Const sGSWin32C` and `sGSPrint64` point at GhostScript binaries on a network share — failure to reach the share at print time will cascade through any feature relying on the global, not just GhostScript-spawning code.

## Coverage

Updated in [[../_coverage]] — `Modules\Main.vb` → `done`, `last-reviewed: 2026-06-18`.
