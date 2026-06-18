---
type: business-rule
title: "Servicedesk email fallback to pvs@jazo.com"
status: needs-review
module: "ICenterLib/iCenter"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\Servicedesk.vb"
last-reviewed: ""
tags: [business-rule, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Servicedesk email fallback — `pvs@jazo.com`

## Rule

When the iCenter Servicedesk widget builds a "new ticket" URL parameter set, the email address parameter falls back to the **hardcoded literal `pvs@jazo.com`** if the user does not have a personal email address. The four trigger conditions are:

1. EmpId equals the guest sentinel (`Common.GuestEmpId`).
2. The employee is a machine-emp id (`Employee.IsMachineEmpId`).
3. `LocalEmailAddress` is `Nothing`.
4. `LocalEmailAddress.Trim = ""`.

Any of these four → `Email = "pvs@jazo.com"`. Otherwise, the operator's `LocalEmailAddress` is used.

## Where it lives

- File: `ICenterLib\iCenter\Servicedesk.vb` lines 44-65
- Symbol: `Servicedesk.GetNewTicketUrl(EmpId)`

## The code (verbatim)

```vb
Dim LMA As String = JIBA.Employee.GetUserInfoByEmpId(EmpId, JIBA.Employee.ReturnField.LocalEmailAddress)
...
If EmpId = Common.GuestEmpId OrElse Employee.IsMachineEmpId OrElse LMA Is Nothing OrElse LMA.Trim = "" Then
    Email = "pvs@jazo.com"
Else
    Email = LMA
End If
```

## Why this is a business rule

- **Guest** (operator logged in as guest, no personal account) — there's no "from" to attribute a Zammad ticket to, so use a catch-all inbox.
- **Machine emp id** — workstation-owner sentinels representing the station, not a person; same reasoning.
- **Missing/empty `LocalEmailAddress`** — most operators don't have a JAZO email (they aren't office staff). Their tickets get routed via the catch-all.

The hardcoded literal makes `pvs@jazo.com` the **default servicedesk requester** for the vast majority of shop-floor-originated tickets. Whoever owns that inbox effectively triages all anonymous shop-floor tickets.

## Triggers / when it fires

- Anywhere the Servicedesk widget builds a "new ticket" URL — likely the Servicedesk menu entry from the main iCenter UI.

## Effects

- `Email` query parameter on the Zammad new-ticket URL gets `pvs@jazo.com`.
- Zammad receives the ticket attributed to that address. Anyone watching `pvs@jazo.com` triages.
- The URL ALSO carries `empid=...&name=...&computername=...` so the real identity is recoverable from the ticket — just not as the requester.

## Edge cases / known exceptions

- `JIBA.Employee.GetUserInfoByEmpId(EmpId, ...)` may throw if EmpId is malformed — the call isn't guarded. Q-259.
- If `pvs@jazo.com` ever stops being monitored, all anonymous tickets go to the void.
- `IsMachineEmpId` semantics: defined in `ISAH.Employee`. Probably "EmpId starts with M" or similar — to be verified.

## Safety classification

- [ ] Touches physical process — no.
- [ ] Drives cost / pricing — no.
- [x] Reversible if wrong? — yes (re-route tickets, change literal).
- [x] Blocks production if it fails? — kind of: if Zammad rejects tickets without a valid `From`, operators can't file feedback through iCenter. Soft block.

## SME questions

- **Q-259 (new):** Document `pvs@jazo.com` — what is it (distribution list / inbox)? Who triages?
- **Q-260 (new):** Move this literal to AppSettings? The Servicedesk* settings already exist (`ServicedeskFeedbackUrl`, `ServicedeskBaseUrl`); add `ServicedeskFallbackEmail`?

Logged in [[../needs-review/_index]].

## Related

- [[../modules/icenterlib-icenter-leaves|`Servicedesk`]] — defining module.
- [[../external-systems/zammad|Zammad]] — destination system.
- [[../mocs/icenterlib-icenter]] — parent MOC.
