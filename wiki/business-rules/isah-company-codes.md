---
type: business-rule
title: "JAZO vs FlowGrill company codes"
status: needs-review
module: "ICenterLib/ISAH"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\Company.vb"
last-reviewed: ""
tags: [business-rule, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# JAZO vs FlowGrill — company codes

## Rule

JAZO operates as **two legal entities** at the ERP level, each identified by:

| Entity | `AdminCode` (T_Admin) | `SelectionCode` (T_Selection) | Default? |
|--------|-----------------------|--------------------------------|----------|
| **JAZO** | `"JAZO"` | `"041"` | yes |
| **FlowGrill** | `"FLOWGRIL"` (note: no `L` at end) | `"042"` | no |

Both codes are **hard-coded** in `ISAH\Company.vb` as `Public Sub New(CompanyName)` mappings + three `CreateByXxx` factory functions. Adding a third entity (or renaming either) requires editing this file in 5 places (enum + two constructors + three factories + `GetCompanyName`).

## Where it lives

- File: `ICenterLib\ISAH\Company.vb` lines 7–80
- Enum: `Public Enum CompanyName { FlowGrill = 1, JAZO = 2 }`

## The code (minimal quote)

```vb
Public Sub New(CompanyName As CompanyName)
    Select Case CompanyName
        Case CompanyName.FlowGrill
            AdminCode     = "FLOWGRIL"
            SelectionCode = "042"
        Case Else
            AdminCode     = "JAZO"
            SelectionCode = "041"
    End Select
End Sub

Public Shared Function CreateBySelectionCode(SelectionCode As String) As Company
    Select Case SelectionCode
        Case "042" : Return New Company(CompanyName.FlowGrill)
        Case "041" : Return New Company(CompanyName.JAZO)
        Case Else  : Return Nothing
    End Select
End Function
```

## Why this is a business rule

The split between JAZO and FlowGrill drives **financial accounting** (which legal entity owns the invoice / cost / inventory) and many workflow paths in iCenter. Examples from prior batches:

- [[../modules/engineering-overview|`FrmOrdersAsBuilt`]] reads the current user's company via `Employee.GetCompany()` and falls back to `Company.JAZO` if none is set (line 38 of `FrmOrdersAsBuilt.vb`).
- The `MultiFinance.GetAdminCodeByOrdType(OrdType)` lookup (called from `Company.CreateByOrdType`) ties order types to companies.
- Outsourcing workflows, PurDoc creation, ShopDoc routing all depend on which entity owns the work.

`CreateByXxx` factories returning `Nothing` for unknown codes is the **fail-closed** safeguard: a corrupted `OrdType` or a manual data-entry typo doesn't silently get assigned to a wrong company.

## Triggers / when it fires

- Anywhere code constructs a `Company` from one of:
  - a known `OrdType` (via `MultiFinance.GetAdminCodeByOrdType`)
  - an `AdminCode` string (`"JAZO"` or `"FLOWGRIL"`)
  - a `SelectionCode` string (`"041"` or `"042"`)
  - the `CompanyName` enum directly
- Defaults to `JAZO` from the `New(CompanyName)` constructor's `Case Else` branch — meaning `New Company(CompanyName.Undefined)` (or any unrecognised enum value) silently becomes JAZO. Q-146.

## Effects

- Controls every downstream workflow that consults `Company.AdminCode` or `Company.SelectionCode`.
- The `CompanyName` enum value is used by callers that branch on entity (`Select Case Company.GetCompanyName()`).

## Edge cases / known exceptions

- **`Case Else` defaults to JAZO** in `New(CompanyName)` — silent. A new `CompanyName.Acme=3` enum entry would silently behave like JAZO. Q-146.
- **`CreateByXxx` returns `Nothing`** for unknown codes — callers must null-check. Several do (`FrmOrdersAsBuilt.vb` line 37 has the fallback `New Company(CompanyName.JAZO)`).
- **Renaming a code in ISAH** (e.g. changing `"FLOWGRIL"` to `"FLOWGRILL"` in `T_Admin`) breaks every consumer here — `CreateByAdminCode("FLOWGRILL")` would return `Nothing`. Coordination required.
- The `"FLOWGRIL"` spelling (without final `L`) is intentional per ISAH's `T_Admin.AdminCode` 8-char limit. Easy to miss.

## Safety classification

- [ ] Touches physical process — no.
- [x] Drives financial / accounting routing → `#needs-review` (data-correctness concern).
- [ ] Reversible if wrong? — Yes but auditable.
- [ ] Blocks production if it fails? — Indirectly (wrong company on an order causes invoicing problems, not shop-floor stoppage).

## SME questions

- **Q-146 (new):** `Company.New(CompanyName)` defaults to JAZO via `Case Else`. Should an unknown enum throw instead?
- **Q-140** (from MOC): if a third legal entity is ever added, this becomes a hard-coded sprawl point. Drive from `T_Selection` instead?

Logged in [[../needs-review/_index]].

## Related

- [[../modules/isah-identity|`ISAH.Company`]] — defining module.
- [[../mocs/icenterlib-isah]] — parent MOC.
- [[sales-team-codes]] — sibling rule: hard-coded codes in `T_Selection`.
