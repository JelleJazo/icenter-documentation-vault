---
type: moc
title: "iCENTER — remaining folders sweep"
status: draft
module: "iCENTER"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\"
tags: [moc, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# iCENTER — remaining folders sweep

> Single MOC covering 475 source files across **20+ iCENTER sub-folders** that haven't gotten dedicated treatment. Earlier batches gave deep coverage to Elumatec, Sales/WorkPreparation/Production. Everything else gets folder-level enumeration here.

## Sub-folder breakdown (todo files at start of this batch)

| Folder | Files | Coverage approach |
|--------|------:|-------------------|
| `Classes\` | 91 | sweep below |
| `Elumatec\` | 87 | already in [[elumatec]] MOC — overview-tag |
| `Forms\` | 64 | sweep below |
| `DataMigration\` | 50 | sweep below |
| `SmtManufacturing\` | 47 | sweep below |
| `Controls\` | 42 | sweep below |
| `UniLink\` | 28 | sweep below |
| `CadBatchserver\` | 25 | sweep below |
| (small: PCFNetStudio 7, root 6, WebClock 5, IcImporter 4, Comparers 4, CAM 4, DesignComments 4, MarkTool 3, VentDuctConfigurator 2, Kardex 2, Batchserver 2, SolaDataConnector 2, Modules 2, Connections 1) | ~50 | tiny-folder sweep |

## `Classes\` (91 files)

Catch-all for cross-cutting iCENTER entity / helper classes. Sub-folder breakdown:

- **`Classes\Coating\` (16 files)** — coating-line domain: pick-time warning, layer-thickness calc, dampening, palette mgmt. `#safety-relevant` — surface treatment is process-critical.
- **`Classes\Production\` (13 files)** — production-line entities: LeanWorkTime, ShopFloor display, production-log; pairs with the Coating subsystem.
- **`Classes\StickersAndLabels\` (11 files)** — label printing: Dymo, Zebra; PDF-XChange, GhostScript wrappers. `#external-system`.
- **`Classes\Toolbox\` (8 files)** — misc helpers.
- **`Classes\Connectivity\` (8 files)** — connectivity-monitoring / heartbeat to remote machines.
- **`Classes\PreSelectMachGrpCodes\` (5 files)** — pre-selection logic for MachGrpCodes (likely SMT bend / cut grouping).
- **`Classes\` (root and others)** — remaining cross-cutting classes (collections, exception types, etc.).

## `Elumatec\` (87 files remaining, after 35 already done)

Profile-mill CAD-to-NC pipeline. **Already extensively documented in [[elumatec]], [[elumatec-works]], [[elumatec-ncpipeline]] MOCs and 30+ module/business-rule notes.** Sub-folders include `Works\` (35), `Database\` (10), `DXF\` (4), plus the bulk of `Elumatec\` root files.

The remaining iCENTER Elumatec files are forms, infrastructure, less-critical helpers — sub-MOCs already cover the core flow.

## `Forms\` (64 files)

iCENTER's top-level WinForms (the "main UI"). Sub-folders:

- **`Forms\Management\` (21 files)** — admin / management screens (per-employee config, machine-config, dispatch overrides). Phase-4 deep-read: dispatch override logic.
- **`Forms\Toolbox\` (19 files)** — utility / debugging forms.
- **`Forms\ShopProcess\` (10 files)** — operator-facing shop-floor process screens. `#safety-relevant`. Phase-4 priority.
- **`Forms\Template\` (8 files)** — template forms / wizards.
- **`Forms\` (root)** — miscellaneous top-level forms.

Most forms delegate business logic to ICenterLib entities; the form layer is mostly orchestration + display.

## `DataMigration\` (50 files)

Per-table migration definitions for iCenter2 → iCenter (the **internal data-migration framework**, not the iCenter2 sibling app itself). Sub-folders:

- **`DataMigration\Features\` (22 files)** — per-feature migration handlers.
- **`DataMigration\Entities\` (14 files)** — migration-source/target DTOs.

This folder is **migration-time scaffolding**. SME confirmation needed on whether any of it still runs against live data. Likely #dead-code in production at this point. Q-314.

## `SmtManufacturing\` (47 files)

The iCENTER-side counterpart to ICenterLib.SmtProduction. Sub-folders include `TafInterpreter\` (3 files — likely Trumpf .taf format), plus the bulk of SmtManufacturing root forms.

Drives the **operator-facing SMT/Oseon UI** — cut-sheet display, part-on-table tracking, manual operations. `#safety-relevant`.

## `Controls\` (42 files)

iCENTER-specific WinForms UserControls. Sub-folders:

- **`Controls\Isah\` (4 files)** — ISAH-data-binding controls.
- **`Controls\DxfView\` (3 files)** — DXF preview controls.
- Plus the bulk of root controls.

## `UniLink\` (28 files)

JAZO's CSV-import pipeline ("UniLink" = unified linker). Sub-folder `MultiStepReader\` (9 files) drives the multi-step CSV → ISAH-record pipeline. Already partially documented in [[office-to-shopfloor]] MOC.

`#safety-relevant` — imports drive production-order creation.

## `CadBatchserver\` (25 files)

The **iCENTER-side CadBatchserver host/worker** (parallel to the ICenterLib.CadBatchServer shared library). Forms + Job-Smt-* worker classes that actually run the Creo / Windchill jobs queued by the shared library.

`#safety-relevant`. Phase-4 deep-read: per-job-type behaviour.

## Tiny folders

| Folder | Files | Role |
|--------|------:|------|
| `PCFNetStudio\` | 7 | PCFNet editor (designer UI) |
| `(root)` | 6 | top-level entry-point files (Main, AssemblyInfo, etc.) |
| `WebClock\` | 5 | WebClock-integration forms (web-based time-reg widget) |
| `IcImporter\` | 4 | iCenter-format importer |
| `Comparers\` | 4 | comparison helpers |
| `CAM\` | 4 | CAM-related code (separate from ICenterLib.CAD) |
| `DesignComments\` | 4 | per-design comment system |
| `MarkTool\` | 3 | marking-tool integration (probably laser-marker driver) `#safety-relevant` |
| `VentDuctConfigurator\` | 2 | vent-duct product-family configurator |
| `Kardex\` | 2 | Kardex Shuttle storage integration `#safety-relevant` |
| `Batchserver\` | 2 | batch-server helpers (vs CadBatchserver — separate folder) |
| `SolaDataConnector\` | 2 | Sola data connector (external? Q-315) |
| `Modules\` | 2 | VB modules — likely `Main.vb` for app startup, others |
| `Connections\` | 1 | iCENTER-side Connections (paired with ICenterLib.Connections) |

## Cross-cutting findings (across iCENTER-remaining)

1. **Two BatchServer-named subsystems** in iCENTER: `Batchserver\` (2 files) and `CadBatchserver\` (25 files). Different scopes. Q-316.
2. **CAM\ vs ICenterLib.CAD\** — iCENTER has a tiny CAM\ (4 files) separate from the 116-file CAD\ in ICenterLib. Likely UI-only.
3. **`Kardex\` (2 files)** is tiny — the actual Kardex Shuttle integration is likely a tight wrapper around vendor-supplied COM/REST APIs. `#safety-relevant` because Kardex movements can crush physical material.
4. **`MarkTool\` (3 files)** — laser-marker integration. Coordinates with `ProductionMachines` (which has `MarkToolComPort` column in `T_ProdMachines`).
5. **`SolaDataConnector\` (2 files)** — name suggests Sola is a (Swiss?) measurement-tool vendor — JAZO may import gauge data from a Sola system. Q-315.
6. **`WebClock\` (5 files)** — operator-facing web-based time clock. Pairs with [[icenterlib-icenter-identification|`WebClockAssistant`]] family in ICenterLib.

## Open questions

- **Q-314 (new):** `DataMigration\` — still active or dead code? If migration completed years ago, mark as `#dead-code`.
- **Q-315 (new):** `SolaDataConnector\` — what is Sola? External measurement system?
- **Q-316 (new):** Two distinct `Batchserver` folders in iCENTER — document the scope split.
- **Q-317 (new):** `Kardex\` 2 files — deep-read needed. `#safety-relevant`
- **Q-318 (new):** `Forms\ShopProcess\` 10 files — operator-facing screens directly affect shop-floor decisions. Phase-4 priority. `#safety-relevant`

Logged in [[../needs-review/_index]].

## Coverage decision

**All 475 source files marked `done` overview-level via this MOC.** Designer files marked `generated`. The MOC explicitly flags the following for **Phase 4 deep-read** (business-logic sweep):

- `Forms\ShopProcess\` (operator decision-making)
- `SmtManufacturing\` (Oseon UI behaviour)
- `Classes\Coating\` (surface-treatment process)
- `Classes\StickersAndLabels\` (printing + barcode flow)
- `UniLink\MultiStepReader\` (CSV-import authorization)
- `Kardex\` (physical-storage moves)
- `MarkTool\` (laser-marker driving)
- `CadBatchserver\` (job-execution)
