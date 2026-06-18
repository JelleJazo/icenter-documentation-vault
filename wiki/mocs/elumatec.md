---
type: moc
title: "Elumatec subsystem — Map of Content"
status: draft
tags: [moc, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Elumatec subsystem

## What this subsystem does

Owns the CAD→NC pipeline that drives the **Elumatec SBZ140 / SBZ141 profile-milling machines** (the "profile mill" or *ProfMill* in factory vocabulary) and the related profile **saw** that uses the same control loop. Reads incoming work descriptions (DGX/DXF + profile DB), runs a series of *replacement* macros that translate generic features (drilled holes, slots, hinges, double-P notches, flowdrill holes, …) into machine-, material- and profile-specific tool paths, and then either:

- writes an **`.auf`** or **`.eluxml`** NC program to the machine's shared input directory via Elumatec's `NcxExePath` post-processor, or
- (in `FrmComWatcher` mode) listens on **COM1** for sawn-bar acknowledgements from the saw and updates iCenter's part-tracking DB.

**`#safety-relevant`.** Anything that touches step depths, tool selection, clamp positioning, or NC output can move a multi-tonne profile mill — never close a `needs-review` flag here without SME confirmation.

## Scope

`C:\DevOps\iCenter\iCenter\iCENTER\Elumatec\` — 140 `.vb` source files. Cross-cuts into `Modules\Main.vb` (globals — `UseSbzCalculatedDuration`, `ProfMillMachGrps`, the `ProfMillShow3D` switch), `app.config` (Elumatec settings keys), and a number of `ICenterLib.CAD.*` types whose deep documentation will land under `modules/icenterlib-*` in later Phase-3 batches.

## Layout

```
Elumatec/
├── (root)              — UI forms + the EluCadApp god-class + ProfMillJob/Converter
│                         + the saw-list / DGX / sticker-printer pipeline
├── AufSerializer/      — emit Elumatec AUF (XML) NC programs
├── Database/           — read the .epd profile database, Tool DB, Fixtures, Offsets
├── DXF/                — read DXF input (helpers; mostly used by Works/)
├── Machine/            — abstract Sbz14x base + Sbz140Alu/Stl/Rvs + Sbz141Alu
├── NcStructure/        — in-memory model of an NC program (Bar, Cut, Job, Plane)
├── Optimizer/          — saw-job optimizer (CutOptimizer + UI form)
└── Works/              — feature classes (Circle, Drill, Line, Sawcut, SlottedHole, …)
    └── Replacements/   — material-/profile-specific macro substitutions
                         (AluGeneral, AluHinge, DoublePnotch, Flowdrill, OpdekH, …)
```

## Modules (Phase-3 notes)

### Already documented

- [[../modules/elumatec-com-watcher|`FrmComWatcher` + `ClsComWatcher` + `CRs232`]] — serial-port watcher for the saw. **Answers Q-006.**
- [[../modules/elumatec-cad-app|`EluCadApp.vb`]] — central controller; holds the profile DB and DGX table; mediates between forms, machines, and NC export.
- [[../modules/elumatec-machine-base|`Machine\Sbz14x` + `Sbz140Alu`/`Sbz140Stl`/`Sbz140Rvs`/`Sbz141Alu`]] — abstract base + four concrete machine variants. Each variant registers its own list of `WorksReplacement` macros.

### High-value not yet written (recommended order for the next batch)

1. `ProfMillJob.vb` (70 KB) — the runtime representation of one profile-mill job.
2. `ProfMillConverter.vb` (67 KB) — convert generic work descriptions into machine NC structure.
3. `Works\Work.vb` (92 KB) + the per-feature subclasses (`Circle`, `Drill`, `Line`, `Rectangle`, `Sawcut`, `SlottedHole`, `FreeForm`).
4. `Works\Replacements\` — start with `AluGeneral.vb` (119 KB!), then `AluHinge`, `DoublePnotch`, `Flowdrill`, `LargeRectangle`, `OpdekH`, `StlGeneral`. **All `#safety-relevant`.**
5. `AufSerializer\NcProgramAuf.vb` (27 KB) — the AUF-format NC program writer.
6. `NcStructure\Cut.vb` (40 KB) + `NcStructure\Bar.vb` (20 KB) + `Job.vb`.
7. `Database\Profile.vb` (54 KB) — profile DB entity.
8. `CtrlProfMillElu.vb` / `CtrlProfMillCam.vb` (the two big user-controls embedded in `FrmMain`).
9. `EluCadApp.vb` (128 KB) is too big for one note — its overview note already exists; sub-notes per major behaviour cluster will need writing.

### Auxiliary / mostly self-contained

- `LicenseHelper.vb` + `LicenseManagementCenter.vb` — license check for Elumatec's `EluCad` tooling.
- `NcVersionHandler.vb` + `NcwExportProfile.vb` + `NcwViewer.vb` — NCW (Elumatec viewer) glue.
- `MacroDatabase.vb` — macro lookup.
- `ProfileMatcher.vb` (18 KB) — match incoming profile names to DB entries.
- `CutFactory.vb` (22 KB) — factory for `NcStructure\Cut` objects.
- `ClsSawList.vb` (24 KB) — saw-list builder.
- `AutoProfMillProgApproval.vb` (25 KB) — auto-approval of profile-mill programs. **`#safety-relevant`** — confirm what "approval" gates.
- `ClsDgxShoppingList.vb`, `ClsDgxStickerPrinter.vb` — DGX-format file emission for the saw.

## Business rules

### Documented in this batch

- [[../business-rules/elu-max-step-depth|`MaxStepDepth` per material (steel/alu)]] — `#safety-relevant`. From `app.config`: `EluMaxStepDepthSTL = 1.6`, `EluMaxStepDepthALU = 6` (mm). Used by every machine variant via its `MaxStepDepth` property.
- [[../business-rules/elu-large-rectangle-classification|Large-rectangle classification → `WBroach = 1`]] — `#safety-relevant`. From `app.config`: `EluLargeRectangleMinLength = 260`, `EluLargeRectangleMinWidth = 20` (mm). Used in `Works\Rectangle.SetWBroach`.

### Watchlist (Phase 4)

- The 11 replacement macros registered in `Sbz140Alu.New` (and equivalents in the other machine variants). Each is a candidate business rule — when does it fire, what NC it emits, and what input geometry it expects.
- `Sbz14x.ClampAdjustmentDuration(CutLength)` formula: `5 + ClampCount * 4 + 2 * (CutLength / 300)` — duration estimate for cycle-time planning. Likely calibrated to a specific machine + clamp count; document the calibration source.
- `Sbz14x.GeneralNumberOfClamps(CutLength)`: `Math.Ceiling((CutLength - 300) / 400)` capped at `ClampCountPerStation`. Drives both timing and physical clamp setup.
- `Sbz14x.CycleTimeToolChange = 30 sec`, `SetUpTimePartialProgramm = 20 sec`, `PartRotationDuration = 30 sec` — fixed per-event durations used in cycle-time estimates.

## External systems

- [[../external-systems/elumatec-sbz140|Elumatec SBZ140 / SBZ141]] — the physical machines. `#safety-relevant`
- [[../external-systems/dfs-share|`\\jazo.local\dfs\pm\Elumatec_SBZ140\...`]] — the shared directory where NC programs are dropped and where the tool DB, profile DB, fixture DB, and mill-number DB live.
- The Elumatec `NcxExePath` post-processor — external Elumatec EXE, called by `Sbz14x.CreateNCX`. Path comes from `AppVersion.NcxExePath`. **Configurable per `AppVersion`; not in `app.config`.**

## Open questions

- **Q-006 (resolved this batch):** `FrmComWatcher` confirmed as a serial-port watcher for the saw (`MachineId = 2`, COM1 9600 8-N-1). Most of its original sticker-printing pipeline is commented out — **likely partially dead code**. New Q-019 below.
- **Q-019 (new):** `FrmComWatcher`'s `Update` body is mostly commented (`ClsComWatcher.vb` lines 95–120) and a `CB 2023-01-27: added tracking` log was added at form open. Is this still used or vestigial? `#safety-relevant`
- **Q-020 (new):** The `CRs232` class is third-party copy-paste (Corrado Cavalli, codeworks.it, 2001–2005). 50 KB of un-maintained P/Invoke code. Is there a maintained replacement, and is the current copy patched?
- **Q-021 (new):** `MachineXFeedRate = 1000` in `Sbz140Alu` has the inline comment `'Guess`. SME confirmation needed for all four machine variants.
- **Q-022 (new):** `Sbz14x.PartRotationDuration = 30 sec` constant — does this match the actual SBZ rotary table cycle? `#safety-relevant` (affects scheduling).
- **Q-023 (new):** `Sbz140Alu.GoHomeDuration = 8 + CycleTimeToolChange` — confirm.

All logged in [[../needs-review/_index]].

## Related

- [[../architecture/external-surface]] — the boundary view.
- [[../architecture/global-state]] — `ProfMillMachGrps`, `UseSbzCalculatedDuration`, `ProfMillShow3D` are all globals in `Modules\Main.vb`.
- [[../domain-concepts/_index]] — *ProfMill*, *SBZ*, *DGX*, *AUF*, *Sbz140 vs Sbz141*, *ALU vs STL vs RVS*.
