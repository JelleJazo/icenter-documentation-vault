---
type: business-rule
title: "Employee local email = UserName@jazo.local"
status: needs-review
module: "ICenterLib/JIBA"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\JIBA\\Employee.vb"
last-reviewed: ""
tags: [business-rule, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Employee local email = `{UserName}@jazo.local`

## Rule

When iCenter asks JIBA "what is this employee's local email?", the answer is **constructed at query time** as `UserName & "@jazo.local"` rather than read from a column. `@jazo.local` is JAZO's internal Active Directory / Exchange domain (not the public `@jazo.com`).

## Where it lives

- File: `ICenterLib\JIBA\Employee.vb` lines 39-40 and 83-84
- Symbol: `JIBA.Employee.GetUserInfoByEmpId(EmpId, ReturnField.LocalEmailAddress)` and `GetUserInfoByUsername(..., ReturnField.LocalEmailAddress)`

## The code

```vb
Case ReturnField.LocalEmailAddress
    ReturnValue = SqlRdr("UserName").ToString.Trim & "@jazo.local"
```

## Why this is a business rule

The result of `LocalEmailAddress` lookups feeds:

- **Servicedesk** ticket-creation URL parameters — see Q-243 path (the fallback when this comes back empty is `pvs@jazo.com`).
- Anywhere iCenter needs to address an employee internally (likely also report distribution).

The constructed nature means that:

- A username change in AD without a database update would silently misroute mail.
- `T_Users.UserName` must be the **sAMAccountName** (not UPN, not "DisplayName") for the local mail to resolve.
- An employee with `UserName` containing a space or special characters produces an invalid email — JAZO must enforce sAMAccountName-style usernames.

## Triggers / when it fires

- Anywhere `JIBA.Employee.GetUserInfoBy*(EmpId|UserName, ReturnField.LocalEmailAddress)` is called.

## Effects

- Returns `<sAMAccountName>@jazo.local`.

## Edge cases

- If `T_Users.UserName` is empty/NULL: returns `"@jazo.local"` (just the suffix).
- The string is **never validated** against the local Exchange directory — a stale username silently produces a wrong-but-syntactically-valid address.
- Distinct from the public-facing `@jazo.com` literal used in [[icenter-servicedesk-fallback-email|`Servicedesk.GetNewTicketUrl`]]. Two domains exist.

## Safety classification

- [ ] Touches physical process — no.
- [ ] Drives cost / pricing — no.
- [x] Reversible if wrong? — yes (re-route, re-issue).
- [ ] Blocks production if it fails? — no, just email-delivery soft failure.

## SME questions

- **Q-265 (new):** Should `LocalEmailAddress` be read from a column rather than constructed? Avoids drift if an employee's local email differs from the sAMAccountName-derived address.
- **Q-266 (new):** Document the `@jazo.local` vs `@jazo.com` split — which is internal-only, which is customer-facing.

Logged in [[../needs-review/_index]].

## Related

- [[../modules/icenterlib-jiba-employee-asset|`JIBA.Employee`]] — defining module.
- [[icenter-servicedesk-fallback-email|Servicedesk pvs@jazo.com fallback]] — the public-domain counterpart.
