---
type: module
title: "ISAH TimeRegistration + TimeRegCollector"
status: done
module: "ICenterLib/ISAH"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\TimeRegistration.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\TimeRegCollector.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# ISAH `TimeRegistration` + `TimeRegCollector`

## Purpose

Two classes that together implement **JAZO's shop-floor time clocking** — the bridge between operators' actions (clock in / change to shop-doc / clock out) and ISAH's `T_TimeRegistration` table.

- **`TimeRegistration`** (1274 lines — **biggest single ISAH file**) wraps the SP-driven time-reg writers (`IP_ins_TimeRegistr`, `IP_Upd_TimeRegistr_v2`, `IP_Ins_PBOOEmployee`) plus all the surrounding business logic: DTR status conversion, assistant cascading, "moving employee" handoff, combined time-reg generation for sheet-metal (Trumpf/Oseon) jobs.
- **`TimeRegCollector`** (147 lines) is the **batch helper** — operators or background jobs add ShopDocs to "started" or "finished" lists plus per-line time-reg rows to a buffer DataTable, then `ProcessCollections()` flushes the lot to ISAH with 1-second pacing between writes.

This is the **most-modified file in the ISAH wrapper** if you weight by per-line-density of `'CB <date>:` comments — multi-year reactive maintenance against shop-floor incidents.

## `TimeRegistration` — top-level surface

```vb
Public Class TimeRegistration
    Public Const AutoTimeRegHourCode As String = "01"     ' cycle-time HourCode

    Enum TypeOfDtrCloseCode
        WorkCode      = 0
        PresenceCode  = 1
        NextDay       = 2
        ReturnSameDay = 3
    End Enum

    ' 19 public + many private methods (Phase-3 deferred to enumerate fully)
End Class
```

Visible public surface (subset):

| Symbol | Role |
|--------|------|
| `Sub ChangeToShopDoc(EmpId, ShopDocCode, HourCode, UseEmpMachGrpCode, AssistantMachGrpCode, EnforceProvidedShopDocCode, MovingEmpTimeReg)` | **The headline mutator** — switch an employee onto a different ShopDoc. Cascades to assistants, closes existing line, inserts a new one. |
| `Function GetExtraShopDocInfo(ShopDocCode)` | look up `(ProdHeaderDossierCode, ProdBOOLineNr, MachGrpCode)` from a ShopDoc |
| `Function GetShopDocsByClusterCode(ClusterCode)` | for cluster-based time-reg (multiple shop-docs grouped into one cluster) |
| `Sub DeletePBOOEByProdBOOLineNr(...)` / `DeletePBOOE(...)` / `GetPBOOE(...)` | PBOO-employee linkage (`T_ProdBOOEmployee`) |
| `Sub GetPbooDetails(ShopDocCode, ByRef ProdHeaderDossierCode, ByRef ProdBoolineNr)` | out-parameter lookup |
| `Sub IP_Upd_TimeRegistr_v2(...)` (×2 overloads) | direct SP wrapper for updating an existing time-reg line |
| `Function InsertAutoTimeRegistr(...)` | insert + immediately update — "auto" time-reg lines with both start and end set |
| `Sub CloseTimeRegLine(EmpId, RegDate, Optional CloseAssistantsTimeRegLine=True)` | close the currently-open work line for an employee |
| `Function GetCurrentTimeInSeconds()` | **`Hour*3600 + Minute*60`** — minute-precision; *Second is intentionally dropped* |
| `Function GetTimeRegExists(ShopDocCode, ...)` | existence check |
| `Sub InsertTimeRegLineByProdHeaderDossierCodeAndMachGrpCode(...)` | direct line insert keyed by PH + MachGrp |
| `Function GetAvailableEmpIdByMachGrpCode(MachGrpCode, CheckDate)` | reverse lookup — who's available on this machine right now |
| `Sub IP_Ins_PBOOEmployee(ProdHeaderDossierCode, ProdBooLineNr, EmpId)` | bind an employee to a PBOO line |
| `Function CreateCombinedTimeRegLines(EmpId, dtSheetPart, StartDate, ShowGui)` | **sheet-metal/Oseon → ISAH bridge** — generate setup+cycle pairs from a TruTops sheet |
| `Function GetEmployeeShopDocsFromTimeRegistration(EmpId)` | which shop-docs has this employee been on |

### `ChangeToShopDoc` — the orchestrator

The most important method in the file. Roughly:

1. Delete any existing PBOO-employee binding for this `EmpId`.
2. **Refuse to act** if the employee isn't physically present (`Not E.IsMachineEmpId AndAlso Not E.GetIsPresent`). Logs Info and returns. Q-182.
3. Resolve `(ProdHeaderDossierCode, MachGrpCode, ProdBOOLineNr)` from the supplied ShopDoc.
4. Optionally substitute the employee's own MachGrp (`UseEmpMachGrpCode = True`) or an assistant-provided one.
5. If not `EnforceProvidedShopDocCode`: re-resolve the ShopDoc by `(ProdHeaderDossierCode, MachGrpCode)` — meaning the original ShopDoc the caller passed can be **silently replaced** with a different one that matches the employee's MachGrp. Logs Info on substitution.
6. **Recursively cascade to assistants** (`ChangeAssistantsToShopDoc` → loops back through `ChangeToShopDoc` with `AssistantMachGrpCode <> Nothing` to break the recursion).
7. `CloseTimeRegLine(EmpId, today)` — close the current line.
8. If "moving employee" mode is on AND the previous `LastUsedWorkChangeShopDocCode` is different from the new one: `DeleteZeroDurationTimeRegLines` — clean up 0-minute "work change" rows.
9. `IP_Ins_TimeRegistr(...)` insert.
10. `IP_Ins_PBOOEmployee(...)` bind.

**Critical surprises:**

- The whole body is wrapped in a single outer `Try…Catch ex As Exception` that logs `Critical` and returns. **No exceptions escape**. If step 8 silently fails, the resulting time-reg may be inconsistent. Q-183.
- **Step 5 silently substitutes the ShopDoc**. The caller passed `ShopDocCode = X`; if `(ProdHeaderDossierCode, EmpMachGrpCode)` resolves to `ShopDocCode = Y`, the function uses Y. Logs Info but the caller has no callback. Q-184.
- **Assistant cascade is recursive** with the `AssistantMachGrpCode <> Nothing` trick to break it — inline comment _"AssistantMachGrpCode misbruiken om uit de recursive loop te springen"_ ("abuse AssistantMachGrpCode to escape the recursive loop"). Fragile.

### Time precision: minute-granularity by design

```vb
Public Function GetCurrentTimeInSeconds() As Integer
    Dim returnValue As Integer = 0
    returnValue = (DateTime.Now.Hour * 3600) + (DateTime.Now.Minute * 60)
    Return returnValue
    'Return DateTime.Now.TimeOfDay.TotalSeconds      ' commented-out: real-time variant
End Function
```

**`Second` is intentionally dropped.** Every time-reg line is anchored to a whole-minute boundary. The commented-out `TimeOfDay.TotalSeconds` would have given sub-minute precision; the developer chose the truncated version. Implication: two consecutive clock-actions within the same minute land on the same `StartTime` — concurrent operators on the same machine can produce ambiguous lines. Documented as [[../business-rules/isah-timereg-minute-granularity]].

### DTR status codes — a 2-char state machine

`DTRStatusCode` is a 2-character ISAH code. Inferred conventions from the `ConvertDtrStatusCode` table (lines 695–727):

| Code | First char | Second char | Meaning (inferred) |
|------|-----------|-------------|---------------------|
| `AO` | A = aanwezig (present) | O = open | clocked in, presence open |
| `AW` | A = aanwezig | W = wachten (closed/wait) | presence closed |
| `IO` | I = indirect | O = open | indirect work, open |
| `IW` | I = indirect | W = closed | indirect work closed |
| `II` | I = indirect | I = interrupted | indirect, day-rollover |
| `OO` | O = order | O = open | order work, open |
| `OW` | O = order | W = closed | order work closed |
| `OI` | O = order | I = interrupted | order, day-rollover |

`ConvertDtrStatusCode(code, type)` defines the transitions:

- **WorkCode close**: `IO → IW`, `OO → OW`
- **PresenceCode close**: `AO → AW`
- **NextDay carry**: `IO → II`, `OO → OI`
- **ReturnSameDay reopen**: `II → IO`, `OI → OO`

Documented as [[../business-rules/isah-dtr-status-codes]].

**Important context**: `Connections.UseIsahNoDtrTimeReg = True` (current production) means the entire DTR-status branch is bypassed — the SP calls skip `@DTRStatusCode`, `@DTRInfoCode`, `@DTRExportInd`. The conversion table is **dead code in production today** (cf. Q-132). The conversions still apply to the test-DB path where `UseIsahNoDtrTimeReg = False`.

### HourCodes

iCenter uses three hardcoded hour-codes when writing time-reg lines:

| HourCode | Meaning |
|----------|---------|
| `"01"` | normal cycle time (`AutoTimeRegHourCode` constant). Also the canonical hour-code for `CreateCombinedTimeRegLines`'s machine-cycle rows. |
| `"02"` | setup time. Used in `CreateCombinedTimeRegLines` for the `MachSetupTime` rows. |
| `"AW"` | "Aanwezig" / presence. Filtered on in `Employee.GetIsPresent` / `GetWorkingHourCode`. |

Documented as [[../business-rules/isah-hourcodes]].

### `IP_Ins_TimeRegistr` — the SP wrapper

Inserts to `T_TimeRegistration` via SP `IP_ins_TimeRegistr`. ~25 parameters:

- Identity / routing: `@EmpId`, `@HourCode`, `@MachCode` (always `""`), `@MachGrpCode`, `@MalfunctionCode` (always `""`), `@ProdHeaderDossiercode`, `@ProdBOOLineNr`, `@ShopDocCode`, `@WorkingHourCode`, `@CostCenterCode` (always `""`), `@ServObjectCode` (always `""`), `@ClusterCode`.
- Timing: `@RegDate`, `@StartDate`, `@StartTime`, `@EndTime`, `@EndDate`.
- DTR (only if `UseIsahNoDtrTimeReg = False`): `@DTRInfoCode = "#"`, `@DTRExportInd = 1`, `@DTRStatusCode`.
- Admin: `@IsahUserCode = Common.APPLISAHUSERCODE` (**= `"ICENTER"` — correct convention**, unlike the production/dossier classes which hardcoded `"ISAH"` — Q-165). Q-185: harmonise.
- `@ProcessInd`, `@ProducedQty = 0`, `@TimeRegInputType = 1` _(comment: `RR 20220725 Fix voor ontbrekende JournaalPosten? Waarde was 2`)_.

The SP returns the new `TimeRegLineNr` via an output parameter (`@New_TimeRegLineNr`).

### `CreateCombinedTimeRegLines` — Oseon → ISAH bridge

Takes a sheet-part DataTable (from TruTops Oseon — see [[../external-systems/trutops-oseon]]) and registers consolidated setup + cycle time-reg lines for an employee. Notable steps:

1. **Hardcoded debug-MachGrp substitution**: `If MachGrpCode = "T40" Then MachGrpCode = "P02"` — the `T40` machine group is reserved for the debugger employee; rewrite to a real `P02` so the rest works. Q-180.
2. EmpIds starting with `"S"` → `DeleteNextDayTimeRegLines` (`S` prefix has special-case handling; Q-186 — what does `S` mean?).
3. Force `StartDate` to `00:01` of the day to "soms worden er geen urenregels onder werktijd weggeschreven waardoor ik na 23:59u nog uren zou moeten wegschrijven" — sometimes no rows write under work-time so a post-23:59 write fails. Working around this by starting at 00:01 then advancing.
4. Look up existing overlapping rows via `GetExistingTimeRegLines(EmpId, regDate, startTime, endTime)`. If overlap found: shift the StartTime forward to `Ceiling(max(EndTime, CalculatedEndTime) / 60) * 60` (next whole minute). Comment: _"Oh jee, overlappende urenregels, dan maar achter de laatste plakken"_ ("Oh dear, overlapping time-reg lines — just paste after the last one").
5. For each unique ShopDocCode: get matching PBOO row (try by `(ShopDoc, MachGrp)` then fall back to `ShopDoc`), then **split MachSetupTime into HourCode `"02"` row and MachCycleTime into HourCode `"01"` row**.
6. Skip the setup row if a matching `"02"` row already exists (`GetTimeRegLinesByShopDocCodeAndMachGrpCode` returns non-empty).
7. Optional GUI mode shows the table in a `FrmDataGridView` and asks "Urenregels inschieten?" before persisting.

The minute-granularity overlap-resolution is a workaround for the integer-precision design choice. `#safety-relevant` — time-reg is the basis for payroll and cycle-time costing.

## `TimeRegCollector` — batch helper

Three private buffers:

```vb
Private SetStartedList   As New ShopDocCollection
Private SetFinishedList  As New ShopDocCollection
Private TimeRegTable     As DataTable    ' built by CreateTimeRegTable
Public  TempDelayForSql  As Integer = 1000
Private ReadOnly _CutSheetOperRegistrationDataService As CutSheetOperRegistrationDataService
```

### `TimeRegTable` schema

Computed columns:

| Column | Type | Definition |
|--------|------|------------|
| `Qty` | Double | (input) |
| `EmpId` | String | (input) |
| `TimeStamp` | DateTime | (input) |
| `ShopDocCode` | String | (input) |
| `MachCycleTime` | Double | (input) |
| `MachSetupTime` | Double | (input) |
| `MachTimeTotal` | Double | **expression** `Qty * MachCycleTime + MachSetupTime` |
| `MachCycleTimeTotal` | Double | **expression** `Qty * MachCycleTime` |
| `OperationNo` | String | (input) |
| `SheetId` | Int32 | (input) |
| `ProdOrdNo` | String | (input) |

The two expression columns autoupdate when input columns change — convenient but means **adding a row doesn't fire a write to ISAH** until `ProcessCollections()` runs.

### Behaviour

| Method | Role |
|--------|------|
| `AddStarted(ShopDoc/ShopDocCode)` | append to `SetStartedList` |
| `AddFinished(ShopDoc/ShopDocCode)` | append to `SetFinishedList` |
| `AddTimeReg(Qty, EmpId, ts, ShopDocCode, cycle, setup, OperationNo, SheetId, ProdOrdNo)` | append to `TimeRegTable` |
| `ProcessCollections()` | flush all three buffers in order: SetStarted → SetFinished → InsertTimeRegLines. Aggregates exceptions into a single `MySystem.ExceptionList`. |

`ProcessCollections()` important details:

1. `SetStarted` calls `SD.SetShopDocStartedInd(True)` for each ShopDoc, then sleeps `TempDelayForSql = 1000` ms between writes. **1-second pacing per write** ([[../business-rules/isah-timereg-write-pacing]] — Q-181).
2. `SetFinished` calls `SD.SetShopDocFinInd(True)` for each — **the same method that's commented-out in `OutsourceOperationsHandler`** (Q-094). This is the live path that *does* finish ShopDocs.
3. `InsertTimeRegLines` groups by EmpId then calls `TimeReg.CreateCombinedTimeRegLines(...)` per employee. On success, calls `_CutSheetOperRegistrationDataService.UpdateTimeRegProcessed(SheetId, OperationNo, True)` to mark the Oseon side as processed.
4. Throws the aggregate exception list at end if non-empty.

The 1-second pacing comment-free in source. Plausible motivation: **avoiding overlap-resolution churn** in `CreateCombinedTimeRegLines` (because `GetCurrentTimeInSeconds` rounds to whole minutes, back-to-back inserts within the same second land on the same minute and re-trigger the overlap logic). With 1s sleep, the inserts spread across timestamps. Q-181 to confirm.

## Business rules surfaced here

- [[../business-rules/isah-dtr-status-codes|DTR status-code 2-char state machine]] — `AO/AW/IO/IW/II/OO/OW/OI` with the 4-type close conversion table. Currently dead in production (`UseIsahNoDtrTimeReg = True`) but documented for the test-DB path.
- [[../business-rules/isah-hourcodes|Time-reg HourCodes]] — `"01"` cycle, `"02"` setup, `"AW"` presence. The `AutoTimeRegHourCode = "01"` constant on `TimeRegistration` is the canonical reference.
- [[../business-rules/isah-timereg-minute-granularity|Minute-granularity time-reg]] — `GetCurrentTimeInSeconds` rounds to whole minutes by design. Drives the overlap-resolution workaround in `CreateCombinedTimeRegLines`.
- [[../business-rules/isah-timereg-write-pacing|`TimeRegCollector.TempDelayForSql = 1000`]] — 1-second pause between ISAH writes.
- **EmpId convention `S*`** — IDs starting with `"S"` get special-case `DeleteNextDayTimeRegLines` handling. Q-186 to document what `S` means (`S` for "service"? "Sander"-special? "machine sub"?).
- **MachGrpCode `T40` → `P02`** debug substitution. Q-180 — the `T40` "debugger employee" MachGrp gets silently rewritten.

## Surprises

1. **`GetCurrentTimeInSeconds` rounds Second to 0** (minute granularity). The commented-out `TimeOfDay.TotalSeconds` would give sub-minute precision but isn't used. Q-187.
2. **`ChangeToShopDoc` silently substitutes ShopDoc** when the supplied one doesn't match the employee's MachGrp (step 5 above). Q-184.
3. **`ChangeToShopDoc` outer `Try…Catch` swallows all exceptions.** A partial mutation (e.g. PBOOE deleted, time-reg insert failed) leaves the database inconsistent and the caller unaware. Q-183.
4. **Assistant cascade uses parameter-as-flag trick** (`AssistantMachGrpCode Is Nothing` decides whether to recurse). Comment in source confirms it's a hack. Q-188 — refactor.
5. **`CreateCombinedTimeRegLines` forces StartDate to 00:01**, then walks forward through overlapping lines. Means: late-night clock writes always reset to early-morning. Cycle-time *amount* is preserved, but the timestamp doesn't reflect when the work actually happened. Q-189, `#safety-relevant` for payroll-audit alignment.
6. **`T40 → P02` MachGrp rewrite** is a *production-side* fix for a *debug* employee. Q-180.
7. **`TimeRegInputType = 1`** with the `RR 20220725` comment — value was 2, changed for journal-post fix. No regression test in source. Q-190.
8. **`IsahUserCode = Common.APPLISAHUSERCODE`** here vs hardcoded `"ISAH"` in production / dossier classes — inconsistent. Q-185 (cross-references Q-165).
9. **`TimeRegCollector.ProcessCollections` aggregates exceptions but doesn't roll back**. A failure during `SetStarted` doesn't prevent `SetFinished` and `InsertTimeRegLines` from running. A ShopDoc could end up *finished* without being *started* in the ISAH log. Q-191.
10. **`TempDelayForSql = 1000` is `Public`** — caller-mutable. Setting it to `0` removes pacing but exposes the minute-granularity overlap problem. Q-192 — should be `Private` or surfaced as an option with explicit warning.

## Open questions

- **Q-179 (new):** `TimeRegistration.IP_Ins_TimeRegistr` correctly uses `Common.APPLISAHUSERCODE` while production/dossier classes hardcode `"ISAH"`. Harmonise (cross-references Q-165).
- **Q-180 (new):** `T40 → P02` debug MachGrp substitution in `CreateCombinedTimeRegLines`. Why production-side fix for a debug employee?
- **Q-181 (new):** Document `TempDelayForSql = 1000` — likely a workaround for the minute-granularity overlap behaviour. Confirm with SME.
- **Q-182 (new):** `ChangeToShopDoc` refuses to act if `Not E.IsMachineEmpId AndAlso Not E.GetIsPresent`. Combined with [[isah-identity|`Employee.GetIsObsolete` fail-closed]] (Q-133), an ISAH outage prevents all clocking. `#safety-relevant`
- **Q-183 (new):** `ChangeToShopDoc`'s outer `Try…Catch` swallows all exceptions. A partial mutation (PBOOE deleted but time-reg insert failed) leaves DB inconsistent. `#safety-relevant`
- **Q-184 (new):** `ChangeToShopDoc` silently substitutes ShopDocCode when MachGrp-resolution finds a different one. Logs Info but no caller callback. `#safety-relevant`
- **Q-185 (new):** Harmonise `IsahUserCode` — TimeRegistration uses `Common.APPLISAHUSERCODE`, production/dossier classes hardcode `"ISAH"`. Pick one.
- **Q-186 (new):** Document EmpId convention `S*` — what does the `S` prefix mean? Why does `S*` need `DeleteNextDayTimeRegLines`?
- **Q-187 (new):** `GetCurrentTimeInSeconds` rounds Second to 0 by design. Document the convention; explain why sub-minute precision was rejected.
- **Q-188 (new):** Refactor `ChangeAssistantsToShopDoc` / `ChangeToShopDoc` to break the recursive-loop trick (`AssistantMachGrpCode Is Nothing` as escape flag).
- **Q-189 (new):** `CreateCombinedTimeRegLines` forces `StartDate = 00:01` and walks forward — time-reg timestamp doesn't reflect when work actually happened. Confirm with payroll/HR. `#safety-relevant`
- **Q-190 (new):** `TimeRegInputType = 1` (was 2) — RR 2022-07-25 fix for missing JournaalPosten. Regression test?
- **Q-191 (new):** `TimeRegCollector.ProcessCollections` doesn't roll back: a `SetStarted` failure doesn't prevent `SetFinished` running. A ShopDoc could be marked finished without ever being started. `#safety-relevant`
- **Q-192 (new):** `TimeRegCollector.TempDelayForSql = 1000` is `Public` (caller-mutable). Setting to `0` removes pacing — surface the trade-off explicitly.

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/icenterlib-isah]] — parent MOC.
- [[isah-identity|`Employee.GetIsPresent` / `GetCurrentTimeReg`]] — TimeRegistration calls these.
- [[isah-shop-and-pur-doc|`ShopDoc.SetShopDocStartedInd` / `SetShopDocFinInd`]] — TimeRegCollector drives both.
- [[isah-production-hierarchy|`PBOO`]] — `IP_Ins_PBOOEmployee` writes to `T_ProdBOOEmployee`; PBOO.SetShopDocStatusCode writes the "20" started status.
- [[../external-systems/trutops-oseon|TruTops Oseon]] — `CreateCombinedTimeRegLines` ingests sheet-part data from Oseon; `_CutSheetOperRegistrationDataService.UpdateTimeRegProcessed` notifies Oseon when done.
- [[icenterlib-connections|`Connections.UseIsahNoDtrTimeReg`]] — the migration flag that's `True` in production today, bypassing the DTR-status-code branches throughout.

## Coverage

`_coverage.md`:
- `ICenterLib\ISAH\TimeRegistration.vb` → `done` (1274-line file; overview-level — per-method enumeration of all 19 publics deferred)
- `ICenterLib\ISAH\TimeRegCollector.vb` → `done`
