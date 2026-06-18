---
type: moc
title: "Domain Concepts — Index"
status: draft
tags: [moc, domain-concept]
created: 2026-06-18
updated: 2026-06-18
---

# Domain Concepts

Vocabulary glossary for the iCenter domain. Each term is one page. Use these as targets for `[[wikilinks]]` from module and business-rule notes.

> Many terms in this codebase are **Dutch** (`werkvoorbereiding` = work preparation, `bewerking` = operation, `tekening` = drawing). Always include the English alias for clarity.

## Watchlist (seeded by the Phase 2 read of `Modules\Main.vb` + `app.config`)

- **iCenter** — the application itself; also a dedicated database (`oICENTER`, `ClsICenter`).
- **iCenter2** — sibling app whose data is read for migration via `iCenter2NewestDataModel` DB connection. Out of code scope.
- **Dossier** — a folder/bundle of work for one engineering job. `Engineering/`, `EngineeringOrders`, `DataMigration\Entities\Dossier.vb`.
- **ProductionDossier** — production-side counterpart. `DataMigration\Entities\ProductionDossier.vb`.
- **BOM (BillOfMaterial)** / **BillOfOperation** — standard ERP terms; entity classes in `DataMigration\Entities\`.
- **Machine group (MachGrp)** — group of similar machines. Codes like `A48`, `S48`, `A01`, `P44`, `EBTV`, `EBAN`.
- **Operation (Bewerking)** — single step in a routing. Codes like `A80`, `S80`, `Y80`, `K80` are "kick-off" operations.
- **Kickoff operation** — the operation that opens a routing. See `KickOffOperations`.
- **Lean mode** — UI mode restricting machine groups to a lean subset. `FrmMain.LeanMode`.
- **IP part / IP batch** — "iCenter Production" part/batch. `SearchByType.IPPart`, `SearchByType.IPBatch`, `IPPartNotReleasedModelname`.
- **ProfMill (Profile Mill)** — Elumatec SBZ-class profile-milling. `ProfMillMachGrps = A48;S48;`.
- **SBZ** — Elumatec machine family ("Stab Bearbeitungszentrum" — profile machining centre).
- **DGX** — Elumatec Door+Gate machine variant. `EluDxfDirDgx`, `EluEdpDirDgx`.
- **SMT (SheetMetal)** — sheet-metal subsystem; uses Trumpf / TruTops Oseon. `SmtMachGrpCodesOverrideGenSetupTime`, `SmtMaxValueOutline`.
- **Oseon** — Trumpf MES product. `OseonAppContext`, `OseonAppContextDataService`.
- **Coating** — surface-treatment subsystem (zinc, paint). `Classes\Coating\`, `CoatingPickTimeWarning`.
- **EBTV** — sub-process code; appears in `leanMachGrpCodes`. `#needs-review` — what does EBTV stand for?
- **Kardex** — vertical-lift parts storage. `Isah2KardexExportPath`.
- **Kanban bin** — physical bin in production. `Classes\StickersAndLabels\KanbanBinLabel.vb`, `SearchByType.KanbanBin`.
- **EngineeringOrder / EngOrd** — engineering-side order. `Engineering\EngineeringOrders`.
- **ProdOrd** — production order.
- **WorkPreparation (Werkvoorbereiding)** — Dutch term for the preparation phase between engineering and production. Dedicated folder `WorkPreparation/`.
- **Werkvoorbereider** — work preparator (role); see `FrmMain.IamWorkPrepper`.
- **JIBA** — internal portal at `jiba.jazo.com`. `oJIBA`, `ClsJIBA`.
- **JZPUBLISH** — name of a service account / user (commented hint in `Modules\Main.vb`). `#needs-review`
- **FGQC** — Finished-Good Quality Control. `FGQCPartCode = "QLTYCNTRL_ALU"`.

## Pages

_(Populated as Phase 3 progresses. Use the [[../_templates/domain-concept|template]]. Most will be 3–10 sentence notes citing 1–3 source paths.)_
