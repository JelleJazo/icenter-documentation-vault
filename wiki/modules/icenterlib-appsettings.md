---
type: module
title: "ICenterLib\\AppSettings.vb — T_ApplicationSettings reader"
status: done
module: "ICenterLib/(root)"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\AppSettings.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `ICenterLib\AppSettings.vb` — `T_ApplicationSettings` reader

## Purpose

Wraps the **iCenter database's `T_ApplicationSettings` table** as a typed key-value store. Adds a small set of `Public Const` / `Public Shared` properties for paths and physical constants that don't (yet) live in the DB. Used everywhere via `AppSettings.GetStringSetting("MyKey")`, `AppSettings.GetIntegerSetting`, etc.

> **Two distinct settings stores live behind ICenterLib**:
> 1. **`T_ApplicationSettings`** — iCenter DB, dynamic, key-value. Read here via `SIP_GetAppSetting` stored procedure. Used for almost everything tunable at runtime.
> 2. **`My.Settings.Properties("...").DefaultValue`** — the consuming assembly's `app.config`. Used for the Elumatec threshold values (`EluMaxStepDepthSTL/ALU`, `EluLargeRectangle*`, …) and other static paths.

## Public surface

```vb
Public Class AppSettings
    ' Path roots — all derived from ICENTER_ROOT_SERVER
    Public Shared ReadOnly Property XML_ROOT_SERVER       As String
    Public Shared ReadOnly Property MODELINFO_ROOT_SERVER As String
    Public Shared ReadOnly Property PV_ROOT_SERVER        As String
    Public Shared ReadOnly Property LST_ROOT_SERVER       As String
    Public Shared ReadOnly Property XLS_ROOT_SERVER       As String
    Public Shared ReadOnly Property PDF_PROD_ROOT_SERVER  As String

    Public Const ICENTER_ROOT_SERVER As String = "\\jazo.local\dfs\TEKDB\TEKENINGENDB\"
    Public Shared NetworkMapping As New Dictionary(Of String, String) From {{"G:", "\\jazo.local\dfs\PM"}}
    Public Const SmtDeburrSpeed As Double = 0.225 / 60   ' m²/sec

    ' Setting accessors (all hit SIP_GetAppSetting against T_ApplicationSettings)
    Public Shared Function GetStringSetting(Name As String)        As String
    Public Shared Function GetIntegerSetting(Name As String)       As Integer
    Public Shared Function GetDateSetting(Name As String)          As Date
    Public Shared Function GetDoubleSetting(Name As String)        As Double
    Public Shared Function GetBooleanSetting(Name As String)       As Boolean
    Public Shared Function GetLongSetting(Name As String)          As Long
    Public Shared Function GetDictionaryFromJSON(Name As String)   As Dictionary(Of String, String)
    Public Shared Function GetDataTableFromJSON(Name As String)    As DataTable
    Public Shared Function GetDecryptedSetting(Name As String)     As String

    ' List-style accessors
    Public Shared Function SettingContainsUsername(Key, Username)  As Boolean
    Public Shared Function SettingContainsValue(Key, Value)        As Boolean
    Public Shared Function SettingContainsValue(Key, Values())     As Boolean
    Public Shared Function StringContainsValue(MyString, Values())  As Boolean
    Public Shared Function SettingContainsMatch(Key, Value)        As Boolean
    Public Shared Function GetSettingAsList(Key As String)         As List(Of String)
    Public Shared Function GetEmailAddresses(Key As String)        As List(Of String)

    Public Shared Sub Update(Key As String, Value As String)

    Public Const APPSETTINGDOCUMENTPREFIX As String = "APPSETTING:"
    Public Shared Function IsInsideMaintenanceWindow()             As Boolean
End Class
```

## The `T_ApplicationSettings` table

Implied schema (from observed `SELECT` / `UPDATE` patterns):

| Column | Type |
|--------|------|
| `SettingName` | string (PK) |
| `SettingValue` | string |
| (special row) | `SettingName = "CipherKey1"` — used by `GetDecryptedSetting` to symmetrically decrypt other rows whose value is the ciphertext |

`SIP_GetAppSetting(@SettingName)` returns one row with `SettingValue`. `Update` runs a raw `UPDATE` against the table; no `INSERT` path is visible — settings must pre-exist.

## Path roots

| Property | Value |
|----------|-------|
| `ICENTER_ROOT_SERVER` | `\\jazo.local\dfs\TEKDB\TEKENINGENDB\` |
| `XML_ROOT_SERVER` | `<root>XML_DIR\` |
| `MODELINFO_ROOT_SERVER` | `<root>modelinfo_dir\` |
| `PV_ROOT_SERVER` | `<root>pv_prod_dir\` (ProductView) |
| `LST_ROOT_SERVER` | `<root>lst_dir\` |
| `XLS_ROOT_SERVER` | `<root>xls_dir\` |
| `PDF_PROD_ROOT_SERVER` | `<root>pdf_prod_dir\` |

All assume the DFS share `\\jazo.local\dfs\TEKDB\TEKENINGENDB\` is mounted / reachable. Build-time changes need a recompile.

## Constants of note

- **`NetworkMapping = {"G:" → "\\jazo.local\dfs\PM"}`** — declared as a `Public Shared` Dictionary. Used (Phase-3 follow-up) to resolve `G:\…` paths into their UNC equivalents.
- **`SmtDeburrSpeed = 0.225 / 60`** — m²/sec. **Process-relevant constant** documented in [[../business-rules/smt-deburr-speed]]. The fact that it lives in `AppSettings.vb` (rather than next to the sheet-metal calculator) is surprising — Q-122.
- **`APPSETTINGDOCUMENTPREFIX = "APPSETTING:"`** — document-prefix marker; Phase-3 follow-up on what consumes it.

## Setting accessors

All eventually delegate to `GetStringSetting`. Type-conversion methods (`GetIntegerSetting`, `GetDoubleSetting`, `GetDateSetting`, `GetLongSetting`, `GetBooleanSetting`) parse the string and return a typed value with a 0-/false-/Date.MinValue fallback on failure.

- `GetStringSetting`: runs `SIP_GetAppSetting`; silently returns `Nothing` on error.
- `GetDecryptedSetting`: runs a custom `SELECT` that fetches BOTH the requested setting AND the `CipherKey1` setting in one query. Decrypts via `Encryption.SecurityController.Decrypt(CipherKey1, ciphertext)`. **The encryption key is stored in the same table as the encrypted data.** Q-111.
- `GetDictionaryFromJSON` / `GetDataTableFromJSON`: deserialise the raw JSON string via Newtonsoft.Json into the requested shape.

## List-style accessors

`T_ApplicationSettings.SettingValue` rows often carry `;`-delimited lists. The list accessors:

- `GetSettingAsList(Key)` — split on `;`, trim, drop empties.
- `SettingContainsValue(Key, Value)` — case-insensitive check via the in-memory list. **Always returns true if the list contains `"*"`** (added implicitly to the check values — line 230). Wildcard convention.
- `SettingContainsMatch(Key, Value)` — checks each list entry as a **regex** against `Value` (case-insensitive). Used for pattern-based permission gates.
- `SettingContainsUsername(Key, Username)` — alias for `SettingContainsValue`.

`GetEmailAddresses(Key)` — splits, then per-entry: tries to construct a `MailAddress` directly; if that fails (it's an EmpId, not an email), looks up the JIBA local email via `JIBA.Employee.GetUserInfoByEmpId`. Returns a list of valid addresses. **Drops invalid entries silently.**

## `IsInsideMaintenanceWindow()`

```vb
Public Shared Function IsInsideMaintenanceWindow() As Boolean
    Dim currentTime As TimeSpan = Date.Now.TimeOfDay
    Dim startTimeString As String = My.Settings.MaintenanceWindowStart
    Dim endTimeString   As String = My.Settings.MaintenanceWindowEnd

    Dim startTime, endTime As TimeSpan
    If Not TimeSpan.TryParse(startTimeString, startTime) Then startTime = New TimeSpan(1, 0, 0)  ' 01:00 fallback
    If Not TimeSpan.TryParse(endTimeString, endTime)     Then endTime   = New TimeSpan(4, 0, 0)  ' 04:00 fallback

    If startTime <= endTime Then
        Return currentTime > startTime AndAlso currentTime <= endTime
    End If
    Return currentTime > startTime OrElse currentTime <= endTime    ' overnight wrap
End Function
```

Two things worth flagging:

1. **Reads `My.Settings`, not the DB.** `My.Settings` resolves to the consuming assembly's `app.config` — meaning **iCenter** must have a `MaintenanceWindowStart/End` setting in its `app.config`, not ICenterLib's. Q-112 — confirm iCenter's `app.config` has these keys (the current `app.config` snapshot didn't list them, so the fallbacks 01:00–04:00 likely fire in practice).
2. The overnight-wrap branch (line 342) handles windows like `22:00-04:00`.

Documented as [[../business-rules/icenterlib-maintenance-window]].

## Surprises

1. **`GetStringSetting` returns `Nothing` on any error** (top-level Catch on line 73). Callers checking `If s Is Nothing` work; callers checking `If s = ""` get a NullReferenceException at runtime.
2. **`GetIntegerSetting` converts via `Double` → `Int32`** (`Convert.ToInt32(GetDoubleSetting(Name))`). Banker's rounding for decimals. If the setting holds a string like `"abc"`, `GetDoubleSetting` returns 0 → `GetIntegerSetting` returns 0. Silent.
3. **`GetDataTableFromJSON` calls Newtonsoft.Json directly** — depends on a NuGet-restored `Newtonsoft.Json` reference (not in iCenter's `packages.config`, must be in ICenterLib's).
4. **`SettingContainsValue` always allows `"*"`** as a wildcard — even if the caller didn't include it. The implicit add of `"*"` to `Values` (line 232) is *aggressive*: any setting list containing `"*"` matches *every* test. Q-123.
5. **`Update` is the only write method** — no `Insert` path. Setting must pre-exist (presumably created manually in the DB).
6. **`IsInsideMaintenanceWindow` reads `My.Settings`** of the consuming assembly. From ICenterLib's perspective, that's the host process's `app.config`. Means: if `iCenter.exe`'s `app.config` lacks `MaintenanceWindowStart/End`, the fallback 1-4 AM applies. Same code called from a different host (e.g. a scheduled job) reads *that* host's `app.config`.
7. **`GetDecryptedSetting` does a 2-row fetch** in a single SQL via two correlated subqueries — clever, single round-trip.

## Business rules surfaced here

- [[../business-rules/smt-deburr-speed|SMT deburr speed = 0.225 m²/min]] — process-relevant constant.
- [[../business-rules/icenterlib-maintenance-window|Maintenance window]] — default 01:00–04:00 if `My.Settings` not set.
- `SettingContainsValue` always-allows `"*"` — list-style permission wildcard (Q-123).

## Open questions

- **Q-111** (from MOC): `GetDecryptedSetting` reads the cipher key from the same table as the encrypted value.
- **Q-112** (from MOC): `IsInsideMaintenanceWindow` reads `My.Settings` of the consuming assembly — confirm iCenter's `app.config` has `MaintenanceWindowStart/End`.
- **Q-122 (new):** `SmtDeburrSpeed = 0.225 / 60` lives in `AppSettings.vb`. Why here and not next to the sheet-metal calculator?
- **Q-123 (new):** `SettingContainsValue` implicitly adds `"*"` to the check values — confirm intentional wildcard semantics.
- **Q-124 (new):** `Update` has no INSERT fallback — settings must pre-exist in `T_ApplicationSettings`. How are new settings created? Manual SQL?

Logged in [[../needs-review/_index]].

## Coverage

`_coverage.md`: `ICenterLib (root)\AppSettings.vb` → `done`.
