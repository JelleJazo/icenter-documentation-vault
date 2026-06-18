---
type: module
title: "Sales\\FrmCustomerTeam.vb — customer-to-team assignment"
status: done
module: "iCENTER/Sales"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Sales\\FrmCustomerTeam.vb"
last-reviewed: 2026-06-18
tags: [module, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `Sales\FrmCustomerTeam.vb` — customer-to-team assignment

## Purpose

WinForms UI for assigning ISAH customers to **sales teams** (the three "klantteams" 031/032/033). One filter combo (CBFilterTeam), one apply-team combo (CBApplyTeam), an editable DataGridView of customers (DGVCustomer), and a date-range filter on last-order-confirmation date.

Reads customer data via `ISAH.Customer.GetByOrdConfDate(filter, till, from)` (or an unfiltered variant when the date-range checkbox is off). Writes back team assignments via the apply-combo action (Phase-3 follow-up to enumerate the exact write API — likely `ISAH.Customer.UpdateTeam` or similar; not visible in the first 80 lines).

## Public surface

| Symbol | Kind | Note |
|--------|------|------|
| `Sales.FrmCustomerTeam` | class | WinForms form |
| `IsahTableName As String = "T_Customer"` | const | the ISAH source table |
| `FormText As String = "Klantteam verdeling"` | const | window title (Dutch: "Customer team distribution") |
| `New()` | constructor | populates both combos, wires header checkbox |

Private members of note: `DTCustomer`, `DVCustomer`, `DTFilter`, `DictFilter`, `MySelectionMode`, `headerCheckBox`.

## Behavior in plain language

Constructor (lines 18–51):
1. Builds a local `DTFilter` DataTable with columns `SelectionCode` and `DisplayName`.
2. Adds a `(DBNull, "Geen")` row meaning "no filter".
3. **Hard-codes three team selection codes**: `"031"`, `"032"`, `"033"` (line 31). For each, fetches the description via `ISAH.Selection(code).GetDescription`; falls back to the code itself if description is missing.
4. Binds the same `DTFilter` to both `CBFilterTeam` and `CBApplyTeam` (the apply-combo gets a `.Copy` so they don't share row-changes).
5. Disables both date-pickers until the date-range checkbox is checked.

`GetData()` (line 68+):
- If `CBLastOrdConfDate` is checked → `ISAH.Customer.GetByOrdConfDate(filter, till, from)`.
- Otherwise → presumably an unfiltered variant (Phase-3 follow-up).

The header-checkbox click-handler (line 57) ends grid edit and bulk-toggles the per-row "Selected" checkbox cells.

## Business rules surfaced here

- [[../business-rules/sales-team-codes|`SalesTeamCodes = {"031","032","033"}`]] — the three team codes are hard-coded; no `app.config` setting, no iCenter-DB lookup. Q-082. `#needs-review` to confirm with SME these are the only three teams.
- `IsahTableName = "T_Customer"` — a strong coupling to the ISAH schema. Renaming the ISAH table breaks this form.

## External systems touched

- [[../external-systems/isah|ISAH]] — `ISAH.Selection`, `ISAH.Customer.GetByOrdConfDate`, `T_Customer` table.
- iCenter [[../architecture/global-state|`DataHandler.Selection.RowSelectionMode`]] (global enum used for the grid selection mode).

## Surprises

1. **Hard-coded team codes.** The `Dim l As New List(Of String) From {"031", "032", "033"}` (line 31) is the entire authoritative list. A future fourth team requires a code change + deploy.
2. **`DTFilter` is declared *both* at module scope (line 9: `Private DTFilter As DataTable = Nothing`) *and* shadowed inside `New()` (line 25)** — the shadowing local is the one actually bound to the combos. The class-level field stays `Nothing` forever. Likely a left-over; harmless in this code path.
3. The first row `{DBNull.Value, "Geen"}` (Dutch for "none") makes the combo's first item a "no filter" sentinel — pattern repeated in several iCenter forms.
4. **`CBApplyTeam.DataSource = DTFilter.Copy`** — copies the DataTable. If a future team is added to the filter combo via UI, the apply combo won't see it. Defensible if both lists are static.
5. Date-pickers are disabled by default; the user has to opt into date-filtered queries by checking `CBLastOrdConfDate`.

## Open questions

- **Q-082** (from MOC): confirm with SME that 031/032/033 are the only team codes and that the codes themselves match ISAH `T_Selection` entries with the same prefix.
- **Q-091 (new):** what does the "apply" action actually call on ISAH? Phase-3 follow-up to find the matching `UpdateTeam`-like method.
- **Q-092 (new):** `DTFilter` is declared at class scope and shadowed in `New()`. Remove the field?

Logged in [[../needs-review/_index]].

## Coverage

`_coverage.md`: `Sales\FrmCustomerTeam.vb` → `done`.
