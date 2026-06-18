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

- [[../mocs/icenter-remaining|`Batchserver/`]] — generic post-`FrmMain` batch host. 3 `.vb` files.
- [[../mocs/icenterlib-cadbatchserver|`CadBatchserver/`]] — CAD job runner (`-m cadbatchserver`). 27 `.vb` files; one note per `Job*`.
- [[../mocs/icenter-remaining|`CAD/`]] — 2 `.vb` files.
- [[../mocs/icenter-remaining|`CAM/`]] — manufacturing-part classes. 4 `.vb` files.
- [[../mocs/icenter-remaining|`Classes/`]] — core domain classes + helpers. **111 files**. Subfolders: `Coating/` (deep-read at [[icenter-coating]]), `Connectivity/`, `PreSelectMachGrpCodes/`, `Production/`, `StickersAndLabels/`, `Toolbox/`.
- [[../mocs/icenter-remaining|`Comparers/`]] — `IComparer` implementations. 4 files.
- [[../mocs/icenter-remaining|`Controls/`]] — custom WinForms user-controls. **78 `.vb`**. Sub: `Isah/`.
- [[../mocs/icenter-remaining|`DataMigration/`]] — Entity + Handler + Factory pattern targeting `iCenter2NewestDataModel`. 50 files. Likely `#dead-code` (Q-314).
- [[../mocs/icenter-remaining|`DesignComments/`]] — 7 files.
- [[../mocs/elumatec|`Elumatec/`]] — SBZ140 / DG profile-mill integration. **140 `.vb`** — biggest single-purpose subsystem. `#safety-relevant`. Subs: `AufSerializer/`, `Database/`, `Machine/`, `NCStructure/`, `Works/Replacements/`.
- [[../mocs/office-to-shopfloor|`Engineering/`]] — 15 files.
- [[../mocs/icenter-remaining|`Forms/`]] — non-main dialogs / shop forms. **127 `.vb`** + 61 `.resx`. Subs include `ShopProcess/`.
- [[../mocs/icenter-remaining|`IcImporter/`]] — 5 files.
- [[icenter-kardex|`Kardex/`]] — Kardex Shuttle integration. 3 files. `#safety-relevant`
- [[icenter-marktool|`MarkTool/`]] — Telesis laser engraver COM-port driver. 4 files. `#safety-relevant`
- [[main-module|`Modules/`]] — `Main.vb`, plus 2 others.
- [[../mocs/icenter-remaining|`PCFNetStudio/`]] — PCFNet integration. 12 files.
- [[../mocs/office-to-shopfloor|`Production/`]] — production-floor logic. 6 files.
- `Resources/` — 354 image / icon assets. Not documented individually; covered by [[../_coverage]] as `config`.
- [[../mocs/office-to-shopfloor|`Sales/`]] — 2 files.
- [[../mocs/smtmanufacturing|`SmtManufacturing/`]] — sheet-metal (Trumpf/Oseon) UI + logic. **63 `.vb`** — MOC + deep-reads of `FlatPatternConverter.vb` + `Part.vb`. `#safety-relevant`
- [[../external-systems/sola-data-connector|`SolaDataConnector/`]] — 2 files.
- [[../mocs/office-to-shopfloor|`UniLink/`]] — 29 files. CSV-import pipeline. `#needs-review`
- [[../mocs/icenter-remaining|`VentDuctConfigurator/`]] — ventilation-duct configurator. 3 files.
- [[../external-systems/webclock|`WebClock/`]] — time-registration UI. 9 files.
- [[../mocs/office-to-shopfloor|`WorkPreparation/`]] — werkvoorbereiding. 5 files.

> **Most entries above are stubs** (not yet written) — that's expected for Phase 2. Phase 3 fills each one. Don't follow the link if it's red unless you're ready to write the note.
