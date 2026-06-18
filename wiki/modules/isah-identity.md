---
type: module
title: "ISAH identity — Employee, User, Customer, Vendor, Company"
status: done
module: "ICenterLib/ISAH"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\Employee.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\User.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\Customer.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\Vendor.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\Company.vb"
last-reviewed: 2026-06-18
tags: [module, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# ISAH identity — `Employee` / `User` / `Customer` / `Vendor` / `Company`

## Purpose

Five small wrapper classes around ISAH's identity-bearing entities. Together they answer "who is the human / who is the organisation behind this row?" for every other ISAH entity.

| Class | Identifier | Wraps |
|-------|------------|-------|
| `Employee` | `EmpId` (string) | `T_Employee` + time-registration helpers via `T_TimeRegistration` |
| `User` | (none — all static) | `T_UserRegistration` + `T_UserRegistrationInfo` joins, plus `T_MemoDetail` for job descriptions |
| `Customer` | not deep-read in this batch | `T_Customer` |
| `Vendor` | not deep-read in this batch | `T_Vendor` |
| `Company` | `AdminCode` + `SelectionCode` | hard-coded JAZO/FlowGrill — no SQL |

## `Employee`

**Source**: `ISAH\Employee.vb` (17 KB)

```vb
Public Class Employee
    Public ReadOnly Property EmpId As String

    Public Sub New(EmpId As String)

    Public Enum GenderCode { Undefined=0, Male=1, Female=2, Neutral=3 }

    Public Function GetIsPresent()           As Boolean
    Public Function GetIsObsolete()          As Boolean
    Public Function GetStatus()              As DataTable      ' SP: JIP_Get_EmpStatus_v2
    Public Function GetCurrentTimeReg()      As DataTable      ' SP: JIP_Get_EmpCurrentTimeReg
    Public Function GetCurrentTimeRegBasic() As DataTable      ' inline SQL, branches on UseIsahNoDtrTimeReg
    ' ... more methods beyond the first 120 lines (Phase-3 follow-up)
End Class
```

### Key behaviours

- **`GetIsPresent()`** checks whether the employee has an open `AW` ("aanwezig" = present) time-registration row for today's `RegDate`. Branches on `Connections.UseIsahNoDtrTimeReg`:
  - **`True` (current production behaviour)**: query checks `Starttime <> 0 AND EndTime = 0` — no DTR status involved.
  - **`False` (legacy)**: query checks `DTRStatusCode = 'AO'`. Dead path. Q-132.
- **`GetIsObsolete()`** returns `True` on any error or no-rows. **Fail-closed** — transient ISAH outage flags every employee obsolete. Q-133.
- **`GetStatus()`** runs SP `JIP_Get_EmpStatus_v2` (JAZO-custom SP).
- **`GetCurrentTimeReg()`** runs SP `JIP_Get_EmpCurrentTimeReg` via `DataHandler.GenericQuery`.
- **`GetCurrentTimeRegBasic()`** has parallel SQL like `GetIsPresent` — branches on `UseIsahNoDtrTimeReg`.
- The `GenderCode` enum is declared but not visibly used in the first 120 lines.

## `User`

**Source**: `ISAH\User.vb` (7 KB) — all `Public Shared` (no instance constructor).

```vb
Public Class User
    Public Shared Function GetUserCodeByEmpId(EmpId As String)        As String
    Public Shared Function GetEmpIdByUserCode(UserCode As String)     As String
    Public Shared Function GetEmpIdByWindowsLogin(WindowsLogin As String) As String
    Public Shared Function GetJobDescription(EmpId As String)         As String
    Private Shared Function GetRawJobDescription(EmpId As String)     As String
End Class
```

### Key behaviours

- **Three bidirectional accessors** between EmpId, UserCode, and WindowsLogin via joins on `T_UserRegistration ⨝ T_UserRegistrationInfo`. The `WindowsLogin` column is the link between AD-authenticated Windows users and their ISAH identities.
- **`GetJobDescription(EmpId)`** reads `T_MemoDetail` filtered on **`MemoTypeCode = 'JEM10' AND LangCode = 'NL'`** (hard-coded Dutch-only). Strips HTML/RTF via `PlainTextHelper`. Q-134.
- **No `New User(...)` constructor.** The class is purely static; identity is passed by argument.

## `Customer`

**Source**: `ISAH\Customer.vb` (5 KB) — not deep-read in this batch. Visible from [[../modules/sales-customer-team|`Sales\FrmCustomerTeam`]]: `ISAH.Customer.GetByOrdConfDate(filter, till, from)` is a Customer-side query.

Phase-3 follow-up.

## `Vendor`

**Source**: `ISAH\Vendor.vb` (1 KB) — not deep-read. Used by `OutsourceOperationsHandler` (VendId column in the outsource pipeline) but as a string identifier; the wrapper is trivially small. Phase-3 follow-up.

## `Company`

**Source**: `ISAH\Company.vb` (3 KB) — **no SQL**, pure model.

```vb
Public Class Company
    Public ReadOnly Property AdminCode     As String
    Public ReadOnly Property SelectionCode As String

    Public Enum CompanyName { FlowGrill=1, JAZO=2 }

    Public Sub New(AdminCode As String, SelectionCode As String)
    Public Sub New(CompanyName As CompanyName)

    Public Shared Function CreateByOrdType(OrdType As String)         As Company
    Public Shared Function CreateByAdminCode(AdminCode As String)     As Company
    Public Shared Function CreateBySelectionCode(SelectionCode As String) As Company

    Public Function GetCompanyName() As CompanyName
End Class
```

Encodes the two JAZO legal entities:

| Company | `AdminCode` | `SelectionCode` |
|---------|-------------|------------------|
| JAZO (default) | `"JAZO"` | `"041"` |
| FlowGrill | `"FLOWGRIL"` | `"042"` |

All three `CreateByXxx` factories return `Nothing` for unknown codes (fail-closed). `CreateByOrdType(OrdType)` delegates to `MultiFinance.GetAdminCodeByOrdType(OrdType)` to resolve the AdminCode first.

Documented as [[../business-rules/isah-company-codes|JAZO vs FlowGrill company codes]].

## Surprises

1. **`Employee.GetIsObsolete`** is fail-closed. A transient ISAH outage during e.g. `FrmOrdersAsBuilt.New()` would filter out every engineer from the assignment combo.
2. **`Employee.GetIsPresent`** uses `DateTime.Now.ToString("yyyyMMdd")` for `RegDate` — assumes the iCenter machine and the ISAH machine agree on date. Time-zone drift would skip an employee's clock-in.
3. **`User` is purely static** while `Employee` is instance-based — asymmetric. A `New User(...)` constructor would let several queries share a UserCode lookup. Not a bug, just a style inconsistency.
4. **`User.GetEmpIdByWindowsLogin`** is the iCenter-side AD ↔ ISAH bridge. Used at iCenter startup (Phase-3 follow-up to find the callsite) to identify the current user as an ISAH employee.
5. **`Company` is hard-coded.** Adding a third legal entity requires editing `Company.vb` everywhere (enum + two constructors + three factories + GetCompanyName). Q-140.
6. **`MemoTypeCode = 'JEM10'`** in `User.GetJobDescription` is undocumented (Q-134). The `JEM` prefix likely stands for "JAZO Employee Memo" but unconfirmed.
7. **All connection objects opened directly via `Connections.ConnectIsah`** — none of these classes use `DataHandler.GenericQuery` consistently. Modernisation candidate.

## Business rules surfaced here

- [[../business-rules/isah-company-codes|JAZO vs FlowGrill company codes]]
- `MemoTypeCode = 'JEM10'` for employee job descriptions, Dutch-only — Q-134 to document.
- `Employee` "present" semantics: open `AW` time-registration row for today.
- `Employee` "obsolete" semantics: fail-closed.

## Open questions

- **Q-132** (from MOC): remove dead DTR-status branches in `Employee.GetIsPresent` and `GetCurrentTimeRegBasic`?
- **Q-133** (from MOC): `Employee.GetIsObsolete` fail-closed — intentional?
- **Q-134** (from MOC): document `MemoTypeCode = 'JEM10'`.
- **Q-140 (new):** Adding a third JAZO company requires editing `Company.vb` in 5 places. Drive from `T_Selection` instead?
- **Q-141 (new):** `Employee.GetIsPresent` uses local `DateTime.Now` for `RegDate`. Time-zone-safe?

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/icenterlib-isah]] — parent MOC.
- [[isah-lookups]] — Selection (used by Company indirectly through 041/042 codes).
- [[isah-shop-and-pur-doc]] — uses Employee for time-tracking.
- [[../business-rules/isah-company-codes]] — companion rule note.

## Coverage

`_coverage.md`:
- `ICenterLib\ISAH\Employee.vb` → `done` (overview level; full method enumeration deferred)
- `ICenterLib\ISAH\User.vb` → `done`
- `ICenterLib\ISAH\Customer.vb` → `needs-review` (not deep-read; used by Sales — Phase-3 follow-up)
- `ICenterLib\ISAH\Vendor.vb` → `needs-review` (small; Phase-3 follow-up)
- `ICenterLib\ISAH\Company.vb` → `done`
