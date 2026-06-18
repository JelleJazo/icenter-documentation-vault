---
type: business-rule
title: "ISAH time-reg HourCodes (01 cycle / 02 setup / AW presence)"
status: needs-review
module: "ICenterLib/ISAH"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\TimeRegistration.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\Employee.vb"
last-reviewed: ""
tags: [business-rule, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# ISAH time-reg `HourCode` — `"01"` cycle, `"02"` setup, `"AW"` presence

## Rule

Every row in `T_TimeRegistration` carries an **`HourCode`** (string). iCenter uses three hardcoded values for the rows it writes:

| `HourCode` | Meaning | Used by |
|-----------|---------|---------|
| `"01"` | normal **cycle** time | `AutoTimeRegHourCode` constant; `CreateCombinedTimeRegLines` for cycle rows; default for `ChangeToShopDoc` callers |
| `"02"` | **setup** time | `CreateCombinedTimeRegLines` setup rows (`MachSetupTime`) |
| `"AW"` | "Aanwezig" / **presence** | filtered by `Employee.GetIsPresent` and `Employee.GetCurrentTimeRegBasic`; the row distinguishes "clocked-in" from "working on order X" |

Other HourCodes presumably exist in `T_HourCode` (vacation, sick, training, etc.) but iCenter never writes them; only `T_TimeRegistration` rows that pass *through* iCenter use the three above.

## Where it lives

- File: `ICenterLib\ISAH\TimeRegistration.vb`
- Constant: `Public Const AutoTimeRegHourCode As String = "01"` (line 7)
- Inserts: `CreateCombinedTimeRegLines` lines 1353 (`"02"`) and 1368 (`"01"`)
- Filter: `Employee.vb` lines 32, 36, 117 (`HourCode = 'AW'`)

## The code (minimal quote)

```vb
' Constant
Public Const AutoTimeRegHourCode As String = "01"

' CreateCombinedTimeRegLines — split setup vs cycle into two rows
If MachSetupTimeTotal > 0 Then
    ISAH.TimeRegistration.CreateTimeRegLine(dtTimeReg, StartDate, EndDate,
        TimeRegShopDocCode, ProdHeaderDossierCode, MachGrpCode, ProdBOOLineNr,
        MachSetupTimeTotal, EmpId, "02")              ' setup row
End If

Dim MachCycleTimeTotal As Double = dtSheetPart.Compute("Sum(MachCycleTimeTotal)", ...)
ISAH.TimeRegistration.CreateTimeRegLine(dtTimeReg, StartDate, EndDate,
    TimeRegShopDocCode, ProdHeaderDossierCode, MachGrpCode, ProdBOOLineNr,
    MachCycleTimeTotal, EmpId, "01")                  ' cycle row
```

```vb
' Employee.GetIsPresent — filter on presence row
"SELECT TimeRegLineNr, EmpId, HourCode FROM T_TimeRegistration
 WHERE EmpId=@EmpId AND HourCode='AW' AND Starttime <> 0 AND EndTime = 0
   AND RegDate=@RegDate"
```

## Why this is a business rule

iCenter explicitly separates **machine setup time** from **machine cycle time** so JAZO's cost-accounting can charge them differently (setup is amortised across the batch; cycle scales with quantity). The split is materialised as **two rows** per ShopDoc-employee-batch combination — one `"01"` row carrying `MachCycleTimeTotal`, one `"02"` row carrying `MachSetupTimeTotal`.

The `"AW"` rows are written by ISAH's own clock-in flow (not by iCenter directly), but iCenter *queries* them to check presence — meaning iCenter's clock-aware logic depends on ISAH's clock-in flow having written an `AW` row with `Starttime <> 0 AND EndTime = 0`. Disconnection from ISAH or a missed clock-in → iCenter believes the employee is not present and refuses to clock work.

## Triggers / when it fires

- `CreateCombinedTimeRegLines` writes both `"01"` and `"02"` rows when registering completed sheet-metal/Oseon work.
- `ChangeToShopDoc(EmpId, ShopDocCode, HourCode, ...)` accepts the HourCode as a caller-supplied parameter — most callers pass `"01"` (cycle).
- Setup-row dedup: `CreateCombinedTimeRegLines` checks for an existing `"02"` row before writing (`GetTimeRegLinesByShopDocCodeAndMachGrpCode(..., "02")` — skips if already present). No equivalent check for `"01"` cycle rows — multiple cycle entries are intentional.

## Effects

- Per-ShopDoc setup time is recorded *once* (deduplicated).
- Per-ShopDoc cycle time accumulates across consecutive clockings.
- ISAH-side reporting and payroll consume the two row types separately.

## Edge cases / known exceptions

- **Stringly-typed.** `"01"`, `"02"`, `"AW"` are bare strings throughout iCenter. A typo (`"O1"` with a letter O instead of zero) would silently land in ISAH and break the reporting joins. Q-194.
- **Hardcoded.** Adding a third "kind of registered time" (e.g. `"03"` for changeover) means editing `CreateCombinedTimeRegLines` and any consumers.
- **Setup-row dedup is per-`(ShopDoc, MachGrp)`.** Reassigning an employee to a different MachGrp re-writes the setup row.
- **`"AW"` is read but never written by iCenter.** If ISAH's own clock-in flow is broken or bypassed, iCenter's presence check fails. Q-195.

## Safety classification

- [ ] Touches physical process — no.
- [x] Drives payroll, cost-accounting, capacity reporting → `#safety-relevant`
- [x] Reversible if wrong? — Yes (correct the row).
- [ ] Blocks production if it fails? — Yes indirectly (a `"AW"`-absent employee can't clock work).

## SME questions

- **Q-194 (new):** HourCodes are bare strings — wrap in an enum / `Public Const` registry on `TimeRegistration` to prevent typos?
- **Q-195 (new):** `"AW"` is read but never written by iCenter. Document the contract with ISAH's own clock-in flow.

Logged in [[../needs-review/_index]].

## Related

- [[../modules/isah-time-registration]] — defining module.
- [[isah-dtr-status-codes]] — sister rule (DTR codes complement HourCodes).
- [[isah-timereg-minute-granularity]] — sister rule (minute precision).
- [[../mocs/icenterlib-isah]] — parent MOC.
