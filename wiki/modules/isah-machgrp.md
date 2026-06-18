---
type: module
title: "ISAH MachGrp — machine-group lookups"
status: done
module: "ICenterLib/ISAH"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\MachGrp.vb"
last-reviewed: 2026-06-18
tags: [module, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `ISAH\MachGrp.vb` — machine-group lookups

## Purpose

Wraps ISAH's **`T_MachGrp`** table — the master list of machine groups (the routing codes like `A01`, `A07`, `P44`, `S01`, `EBAN`, …). Provides read-only lookups for description, department code, lead time, and the `IsTrackOperation` classifier.

Referenced from almost every iCENTER module that deals with routings: `Modules\Main.vb` (`ProfMillMachGrps`, `SawListMachGrps`), [[../modules/production-profile-cut-items|`ProductionProfileCutItemsHandler`]], [[../modules/workprep-outsource-operations|`OutsourceOperationsHandler`]] (`MachGrp.GetDeptCode`), and [[../modules/elumatec-machine-base|`Sbz14x.GetMachGrpCode`]].

## Public surface

```vb
Public Class MachGrp
    Public Property MachGrpCode As String
    Public ReadOnly Property TrackOperationRegex As Regex = New Regex("TR\d\d")

    Public Sub New(MachGrpCode As String)

    Public Shared Function GetMachGrpCodes(IncludeObsolete As Boolean) As DataTable
    Public Function GetDescription()       As String
    Public Function GetStandLeadTime()     As Integer
    Public Function GetInfo()              As DataTable
    Public Function GetDeptCode()          As String
    Public Shared Function GetDeptCode(MachGrpCode As String) As String     ' overload — static convenience

    Public ReadOnly Property IsTrackOperation As Boolean
    Public Overrides Function ToString() As String                          ' returns MachGrpCode
End Class
```

## Schema fragment

The `GetInfo` query selects from `T_MachGrp`:

| Column | Type | Notes |
|--------|------|-------|
| `MachGrpCode` | string | primary key |
| `Description` | string | obsolete-marker convention: `~%` prefix |
| `StandMachSetupTime` | numeric | standard setup time |
| `StandOccupationCycleTime` | numeric | standard occupation cycle time |
| `DeptCode` | string | which department owns this machine group |
| `StandLeadTime` | integer | standard lead time |
| (`ExtOperInd`, `MachGrpBarColor`, `VisiblePlanBoardInd` — visible in `GetMachGrpCodes` only) | | |

## Behaviour highlights

### `GetMachGrpCodes(IncludeObsolete)` (Shared)

Returns the full list. The query:

```sql
SELECT TRIM(MachGrpCode) AS MachGrpCode, Description, ExtOperInd, MachGrpBarColor, VisiblePlanBoardInd
FROM T_MachGrp WHERE NOT TRIM(MachGrpCode) = ''
[ AND Description NOT LIKE '~%' ]   -- only when IncludeObsolete = False
```

**The `~` prefix convention** marks an obsolete machine group. The factory keeps the row (operators may still see it in historical data) but the description's first character is `~` so it sorts to the bottom and is filterable. Q-135.

### `GetInfo()` / `GetDescription()` / `GetDeptCode()` / `GetStandLeadTime()`

All thin accessors over `GetInfo()`'s single-row DataTable. **No caching** — each call hits ISAH.

### `IsTrackOperation`

```vb
Public ReadOnly Property TrackOperationRegex As Regex = New Regex("TR\d\d")

Public ReadOnly Property IsTrackOperation As Boolean
    Get
        Return TrackOperationRegex.Match(MachGrpCode).Success
    End Get
End Property
```

A `MachGrpCode` matching `TR\d\d` (e.g. `TR01`, `TR99`) is classified as a **"track operation"**. Phase-3 follow-up: find the callsite to understand what the consequence of being a track-op is.

Documented as [[../business-rules/isah-track-operation-pattern]].

## Surprises

1. **`GetMachGrpCodes` returns `DataTable` even on error** — the inner `Catch` is empty (line 32-34), so an SP failure returns whatever the empty `Dt` is (zero-row DataTable). Callers that test `.Rows.Count > 0` work; ones that don't crash on missing columns.
2. **`MachGrpCode` is a writeable `Property`**, not a read-only one (unlike `Employee.EmpId` which is `ReadOnly Property`). Mutating `MachGrp.MachGrpCode` mid-life doesn't recompute anything since there's no cache — but it's an asymmetry worth noting.
3. **`TrackOperationRegex`** is constructed as a `ReadOnly Property` returning `New Regex(...)` — but each access creates a *new* `Regex` instance because of the `Get` semantics. The `Public Class`-level `TrackOperationRegex` should be a `Public Shared ReadOnly` field for the regex to be compiled once. Q-144.
4. **`GetInfo` selects six columns** but `GetMachGrpCodes` selects five different ones — the two queries return non-overlapping shapes. Two callers wanting "the row" must call the right method.
5. **`Description NOT LIKE '~%'`** is anchored to the *first* char; `LIKE '%~%%'` would catch obsoletes-in-the-middle (none should exist by convention). The convention is fragile to operator typos.

## Business rules surfaced here

- [[../business-rules/isah-track-operation-pattern|`TR\d\d` track-operation pattern]]
- Obsolete-machine-group `~` prefix convention. Q-135.
- Per-machine-group: SetupTime, OccupationCycleTime, LeadTime — these feed scheduling estimates (Phase-4 follow-up for callsites).

## Open questions

- **Q-135** (from MOC): document `~`-prefix obsolete convention.
- **Q-144 (new):** `TrackOperationRegex` recompiles on each access. Promote to `Public Shared ReadOnly Field`?
- **Q-145 (new):** Document downstream consequences of `IsTrackOperation = True`. What does a "track op" trigger?

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/icenterlib-isah]] — parent MOC.
- [[../business-rules/isah-track-operation-pattern]] — companion rule.
- [[../modules/elumatec-machine-base|`Sbz14x.GetMachGrpCode`]] — calls `prodMachines.GetMachineInfoByName(..., "MachGrpCodes")` to find the routing code for an Elumatec machine.
- [[../modules/production-profile-cut-items]] — operations 1/9/31 map to MachGrpCodes.

## Coverage

`_coverage.md`: `ICenterLib\ISAH\MachGrp.vb` → `done`.
