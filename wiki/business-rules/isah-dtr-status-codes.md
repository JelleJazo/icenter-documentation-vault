---
type: business-rule
title: "DTR status codes — 2-char state machine (AO/AW/IO/IW/II/OO/OW/OI)"
status: needs-review
module: "ICenterLib/ISAH"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\TimeRegistration.vb"
last-reviewed: ""
tags: [business-rule, needs-review, dead-code]
created: 2026-06-18
updated: 2026-06-18
---

# DTR status codes — 2-char state machine

> **Currently dead in production.** `Connections.UseIsahNoDtrTimeReg = True` bypasses every DTR-status-code branch in iCenter. This rule documents the *test-DB path* and the historical pre-migration behaviour. If anyone ever flips `UseIsahNoDtrTimeReg = False`, this rule is suddenly live again.

## Rule

iCenter's pre-migration ISAH connection used **2-character `DTRStatusCode` values** on `T_TimeRegistration` rows to track an employee's daily time-registration state. Eight values are referenced in `TimeRegistration.ConvertDtrStatusCode`:

| Code | First char | Second char | Meaning (inferred from transitions) |
|------|-----------|-------------|---------------------------------------|
| `AO` | A = *aanwezig* (present) | O = open | clocked in, presence open |
| `AW` | A = aanwezig | W = *wachten* / closed | presence closed |
| `IO` | I = *indirect* | O = open | indirect work, open |
| `IW` | I = indirect | W = closed | indirect work closed |
| `II` | I = indirect | I = interrupted (day-rollover) | indirect carried to next day |
| `OO` | O = *order* | O = open | order work, open |
| `OW` | O = order | W = closed | order work closed |
| `OI` | O = order | I = interrupted | order carried to next day |

Transitions (`ConvertDtrStatusCode(code, type)`):

| Close type | Source → Target |
|-----------|-----------------|
| `WorkCode` | `IO → IW`, `OO → OW` |
| `PresenceCode` | `AO → AW` |
| `NextDay` | `IO → II`, `OO → OI` |
| `ReturnSameDay` | `II → IO`, `OI → OO` |

`Employee.GetIsPresent` (cf. [[../modules/isah-identity]]) used to filter `DTRStatusCode = 'AO'` on the legacy path.

## Where it lives

- File: `ICenterLib\ISAH\TimeRegistration.vb` lines 695–727 (the conversion table)
- File: `ICenterLib\ISAH\Employee.vb` lines 35–37 (the legacy presence-check)
- Toggle: `ICenterLib\Connections.vb` `Public Const UseIsahNoDtrTimeReg As Boolean = True`

## The code (minimal quote)

```vb
Private Function ConvertDtrStatusCode(DtrStatusCode As String, Type As TypeOfDtrCloseCode) As String
    Dim ReturnValue As String = ""

    Select Case Type
        Case TypeOfDtrCloseCode.PresenceCode
            Select Case DtrStatusCode
                Case "AO" : ReturnValue = "AW"
            End Select
        Case TypeOfDtrCloseCode.WorkCode
            Select Case DtrStatusCode
                Case "IO" : ReturnValue = "IW"
                Case "OO" : ReturnValue = "OW"
            End Select
        Case TypeOfDtrCloseCode.NextDay
            Select Case DtrStatusCode
                Case "IO" : ReturnValue = "II"
                Case "OO" : ReturnValue = "OI"
            End Select
        Case TypeOfDtrCloseCode.ReturnSameDay
            Select Case DtrStatusCode
                Case "II" : ReturnValue = "IO"
                Case "OI" : ReturnValue = "OO"
            End Select
    End Select
    Return ReturnValue
End Function
```

## Why this is a business rule

DTR (*Daily Time Registration*) is ISAH's older time-clocking model. The status code distinguishes:

- **Presence vs work**: an employee may be clocked-in (present) but not on a specific order — `A*` vs `I*` vs `O*`.
- **Open vs closed**: only one row per employee per day per status type may be "O" (open) at a time. Closing a row converts to its "W" form.
- **Day-rollover**: shop-floor workflows that span midnight need an "interrupted" intermediate state (`II` / `OI`) that the next-day re-open converts back to active (`IO` / `OO`).

iCenter's migration to the **no-DTR model** (`UseIsahNoDtrTimeReg = True`) replaced this with a simpler `StartTime <> 0 AND EndTime = 0` open/closed test. The code remains as fallback for the test DB and for historical compatibility.

## Triggers / when it fires

- **Currently never** in production (`UseIsahNoDtrTimeReg = True`).
- In the legacy path: every `IP_Ins_TimeRegistr` call sets `@DTRStatusCode`, `@DTRInfoCode = "#"`, `@DTRExportInd = 1` (`TimeRegistration.vb` lines 775–779). Every `CloseTimeRegLine` reads `CurrDtrStatusCode` from `T_TimeRegistration` and writes back the converted target.

## Effects

When live: `T_TimeRegistration.DTRStatusCode` reflects the employee's current clocking state. ISAH-side DTR exports consume this column.

## Edge cases / known exceptions

- **Conversion table is non-total.** `AO` on a `WorkCode` close returns `""` (empty string), not an error. The caller gets a blank status. Same for `IW` on any conversion type. Q-193.
- **No bidirectional validation.** Closing an already-closed line (`IW → ???`) silently returns `""`. No assertion that the input is in an open state.
- **`UseIsahNoDtrTimeReg = True`** kills the entire branch. Any DTR-related query in the test DB will see no rows because no DTR codes are written.

## Safety classification

- [ ] Touches physical process — no.
- [x] Drives payroll / HR reporting → `#needs-review` (if ever re-enabled).
- [x] Currently dead → `#dead-code`
- [ ] Reversible if wrong? — Yes (toggle the const).

## SME questions

- **Q-132** (cross-listed): document the `UseIsahNoDtrTimeReg = True` migration as complete; remove the dead branches?
- **Q-193 (new):** Conversion table is non-total — `AO` on `WorkCode` returns `""`. Should this throw or stay silent?

Logged in [[../needs-review/_index]].

## Related

- [[../modules/isah-time-registration]] — defining module.
- [[isah-hourcodes]] — sister rule (HourCodes "01"/"02"/"AW" complement DTR codes).
- [[icenterlib-connections|`Connections.UseIsahNoDtrTimeReg`]] — the master switch.
- [[../mocs/icenterlib-isah]] — parent MOC.
