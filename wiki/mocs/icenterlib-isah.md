---
type: moc
title: "ICenterLib/ISAH — Map of Content"
status: draft
tags: [moc, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# ICenterLib/ISAH

## What this hub covers

`ICenterLib\ISAH\` — 65 `.vb` files that wrap the **ISAH ERP database** as typed entity classes. Every Sales/WorkPreparation/Production note we've written already references `oISAH.*` or `ICenterLib.ISAH.*` for some lookup or mutation. Most files map 1:1 to an ISAH table or a tight group of tables.

> The folder is the **most-referenced subsystem** in ICenterLib from iCENTER's perspective. iCenter cannot operate without ISAH being reachable.

ISAH is accessed via three patterns visible in this folder:

1. **Direct SQL** via `Connections.ConnectIsah()` — most queries; entities open their own connection, run inline `SELECT`/`UPDATE`, close.
2. **Stored procedures** — many are prefixed `IP_*`, `JIP_*`, `SIP_*`. The `JIP_` prefix is JAZO-custom (cf. `JZ_*` tables).
3. **`DataHandler.GenericQuery`** — the parameterised wrapper used in newer code.

## Scope

65 `.vb` files; ~330 KB of source. Largest 10:

| File | KB | Role |
|------|---:|------|
| `TimeRegistration.vb` | 72 | shop-floor time clocking + work-time tracking |
| `Part.vb` | 49 | ISAH parts (T_Part), including SmtPart/Profile/etc. |
| `ProductionHeader.vb` | 45 | production-header workflow |
| `Icenter2Isah.vb` | 38 | iCenter2 ↔ ISAH sync layer |
| `PBOO.vb` | 31 | Production Bill of Operations |
| `DossierDetail.vb` | 28 | dossier-detail line items |
| `BillOfMat.vb` | 26 | bill of materials |
| `DossierMain.vb` | 19 | dossier-main (sales / production order shell) |
| `Employee.vb` | 17 | employees (T_Employee) |
| `PBOS.vb` | 11 | Production Bill of Services (?) |

## Documented in this batch

| File(s) | Module note |
|---------|-------------|
| `Selection.vb`, `Setting.vb` | [[../modules/isah-lookups]] |
| `Employee.vb`, `User.vb`, `Customer.vb`, `Vendor.vb`, `Company.vb` | [[../modules/isah-identity]] |
| `ShopDoc.vb`, `PurDoc.vb` | [[../modules/isah-shop-and-pur-doc]] |
| `MachGrp.vb` | [[../modules/isah-machgrp]] |

The big domain files (TimeRegistration, Part, ProductionHeader, Icenter2Isah, PBOO, DossierDetail, BillOfMat, DossierMain) are deferred to subsequent batches — each warrants its own deep-dive note.

## ISAH schema fragments seen so far

| Table | Used by | Notes |
|-------|---------|-------|
| `T_Employee` | Employee, User | core HR record |
| `T_UserRegistration`, `T_UserRegistrationInfo` | User | iCenter login mapping |
| `T_TimeRegistration` | Employee, TimeRegistration | shop-floor clocking |
| `T_Customer` | Customer, FrmCustomerTeam | |
| `T_MachGrp` | MachGrp | machine-groups |
| `T_Selection` | Selection | code-tables (the 031/032/033 sales teams, 041/042 company codes, dept codes, etc.) |
| `T_ShopDoc` | ShopDoc | shop-floor work documents |
| `T_ProductionHeader` | ProductionHeader, ShopDoc, IPBatchCollector | production-job header |
| `T_ProdBillOfOper` | PBOO, ShopDoc | per-operation lines under a ProductionHeader |
| `T_PurchaseDocument` | PurDoc | purchase orders |
| `T_DossierMain` | DossierMain | sales/order-dossier shell |
| `T_DossierDetail` | DossierDetail | per-line dossier items |
| `T_ProdHeadDosDetLink` | (joins) | links ProductionHeader ↔ DossierDetail |
| `T_MemoDetail` | User.GetJobDescription | typed memos (with `MemoTypeCode`, `LangCode`) |
| `T_ApplicationSettings` | — *(this is the iCenter DB, not ISAH)* — listed for comparison |

Plus JAZO-custom `JZ_ProdRefNr` (already seen via `IPBatchCollector`).

## Business rules surfaced in this batch

- [[../business-rules/isah-company-codes|JAZO vs FlowGrill company codes]] — `JAZO = AdminCode "JAZO" / SelectionCode "041"`, `FlowGrill = "FLOWGRIL" / "042"`. Hard-coded in `ISAH.Company`.
- [[../business-rules/isah-track-operation-pattern|`TR\d\d` track-operation pattern]] — `MachGrpCode` matching the regex `TR\d\d` is classified as a "track operation" by `MachGrp.IsTrackOperation`.

## Notable findings from this batch

1. **`Employee.GetIsPresent`** branches on the migration constant `Connections.UseIsahNoDtrTimeReg`. When true (which it currently is — `Public Const = True`), the query checks `Starttime <> 0 AND EndTime = 0` (no DTR status). When false, it would use `DTRStatusCode = 'AO'`. The DTR pre-migration path is dead code today. Q-132.
2. **`Employee.GetIsObsolete`** returns `True` if `IP_sel_EmployeeRecord` fails or yields no rows — i.e. **fail-closed** ("treat as obsolete on error"). Important for downstream UI that filters out obsolete employees: a transient ISAH outage would make every employee appear obsolete. Q-133.
3. **`Employee.GetCurrentTimeRegBasic`** has parallel SQL: a `T_TimeRegistration`-direct query (`UseIsahNoDtrTimeReg = True` path) and an old DTR-status-aware variant (Phase-3 follow-up; truncated in this read). Same pattern as `GetIsPresent`.
4. **`User.GetJobDescription`** reads `T_MemoDetail` filtered on `MemoTypeCode='JEM10' AND LangCode='NL'` — Dutch-only memo with a magic memo-type code. Q-134.
5. **`Selection`** is the universal code-table accessor. Backed by `IP_sel_SelectionRecord` SP. Used widely (e.g. the `031/032/033` sales team codes in [[../modules/sales-customer-team]] each construct a `Selection`).
6. **`MachGrp`** filters out obsolete entries by `Description NOT LIKE '~%'` — the `~` prefix convention marks obsolete machine groups. Q-135.
7. **`MachGrp.IsTrackOperation`** uses a regex (`TR\d\d`) to detect track operations. Phase-4 candidate for a domain-concept note.
8. **`ShopDoc.SetShopDocFinInd`** calls SP `JIP_Upd_MyShopDoc` and writes a Log entry. **`SetShopDocStartedInd`** is a thin wrapper that delegates to `PBOO.SetOperStartInd(ShopDocCode, True)` — the "started" flag actually lives on `T_ProdBillOfOper` not `T_ShopDoc`. Q-136.
9. **`Company`** hard-codes the two JAZO entities: `JAZO` (admin "JAZO", selection "041") and `FlowGrill` (admin "FLOWGRIL", selection "042"). All three factory methods (`CreateByOrdType`, `CreateByAdminCode`, `CreateBySelectionCode`) return `Nothing` for unknown values — fail-closed.
10. **`PurDoc.CreateByPurOrdNr`** is the lookup-by-display-id factory. Returns `Nothing` if no matching `T_PurchaseDocument.PurOrdNr`. The PurDocCode (numeric internal id) is the primary identifier in the rest of the code.
11. **`Setting.GetStandCapacityType`** is the only method in the file — and it uses an **output parameter** (`SqlParameter Direction = Output`). Returns the int via the SP's `@StandCapacityType` out-param. Older pattern; rest of ICenterLib uses `ExecuteScalar`/`ExecuteReader`.

## Open questions

- **Q-132 (new):** `Connections.UseIsahNoDtrTimeReg = True` means the DTR-status time-reg query path is dead code. Confirm with SME this migration is complete; remove the dead branches?
- **Q-133 (new):** `Employee.GetIsObsolete` is fail-closed (returns True on error). A transient ISAH outage would tag every employee as obsolete. Intentional?
- **Q-134 (new):** `User.GetJobDescription` filters on `MemoTypeCode='JEM10' AND LangCode='NL'`. Document the JEM10 memo-type. Hard-coded Dutch-only.
- **Q-135 (new):** `MachGrp` obsolete-detection by `Description LIKE '~%'`. Is the `~` prefix convention documented anywhere?
- **Q-136 (new):** `ShopDoc.SetShopDocStartedInd` writes the "started" flag to `T_ProdBillOfOper`, not `T_ShopDoc`. Confirm intentional and document the PBOO/ShopDoc relationship.
- **Q-137 (new):** ISAH SP naming uses three prefixes: `IP_*` (stock ISAH?), `JIP_*` (JAZO-custom?), `SIP_*` (iCenter DB, not ISAH). Document the convention.

Logged in [[../needs-review/_index]].

## Documentation roadmap (next ISAH batches)

1. **DossierMain + DossierDetail + DossierDetailExtra** — the sales/order dossier hierarchy.
2. **ProductionHeader + PBOO + PBOS + PBOM** — the production hierarchy (header → bills of operations / services / materials).
3. **BillOfMat + BillOfOper + Part** — the parts-and-routings layer.
4. **TimeRegistration + TimeRegCollector + Employee** (deep) — clocking workflow.
5. **Icenter2Isah** — the sync layer between iCenter2 and ISAH (likely big and central).
6. **PartDispatch + PartDispatchCollectorDataService** — picking/dispatch flow.
7. **CallRegistration + Servicedesk** — service-desk integration.
8. **DataServices/** and **Helpers/** subfolders.

## Related

- [[icenterlib|ICenterLib top-MOC]] — parent.
- [[../modules/icenterlib-connections]] — how every ISAH class gets its connection.
- [[../mocs/office-to-shopfloor]] — every module there hits one or more ISAH classes.
- [[../external-systems/isah|ISAH external system]] — the boundary description.
