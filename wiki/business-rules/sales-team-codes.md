---
type: business-rule
title: "Sales team codes (031 / 032 / 033)"
status: needs-review
module: "iCENTER/Sales"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Sales\\FrmCustomerTeam.vb"
last-reviewed: ""
tags: [business-rule, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Sales team codes — 031 / 032 / 033

## Rule

The sales-side customer-management UI (`FrmCustomerTeam`) recognises **exactly three** sales-team selection codes: `"031"`, `"032"`, `"033"`. The codes are hard-coded as a `List(Of String)` in the form constructor; their descriptions are pulled at runtime from ISAH `T_Selection` via `ISAH.Selection(code).GetDescription`, with the code itself used as a fallback if no description exists.

The list is the authoritative input source for both the **filter combo** (`CBFilterTeam`, "show customers in team X") and the **apply combo** (`CBApplyTeam`, "assign selected customers to team X").

## Where it lives

- File: `Sales\FrmCustomerTeam.vb`
- Symbol: `FrmCustomerTeam.New()` lines 31–39

## The code (minimal quote)

```vb
Dim l As New List(Of String) From {"031", "032", "033"}
For Each s As String In l
    Dim Selection As New ISAH.Selection(s)
    Dim Description As String = Selection.GetDescription
    If Description Is Nothing OrElse String.IsNullOrEmpty(Description) Then
        Description = s
    End If
    DTFilter.Rows.Add({s, Description})
Next
```

## Why this is a business rule

`031 / 032 / 033` are JAZO's internal sales-team identifiers; the codes also live in ISAH as `T_Selection` entries (so the description lookup succeeds). Code lookups in this category drive:
- which sales-team a customer is *visible to* (filter combo)
- which sales-team a customer is *assigned to* (apply combo)

Adding or renaming a team requires a code change in this file + redeploy.

## Triggers / when it fires

- Once per `FrmCustomerTeam.New()` (when the form opens).
- Inputs: none (codes are literals); ISAH must be reachable for description lookup.

## Effects

- Populates the form's two ComboBoxes with `(SelectionCode, DisplayName)` rows.
- Drives the SQL filter passed to `ISAH.Customer.GetByOrdConfDate(filter, …)` when the user selects a team.

## Edge cases / known exceptions

- **A fourth team can never be selected in iCenter** without a source change.
- **If ISAH descriptions are removed** (e.g. cleaning up `T_Selection`), the combo shows the bare code (`031`) — still functional, just less friendly.
- The leading `(DBNull, "Geen")` row (= "none") is the team-filter sentinel meaning "no team filter" — *not* a fourth team.

## Safety classification

- [ ] Touches physical process — no.
- [ ] Reversible if wrong? — Trivially, if the list changes.
- [ ] Blocks production if it fails? — No (sales-only).

Marked `#needs-review` for SME confirmation, not safety.

## SME questions

- **Q-082**: confirm 031/032/033 are the only sales teams. Surface their human-friendly names (since ISAH descriptions may be terse).
- **Q-091**: what does the apply-team action actually call on ISAH? (Tracked under [[../modules/sales-customer-team]].)

Logged in [[../needs-review/_index]].

## Related

- [[../modules/sales-customer-team]] — the module.
- [[../mocs/office-to-shopfloor]] — workflow hub.
