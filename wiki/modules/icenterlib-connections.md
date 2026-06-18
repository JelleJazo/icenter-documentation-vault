---
type: module
title: "ICenterLib\\Connections.vb — DB and HTTP connection factories"
status: done
module: "ICenterLib/(root)"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\Connections.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `ICenterLib\Connections.vb` — DB + HTTP connection factories

## Purpose

A `NotInheritable` static class that **owns every outbound connection-string in iCenter**. 12 SQL database factories + 4 HTTP-client factories + 1 SMTP factory + 1 zero-code parameters bag. The `Connections.ConnectXxx()` pattern is the universal entry-point used everywhere else in iCenter/ICenterLib for cross-system calls.

> **`#safety-relevant`.** Every connection method carries its credentials as a hard-coded literal. **No secret store, no env-var overrides, no Windows-auth fallback.** Auditing this file is the single biggest security finding so far in the wiki. Q-107.

## Public surface

```vb
Public NotInheritable Class Connections
    Public Shared UseIsahTestDb As Boolean = False         ' runtime-mutable test/prod toggle (Q-110)
    Private Const UseSql2 As Boolean = False

    ' Migration constants
    Public Const UseIsahHtml As Boolean = True
    Public Const UseIsahNoDtrTimeReg As Boolean = True

    Public Shared ReadOnly Property SqlServerHostName As String      ' JGSDS or JZSQL02
    Public Shared ReadOnly Property IsahDataSource As String         ' JGISAH\ISAHSQLSERVER or JGISAH02\ISAHSQLSERVER

    ' SQL connection factories
    Public Shared Function ConnectJIBA()                  As SqlConnection
    Public Shared Function ConnectJIBAAsJibaUser()        As SqlConnection
    Public Shared Function ConnectICenter()               As SqlConnection
    Public Shared Function ConnectICenter2(DataBaseName)  As SqlConnection
    Public Shared Function ConnectICenter2Development(...)As SqlConnection
    Public Shared Function ConnectWindchill(HostName)     As SqlConnection
    Public Shared Function ConnectProductDb()             As SqlConnection
    Public Shared Function ConnectProductDbAsProductDb()  As SqlConnection
    Public Shared Function ConnectIsah()                  As SqlConnection
    Public Shared Function ConnectIsahSA()                As SqlConnection
    Public Shared Function ConnectTruTopsOseon(ctx)       As SqlConnection
    Public Shared Function ConnectKardex()                As SqlConnection

    ' SMTP
    Public Shared Function ConnectMX()                    As SmtpClient

    ' HTTP clients (URLs from T_ApplicationSettings)
    Public Shared Function GetProdexHttpClient()          As HttpClient
    Public Shared Function GetIsahCustomisingHttpClient() As HttpClient
    Public Shared Function GetElfsquadHttpClient()        As HttpClient
    Public Shared Function GetCrystalReportHttpClient()   As HttpClient

    Public Shared Function GetZeroCodeParameters()        As Dictionary(Of String, Object)
End Class
```

## SQL hosts

| Property | Value (`False` branch) | Value (`True` branch) |
|----------|------------------------|------------------------|
| `SqlServerHostName` | `"JGSDS"` (when `UseSql2 = False`, the default) | `"JZSQL02"` |
| `IsahDataSource` | `"JGISAH\ISAHSQLSERVER"` (when `UseIsahTestDb = False`) | `"JGISAH02\ISAHSQLSERVER"` |

- `UseSql2` is a `Private Const` — toggle requires a recompile.
- `UseIsahTestDb` is a **`Public Shared` field** — runtime-mutable. Set by debug code; **never expected to be `True` in production**. Q-110.

Other notable hostnames:
- `ConnectICenter2Development` connects to **`10.11.70.32`** directly (raw IP, not hostname). Likely a developer machine. Q-118.
- `ConnectMX` connects to `mail.jazo.nl` port `10025` (was `587`, changed CB 2020-04-06).
- `ConnectKardex` connects to `<SqlServerHostName>` → `PowerPick` database.
- `ConnectTruTopsOseon(ctx)` reads `DatabaseFullName` + `InitialCatalog` from the `OseonAppContext` parameter — only connection method that's context-driven.
- `ConnectWindchill(hostname)` accepts a hostname parameter (Windchill server isn't `JGSDS`).

## Hard-coded credentials (per Q-107)

| Method | User | Password | DB / Service |
|--------|------|----------|--------------|
| `ConnectJIBA` | `iCenter` | `p2yeXeC7` | `JIBA` |
| `ConnectJIBAAsJibaUser` | `JibaUser` | `gQny4prh` | `JIBA` |
| `ConnectICenter` | `icenter` | `p2yeXeC7` | `iCenter` |
| `ConnectICenter2` | `iCenter2User` | `j2hvTEJ1tunz1` | `<param>` (iCenter2 family) |
| `ConnectICenter2Development` | `sander-h` | `UFVSaBfh9oy4ydsNqhsR` | `<param>` (dev IP `10.11.70.32`) |
| `ConnectWindchill` | `icenter` | `p2yeXeC7` | `wtuser` |
| `ConnectProductDb` | `icenter` | `p2yeXeC7` | `ProductDb` |
| `ConnectProductDbAsProductDb` | `ProductDb` | `Pr0ductDb` | `ProductDb` |
| `ConnectIsah` | `icenter` | `p2yeXeC7` | `JazoZevenaarDb` (or `TestUpdate` if `UseIsahTestDb`) |
| `ConnectIsahSA` | **`sa`** | (test: `p2yeXeC7`; **prod: `koyTRedgh&*(kl:[`**) | `JazoZevenaarDb` (or `Test47`) |
| `ConnectTruTopsOseon` | `Icenter` | `p2yeXeC7` | `<ctx-driven>` |
| `ConnectKardex` | `Kardex` | `K@rdex951` | `PowerPick` |
| `ConnectMX` | `JAZO\JazoApp` | `p#4Th@s1wa` | `mail.jazo.nl:10025` |

Three observations:
1. The same password `p2yeXeC7` is shared across iCenter, JIBA, Windchill, ProductDb, ISAH, and TruTops Oseon. If any of these databases is compromised, all are.
2. The `sa` connection to ISAH (`ConnectIsahSA`) is a database-administrator login. Used by methods that need elevated privileges. Q-119.
3. **All connections set `TrustServerCertificate = True`** (where TLS is in scope) — disables certificate-chain validation. Mostly fine because the SQL servers are on the JAZO LAN, but worth flagging.

## HTTP client factories

All read base URLs from `T_ApplicationSettings` (via `AppSettings.GetStringSetting(<key>)`):

| Method | Setting key | Notes |
|--------|-------------|-------|
| `GetProdexHttpClient` | `ProdexApiBaseUrl` | commented dev fallback `http://localhost:44398/api/` |
| `GetIsahCustomisingHttpClient` | `IsahCustomisingApiBaseUrl` | |
| `GetElfsquadHttpClient` | `ElfsquadApiBaseUrl` | commented dev fallback `http://localhost:5184/api/` |
| `GetCrystalReportHttpClient` | `ElfsquadApiBaseUrl` (sic — read but **overwritten** with hard-coded `https://crystalreportapi.jazo.com/api/`) | Q-120 |

`GetCrystalReportHttpClient` reads `ElfsquadApiBaseUrl` and then **overwrites** it with the Crystal-Reports URL. The read is dead code; the hard-coded URL is the live value. Q-120.

`GetZeroCodeParameters()` returns a `Dictionary(Of String, Object)` with **three hard-coded credentials** (API key, username, base64-encoded password) for a "ZeroCode" service. The commented-out `If Not Value.ContainsKey(...)` block (lines 213–223) shows an earlier conditional-add pattern that was replaced with always-add. Q-108.

## Behavior in plain language

Each `ConnectXxx()` method:
1. Constructs a new `SqlConnection`.
2. Checks `SC.State = ConnectionState.Open` (always `False` for a fresh connection — defensive only).
3. Sets `ConnectionString` from hard-coded literals + the host/datasource property.
4. Calls `SC.Open()`.
5. Returns the open connection.

**Callers own the disposal.** None of the factory methods wrap in `Using`. Connection leaks are entirely the caller's responsibility.

## Surprises

1. **The `If Not SC.State = ConnectionState.Open` guard is meaningless** for a freshly-`New`ed connection — `State` is always `Closed` until `Open` succeeds. Possibly copy-paste from a connection-pool wrapper that no longer exists.
2. **No connection pooling at this layer** — every call to `ConnectICenter()` etc. opens a new SQL connection. .NET pools at the driver level so this is fine, but profiling has to account for the `Open` cost.
3. **Connection timeout is `30`** seconds for every SQL connection. Identical, hard-coded.
4. **`ConnectICenter` has a commented alternate** (`Initial Catalog=iCenterTest`) — easy switch but requires source edit. Q-121.
5. **`ConnectIsahSA` test branch uses `TestUpdate` *and* `Test47` catalogs** depending on which method you read (`ConnectIsah` uses `TestUpdate`, `ConnectIsahSA` uses `Test47` in test mode). The two test catalogs diverge — Phase-3 follow-up on what each represents.
6. **No connection method wraps the open in try/catch** — open failures bubble straight to the caller. Most callers don't handle this, so a temporarily-unreachable DB → MsgBox in the user's face (or worse, in `oICENTER`'s field-init at process startup).
7. The trivia `Public Const UseIsahHtml = True` and `UseIsahNoDtrTimeReg = True` (lines 11–12) are **migration flags** noted with the comment `'Settings for Isah migration`. They appear to be vestigial after the migration completed but are still referenced by methods elsewhere (Phase-3 follow-up).

## Open questions

- **Q-107** (from MOC): hard-coded DB credentials throughout. `#safety-relevant`
- **Q-108** (from MOC): `GetZeroCodeParameters` hard-coded API key + password. `#safety-relevant`
- **Q-110** (from MOC): `UseIsahTestDb` is process-mutable. Confirm production never sets it.
- **Q-118 (new):** `ConnectICenter2Development` connects to raw IP `10.11.70.32` as user `sander-h`. Developer machine? Remove from production builds? `#safety-relevant`
- **Q-119 (new):** `ConnectIsahSA` uses ISAH's `sa` login. Which iCenter operations actually require sa privileges? Can they be moved to a least-privilege user? `#safety-relevant`
- **Q-120 (new):** `GetCrystalReportHttpClient` reads `ElfsquadApiBaseUrl` then overwrites it with a hard-coded URL. Remove the dead read. `#needs-review`
- **Q-121 (new):** `ConnectICenter` has a commented-out alternate connection string for `iCenterTest`. Document the iCenterTest catalog's purpose. `#needs-review`

Logged in [[../needs-review/_index]].

## Coverage

`_coverage.md`: `ICenterLib (root)\Connections.vb` → `done`.
