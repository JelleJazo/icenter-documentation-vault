---
type: module
title: "ICenterLib\\Common.vb — shared helpers and constants"
status: done
module: "ICenterLib/(root)"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\Common.vb"
last-reviewed: 2026-06-18
tags: [module, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `ICenterLib\Common.vb` — shared helpers and constants

## Purpose

The **static-helper grab-bag** at ICenterLib's root. ~533 lines, all `Public Shared` — no instance state. Centralises:

- application-wide culture / locale.
- the magic constants every other part of iCenter reads.
- user-identity helpers (`GetUsername`, `GetComputername`) with terminal-server awareness.
- version reporting (`GetApplicationVersion`).
- file-system housekeeping (`CreateDirectory`, `DeleteFolder`, `DeleteFolderIfEmpty`).
- launchers for external programs (`OpenWebPage`, `OpenInChrome`, `OpenInNotepadPP`, `OpenClickOnceApplication`).
- type-coercion helpers (`GetAsStringValue`, `GetAsBooleanValue`, `GetAsIntegerValue`).
- SQL helpers (`GetSqlInString`, `SqlParameterValue`, `GetTableData`).

Everything is referenced via `ICenterLib.Common.<X>` from iCENTER (the qualifier is often dropped because of the `Import ICenterLib` in `iCenter.vbproj`).

## Constants (the load-bearing ones)

| Const | Value | Meaning |
|-------|-------|---------|
| `APPLISAHUSERCODE` | `"ICENTER"` | ISAH user that iCenter logs in as |
| `IPPARTPREFIX` | `"IA"` | prefix for iCenter Production parts |
| `IPPARTCOPYPREFIX` | `"IAK"` | prefix for IPpart copies (K = kopie) |
| `IPPRNPREFIX` | `"PRN"` | prefix for production reference numbers |
| `DATETIMEFORMAT` | `"yyyy-MM-dd HH:mm:ss.fff"` | canonical timestamp format |
| `GuestEmpId` | `"0000"` | sentinel "no user assigned" employee ID |
| `PdfCreatorExportDir` | `"M:\_PDFs\"` | hard-coded per-user PDF export dir |
| `WorkViewDefaultFromStatusCode` | `"40"` | default work-view lower status bound |
| `WorkViewDefaultTillStatusCode` | `"49"` | default work-view upper status bound |
| `APPSETTINGDOCUMENTPREFIX` | `"APPSETTING:"` | document-prefix marker (Phase-3 follow-up) |

The IPPART/IPPRN prefixes and the 40-49 status range are documented as business rules: [[../business-rules/icenter-part-code-prefixes]] and [[../business-rules/icenter-status-code-default-range]].

## Cultures

```vb
Public Shared ReadOnly Property ApplicationCulture As CultureInfo
    Get
        Return New System.Globalization.CultureInfo("en-US")
    End Get
End Property

Public Shared ReadOnly Property DefaultLangCode As String = "nl-NL"

Public Shared ReadOnly Property WebServiceCulture As CultureInfo
    Get
        Dim CI As New CultureInfo("nl-NL")
        CI.NumberFormat.NumberDecimalSeparator = ","
        CI.NumberFormat.NumberGroupSeparator = "."
        Return CI
    End Get
End Property
```

- **`ApplicationCulture` = en-US** — used for `Double.Parse` / `ToString` *inside the application* (decimals, no thousands separator).
- **`DefaultLangCode` = "nl-NL"** — UI language.
- **`WebServiceCulture` = nl-NL with Dutch number formatting** — used when iCenter calls a web service expecting Dutch number formatting (`,` decimal, `.` thousands).

The dual-culture model is the source of subtle bugs elsewhere — `Double.Parse(value, Common.ApplicationCulture)` is the safe pattern; bare `Double.Parse(value)` (which uses the thread culture, typically Dutch) can flip the meaning of `"1.6"` (1.6 vs 16). Many examples flagged in earlier batches.

## User / computer identity

`GetUsername()` — `Environment.USERNAME`.

`GetUsername(EmpId)` — `ISAH.User.GetUserCodeByEmpId(EmpId)` (or current Windows user if EmpId is blank / `"0000"`).

`GetComputername()` — `MySystem.Environment.HostName`. **Special-cased for terminal servers**:

```vb
If Result.StartsWith("JZTS") OrElse Result.StartsWith("JZRAS") OrElse GetIsAdsServer(Result) Then
    Dim ClientName As String = My.Application.GetEnvironmentVariable("CLIENTNAME")
    If ClientName IsNot Nothing AndAlso ClientName <> "" Then
        Result = ClientName.ToUpper
    End If
End If
```

- `JZTS*` — JAZO Terminal Server
- `JZRAS*` — JAZO RAS server
- `JZADS*` — JAZO Application Delivery Server (recognised by `GetIsAdsServer`)

On these, `CLIENTNAME` (set by the TS / Citrix session) replaces the server name. Without this iCenter would tag every TS-session user with the server's hostname.

`IsSharedWindowsAccount()` — returns `True` if `USERNAME = "PVS"`. **Only `PVS`** is recognised as shared. Q-115. The same `PVS` account hosts the XWiki credentials in `app.config` (Q-015).

`GetSessionName()` — `Environment.SESSIONNAME` wrapped in a try/catch.

## Version reporting

`GetApplicationVersion()`:
1. Try `ApplicationDeployment.CurrentDeployment.CurrentVersion.ToString` (ClickOnce-deployed instance).
2. Fall back to `ICENTERVERSION_MAJOR/MINOR/BUILD/REVISION` env vars, joined as `M.m.B.R`.
3. Else: return `"DEBUG MODE"`.

The env-var fallback is used by build / batch scripts that set the version before launching iCenter.exe outside ClickOnce.

## SQL helpers

| Method | Behaviour |
|--------|-----------|
| `GetSqlInString(CommaSeparatedValue)` | takes `"A,B,C"` → returns `'A','B','C'` (concatenated). **String concat — vulnerable to `'`-injection** if any value contains `'`. Q-114. |
| `SqlParameterValue(Value)` | nulls → `DBNull.Value`, else passes through. Used to bind parameters cleanly. |
| `GetTableData(TableName)` | builds a `SqlConnection` to `Connections.ConnectICenter`, runs `EXEC('SELECT * FROM ' + @TableName)`, returns a `DataTable`. **Server-side SQL string concat** — fine if `TableName` is internal, but `EXEC` always feels sketchy. Q-109. |

## File / process launchers

- `DeleteFile(filepath)`, `DeleteFiles(folder, pattern)`, `DeleteFolder(folder)`, `DeleteFolderIfEmpty`, `CreateDirectory` — silent-success try/catch wrappers.
- `GetSubDir(root, modelname)` → `root\<first-5-chars-of-modelname>` — used to shard model files into per-prefix subfolders.
- `GetTempFileFolder()` → `%TEMP%\iCenter\` (or `%TEMP%\iCenter\<computername>` under RAS).
- `GetLocalTempFileFolder()` → `c:\temp` (or `c:\temp\<computername>` under RAS).
- `OpenParentFolderNewWindow(filepath)` → `explorer.exe /select,<filepath>`.
- `OpenFile(path)` → `Process.Start` with `UseShellExecute = True`.
- `OpenWebPage(url[, useIE])` → if `useIE = True` → `iexplore.exe`. Otherwise spawns the default browser via `UseShellExecute`. Validates URL with `Uri.IsWellFormedUriString`.
- `OpenInNotepadPP(filepath)` → tries `notepad++.exe`, falls back to `notepad.exe`.
- `OpenInChrome(url, maximized, asApp)` → reads chrome path from `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\App Paths\chrome.exe`, launches with `--app=<url>` and optionally `--start-maximized` (commented "But this has no effect…"). Throws if Chrome isn't installed.
- `OpenClickOnceApplication(FullURI)` → `presentationhost.exe -launchapplication <uri>`.

## Type-coercion helpers

`GetAsStringValue(Value)` — culture-aware ToString for Int16/32/64/Double/Decimal/Boolean (booleans as `"1"`/`"0"`).

`GetAsBooleanValue(Value)` — accepts `"1"`, `"TRUE"`, `"YES"`, `"WAAR"` (Dutch true) as true, anything else as false. Nulls → false.

`GetAsIntegerValue(Boolean)` → 1 or 0.

## Other

`IsPcfAdminDossier(DossierCode)` — true if the dossier's `QuotNr` OR `OrdNr` starts with `"PCF.ADM"` (PCF = Production Configuration Form admin?). Reads `ISAH.DossierMain`. Used for permission gates.

`GetBaseModelname(Modelname)` — first 7 characters.

`GetDrawingNrFromObjectName(name)` — strip extension (everything before first `.`).

`UppercaseFirstLetter(val)` — title-case for one char.

`GetBarCodeFontName()` — tries `IDAutomationHC39M`, `IDAHC39M Code 39 Barcode`, `Arial` in order; returns first installed.

`GetWeekYearValue(date, delimiter)` — ISO week + year string.

`GetNextDate(from, dayOfWeek)` — next occurrence of a weekday.

`GetTimeStamp()` → `yyyy-MM-dd HH:mm:ss`.

`GetStatusCodeColor(StatusCode)` — colour mapping for "04"→Tomato, "05"→Turquoise, "06"→Violet, "07"→Purple, "08"→Pink, "09"→Fuchsia. Q-116 — what do these status codes mean to SMEs?

`MessageType` enum: `None=0, Information=5, Warning=10, Critical=15`.

## Surprises

1. **`GetTableData` uses server-side `EXEC` with table-name interpolation.** Safe today (no untrusted callers visible) but the pattern is a smell. Q-109.
2. **`GetSqlInString` string-concatenates values into a SQL fragment** — caller must guarantee no apostrophes. Q-114.
3. The **terminal-server / RAS / ADS branch** in `GetComputername` is essential context for any code that uses `GetComputername()` for per-machine config — it returns the *client* name, not the *server* name.
4. **`OpenWebPage`** still keeps an Internet-Explorer-specific code path. Likely dead-but-defended (IE is end-of-life since 2022). Q-117.
5. **`GetApplicationVersion`'s env-var fallback** is the only way `iCenter -m cadbatchserver` (which isn't ClickOnce-deployed) reports its version. Build scripts must set the four `ICENTERVERSION_*` env vars.
6. **`MessageType` enum values jump (0, 5, 10, 15)** — leaves room for intermediate severities. Most call sites pass a `MsgBoxStyle` instead.

## Business rules surfaced here

- [[../business-rules/icenter-part-code-prefixes|IA/IAK/PRN part-code prefixes]]
- [[../business-rules/icenter-status-code-default-range|Default work-view status range 40-49]]
- `GuestEmpId = "0000"` — sentinel ID used in several forms (e.g. `"0000 Niemand"` in `FrmOrdersAsBuilt`).
- `IsPcfAdminDossier` permission gate: starts-with `"PCF.ADM"` on QuotNr OR OrdNr.

## Open questions

- **Q-109** (from MOC): `GetTableData` server-side EXEC pattern.
- **Q-114** (from MOC): `GetSqlInString` apostrophe vulnerability.
- **Q-115** (from MOC): is `PVS` the only shared Windows account?
- **Q-116 (new):** `GetStatusCodeColor` colour mapping — what do status codes 04..09 mean to SMEs?
- **Q-117 (new):** `OpenWebPage(url, UseInternetExplorer:=True)` is still callable — confirm no live callers (IE end-of-life 2022). `#dead-code` candidate.

Logged in [[../needs-review/_index]].

## Coverage

`_coverage.md`: `ICenterLib (root)\Common.vb` → `done`.
