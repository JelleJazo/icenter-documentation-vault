---
type: moc
title: "Elumatec Works/ pipeline — Map of Content"
status: draft
tags: [moc, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Elumatec `Works/` pipeline

## What this hub covers

`iCENTER\Elumatec\Works\` is the heart of Elumatec NC translation. 40 `.vb` files split into two layers:

1. **Feature classes** (`Works\*.vb` at the root) — one class per *kind* of machining feature. All inherit from `Work` (abstract).
2. **Replacement macros** (`Works\Replacements\*.vb`) — per-profile, per-material rewrites. Each implements `WorksReplacement.ApplyTo(Cut, BIdentNo)` and is registered into a machine's `WorksReplaceList` (see [[../modules/elumatec-machine-base]]).

The full pipeline (per cut):

```
Generic Cut (list of Work features)
   │
   │  for each WorksReplacement in machine.WorksReplaceList   ← order matters
   │       replacement.ApplyTo(cut, BIdentNo)                  ← may add/remove/rewrite Works
   │
   ▼
Specialised Cut → NC output (AUF / EluXml)
```

A `Cut` represents one physical bar (one profile-mill workpiece). `BIdentNo` is the profile ID from the `.epd` profile database (e.g. `"100142"` = P-profiel). Most replacement macros dispatch on `BIdentNo` — large parts of the system are "if profile X then do Y".

`#safety-relevant`. Any rewrite that adds/removes/changes a `Work` ultimately changes the NC program sent to the SBZ140/141 mill.

## Feature classes (in `Works\`)

| File | KB | Inherits | Role |
|------|---:|----------|------|
| [[../modules/elumatec-work-base|`Work.vb`]] | 92 | (abstract) | base class — fields `WX1/WY1/WDepth/WW1/WToolID/WSide/...`, `Sides` enum, `RuntimeManipulations` hook, ~25 `MustOverride` methods |
| `Circle.vb` | 20 | `Work` | round through-hole / pocket |
| `Drill.vb` | 10 | `Work` | drilled hole |
| `Line.vb` | 12 | `Work` | linear cut |
| `Rectangle.vb` | 17 | `Work` | rectangular pocket. **Documented in part by [[../business-rules/elu-large-rectangle-classification]]** |
| `SlottedHole.vb` | 19 | `Work` | oblong slot |
| `Sawcut.vb` | 14 | `Work` | saw operation |
| `FreeForm.vb` | 24 | `Work` | arbitrary polyline pocket (with optional arc segments). Used as a fallback / "express anything" target by replacements |
| `FreeFormPoint.vb` | 9 | (point struct) | one point in a `FreeForm` polyline |
| `DxfFreeForm.vb` | 1 | `Work` | freeform from DXF |
| `Macro.vb` | 10 | `Work` | references a stored macro (e.g. `EC00601`) from `AutoReplaceMacros.ncd` |
| `Group.vb` | 12 | `Work` | container — a group of `Work`s treated as a unit |
| `Deburr.vb` | 7 | `Work` | deburr / countersink operation |
| `FrmTestDrwProfile.vb` | 4 | Form | test/debug dialog for drawing profiles. Likely dev-only. `#needs-review` (dead code?) |

## Replacement macros (in `Works\Replacements\`)

> Each replacement implements `WorksReplacement.ApplyTo(Cut, BIdentNo)`. They run in registration order; see [[../modules/elumatec-machine-base]] for the per-machine list. **All `#safety-relevant`.**

| File | KB | Description | Registered by |
|------|---:|-------------|---------------|
| [[../modules/elumatec-works-replacement-base|`WorksReplacement.vb`]] | 2 | abstract base | — |
| `Flowdrill.vb` | 16 | Replace circles ≥ Ø9.3 mm with flow-drill macro `EC00601` (with countersink) or `EC00054` (without). Heavy profile-specific corrections for `BIdentNo` 100142, 100381. **Documented in [[../modules/elumatec-replacement-flowdrill]]** | Sbz140Alu, Sbz141Alu |
| `DoublePnotch.vb` | 19 | Replace double-P notch joints | Sbz140Alu, Sbz141Alu |
| `AluSinglePnotch.vb` | 6 | Single-P-on-double-P (when no undersill, beside door) | Sbz140Alu, Sbz141Alu |
| `LargeRectangle.vb` | 13 | Replace large rectangles in door-needle / opdekje profiles with FreeForm. **Profile-restricted to 100116/100285/100103, hard-coded >200×>20**. **Documented in [[../modules/elumatec-replacement-large-rectangle]]** | Sbz140Alu, Sbz141Alu |
| `OpdekH.vb` | 3 | "Opdek-H" profile rotation rule (if a thumb-hole "uitraveling" exists, rotate profile 90°) | Sbz140Alu, Sbz141Alu |
| `RDHS27Notch.vb` | 4 | HS27 grille-door plank ("roosterdeur") notch handler | Sbz140Alu, Sbz141Alu |
| `AluHinge.vb` | 18 | Thumb-holes ("duimgaten") in (Double) P-profile | Sbz140Alu, Sbz141Alu |
| `AluSRkom.vb` | 6 | HS27 sham-grille-plank ("schijnroosterplank") keyway | Sbz140Alu, Sbz141Alu |
| `AluHUPO.vb` | 4 | Hex blind-rivet nuts ("Blinkdklink moeren"). Naming likely a typo of "Blindklink" | Sbz140Alu, Sbz141Alu |
| `ExtraLength.vb` | 3 | Add over-length to short parts (delegates to `ExtraLengthMacro` via `ExtraLengthMacroFactory`) | Sbz140Alu, Sbz141Alu |
| `ExtraLengthMacro.vb` | 1 | macro for extra length | — |
| `ExtraLengthMacroFactory.vb` | 2 | factory for `ExtraLengthMacro` | `ExtraLength` |
| [[../modules/elumatec-replacement-alu-general|`AluGeneral.vb`]] | **119** | **catch-all** — one giant `Select Case BIdentNo` with profile-specific tweaks. Documented as overview only | Sbz140Alu, Sbz141Alu (last in list) |
| `StlGeneral.vb` | 13 | catch-all for steel variants | Sbz140Stl, Sbz140Rvs |
| `StlHinge.vb` | 3 | steel-variant hinge | Sbz140Stl, Sbz140Rvs |
| `StlDoublePnotch.vb` | 6 | steel-variant double-P notch | Sbz140Stl, Sbz140Rvs |
| `StlFlowDrill.vb` | 5 | steel-variant flow-drill | Sbz140Stl, Sbz140Rvs |
| `DoorPlankCalibration.vb` | 6 | door-plank calibration ("kalibratie") replacement | _(check registrations)_ |
| `DoorPlankCalibrationMessageCutOff.vb` | 1 | message component | — |
| `IDoorPlankCalibrationMessage.vb` | <1 | interface | — |
| `UCDoorPlankCalibration.vb` | 14 | user-control for door-plank calibration UI | embedded into a form |
| `RuntimeManipulation.vb` | 1 | single manipulation instruction class | `Work.RuntimeManipulations` |
| `RuntimeManipulationInstruction.vb` | <1 | instruction sub-type | — |
| `RuntimeManipulations.vb` | 6 | collection of `RuntimeManipulation` on a `Work` | `Work` property |
| `WorksTranslation.vb` | <1 | translation strings | — |

## The macro-file dependency

Most replacements call `GetWorksByMacro(oCut, "EC<id>")` (or `GetGroupByMacro`) on the base class. This reads from a macro database file referenced by `ProfMillConverter.AutoReplacementMacroFile` — which resolves to the `app.config` setting `AutoReplaceMacros = Profiles\Resources\AutoReplaceMacros.ncd`.

> The macro `.ncd` file lives next to iCenter. **Changing macros means changing an external file**, not just `app.config` or code. Specific macro IDs known to be used: `EC00601` (flow-drill with countersink), `EC00054` (flow-drill without). More macro IDs are scattered through `AluGeneral` and other replacements. **`#safety-relevant`** — macro file is part of the deployment surface and should be version-controlled with iCenter.

## Profile IDs (BIdentNo) referenced in replacements

Partial list found by reading the replacements:

| BIdentNo | Used in | Note (Dutch) |
|----------|---------|-------|
| `100103` | LargeRectangle, AluGeneral | (opdekje) |
| `100104` | AluGeneral | deurplank / deurframe NS |
| `100105` | AluGeneral | deurplank / deurframe NS |
| `100114` | AluGeneral | deurplank + scharniergaten |
| `100115` | AluGeneral | deurplank / deurframe NS |
| `100116` | LargeRectangle | deurnaaldprofiel |
| `100142` | Flowdrill, AluGeneral | P-profiel |
| `100143` | AluGeneral | deurplank + scharniergaten |
| `100180` | AluGeneral | Koker 40×20×2 (ontwateringsgaten) |
| `100285` | LargeRectangle | (opdekje variant) |
| `100292`, `100293`, `100294` | AluGeneral | deurplank |
| `100381` | Flowdrill | (paired with 100142) |
| `100999` | `EluCadApp.UnknownBIdentNo` | reserved sentinel |

Build the full list (Phase 4) by `grep BIdentNo iCENTER\Elumatec\Works\Replacements\*.vb`. Each is a [[../domain-concepts/_index|domain concept]] candidate — needs a human-readable name from SME.

## Business rules surfaced in this batch

- [[../business-rules/elu-large-rectangle-classification|Large-rectangle `WBroach` classification]] — **separate** from the LargeRectangle replacement. Different thresholds (260×20 vs 200×20), different scope (all profiles vs three specific door-needle profiles), different effect (`WBroach = 1` flag vs full FreeForm rewrite). The note has been updated to flag the distinction.
- (new) [[../business-rules/elu-flowdrill-replacement|Flow-drill replacement rule]] — circles/drills of Ø9.3 mm (or any circle deeper than 6 mm) get rewritten as flow-drill macros. Profile-specific recovery branches for misrecognised holes.
- (new) [[../business-rules/elu-largerect-freeform-replacement|Large rectangle → FreeForm for door-needle profiles]] — when on profile 100116, 100285, or 100103.

## Open questions (added this batch)

- **Q-034:** the macro-file path `Profiles\Resources\AutoReplaceMacros.ncd` is relative — to what? Likely the EluCad install root via `AppVersion`. Confirm. `#safety-relevant`
- **Q-035:** what does `BIdentNo = "100381"` represent? Paired with 100142 in the Flowdrill rear-side recovery branch (`Flowdrill.vb` line 120). `#needs-review`
- **Q-036:** the order of replacement registration in `Sbz140Alu.New` matters (`AluGeneral` last so it doesn't undo prior, specific replacements). Confirm with SME no recent registration was added out of order.
- **Q-037:** Stl-side machines (`Sbz140Stl`, `Sbz140Rvs`) register a much smaller replacement list than Alu. Is that intentional, or is the steel side just under-implemented?
- **Q-038:** `FrmTestDrwProfile.vb` looks dev-only. Confirm `#dead-code` candidate.

All logged in [[../needs-review/_index]].

## Related

- [[elumatec|Elumatec subsystem MOC]]
- [[../modules/elumatec-machine-base]] — registers the replacement lists
- [[../modules/elumatec-cad-app]] — owns `ProfileDbFile` and `AutoReplacementMacroFile` access
- [[../business-rules/_index]]
