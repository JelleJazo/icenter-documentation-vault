---
type: external-system
title: "iCenter SQL database"
status: stub
tags: [external-system, needs-content, safety-relevant]
created: 2026-06-18
updated: 2026-06-18
---

# iCenter SQL database

iCenter's **own SQL Server database** (distinct from the ISAH ERP database). Accessed via `ICenterLib.ICenter.*` and the global wrappers `oICENTER` / `ClsICenter`.

> **Stub** — populate hostname, instance, version, hosting, backup policy.

## Quick links

- [[../mocs/icenterlib-icenter|ICenterLib/iCenter MOC]] — 24 entity classes wrapping this DB.
- [[../modules/icenterlib-connections|`Connections.ConnectICenter()`]].
- [[../modules/icenterlib-appsettings|AppSettings table]] — `T_ApplicationSettings`.

## Tables seen (partial)

`T_ProdMachines`, `T_ProdMachinesMP`, `T_ProdMachineWorkTime`, `T_ProdMachineWorkTimeRegistration`, `T_IPorderLines`, `T_IPbatchLines`, `T_IPpacketLines`, `T_IPpartLines`, `T_IPpartDispatch`, `T_IPorderRef`, `T_TimeRegistration`, `T_DgxStickers`, `T_CoatingLayerThickness`, `T_SmtMaterials`, `T_Operations`, `T_DossierContactFav`, `T_WebClockAssistant`, `T_ProductionProfileCutItem`, `T_MarkToolLogMessages`, `T_LogMessageType`, `T_ApplicationSettings`.

## SME questions

See [[../needs-review/_index]].
