---
type: business-rule
title: "Time-reg minute granularity (Second is dropped)"
status: needs-review
module: "ICenterLib/ISAH"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\TimeRegistration.vb"
last-reviewed: ""
tags: [business-rule, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Time-reg minute granularity — `Second` is intentionally dropped

## Rule

Every `T_TimeRegistration` row's `@StartTime` / `@EndTime` integer is **minutes-since-midnight × 60**, never seconds-since-midnight. `TimeRegistration.GetCurrentTimeInSeconds` computes the value as `Hour * 3600 + Minute * 60` — explicitly dropping the `Second` component. Two clock-actions within the same minute land on the **same `StartTime`**.

This is a design choice, not an oversight: the commented-out `Return DateTime.Now.TimeOfDay.TotalSeconds` on line 881 of `TimeRegistration.vb` shows that the developer considered sub-minute precision and rejected it.

## Where it lives

- File: `ICenterLib\ISAH\TimeRegistration.vb` lines 877–882
- Function: `Public Function GetCurrentTimeInSeconds() As Integer`

## The code (minimal quote)

```vb
Public Function GetCurrentTimeInSeconds() As Integer
    Dim returnValue As Integer = 0
    returnValue = (DateTime.Now.Hour * 3600) + (DateTime.Now.Minute * 60)
    Return returnValue
    'Return DateTime.Now.TimeOfDay.TotalSeconds       ' rejected alternative
End Function
```

## Why this is a business rule

JAZO's shop-floor reporting and payroll use minute resolution — sub-minute precision adds noise without value when an operator manually clocks in/out. The choice also aligns with ISAH's `T_HourCodeTime` rate-card resolution (per-minute rates).

Consequences:

1. **Two operators clocking the same machine in the same minute** create two time-reg rows with identical `StartTime` — ISAH-side reports must handle the tie.
2. **`CreateCombinedTimeRegLines` overlap-resolution** (lines 1276–1288 of `TimeRegistration.vb`) rounds the recovery `StartTime` forward to the next whole minute: `NewStartTime = Ceiling(max(EndTime, CalculatedEndTime) / 60) * 60`. Without minute-rounding this advance would be sub-minute and not robust.
3. **`TimeRegCollector.TempDelayForSql = 1000`** (1 second between writes — see [[isah-timereg-write-pacing]]) likely exists *because* of this granularity: back-to-back writes within the same second hit the same `StartTime`. The 1s pacing pushes consecutive writes onto different timestamps (though not guaranteed onto different *minutes* — Q-196).

## Triggers / when it fires

- Every `IP_Ins_TimeRegistr` writer call ultimately uses `GetCurrentTimeInSeconds()` to compute `@StartTime` and `@EndTime` (unless the caller supplies its own integer).
- The overlap-resolver in `CreateCombinedTimeRegLines` consumes the granularity directly.

## Effects

- `T_TimeRegistration.StartTime` and `EndTime` columns are integers in `[0, 86400]` always evenly divisible by 60.
- Per-row duration calculations (`EndTime - StartTime`) always come out in whole-minute multiples.

## Edge cases / known exceptions

- **Same-minute collisions** produce two rows with `StartTime = EndTime`. Reports must disambiguate by `TimeRegLineNr` (the surrogate PK).
- **Sub-minute work** (e.g. a 30-second operation) registers as 0 duration. Not iCenter's concern — operators are expected to batch.
- **`CreateCombinedTimeRegLines` forces StartDate to 00:01** — see Q-189 in [[../modules/isah-time-registration]]. Combined with minute-granularity, this means the second-precision of when work actually happened is twice-discarded.

## Safety classification

- [ ] Touches physical process — no.
- [x] Drives payroll / cost reporting → `#safety-relevant` (precision affects per-minute rate billing).
- [ ] Reversible if wrong? — Trivially (uncomment line 881) but downstream reporting / ISAH rate-card scheduling may not handle sub-minute values.
- [ ] Blocks production if it fails? — No.

## SME questions

- **Q-187** (cross-listed): document the minute-granularity design choice; explain why sub-minute precision was rejected.
- **Q-196 (new):** With `TempDelayForSql = 1000`, two writes 1.0 second apart can still land in the same minute. Bump to 60000 ms or accept the collision?

Logged in [[../needs-review/_index]].

## Related

- [[../modules/isah-time-registration]] — defining module.
- [[isah-timereg-write-pacing]] — sister rule (`TempDelayForSql = 1000`).
- [[isah-hourcodes]] — sister rule.
- [[../mocs/icenterlib-isah]] — parent MOC.
