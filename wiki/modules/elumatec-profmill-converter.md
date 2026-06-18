---
type: module
title: "ProfMillConverter — the pipeline orchestrator"
status: done
module: "iCENTER/Elumatec"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\ProfMillConverter.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `ProfMillConverter.vb` — the Elumatec pipeline orchestrator

## Purpose

The single class that drives the whole CAD-→-NC pipeline for one model. Takes a `Modelname` (Creo model), reads its raw `Cut` from the model XML, validates the profile and material, picks the target machine(s), and runs a 17-step `ConvertCut` pipeline that mutates the Cut into NC-ready form, then builds a `ProfMillJob` and triggers post-processing.

**This is the answer to Q-031** ("where does `Sbz14x.MaxStepDepth` get consumed?") and Q-034 ("where does `AutoReplaceMacros.ncd` resolve to?"). See the [Resolved questions](#resolved-questions) section.

## Public surface

```vb
Public Class ProfMillConverter
    Public Sub New(Modelname As String)
    Public Function Optimize() As Boolean
    Public Sub SetWBroach(ByRef machine, ByRef cut)        ' wrapper around Work.SetWBroach
    Public Sub SetWContour(ByRef machine, ByRef cut)       ' wrapper around Work.SetWContour
    Public Sub SetWorkNos(ByRef cut)                       ' renumber Works
    Public Function UpdatePartXml(machine, cuts) As Boolean
    Public ReadOnly Property ProfMillJob As ProfMillJob    ' the last optimised job
    Public Shared ReadOnly Property AutoReplacementMacroFile As String   ' resolves the macro file path
End Class
```

Everything else is `Private`. The big function is `ConvertCut(machine, cut)` (line 114), invoked from `Optimize()` once per target machine.

## `Optimize()` — entry point

`Optimize()` (line 23-112) is the single way to drive the pipeline for a model. Steps:

1. **Read raw Cut**: `_RawCut = CutFactory.ReadFromMonoXml(_Modelname)`.
2. **Ensure the model has been read into `ICenterObjects`** (the [[../architecture/global-state|global cache]]) — if not, call `MyMainForm.ReadModelDataFromXML(_Modelname)`.
3. **Delete any prior manufacturing target files** via `ICenterObject.DeleteManufacturingTargetFiles()`.
4. **Validation**:
   - Reject if `_RawCut.Status = Cut.StatusCode.MissingProeManufData` (throws `"MissingProeManufData"`).
   - Reject if the current `BIdentNo` (from EluCad) differs from `_RawCut.BIdentNoRecognition` (unless current is the `UnknownBIdentNo = "100999"` sentinel). Throws `"current BIdentNo <> recognized BIdentNo: re-promote"`.
   - Reject if `Materiaal` (material) is empty.
   - Reject if `oProfile.MName = UnknownBIdentNo`.
   - Reject if `oProfile.ProfMillMachineId = -1` (first attempts `UpdateProfileMachineId(Materiaal)` to learn it).
5. **Pick target machine**: `Sbz14x.GetSbzMachineByProfileMachineId(oProfile.ProfMillMachineId)`.
6. **Dual-emit decision** (lines 76-80): if the chosen machine is `Sbz140Alu` AND `oEluCad.AppVersion.UseFileBasedSettings`, **also add `Sbz141Alu`** to the target list. So one Optimize run can emit NC programs for both SBZ140 and SBZ141 from the same model. Documented in [[../business-rules/elu-dual-emit-sbz140-sbz141]].
7. **For each target machine**, re-read a fresh `Cut` (so machines start independent), set `CStation = oProfile.GetPreferredSbzStation(machine)`, apply the EluXml `CCopies = 1` hack (lines 87-90, _"tijdelijke fix omdat signature niet klopt"_, Q-067), then call `ConvertCut(machine, cut)`. Per-machine exceptions are aggregated into a `MySystem.ExceptionList`.
8. Throw the aggregate exception list if any machine failed; otherwise return overall success.

## `ConvertCut(machine, cut)` — the 17-step pipeline

This is the heart of the Elumatec pipeline. Each step mutates the in-memory `Cut` in place. Order matters (commented `'CB <date>:` notes document past reorderings).

| # | Step | What it does | Notes |
|---|------|--------------|-------|
| 1 | `ReplaceSpecialWorks(machine, cut)` | For each `WorksReplacement` in `machine.WorksReplaceList`, call `ApplyTo(cut, cut.BIdentNoRecognition)` | See [[../mocs/elumatec-works]] for the 11+ Alu / 4 Stl replacements |
| 2 | `RemoveSawCuts(cut)` | Set `WActive = 0` on every `Sawcut` feature — these are produced elsewhere | Saw cuts have a separate pipeline outside the profile-mill |
| 3 | `AssignTools(machine, cut)` | Pick a `WToolID` for each Work from `machine.GetTools` | Includes "drill→bore" conversions for some features |
| 4 | `RotateWorksOnBottomside(machine, cut)` | If features exist on the bottom side, rotate so the machine can reach them | Side-wissel (side-switch) |
| 5 | `DeleteDuplicates(cut)` | Remove identical features | |
| 6 | `SortWorks(cut)` | Reorder features for cycle-time / tool-change minimisation | |
| 7 | `SetWDepthPost(machine, cut)` | "Naloop" depth — extra through-cut depth | |
| 8 | **`SetMaxStepDepth(machine, cut)`** | **Set `MaxStepDepth` per Work, then call `Work.SplitSteps(MaxStepDepth)`.** Primary source = `machine.ToolDb.GetMaxCut(WToolID)` (per-tool); fallback = `machine.MaxStepDepth` (the `EluMaxStepDepth*` app.config setting). Hard-coded Forster-profile override at line 918. **Q-031 ANSWER.** | See [[../business-rules/elu-tool-max-cut-depth]] |
| 9 | `SetWBroach(machine, cut)` | For each Work with `WBroachDefined = False`, call `Work.SetWBroach(machine.GetTools)` | Includes the `EluLargeRectangle*` rule on `Rectangle` (see [[../business-rules/elu-large-rectangle-classification]]) |
| 10 | `SetWHelixIntr(cut)` | Read `DefaultWHelixIntr` from `AppSettings`, apply to non-replaced Works | New `#business-rule` candidate (Q-070) |
| 11 | `SetWMillDir(machine, cut)` | Set milling direction (`-1` = down-cut / climb milling). Per CB 2023-02-28, **applies to aluminium as well** (previously Stl/Rvs only) | Q-068 |
| 12 | `SetWRotationOfFlowDrillWorksOnBackside(machine, cut)` | Don't flow-drill on backside via side-wissel — burrs interfere with clamp re-positioning | |
| 13 | `SetWorkNos(cut)` | Sequentially renumber Works | |
| 14 | `ReplaceMill(cut, machine.GetTools)` — **only for `Sbz140Stl` / `Sbz140Rvs`** | Replace ordinary mills with bore-mills, or add extra depth | Steel-side only |
| 15 | `SetWContour(machine, cut)` | Set contour mode per Work | Moved here from earlier per `CB 2016-06-08` |
| 16 | `MinimizeMillTools(machine, cut)` | Consolidate mill tool changes | |
| 17 | `SuppressMultiSidedMacros(cut)` | **Entire body commented out** — empty Try/Catch only | Dead-code, but the wrapping `Try` is live (Q-069) |

After the in-place mutation steps, `ConvertCut`:

18. `GetSplitCuts(cut, machine)` → produces a `List(Of Cut)` (a cut may split into multiples for production reasons; private 270-line function).
19. For each split cut:
    - `RepairOriginRemoval(cut)` — repair if a feature removed the origin (e.g. steel "bovenstijl vaste deur"). Was earlier in the pipeline but moved here per CB 2021-10-27 because the original ordering caused `MaxRotation` miscalculation when `Active` flags weren't final.
    - `SetGroupNames(MyCounter)` — sequential group naming.
    - `UpdatePlanesFromWorks()` — derive plane definitions from the features.
    - `CmpCActive()` — recompute whether the cut is active.
    - Update `_ContainsOnlySuppressedByConverter` flag.
20. `UpdatePartXml(machine, cuts)` — persist the result.
21. Build a new `ProfMillJob` from the stored XML, then `PostProcessAndImport()` (runs the NCX post-processor and imports the result).

Return value: AND of `UpdatePartXml` and `PostProcessAndImport`.

## `AutoReplacementMacroFile` — Q-034 answered

```vb
Public Shared ReadOnly Property AutoReplacementMacroFile As String
    Get
        Dim s As String = ICenterLib.CAD.Creo.Environment.GetProManufDir
        If Not s.EndsWith("\") Then s &= "\"
        s &= My.Settings.Properties("AutoReplaceMacros").DefaultValue
        Return s
    End Get
End Property
```

So `AutoReplaceMacros = "Profiles\Resources\AutoReplaceMacros.ncd"` (from `app.config`) is concatenated to `ICenterLib.CAD.Creo.Environment.GetProManufDir` (the **Creo Pro/Manufacture installation directory**). The macro database lives inside the Creo install — **changing it requires write access to the Creo installation tree**, not the iCenter deployment. Confirms Q-034 and is a key piece of context for any future macro-file change request.

## Business rules surfaced here

- **[[../business-rules/elu-tool-max-cut-depth]]** (new) — per-tool `TMaxCut` is the *primary* source of `MaxStepDepth` per Work. `EluMaxStepDepth*` app.config thresholds are the *fallback* (only when `UseTMaxCut = False` — which the inline comment shows is currently `True`).
- **[[../business-rules/elu-max-step-depth]]** (corrected) — note revised to flag that the app.config values are fallback-only.
- **[[../business-rules/elu-forster-thumbhole-step-depth]]** (new) — hard-coded `MaxStepDepth = 6` override for `Rectangle` features on Forster profiles where `WY1 ∈ (48, 50)`, `WDepth > 10`, `WW3 = 5`. The condition appears only on the fallback path so it currently does nothing in production (TMaxCut takes over) — but it documents an intent. Q-065.
- **[[../business-rules/elu-dual-emit-sbz140-sbz141]]** (new) — `Sbz140Alu` jobs additionally emit for `Sbz141Alu` when `AppVersion.UseFileBasedSettings`. One model → two NC files in the same Optimize run.
- **`ReplaceMill` is steel-only** — `Sbz140Stl` and `Sbz140Rvs` get mills replaced by bore-mills or extra depth; aluminium does not. Phase-4 candidate.
- **`SetWMillDir = -1`** — climb-mill direction for **all** machines (per CB 2023-02-28 change from Stl/Rvs-only). Q-068.
- **`DefaultWHelixIntr`** read from iCenter DB `AppSettings` — applied to non-replaced Works. Phase-4 candidate.

## Surprises

1. **`SuppressMultiSidedMacros` body is entirely commented out** (lines 1330-1334). Wrapping `Try` is live, so it would log any exception from the empty body — but it can't throw. Looks like the feature was disabled but the call site kept. Q-069.
2. **`MaxStepDepth` defaults to per-tool `TMaxCut`**, not the `EluMaxStepDepth*` app.config thresholds. My Phase-3a-2 business-rule note ([[../business-rules/elu-max-step-depth]]) needs correction — done in this batch.
3. **The Forster-profile override** (lines 917-922): `If TypeOf oWork Is Rectangle AndAlso oWork.WY1 > 48 AndAlso oWork.WY1 < 50 AndAlso oWork.WDepth > 10 AndAlso CType(oWork, Rectangle).WW3 = 5 Then MaxStepDepth = 6`. The geometry parameters identify a specific Forster-profile thumb-hole. **The condition is inside the fallback `Else` branch** so it doesn't fire when `UseTMaxCut = True` — i.e. in production today, **this override is dead code**. Q-065 to confirm intent.
4. **Reads `CutFactory.ReadFromMonoXml` twice** (line 24 + line 84): once for validation, once per target machine. Each read parses the XML again. Slow but isolates per-machine state.
5. **Dual-emit gate** is opaque — `oEluCad.AppVersion.UseFileBasedSettings`. Phase-3 follow-up to document what determines that flag.
6. **Pipeline order is reordered via inline comments** (CB 2016-06-08 moved `SetWContour`; CB 2021-10-27 moved `RepairOriginRemoval`). Each reorder fixed a bug. Suggests the ordering is delicate — Q-029 reinforced.
7. **`UpdatePartXml` succeeds + `PostProcessAndImport` fails** is reported as a single `success And postProcessed` boolean — caller can't tell which half failed.

## Open questions

- **Q-031 — RESOLVED.** `ProfMillConverter.SetMaxStepDepth` (line 907) is the only callsite. Primary source is per-tool `TMaxCut`, fallback is `Sbz14x.MaxStepDepth`.
- **Q-034 — RESOLVED.** `AutoReplacementMacroFile` = `Creo.Environment.GetProManufDir + app.config[AutoReplaceMacros]`. Macro file lives in Creo's pro-manuf directory, not iCenter's.
- **Q-065 (new):** The Forster-profile thumb-hole override at `SetMaxStepDepth` line 918 lives in the `Else` branch — does it currently fire (i.e. is `UseTMaxCut = False` anywhere in production)? If never, document as dead code. `#safety-relevant`
- **Q-066 (new):** Dual-emit Sbz140Alu→Sbz141Alu depends on `AppVersion.UseFileBasedSettings`. What determines that flag, and how do operators know which machines will receive output? `#safety-relevant`
- **Q-067 (new):** `MyCut.CCopies = 1` is set for EluXml-type machines with comment _"tijdelijke fix omdat signature niet klopt"_. Has the signature mismatch been fixed; can the override be removed? `#needs-review`
- **Q-068 (new):** `SetWMillDir = -1` now applies to aluminium per CB 2023-02-28. Confirm this matches current factory practice. `#safety-relevant`
- **Q-069 (new):** `SuppressMultiSidedMacros` body is commented out; only the empty `Try`/`Catch` remains. Remove or restore? `#needs-review`
- **Q-070 (new):** `SetWHelixIntr` reads `DefaultWHelixIntr` from iCenter DB `AppSettings`. Document this as a business rule. `#safety-relevant`

Logged in [[../needs-review/_index]].

## Resolved questions

- **Q-031 — `Sbz14x.MaxStepDepth` callsites** → `ProfMillConverter.SetMaxStepDepth` (line 907) only.
- **Q-034 — `AutoReplaceMacros.ncd` path** → `Creo.Environment.GetProManufDir + app.config[AutoReplaceMacros]`.

## Related

- [[../mocs/elumatec-ncpipeline|NC pipeline MOC]]
- [[../mocs/elumatec-works|Works/Replacements MOC]]
- [[elumatec-profmill-job|ProfMillJob]] — the output container that ConvertCut creates and triggers
- [[elumatec-machine-base|`Sbz14x`]] — provides `MaxStepDepth`, `ToolDb`, `WorksReplaceList`, `CreateNCX`
- [[../business-rules/elu-max-step-depth]] (corrected this batch)
- [[../business-rules/elu-tool-max-cut-depth]] (new)
- [[../business-rules/elu-forster-thumbhole-step-depth]] (new)
- [[../business-rules/elu-dual-emit-sbz140-sbz141]] (new)

## Coverage

`_coverage.md`: `Elumatec\ProfMillConverter.vb` → `done`.
