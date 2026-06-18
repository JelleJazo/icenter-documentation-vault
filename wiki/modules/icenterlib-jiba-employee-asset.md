---
type: module
title: "JIBA Employee + Asset — identity lookups"
status: done
module: "ICenterLib/JIBA"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\JIBA\\Employee.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\JIBA\\Asset.vb"
last-reviewed: 2026-06-18
tags: [module, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# JIBA `Employee` + `Asset` — identity lookups

> **The bridge from a Windows session to a JAZO identity.** `Asset` resolves the computer → asset record (and via `FindMyEmpIdByComputerName`, the auto-assigned employee). `Employee` resolves EmpId / Windows-Username to full name / dept / email.

## `JIBA.Employee`

Three shared functions, all backed by the same `T_Users ⨝ T_Employee` join (filtered to `EmployeeObsInd = 0`):

```vb
Public Enum ReturnField
    FullName, UserName, EmpID, LocalEmailAddress, GenderCode, FirstName, DeptCode, OfficePhone
End Enum

Shared GetUserInfoByEmpId(EmpId, ReturnField)     As String
Shared GetUserInfoByUsername(UserName, ReturnField) As String     ' strips "JAZO\" prefix
Shared GetEmployeeInfoByEmpId(EmpId, ReturnField) As String       ' T_Employee only (no Users join)
```

The **`LocalEmailAddress` return value** is computed as `UserName & "@jazo.local"` — **the JAZO AD-style local mail domain**, hardcoded. Distinct from the public `@jazo.com` used by Servicedesk for FROM-fallback.

### Anti-pattern: column-at-a-time selection

The query SELECTs many columns, then the `Select Case ReturnField` returns only one. Callers wanting multiple columns make N queries. Q-261.

### Error handling

Top-level `Catch ex As Exception` → `Return ""` (silent empty-string). Inner `Try/Catch` in `GetEmployeeInfoByEmpId` pops `MsgBox(ex.ToString, MsgBoxStyle.Critical)` — **headless-incompatible** (Q-219).

## `JIBA.Asset`

```vb
Public Enum AssetField : DeptCode, EmpId : End Enum

Shared GetAssetIdByMacAddress(MacAddress As String)           As Long      ' AssetActive = 1
Function GetInfo(ReturnField As AssetField)                   As String
Shared GetAssetInfoByComputername(ComputerName, ReturnField)  As String    ' WHERE Name = @Name
Shared GetAssetRecord(Computername)                           As DataTable ' SELECT *
Shared UpdateAssetWebClockEnable(AssetId, Value As Boolean)
Shared FindMyEmpIdByComputerName(ComputerName)                As String    ' SP SIP_Get_AssetByName
```

### `FindMyEmpIdByComputerName(ComputerName)`

Strips `.jazo.local` suffix from FQDN, then runs SP `SIP_Get_AssetByName(@Name)` to resolve the assigned EmpId. **The JAZO standard for "which employee owns this asset"** — used by auto-login flows. Pops `MsgBox` on error.

### `GetAssetIdByMacAddress(MacAddress)`

`SELECT AssetId FROM T_Asset WHERE MacAddress=@MacAddress AND AssetActive = 1`. The MAC-address-based identification path — used when the host has multiple identity options (e.g., remote-desktop sessions).

### `UpdateAssetWebClockEnable(AssetId, Value)`

`UPDATE T_Asset SET WebclockEnabled=@WebclockEnabled WHERE AssetId=@AssetId`. The boolean toggle for **whether this workstation may show the WebClock widget** — admin-only setting.

## Surprises

1. **MsgBox in data-layer** (Q-219 reappears).
2. **`Common.GetComputername` / FQDN handling**: `FindMyEmpIdByComputerName` explicitly strips `.jazo.local`; `GetAssetInfoByComputername` does NOT — caller must trim. Inconsistent. Q-263.
3. **`AssetActive`** column filter is on `GetAssetIdByMacAddress` only — other methods return any row regardless of active status. If a decommissioned asset is queried by name, it returns. Q-264.
4. **`UserName` interpolation in Email**: `UserName & "@jazo.local"`. If UserName contains a space, `@`, or `;`, the result is malformed. UserName must be a clean AD sAMAccountName.

## Business rules

- [[../business-rules/jiba-local-email-domain|`@jazo.local`]] — defining file.
- **`JAZO\` Windows-domain prefix** stripped from usernames before lookup.
- **`SIP_Get_AssetByName`** = the canonical asset-to-employee resolver.
- **`AssetActive` only checked by MAC-lookup** — name/AssetId lookups return inactive assets.

## Open questions

- **Q-261 (existing):** Consolidate Employee.GetUserInfoBy* methods.
- **Q-262 (existing):** Hardcoded `JAZO\` domain prefix.
- **Q-263 (new):** FQDN handling inconsistent (`.jazo.local` stripped in one method, not others).
- **Q-264 (new):** `AssetActive=1` filter only on MAC-lookup. Name lookups can return deactivated assets.

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/icenterlib-jiba]] — parent.
- [[icenterlib-icenter-identification|FrmIdentification]] — calls `Asset.GetAssetInfoByComputername` to pick default dept.
- [[icenterlib-icenter-leaves|Servicedesk]] — uses `Employee.GetUserInfoByEmpId(..., LocalEmailAddress)`.

## Coverage

- `JIBA\Employee.vb` → `done`
- `JIBA\Asset.vb` → `done`
