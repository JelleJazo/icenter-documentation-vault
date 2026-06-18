---
type: external-system
title: "ISAH ERP"
status: stub
tags: [external-system, needs-content, safety-relevant]
created: 2026-06-18
updated: 2026-06-18
---

# ISAH ERP

**The primary ERP database** iCenter integrates with. Accessed via `ICenterLib.ISAH.*` and the global wrappers `oISAH` / `ClsISAH`.

> **Stub** — populate connection details, version, hosting, contact, retry behavior.

## Quick links

- [[../mocs/icenterlib-isah|ICenterLib/ISAH MOC]] — the 63-file folder wrapping this system.
- [[../modules/icenterlib-connections|`Connections.ConnectIsah()`]] — connection-string source.
- [[../business-rules/isah-company-codes|JAZO / FlowGrill company codes]].
- [[../modules/isah-icenter-to-isah|iCenter → ISAH sync layer]].

## Tables seen (partial)

`T_Employee`, `T_Users`, `T_TimeRegistration`, `T_Customer`, `T_MachGrp`, `T_Selection`, `T_ShopDoc`, `T_ProductionHeader`, `T_ProdBillOfOper`, `T_PurchaseDocument`, `T_DossierMain`, `T_DossierDetail`, `T_ProdHeadDosDetLink`, `T_MemoDetail`, `T_Part`, `T_PartDispatch`, `T_PartSelection`, `T_Surcharge`, `T_DeliveryLine`, `T_PurDocPartLine`, `T_CallRegistration`, `T_IsahFieldML`, `T_Design`, `T_DossierContactFav` (iCenter side), plus JAZO-custom `JZ_*` family (`JZ_jConfigParam`, `JZ_ProdLeadTime`, `JZ_PartDispatch`, `JZ_AutomationJob`).

## SP naming conventions

- `IP_*` — stock ISAH stored procedures.
- `JIP_*` — JAZO ISAH-custom procedures.
- `SIP_*` — ?? (Q-137 / Q-214).
- `IPX_*` / `JIPX_*` — extension variants (Q-214).

## SME questions

See [[../needs-review/_index]] for the full open queue. Many ISAH-side questions remain (Q-137 SP-naming, Q-153 ST_-prefix, Q-185 ISAH-vs-ICENTER user codes, …).
