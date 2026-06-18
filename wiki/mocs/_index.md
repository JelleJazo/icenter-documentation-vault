---
type: moc
title: "Maps of Content — Index"
status: draft
tags: [moc]
created: 2026-06-18
updated: 2026-06-18
---

# Maps of Content (MOCs)

A MOC is a subsystem hub: a single page that gathers everything (modules, rules, external systems, concepts, open questions) relevant to one subsystem. They are the navigation backbone of the wiki.

## Subsystem MOCs (one per logical area, to be authored in Phase 3)

Confirmed from the Phase 2 architecture pass — each maps to a top-level `iCENTER\` folder cluster:

- **[[elumatec|Elumatec subsystem]]** — `Elumatec/` (+ `Modules/`, `Production/ProfileMilling`) — CAD→NC pipeline for SBZ140 / SBZ141 machines + the saw COM watcher. `#safety-relevant`. **Hub written 2026-06-18.**
  - **[[elumatec-works|Elumatec Works/Replacements pipeline]]** — feature classes (Circle/Drill/Rectangle/...) + 19 replacement macros. `#safety-relevant`. **Sub-hub written 2026-06-18.**
- **Sheet-metal (SMT) subsystem** — `SmtManufacturing/`, `Modules\Main.vb` Oseon services, parts of `CadBatchserver/Job*Smt*` — Trumpf / TruTops Oseon integration. `#safety-relevant`
- **CAD pipeline** — `CAD/`, `CadBatchserver/`, `CAM/`, Creo integration. `#safety-relevant`
- **Engineering → Production handoff** — `Engineering/`, `WorkPreparation/`, `Production/`, ProductionDossier entities.
- **Sales / Order intake** — `Sales/`, parts of `Forms/`, ProductDb integration.
- **Coating subsystem** — `Classes\Coating\` (12+ files), `CoatingPickTimeWarning`, Kardex job flow.
- **Stickers / labels / printing** — `Classes\StickersAndLabels\`, Dymo, GhostScript, PDF-XChange.
- **Data migration** — `DataMigration/` (50 files), `iCenter2NewestDataModel`.
- **Kardex / storage** — `Kardex/`, `Isah2KardexExportPath`. `#safety-relevant`
- **Time registration** — `WebClock/`, `Classes\Production\LeanWorkTime`, ISAH.TimeRegistration.
- **External systems hub** — points to [[../external-systems/_index]] (already a MOC).
- **Build / deploy / signing** — points to [[../architecture/build-and-deploy]].

Each MOC links to:
- All [[../modules/_index|modules]] in that subsystem
- All [[../business-rules/_index|business rules]] it owns
- All [[../external-systems/_index|external systems]] it touches
- Any [[../needs-review/_index|open questions]] for the subsystem
