---
type: module
title: "IPBatchCollector — ISAH production-BOM query"
status: done
module: "iCENTER/WorkPreparation"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\WorkPreparation\\IPBatchCollector.vb"
last-reviewed: 2026-06-18
tags: [module, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `IPBatchCollector.vb` — ISAH production-BOM query

## Purpose

A thin wrapper around a **single SQL query** that pulls "production bill of operations" rows from ISAH for a given search value. Used by [[workprep-outsource-operations|`OutsourceOperationsHandler.Init`]] (with `""` to fetch all) as the data source for the outsourcing UI.

## Public surface

```vb
Public Class IPBatchCollector
    Public Function GetProdBillOfOper(SearchValue As String) As DataTable
End Class
```

One method, no state.

## The query

Targets the **ISAH database** (via `Connections.ConnectIsah`) — joins six tables:

- `T_ProductionHeader` (PH) — production-header rows
- `JZ_ProdRefNr` (PRN) — JAZO-custom production-reference numbers
- `T_ProdHeadDosDetLink` (PHDDL) — production-header ↔ dossier-detail link
- `T_DossierDetail` (DD)
- `T_DossierMain` (DM)
- `T_ProdBillOfOper` (PBOO) — the production bill-of-operations rows
- `T_MachGrp` (MG)

Returns columns: `OrdNr`, `DetailCode`, `DetailSubCode`, `ProdRefNr`, `Description`, `ProdHeaderDossierCode`, `MachGrpCode`, `EndDate`, `MGDescription`, `DossierCode`.

Search semantics:
- If `@SearchValue <> ''` → filter to rows matching `ProdHeaderDossierCode` OR `DM.OrdNr` OR `CONVERT(nvarchar, PRN.ProdRefNr)` equal to the search value.
- If `@SearchValue = ''` → **no rows** match the filter, but the SELECT then runs against the populated `@T_ProdHeaderDossierCode` table variable which is empty in that case. **Effectively returns no rows.** Q-098 — confirm with SME whether the empty-search case really should return nothing.

## Surprises

1. **Empty-`SearchValue` returns empty DataTable.** Callers expecting "all rows" must pass *something*. The `OutsourceOperationsHandler.Init` call passes `""`, suggesting either (a) the empty case is supposed to populate, or (b) the outsource UI starts empty and waits for the user to enter a search. Q-098.

2. **A commented-out earlier version** at the bottom of the file (lines 57–97) has the same query but without the `T_ProductionHeader`/`JZ_ProdRefNr` inner join in the populate step. The live version (lines 5–46) was apparently a tightening — only show rows whose ProdHeaderDossier *also* has a ProdRefNr (i.e. it's been "released" / assigned a JAZO ref-nr).

3. **Parameterised via `Dictionary(Of String, Object)`** → `DataHandler.GenericQuery.ExecuteSelect` — defends against SQL injection (good, since `SearchValue` may come from a textbox).

4. **`T_Code_ProdHeadDossier`** is a *user-defined type* in ISAH; the `@T_ProdHeaderDossierCode TABLE` variable uses it as the column type. The class therefore won't compile/execute against an ISAH instance that lacks the UDT.

5. **`JZ_*` prefix** = JAZO-custom tables; ISAH base tables use `T_*`. The mixed naming is a JAZO convention. Phase-3 follow-up to enumerate all `JZ_*` tables referenced from iCENTER.

## Business rules surfaced here

- The query **only surfaces production headers that have a `JZ_ProdRefNr` entry**. Rows without one are invisible to outsourcing — meaning the WorkPreparation team can't outsource un-numbered jobs. Q-099.

## External systems touched

- [[../external-systems/isah|ISAH]] via `Connections.ConnectIsah` (the global ISAH connection — Phase-3 follow-up to find where this is initialised).

## Open questions

- **Q-098 (new):** empty `SearchValue` returns no rows — confirm intentional. `#needs-review`
- **Q-099 (new):** filter requires a `JZ_ProdRefNr` row — confirm rule "only jobs with assigned ref-nr are outsourceable". `#needs-review`

Logged in [[../needs-review/_index]].

## Coverage

`_coverage.md`: `WorkPreparation\IPBatchCollector.vb` → `done`.
