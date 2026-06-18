---
type: module
title: "ICenterLib/iCenter ProductionMachines + Client + MP"
status: done
module: "ICenterLib/iCenter"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\ProductionMachines.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\Client.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\ProductionMachineMultiPurpose.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `ProductionMachines.vb` + `Client.vb` + `ProductionMachineMultiPurpose.vb`

## Purpose

The central iCenter-DB wrapper for **production machines** (`T_ProdMachines`), their **multi-purpose siblings** (`T_ProdMachinesMP`), their accumulated **work-time** (`T_ProdMachineWorkTime[Registration]`), and the **Windows-client → machine** binding (`PrefClient`). Where shop-floor reality meets database state.

## ProductionMachines public surface

960 lines, 25+ methods. Themes:

### Listing / lookup

```vb
GetMachinesByGroup(machineGroup As Integer, Optional includeDisabled = False) As DataTable
GetRecord(MachineId As Integer)                                                As DataTable
GetMachineByClientName(ClientName, MachineMustBeEnabled)                       As DataTable
GetMachinesWithActiveBatchId(MachineType, MachineGroup, ActiveBatchId)         As DataTable
Shared GetMachineEmpIds()                                                      As DataTable   ' distinct InitialMachineEmpId
Shared GetMyMachineId()                                                        As Integer     ' resolves via Common.GetComputername → PrefClient
GetMachineInfoByClientName(ClientName, ReturnValue, Optional MachineMustBeEnabled=True) As Object
GetMachineInfo(MachineId, ReturnValue)                                         As Object
GetMachineInfoByName(MachineName, ReturnValue)                                 As Object
GetMachineByVendorWorkPlaceId(WorkPlaceId As Integer)                          As Integer
GetPrefClientsByMachineEmpId(MachineEmpId As String)                           As String
GetMachineByActiveBatch(BatchId, FilterNextOper, ReturnValue)                  As String
GetMPRecord(Id)                                                                As DataTable
Shared GetProdMachinesByEmpId(Employee As ISAH.Employee)                       As DataTable
```

### Per-column typed accessors

```vb
GetIntegerSettingValue(MachineId, ColumnName)                                  As Integer
GetBooleanSettingValue(MachineId, ColumnName)                                  As Boolean
GetStringSettingValue(MachineId, ColumnName)                                   As String
SettingContainsValue(MachineId, Key, Values())                                 As Boolean      ' via AppSettings.StringContainsValue
```

All three execute `SELECT * FROM T_ProdMachines WHERE MachineId=@MachineId` then extract `ReturnField/ColumnName` from the reader. **Untyped string ColumnName → SQL-column lookup** — the dynamic-column anti-pattern (compare Q-170 / Q-180).

### Mutations

```vb
Public Sub UpdateActiveBatchId(MachineId, ICbatchId, FilterNextOper, CreatedOn)        ' clears LatestLineNr too
Public Sub UpdatePrefDocType(MachineName, PrefDocType)
Public Sub UpdateByMPId(Id, MachineId)                                                 ' copies T_ProdMachinesMP row into matching ProdMachine
Public Sub UpdateMPSettings(... 10 columns ...)
Public Sub UpdateLatestLineNr(MachineId, LatestLineNr)
Public Sub UpdateProdMachineWorkTimer(MachineId, TotalTime)                            ' upsert: INSERT or UPDATE T_ProdMachineWorkTime
Public Sub InsertProdMachineWorkTimeRegLine(MachineId, IPPartId, Modelname, GenName, TotalTime)
Public Sub CloseProdMachineWorkTimeRegLine(MachineId)                                  ' SET EndTime=GETDATE() WHERE EndTime IS NULL
Public Function RestoreInitialEmpId(MachineId As Integer) As String                    ' SET MachineEmpId=InitialMachineEmpId
Public Sub SaveMachineEmpId(MachineId, EmpId)
Public Sub DeleteDgxSticker(IbId, LineNr, MachineId)
```

### Misc

```vb
Public Function GetWorkTimePerPart()                                As DataTable        ' joins ProdMachineWorkTimeRegistration with DATEDIFF
Public Function GetDgxStickers()                                    As DataTable        ' SELECT * FROM T_DgxStickers
Public Shared Function GetCurrentTasksMessage(DTCurrentTasks) As String                 ' Dutch UI message builder
```

## Behavior — key patterns

### `GetMyMachineId()`
Resolves the **current Windows machine → MachineId** via `GetMachineInfoByClientName(Common.GetComputername, "MachineId")`. Returns `-1` if no row matches the host computer name. This is **the** way an iCenter client discovers which production machine it represents.

### `RestoreInitialEmpId(MachineId) → String`
Resets `MachineEmpId = InitialMachineEmpId` and returns the restored value. Used when an operator logs off — the machine reverts to its baseline "owner" (typically the team lead or station owner).

### `UpdateProdMachineWorkTimer(MachineId, TotalTime)` — upsert in 4 lines
```sql
IF EXISTS (SELECT 1 FROM T_ProdMachineWorkTime WHERE MachineId=@MachineId)
BEGIN UPDATE ... SET TotalTime=@TotalTime, StartTime=GETDATE() WHERE MachineId=@MachineId END
ELSE BEGIN INSERT INTO T_ProdMachineWorkTime (MachineId, TotalTime) Values (@MachineId, @TotalTime) END
```

When `TotalTime=0`, it switches to `SET TotalTime=0, StartTime=NULL` — the **timer-reset path**. **Side-effect**: resetting clears `StartTime` to NULL so subsequent DATEDIFFs against it return NULL.

### `UpdateMPSettings`
The 10-parameter monster: applies MP-edit to a `T_ProdMachines` row from the MP-config UI. `ICenterOperId = -1` is the **sentinel for "no operation linked"** — converted to `DBNull.Value`. Other null sentinels: `MachineEmpId Is Nothing → DBNull`, `PrefDocType Is Nothing → DBNull`.

### `GetCurrentTasksMessage(DTCurrentTasks)`
Builds the Dutch UI message **`"Je bent al ingeklokt op station:" + bullet-list + "Stop daar de bewerking en probeer het hier opnieuw."`** when a clock-in collision is detected (employee already clocked in elsewhere). Bullet labels prefer `Employee(InitialMachineEmpId).GetEmpFullName` if available, else fall back to `MachineDescription`.

## Client.vb — the Windows-client wrapper

168 lines. Wraps the per-Computername view of `T_ProdMachines`:

```vb
Public Class Client
    Public Const DynamicWorkplacePrefix As String = "WP"

    Public Sub New(Computername As String)
    Public Function SmartUpdateBySession() As Boolean                ' ADS-server check → optional FrmProdMachineSelector
    Public Function GetPrefLabelPrinter() As String
    Public Shared Function GetMachineIdFromName(Value As String) As Integer ' parses "WP123" → 123
    Public Shared Function CreateByMachineId(MachineId As Integer) As Client
    Public Shared Function GetAllLabelPrinters() As DataTable
    Public Shared Function GetRecord(Optional MachineId=-1, Optional PrefClient=Nothing) As DataTable
End Class
```

`DynamicWorkplacePrefix = "WP"` defines a **dynamic-workplace naming convention** for transient/virtual stations: a Windows machine named `WP123` resolves to MachineId 123 (after `GetMachineIdExist` verifies the row exists). Documented as [[../business-rules/icenter-dynamic-workplace-prefix]].

`GetRecord` projects 38 columns — the canonical "give me every property of this machine" SELECT. **Three printer columns** (`PrefPrinter`, `PrefLabelPrinter`, `PrefLabelPrinter2`) get the substring **`@PrefClient`** replaced with the machine's `PrefClient` — a per-client printer-name template (e.g., `\\@PrefClient\labelprinter`).

## ProductionMachineMultiPurpose — the MP-side query

Single method `GetProdMachineMP()` joins `T_ProdMachinesMP + T_Operations` to return:

```sql
SELECT PM.*, O.DeptCode,
  (SELECT TOP 1 MachineName FROM T_ProdMachines PM2 WHERE PM2.MachineEmpId=PM.MachineEmpId) AS HourCode
FROM T_ProdMachinesMP PM
INNER JOIN T_Operations O ON PM.ICenterOperId=O.OperId
WHERE MultiPurposeDisabled=0
```

The correlated subquery looks up the **hour-code via the canonical ProdMachine that owns this MachineEmpId** — assumes 1:1 between `T_ProdMachinesMP.MachineEmpId` and a `T_ProdMachines` row's `MachineName`. Q-225 — if multiple ProdMachines share a MachineEmpId, the `TOP 1` is non-deterministic.

`MultiPurposeDisabled = 0` is the active-filter (same boolean-disable pattern as `T_ProdMachines.MachineDisabled`).

## Surprises

1. **`UpdateMPSettings` doesn't actually update `MachineEmpId`** — Q-226. The SQL has `MachineEmpId=@MachineEmpId` listed, but check carefully: it's `, MachineEmpId=@MachineEmpId` in the UPDATE clause. So it DOES — false alarm; confirmed by re-reading line 698.
2. **`GetMachineByActiveBatch`** uses `FilterNextOper = Nothing OrElse = ""` → `DBNull` sentinel. Inconsistent with `UpdateActiveBatchId` which only checks `= Nothing`.
3. **`SettingContainsValue`** delegates to `AppSettings.StringContainsValue` for comma-separated value parsing — moves the comma-split logic out of the entity. Good split. See [[icenterlib-appsettings]].
4. **`GetMachineEmpIds`** uses `WHERE InitialMachineEmpId IS NOT NULL AND MachineEmpId IS NOT NULL` (both NOT NULL) but only returns `InitialMachineEmpId DISTINCT`. So "an employee with at least one assigned ProdMachine where they were the initial owner AND currently assigned to someone". Subtle.
5. **`Log.NewEntry(ex.ToString, MsgBoxStyle.Critical)` everywhere.** If `Log.NewEntry` actually pops MsgBoxes when style=Critical (likely), this means **any DB exception in a ProductionMachines method blocks with a modal box** — kills headless / batch usage. Q-219.
6. **`InsertProdMachineWorkTimeRegLine` calls `CloseProdMachineWorkTimeRegLine(MachineId)` first** to close any open row, then inserts a new one. **Two separate operations, no transaction** — if the close succeeds but the insert fails, the machine ends up with no open row (orphans the next time-reg). Q-227 `#safety-relevant`.

## Business rules surfaced

- [[../business-rules/icenter-dynamic-workplace-prefix|`WP`-prefix dynamic workplaces]] — `Client.DynamicWorkplacePrefix`.
- **`ICenterOperId = -1` sentinel** = "no operation linked"; converted to DBNull on save.
- **`MultiPurposeDisabled = 0` active-filter** on `T_ProdMachinesMP`.
- **`@PrefClient` printer-name template** — three printer columns get the marker substituted with the machine's `PrefClient`.
- **Reset-timer** (`UpdateProdMachineWorkTimer(MachineId, 0)`) **also nulls `StartTime`**.

## Open questions

- **Q-219 (new):** `Log.NewEntry(msg, MsgBoxStyle.Critical)` — does this actually pop a MsgBox? Headless impact.
- **Q-220 (new):** `Common.GetIsAdsServer(computername)` — what is ADS?
- **Q-225 (new):** `ProductionMachineMultiPurpose.GetProdMachineMP` — correlated subquery uses `TOP 1` without ORDER BY. Non-deterministic if multiple ProdMachines share `MachineEmpId`.
- **Q-227 (new):** `InsertProdMachineWorkTimeRegLine` runs `CloseProdMachineWorkTimeRegLine` then `INSERT` in two separate transactions. Orphan risk if insert fails. `#safety-relevant`

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/icenterlib-icenter]] — parent MOC.
- [[icenterlib-icenter-batch-hierarchy|IPorder → IPbatch → IPpacket → IPpart]] — links via `T_ProdMachines.ActiveICenterBatchId`.
- [[icenterlib-icenter-identification|FrmIdentification]] — uses `GetProdMachineMP` to build the machine-emp picker.
- [[isah-identity|ISAH.Employee]] — `T_ProdMachines.MachineEmpId` / `InitialMachineEmpId` reference `Employee.EmpId`.

## Coverage

- `iCenter\ProductionMachines.vb` → `done`
- `iCenter\Client.vb` → `done`
- `iCenter\ProductionMachineMultiPurpose.vb` → `done`
