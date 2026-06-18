---
type: business-rule
title: "Maintenance window (default 01:00 - 04:00)"
status: needs-review
module: "ICenterLib/(root)"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\AppSettings.vb"
last-reviewed: ""
tags: [business-rule, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Maintenance window — default 01:00 to 04:00

## Rule

`AppSettings.IsInsideMaintenanceWindow()` returns `True` when the **current local time** falls inside a configurable maintenance window. The window is read from the **consuming assembly's `My.Settings`** (`MaintenanceWindowStart` and `MaintenanceWindowEnd`). If either setting is missing or unparseable, the window defaults to **01:00 to 04:00**. The function correctly handles overnight wraps (e.g. `22:00 → 06:00`).

## Where it lives

- File: `ICenterLib\AppSettings.vb` lines 321–343
- Symbol: `Public Shared Function IsInsideMaintenanceWindow() As Boolean`

## The code (minimal quote)

```vb
Public Shared Function IsInsideMaintenanceWindow() As Boolean
    Dim currentTime As TimeSpan = Date.Now.TimeOfDay

    Dim startTimeString As String = My.Settings.MaintenanceWindowStart
    Dim endTimeString   As String = My.Settings.MaintenanceWindowEnd

    Dim startTime, endTime As TimeSpan

    If Not TimeSpan.TryParse(startTimeString, startTime) Then
        startTime = New TimeSpan(1, 0, 0) ' fallback default 01:00
    End If
    If Not TimeSpan.TryParse(endTimeString, endTime) Then
        endTime = New TimeSpan(4, 0, 0)   ' fallback default 04:00
    End If

    If startTime <= endTime Then
        Return currentTime > startTime AndAlso currentTime <= endTime
    End If

    Return currentTime > startTime OrElse currentTime <= endTime   ' overnight wrap
End Function
```

## Why this is a business rule

iCenter uses a maintenance window to **defer non-critical work** (e.g. heavy batch jobs, mass updates) to a time when operators aren't actively using the system. Callers that gate behaviour on `IsInsideMaintenanceWindow()` will silently change behaviour between maintenance and non-maintenance hours.

Phase-3 follow-up: enumerate callsites to find which operations are gated on this.

## Triggers / when it fires

- Any time a caller invokes `AppSettings.IsInsideMaintenanceWindow()`. Returns a fresh evaluation against the wall-clock each call (no caching).
- Inputs: process-local `Date.Now`, and the consuming assembly's `My.Settings.MaintenanceWindowStart / MaintenanceWindowEnd`.

## Effects

Depends entirely on the caller. Some likely use cases (Phase-3 follow-up to confirm):
- skip heavy CAD-batchserver jobs outside maintenance hours
- defer reports / lock cleanups to maintenance hours
- silently swallow non-critical exceptions during maintenance to avoid waking operators

## Edge cases / known exceptions

- **Uses `My.Settings` of the consuming assembly.** When called from iCenter, that's `iCenter.exe`'s `app.config`. The current snapshot of iCenter's `app.config` (last reviewed 2026-06-18) **does not list** `MaintenanceWindowStart` or `MaintenanceWindowEnd` settings — so the **default 01:00–04:00 window applies in production today**. Q-112.
- The `My.Settings` lookup goes via the auto-generated `My.MySettings` strongly-typed wrapper. If the setting names don't exist on `My.MySettings`, this won't compile. Means: either iCenter's `My.MySettings` *does* have these properties (with empty defaults) or the build refers to settings declared elsewhere — Phase-3 follow-up.
- The comparison uses strict `>` for the lower bound (`currentTime > startTime`) — a time exactly equal to `startTime` is **outside** the window. Asymmetric: the upper bound uses `<=`.
- Time-zone insensitive: uses `Date.Now.TimeOfDay`. Works as expected on a Dutch-time JAZO server but not on a UTC-configured host.

## Safety classification

- [ ] Touches physical process — no.
- [x] Behaviour-gating → `#needs-review`
- [ ] Reversible if wrong? — Yes.
- [ ] Blocks production if it fails? — Only if a caller specifically depends on it.

## SME questions

- **Q-112** (from MOC): confirm iCenter's `app.config` has `MaintenanceWindowStart/End` settings, or accept the 01-04 default.
- **Q-129 (new):** Enumerate callsites of `IsInsideMaintenanceWindow` and document what each one gates.

Logged in [[../needs-review/_index]].

## Related

- [[../modules/icenterlib-appsettings]] — defining module.
