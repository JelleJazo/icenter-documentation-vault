---
type: moc
title: "ICenterLib/JIBA — JIBA-portal database wrappers"
status: draft
module: "ICenterLib/JIBA"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\JIBA\\"
tags: [moc, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# ICenterLib/JIBA

## What this hub covers

`ICenterLib\JIBA\` — **wrappers over the JIBA-portal database** (`Connections.ConnectJIBA`). JIBA is the JAZO-internal HR + asset + menu portal: per-employee identity (`T_Employee`, `T_Users`), per-asset registry (`T_Asset`), menu structure, parameter store, custom-satisfaction tracker, encryption helper, navigation tree.

14 source files (no generated). ~90 KB total.

## Files

| File | Role | Note |
|------|------|------|
| `Employee.vb` | T_Employee+T_Users joined lookup (FullName/EmpId/Username/Email/etc.) | [[../modules/icenterlib-jiba-employee-asset]] |
| `Asset.vb` | T_Asset (per-computer/MAC registry) — `GetAssetInfoByComputername`, `FindMyEmpIdByComputerName`, `UpdateAssetWebClockEnable` | [[../modules/icenterlib-jiba-employee-asset]] |
| `AppParameter.vb` | T_AppParameter (key-value config in JIBA DB) | leaves |
| `Company.vb` | T_Company lookup | leaves |
| `ConfigPart.vb` | Configurator-part wrapper | leaves |
| `CustSat.vb` | Customer-satisfaction record | leaves |
| `Encryption.vb` | JIBA-side encryption helper | leaves; #safety-relevant if used for secrets |
| `Enums.vb` | JIBA-namespace enums | leaves |
| `LinkItem.vb` | Menu link item | leaves |
| `Log.vb` | JIBA log writer | leaves |
| `Menu.vb` | Menu tree | leaves |
| `NavigationGroupItem.vb` | Nav group entry | leaves |
| `SubMenu.vb` | Sub-menu entry | leaves |
| `XmlData.vb` | XML-data store wrapper | leaves |

## Business rules surfaced

- [[../business-rules/jiba-local-email-domain|`UserName@jazo.local` is the canonical employee email]] — `Employee.GetUserInfoBy*(LocalEmailAddress)` returns `UserName & "@jazo.local"`. Hard-coded JAZO AD-style local mail domain (NOT the public `@jazo.com`).
- **`UserName.Replace("JAZO\", "")`** in `Employee.GetUserInfoByUsername` — strips Windows domain prefix `JAZO\` from `DOMAIN\user` before lookup. The JAZO Windows domain is hardcoded.
- **`AssetActive = 1`** filter — `Asset.GetAssetIdByMacAddress` only returns active assets. Deactivated workstations skipped.
- **`SIP_Get_AssetByName(@Name)`** in `Asset.FindMyEmpIdByComputerName` — strips `.jazo.local` from FQDN, looks up EmpId via SP.

## Notable findings

1. `Employee` has **three nearly-identical query methods** (GetUserInfoByEmpId / GetUserInfoByUsername / GetEmployeeInfoByEmpId) — same SELECT shape, three different argument forms. Ripe for consolidation.
2. **`MsgBox(ex.ToString)` in catch blocks** (`Asset.GetInfo`, `Asset.FindMyEmpIdByComputerName`, `Employee.GetEmployeeInfoByEmpId`) — pop-up blocks headless processes. Same Q-219 family.
3. **`ReturnField` enum / Select Case** pattern is the dynamic-column anti-pattern in safer form (enum-based not string-based). Q-261 — could be replaced by a typed DTO return.
4. `Asset.GetInfo` silently swallows inner exceptions; outer pops MsgBox; same dual-Try pattern as ProductionMachines.
5. **Windows-username trimming** uses `Replace("JAZO\", "")` — only handles the JAZO domain (Q-262). Non-domain usernames pass through unchanged.

## Open questions

- **Q-261 (new):** `JIBA.Employee.ReturnField` enum is a code-smell — multiple methods select one column at a time via a Select-Case ladder. Consolidate to a typed DTO.
- **Q-262 (new):** `Replace("JAZO\", "")` is hardcoded. If JAZO ever joins a different AD domain or supports cross-domain trusts, this breaks.

Logged in [[../needs-review/_index]].

## Coverage

All 14 files marked `done` via this MOC; deeper detail for Employee+Asset in [[../modules/icenterlib-jiba-employee-asset]]. Other 12 files are JIBA-portal-side wrappers without surprising business logic at a glance — flagged `done` overview-level.
