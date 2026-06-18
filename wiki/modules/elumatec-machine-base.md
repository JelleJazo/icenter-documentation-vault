---
type: module
title: "Elumatec machine variants (Sbz14x family)"
status: done
module: "iCENTER/Elumatec/Machine"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\Machine\\Sbz14x.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\Machine\\Sbz140Alu.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\Machine\\Sbz140Stl.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\Machine\\Sbz140Rvs.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\Machine\\Sbz141Alu.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Elumatec machine variants — `Sbz14x` family

## Purpose

`Sbz14x` is the abstract base class for the four supported Elumatec profile-mill variants. Each concrete subclass declares its **material identity**, its **per-machine cycle-time constants**, and **the ordered list of `WorksReplacement` macros** that translate generic feature requests into machine-specific tool paths. The base class owns the **NC-program generation pipeline** that runs Elumatec's external post-processor.

## Type hierarchy

```
MustInherit Sbz14x                     (abstract base)
├── Sbz140Alu     MachineName = "SBZ140_ALU"
├── Sbz140Stl     MachineName = "SBZ140_STL"
├── Sbz140Rvs     MachineName = "SBZ140_RVS"
└── Sbz141Alu     MachineName = "SBZ141_ALU"
```

`Sbz14x.KnownMachineNames` (static field) holds those four names; `GetSbzMachineByName` and `GetSbzMachineByProfileMachineId` are the two factories. All concrete machines hard-code their `MachineName`.

Material code legend:
- **ALU** — aluminium.
- **STL** — steel.
- **RVS** — Dutch *roestvrij staal* = stainless steel.

> The two SBZ140 steel variants (`Sbz140Stl` and `Sbz140Rvs`) both read `MaxStepDepth` from the same `EluMaxStepDepthSTL` setting. The distinction between them at the *machine* level is non-obvious — likely down to which tool DB / fixture DB they reference. `#needs-review`

## Abstract surface (must be overridden)

```vb
Public MustOverride Property WorksReplaceList As List(Of WorksReplacement)
Public MustOverride ReadOnly Property MaxStepDepth As Double               ' mm
Public MustOverride ReadOnly Property Name As String                       ' MachineName
Public MustOverride ReadOnly Property MachineXFeedRate As Double           ' mm/sec
Public MustOverride ReadOnly Property ToolRetractFeed As Double            ' mm/sec
Public MustOverride ReadOnly Property GoHomeDuration As Double             ' sec
Public MustOverride ReadOnly Property ClampCountPerStation As Integer
Public MustOverride ReadOnly Property NcType As EluCadApp.NcType           ' Auf | EluXml
Public MustOverride ReadOnly Property AutonomousClampPositioning As Boolean
```

## Per-machine values (verified from source, 2026-06-18)

| Property | Sbz140Alu | Sbz140Stl | Sbz140Rvs | Sbz141Alu |
|----------|-----------|-----------|-----------|-----------|
| `MaxStepDepth` | `EluMaxStepDepthALU` (6 mm) | `EluMaxStepDepthSTL` (1.6 mm) | `EluMaxStepDepthSTL` (1.6 mm) | `EluMaxStepDepthALU` (6 mm) |
| `MachineXFeedRate` | `1000 'Guess` | _(see file)_ | _(see file)_ | _(see file)_ |
| `GoHomeDuration` | `8 + CycleTimeToolChange` (= 38 sec) | _(see file)_ | _(see file)_ | _(see file)_ |
| `ClampCountPerStation` | `5` | _(see file)_ | _(see file)_ | _(see file)_ |
| `ToolRetractFeed` | `1000` mm/sec | _(see file)_ | _(see file)_ | _(see file)_ |
| `AutonomousClampPositioning` | `False` | _(see file)_ | _(see file)_ | _(see file)_ |
| `NcType` | `Auf` | _(see file)_ | _(see file)_ | _(see file)_ |
| Tool DB path | `\\jazo.local\dfs\pm\Elumatec_SBZ140\Directories\database\SBZ140_ALU.nct` | _(see file)_ | _(see file)_ | _(see file)_ |

**`Sbz140Alu`'s `MachineXFeedRate = 1000 'Guess` is explicitly labelled a guess in the source** (`Sbz140Alu.vb` line 27). `#safety-relevant` `#needs-review` — feed rate drives cycle-time estimates and toolpath timing. **Q-021** in [[../needs-review/_index]].

## Shared (non-overridable) constants

| Const / property | Value | Used for |
|------------------|-------|----------|
| `PartRotationDuration` | `30 sec` | duration estimate when the saw rotates a part on the rotary table |
| `SetUpTimePartialProgramm` | `20 sec` | one-time setup before running each part of an NC program |
| `CycleTimeToolChange` | `30 sec` | duration estimate for each tool change |
| `ClampAdjustmentDuration(CutLength)` | `5 + ClampCount * 4 + 2 * (CutLength / 300) sec` | global clamp-positioning duration; uses `GeneralNumberOfClamps(CutLength)` |
| `GeneralNumberOfClamps(CutLength)` | `Ceiling((CutLength - 300) / 400)` capped at `ClampCountPerStation`, floored at 2 | physical clamp count |

All of these feed cycle-time estimation. Per [[../architecture/global-state]] `UseSbzCalculatedDuration = True` means iCenter prefers the SBZ-side calculated duration over its own estimate — these constants therefore inform **estimates** more than they drive machine behaviour, but estimates flow into production planning (ISAH).

## What each machine constructor sets up

Each concrete `Sbz140*`/`Sbz141*` constructor:

1. Calls `MyBase.New(toolDbPath)` with the variant-specific tool DB UNC path. Base reads the tool DB (`ReadToolDatabase` with 5 retries × 1 s wait — handles `IOException` from a parallel-running post-processor; see `Sbz14x.vb` lines 89–105).
2. Appends a sequence of `WorksReplacement` macros to `WorksReplaceList`. **The order matters** — replacements run in registration order during NC export.

For example `Sbz140Alu.New` registers (in order):

1. `Flowdrill` — flow-drilled holes (heat-formed bushings).
2. `DoublePnotch` — double-P notch joint.
3. `AluSinglePnotch` — single-P-on-double-P joint (when no undersill).
4. `LargeRectangle` — large rectangles in front panel of door needle become free-form with controlled milling direction. **Uses the `EluLargeRectangle*` thresholds — see [[../business-rules/elu-large-rectangle-classification]].**
5. `OpdekH` — "opdek H profile" rotation rule.
6. `RDHS27Notch` — HS27 grille-door plank notch.
7. `AluHinge` — thumb holes in (Double) P-profile.
8. `AluSRkom` — HS27 sham-grille-plank keyway.
9. `AluHUPO` — hex blind rivet nuts (Blinkdklink moeren).
10. `ExtraLength` — add over-length for short parts.
11. `AluGeneral` — general replacements (catch-all).

This list is a **direct map of the alu-side operational features the SBZ140 supports through iCenter**. Each replacement deserves its own [[../business-rules/_index|business-rule]] note during Phase 4. All `#safety-relevant`.

## NC program generation — `CreateNCX(NcwFile, OutputFile, IncludeViewable)`

Inherits from `Sbz14x` (lines 113–209). Spawns Elumatec's external `NcxExePath` post-processor in sequence:

- **For `NcType.Auf`** — runs `NcxExePath` *four* times with different post-processor configurations:
  1. `PostProcPathAuf` → opdrachtgedeelte (order section)
  2. `PostProcPathBea` → bewerkingen (operations)
  3. `PostProcPathKon` → contouren (contours)
  4. `PostProcPathNcw` → viewable `.nco` (only if `IncludeViewable = True`)

  Each invocation receives `w=NcwFile t=ToolDbFilepath c=PostProcPathConfig p=<one of the post-procs> m=PostProcPathMachin o=OutputFile`. All synchronous (`WaitForExit`).

- **For `NcType.EluXml`** — single invocation with `PostProcPathEluXml` and trailing `/p` flag.

If `OutputFile` doesn't exist after all calls, returns `Nothing`.

**Surprises:**

- All `Catch ex As Exception` blocks swallow errors and return `Nothing`. The caller has to test for `Nothing` to detect a failed post-processor run.
- The post-processor EXE path comes from `AppVersion`, not `app.config` — meaning the **installed Elumatec EluCad version** determines which post-processor runs. Switching EluCad versions can silently change NC output. `#safety-relevant`
- `UseShellExecute = True` with `WindowStyle = Hidden` — the post-processor *could* pop up its own UI under some configurations.

## Static helpers

- `GetMachGrpCode(MachineName)` — read the first machine-group code from `prodMachines.MachGrpCodes` (split on `";"`, uppercase). Used to map a machine to a routing.
- `GetSbzMachineByName(MachineName)` — factory, returns the right concrete class.
- `GetSbzMachineByProfileMachineId(profMillMachineId)` — go via `prodMachines.GetMachineInfo(id, "MachineName")` then the above.
- `GetKnownMachines()` — instantiate all four.

## Business rules surfaced here

- [[../business-rules/elu-max-step-depth|`MaxStepDepth` per material]] — `EluMaxStepDepth*` thresholds. **`#safety-relevant`**
- [[../business-rules/elu-large-rectangle-classification|Large rectangle → broach classification]] — `EluLargeRectangle*` thresholds. **`#safety-relevant`**
- The ordered list of `WorksReplacement` registrations in each `Sbz*.New` constructor — each is a candidate rule for Phase 4.
- The clamp-count formula `Ceiling((CutLength - 300) / 400)` and its hard-coded floor of 2 are physical-setup rules. `#needs-review` whether these reflect SBZ140 reality.

## External systems touched

- [[../external-systems/elumatec-sbz140|Elumatec SBZ140 / SBZ141]] — via NC programs and the per-machine tool DB. **`#safety-relevant`**
- [[../external-systems/dfs-share|`\\jazo.local\dfs\pm\Elumatec_SBZ140\Directories\database\*.nct`]] — tool DBs.
- Elumatec EluCad post-processor (`AppVersion.NcxExePath`) — external EXE.

## Domain concepts

- *SBZ*, *Sbz140 / Sbz141*, *ALU / STL / RVS*, *ProfMill*, *NCW*, *AUF*, *EluXml*, *Tool DB*, *Clamp station*, *Step depth*. See [[../domain-concepts/_index]].

## Open questions

- **Q-021:** `Sbz140Alu.MachineXFeedRate = 1000 'Guess`. Confirm all four machine-variant feed rates with SME. `#safety-relevant`
- **Q-022:** `PartRotationDuration = 30 sec` constant — calibration source? `#safety-relevant`
- **Q-023:** `Sbz140Alu.GoHomeDuration = 8 + CycleTimeToolChange` — confirm.
- **Q-027 (new):** Difference between `Sbz140Stl` and `Sbz140Rvs`? Both read the same `EluMaxStepDepthSTL` setting; what distinguishes them at the machine? `#needs-review`
- **Q-028 (new):** `CreateNCX` Auf path runs the post-processor with the same `OutputFile` 3–4 times. Confirm the post-processor *appends* and isn't *overwriting* between calls. `#safety-relevant`
- **Q-029 (new):** `WorksReplaceList` order — what happens if a user-added macro changes the order? Confirm with SME that the order is intentional and load-bearing.

## Coverage

`_coverage.md`:

- `Elumatec\Machine\Sbz14x.vb` → `done`
- `Elumatec\Machine\Sbz140Alu.vb` → `done`
- `Elumatec\Machine\Sbz140Stl.vb` → `needs-review` (file values not fully transcribed in this batch)
- `Elumatec\Machine\Sbz140Rvs.vb` → `needs-review` (same)
- `Elumatec\Machine\Sbz141Alu.vb` → `needs-review` (same)
