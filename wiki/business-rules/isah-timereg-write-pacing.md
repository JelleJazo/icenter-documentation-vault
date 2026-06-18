---
type: business-rule
title: "TimeRegCollector 1-second write pacing"
status: needs-review
module: "ICenterLib/ISAH"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\TimeRegCollector.vb"
last-reviewed: ""
tags: [business-rule, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `TimeRegCollector.TempDelayForSql = 1000` — 1-second pause between writes

## Rule

`TimeRegCollector.ProcessCollections()` sleeps **1000 ms between every ISAH write** during a batch flush. The same field is consulted three times:

- Between consecutive `SetShopDocStartedInd(True)` calls in `SetStarted()`.
- Between consecutive `SetShopDocFinInd(True)` calls in `SetFinished()`.
- Between consecutive `CreateCombinedTimeRegLines(...)` per employee in `InsertTimeRegLines()`.

For a flush with 10 started + 10 finished + 5 employee time-reg batches, the pacing alone adds **24 seconds** to the run (24 inter-write sleeps × 1 s).

The `Public TempDelayForSql As Integer = 1000` field is **caller-mutable** — a code path that wants to disable pacing can set it to 0 before calling `ProcessCollections()`.

## Where it lives

- File: `ICenterLib\ISAH\TimeRegCollector.vb`
- Field: line 11 `Public TempDelayForSql As Integer = 1000`
- Consumers: lines 56, 73, 95 (each preceded by `If TempDelayForSql > 0 Then Threading.Thread.Sleep(TempDelayForSql)`)

## The code (minimal quote)

```vb
Public Class TimeRegCollector
    ' ...
    Public TempDelayForSql As Integer = 1000
    ' ...
    Private Sub SetStarted()
        For Each SD As ShopDoc In SetStartedList
            Try
                SD.SetShopDocStartedInd(True)
            Catch ...
            End Try
            If TempDelayForSql > 0 Then
                Threading.Thread.Sleep(TempDelayForSql)
            End If
        Next
    End Sub
End Class
```

## Why this is a business rule

No comment explains the 1-second pause, but the likely motivation is **collision avoidance** with the [[isah-timereg-minute-granularity|minute-granularity rule]]:

- `TimeRegistration.GetCurrentTimeInSeconds` rounds the current time to the nearest minute boundary.
- Back-to-back `IP_Ins_TimeRegistr` writes inside the same second produce rows with **identical `StartTime`**.
- `CreateCombinedTimeRegLines`'s overlap-resolver then has to walk every existing row forward, shifting timestamps for no real benefit.
- Pacing the writes by 1 s spreads them across different *seconds* (though not necessarily different *minutes* — Q-196).

The variable is named `TempDelayForSql` with the `Temp` prefix — looks like the developer expected to remove or replace it later. Currently load-bearing in production.

## Triggers / when it fires

- Every call to `ProcessCollections()` — typically batch flushes from CadBatchserver jobs or scheduled imports.

## Effects

- ~1 s extra wall-clock per started ShopDoc, per finished ShopDoc, and per per-employee time-reg batch in the flush.
- Avoids overlap-resolution churn in `CreateCombinedTimeRegLines`.

## Edge cases / known exceptions

- **`TempDelayForSql = 0`** disables pacing. The collision behaviour returns; `CreateCombinedTimeRegLines` will reshuffle timestamps. Q-192 — surface the trade-off explicitly.
- **Large batches** scale linearly: 100 started + 100 finished = ~200 s added. CadBatchserver jobs that ingest large Oseon sheets pay this cost.
- **Failure in `SetStarted` doesn't skip the sleep** — the `Try…Catch` is *inside* the per-iteration block, so exceptions are caught and the sleep still happens. The whole flush always takes minimum N seconds.

## Safety classification

- [ ] Touches physical process — no.
- [x] Production-runtime cost (large flushes) → `#safety-relevant` (batch windows / SLA concern).
- [x] Reversible if wrong? — Yes (set to 0) but the underlying collision returns.
- [ ] Blocks production if it fails? — No.

## SME questions

- **Q-181** (cross-listed): document the motivation for `TempDelayForSql = 1000`. Confirm with SME.
- **Q-192** (cross-listed): make `TempDelayForSql` `Private` or surface as an explicit option with a warning.
- **Q-196** (cross-listed): with 1-second pacing, two writes can still land in the same *minute*. Bump to 60 s or accept?

Logged in [[../needs-review/_index]].

## Related

- [[../modules/isah-time-registration]] — defining module.
- [[isah-timereg-minute-granularity]] — sister rule.
- [[../mocs/icenterlib-isah]] — parent MOC.
