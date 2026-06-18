---
type: moc
title: "Modules — Index"
status: draft
tags: [moc, modules]
created: 2026-06-18
updated: 2026-06-18
---

# Modules

One note per major source file or tight group of files. Each entry mirrors a row in [[../_coverage|_coverage.md]].

## Companion projects (added 2026-06-18)

- **TruTopsLib** — `C:\DevOps\iCenter\iCenter\TruTopsLib\` (65 files). 4 sub-areas: `(root)`, `GeoInterpreter/`, `PMI/`, `TopsFile/`. Mixed `.vb` + `.cs`. Phase-3 notes will live under `modules/trutopslib-*`.
- **ICenterLib** — `C:\Users\jelle-r\source\repos\JIBA\iCenter And Tools\ICenterLib\ICenterLib\` (723 files across 35 top-level folders). Phase-3 notes under `modules/icenterlib-*`. High-priority sub-areas for Elumatec-related work: `CAD/`, `ICenter/`, `ISAH/`, `SmtProduction/`, `PCFNet/`, `Production/`.

## Grouped by `iCENTER\` sub-folder

> Counts reflect the Phase 1 inventory of `.vb` source files (excluding `.Designer.vb` partials and `.resx`). See [[../_coverage]] for the authoritative file list per folder.

### Root files

- [[main-module|`Modules\Main.vb`]] — process entry, god-module of globals.
- `ApplicationEvents.vb` — empty `My.MyApplication` partial. (no note needed; covered in [[../architecture/entry-points]])
- `FrmMain.vb` — the main UI form (15 216 lines). **Will require multiple notes.** Phase-3 plan: split into `frm-main/_index`, `frm-main/treeview`, `frm-main/search`, `frm-main/print`, etc.

### Per-subsystem (one note per folder index; deep notes as needed)

- [[batchserver|`Batchserver/`]] — generic post-`FrmMain` batch host. 3 `.vb` files.
- [[cad-batchserver|`CadBatchserver/`]] — CAD job runner (`-m cadbatchserver`). 27 `.vb` files; one note per `Job*`.
- [[cad|`CAD/`]] — 2 `.vb` files.
- [[cam|`CAM/`]] — manufacturing-part classes. 4 `.vb` files.
- [[classes|`Classes/`]] — core domain classes + helpers. **111 files** — needs sub-MOC. Subfolders: `Coating/`, `Connectivity/`, `PreSelectMachGrpCodes/`, `Production/`, `StickersAndLabels/`, `Toolbox/`.
- [[comparers|`Comparers/`]] — `IComparer` implementations. 4 files.
- [[controls|`Controls/`]] — custom WinForms user-controls. **78 `.vb`**. Sub: `Isah/`.
- [[data-migration|`DataMigration/`]] — Entity + Handler + Factory pattern targeting `iCenter2NewestDataModel`. 50 files. Sub-MOC needed.
- [[design-comments|`DesignComments/`]] — 7 files.
- [[elumatec|`Elumatec/`]] — SBZ140 / DG profile-mill integration. **140 `.vb`** — biggest single-purpose subsystem. `#safety-relevant`. Subs: `AufSerializer/`, `Database/`, `Machine/`, `NCStructure/`, `Works/Replacements/`.
- [[engineering|`Engineering/`]] — 15 files.
- [[forms|`Forms/`]] — non-main dialogs / shop forms. **127 `.vb`** + 61 `.resx`. Subs include `ShopProcess/`.
- [[ic-importer|`IcImporter/`]] — 5 files.
- [[kardex|`Kardex/`]] — Kardex Shuttle integration. 3 files. `#safety-relevant`
- [[mark-tool|`MarkTool/`]] — drawing-marker tool. 4 files.
- [[modules-folder|`Modules/`]] — `Main.vb`, plus 2 others. (See [[main-module]].)
- [[pcf-net-studio|`PCFNetStudio/`]] — PCFNet integration. 12 files.
- [[production|`Production/`]] — production-floor logic. 6 files.
- `Resources/` — 354 image / icon assets. Not documented individually; covered by [[../_coverage]] as `config`.
- [[sales|`Sales/`]] — 2 files.
- [[smt-manufacturing|`SmtManufacturing/`]] — sheet-metal (Trumpf/Oseon) UI + logic. **63 `.vb`** — sub-MOC needed. `#safety-relevant`
- [[sola-data-connector|`SolaDataConnector/`]] — 2 files.
- [[uni-link|`UniLink/`]] — 29 files. Likely an external system bridge. `#needs-review`
- [[vent-duct-configurator|`VentDuctConfigurator/`]] — ventilation-duct configurator. 3 files.
- [[web-clock|`WebClock/`]] — time-registration UI. 9 files.
- [[work-preparation|`WorkPreparation/`]] — werkvoorbereiding. 5 files.

> **Most entries above are stubs** (not yet written) — that's expected for Phase 2. Phase 3 fills each one. Don't follow the link if it's red unless you're ready to write the note.
