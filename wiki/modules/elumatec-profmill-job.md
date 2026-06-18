---
type: module
title: "ProfMillJob — runtime job representation"
status: done
module: "iCENTER/Elumatec"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\ProfMillJob.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `ProfMillJob.vb` — runtime job representation

> **Status: overview note.** 1352 lines, ~60 public methods across 12 behavioural clusters. This page documents the *shape* and the *cluster* responsibilities; per-method deep-dives are deferred until specific business-rule notes need them.

## Purpose

The **runtime container** for one Elumatec profile-mill job. Holds the `NcStructure` `List(Of Job)` plus the target machine, tool DB, NCX log, cached serialised forms, release-level, and a number of state flags. Built by [[elumatec-profmill-converter|`ProfMillConverter.Optimize()`]] once per target machine.

Where `Job` / `Bar` / `Cut` / `Plane` are pure data classes ([[elumatec-ncstructure-hierarchy]]), `ProfMillJob` is the **behaviour-bearing wrapper**: import / export / post-process / cycle-time / refactor / merge. It's also the point where Elumatec connects back into the wider iCenter app (ManufPart, Windchill release-level, batchserver toolkit, runtime manipulations).

## State

```vb
Public KeepAufAsSeperateFile As Boolean = True              ' debug aid

Private lJobs As New List(Of Job)                           ' the NcStructure jobs
Private oMachine As Machine.Sbz14x                           ' target machine
Private oTools As DataTable                                  ' = oMachine.GetTools at construction
Private dtNcxLog As New DataTable("NcxLog")                  ' NCX (post-processor) log rows
Private oEluCad As New EluCadApp
Private oBatchServer As New Batchserver.BatchserverToolkit   ' status updates back to CAD batchserver

Public ReadOnly Property Modelname As String
Private _EcwString As String = Nothing                        ' cached ECW serialisation
Private _NCAsString As String = Nothing                       ' cached NC serialisation
Private _ReleaseLevel As ICenterLib.CAD.PLM.EPMDocument.ReleaseLevel = Work
Private _InvalidNCFile As Boolean = False
Private _IgnoreCnc As Boolean = False
Private _ContainsFullLengthOper As Boolean = False
Private _AutoApproveProfMillVersion As Integer = 0
Private _AppVersionId As Integer = 1
Private _NcVersion As Integer = 0
Private _MachineTimeStamp As DateTime

Public MasterProfMillJob As ProfMillJob                       ' set when merged into a master
Private ReadOnly Property ManufPart As CAM.ManufPart
Public Const ManufPartManufType As CAM.ManufPart.ManufType = CAM.ManufPart.ManufType.NCW

Public Property RuntimeManipulationInitialStatus As Replacement.RuntimeManipulations.Status
Public Property AllowRuntimeManipulationsHandling As Boolean = False
Public Property Workpiece As Workpiece                        ' the part's bounding box / material
```

The constructor `New(Modelname, MachineName, ManufPart)` resolves the machine via `Sbz14x.GetSbzMachineByName(MachineName)` and the EluCad version via `AppVersion.GetCurrentVersionByMachineName(MachineName)`. If `oMachine Is Nothing`, logs `Critical` and exits early — the resulting `ProfMillJob` is unusable.

## NCX log schema

`dtNcxLog` is a per-job log of post-processor events:

| Column | Type | Used for |
|--------|------|----------|
| `type` | String | event type |
| `code` | String | error/warning code |
| `work` | String | which Work it relates to |
| `description` | String | human-readable |
| `job` | String | which Job |
| `ignore` | Boolean | operator-toggled "I've seen this, ignore it" |

Populated by `ProcessNCLogElement(lNcxLogs As XmlNodeList)` (line 256) which iterates the NCX `.err` file output. Surfaced to UI via `GetNcxErrLogAsExceptionList` / `GetNcxLogsFromErrFile` (lines 683, 701).

## Public surface — by cluster

### Construction / loading (5 methods)

| Method | Role |
|--------|------|
| `New(Modelname, MachineName, ManufPart)` | constructor |
| `BuildFromStoredXml() As Boolean` | rebuild `lJobs` from the iCenter DB's stored part XML |
| `BuildFromManufPartXml() As Boolean` | rebuild from the `CAM.ManufPart`'s XML |
| `SetJobs(oJobs As List(Of Job))` | replace jobs (clears caches + flags) |
| `GetJobs() As List(Of Job)` | accessor |

### NC export (4 methods)

| Method | Role |
|--------|------|
| `ExportToEcw(EcwFileName As String) As Boolean` | write `.ecw` (the in-memory hierarchy as text) |
| `ExportToNcw(NcwFileName, NcwExportProfile) As Boolean` | write `.ncw` (input to post-processor) |
| `ExportNC(filepath, AddIaNr As Long) As Boolean` | write the final `.auf` / `.eluxml` with optional IA-number patch |
| `ExportProfilesToDxf(Folderpath)` | export all bars' profiles as DXF |

### Import (2 methods)

| Method | Role |
|--------|------|
| `ImportEcw(filepath As String) As Boolean` | read `.ecw` back into `lJobs` (via [[elumatec-elucadfile|`EluCadFile`]]) |
| `ImportNcx(OutputFile As Object)` | import the post-processor output, populating the NCX log |

### NCX post-processing (6 methods)

| Method | Role |
|--------|------|
| `PostProcessAndImport() As Boolean` | run NCX, import the result, log errors |
| `PostProcess(NcwFile As String) As String` | run NCX only |
| `RuntimePostProcess() As Integer` | runtime version |
| `CreateNCX(NcwFile, OutputFile, IncludeViewable) As String` | delegate to `Sbz14x.CreateNCX` (see [[elumatec-machine-base]]) |
| `DeleteNcxFiles(NCFilePath As String)` | clean up |
| `GetNCErrorFilepath(NCFilepath As String) As String` | path of the `.err` file |

### Manual AUF override (2 methods)

`GetManualAufPath(ecwFilePath As String) As String` and `GetManualAufExists(...)` — check whether an operator has placed a hand-edited `.auf` in the **`ManualProgFolder`** (`\\jazo.local\dfs\pm\Elumatec_SBZ140\Aanpassen\` — declared in `EluCadApp`). If yes, the manual file overrides the generated one. `#safety-relevant` — operators can override iCenter's output. Q-071.

### Cycle time (4 methods)

| Method | Role |
|--------|------|
| `GetCycleTime(AddIaNr As Long) As CycleTime` | full cycle-time bundle |
| `ComputeCycleTime(AufString, AddIaNr) As Double` | compute from an AUF string |
| `ComputeCycleTime(AddIaNr) As Double` | compute from `lJobs` |
| `ShowCycleTimeReport(AddIaNr)` | UI: show the XML report |

These feed the global [[../architecture/global-state|`UseSbzCalculatedDuration = True`]] — when enabled, iCenter prefers these durations over ISAH-side estimates for production planning.

### Production-machine timing (1 method)

- `StartProdMachineWorkTimer(AddIaNr, IPPartId As Long)` — kicks off a work-timer on a specific production machine for a specific IP part.

### Runtime manipulations (4 methods)

- `WorkUpdatedHandler(Cut, Work)` — event hook called when a Work changes mid-run.
- `GetRuntimeManipulationsHandled() As Boolean`
- `GetRuntimeManipulations() As Replacement.RuntimeManipulations` — aggregate across all jobs/bars/cuts
- `ProcessRuntimeManipulation(lJobs, InitialStatus)` (private)

Runtime manipulations let replacements queue post-emit changes that operators can apply or reject — see [[elumatec-work-base|Work]] `RuntimeManipulations` property.

### Job navigation (5 methods)

- `GetGroupByName(Name | Names)` — flatten matching groups across all jobs.
- `GetCutByCNo(CNo As Integer) As Cut` — lookup.
- `GetContainsRevokedWorks() As Boolean` — branch helper.
- `GetWorkpiece() As Workpiece` (private)
- `GetModelBoundingBox() As ICenterLib.CAD.Geometry.BoundingBox` — for UI display.

### Job merging (1 method)

`MergeJobs(SourceProfMillJob)` — combine another `ProfMillJob`'s jobs into this one (sets the other's `MasterProfMillJob = Me`). Used when iCenter needs to consolidate sub-jobs into a single submission.

### Status / state flags (8 methods)

- `SetCutsOutdated(Value)`, `SetCutsNcVersionOutdated(Value)` — cascade flags into all cuts.
- `GetOutdatedCuts()`, `GetOutdatedManufacturingCuts()`, `GetNcVersionOutdatedCuts()` — filter helpers.
- `SetReleaseLevel(NewReleaseLevel, MfgApprovalType)` — update Windchill PLM release-level; can trigger auto-approval.
- `SetIgnoreCnc(Value)` — toggle the `_IgnoreCnc` flag.
- `UpdateStatus(BatchServerId, Status, Optional Message)` (private) — push status to `Batchserver.BatchserverToolkit`.

### Misc (5 methods)

- `RefactorToMacros() As String` — rewrite the in-memory model to use macros where patterns repeat. Heavy operation.
- `GetGenSourceWarnings() As DataTable` — UI: warnings during generation.
- `HasMissingProdDocs() As Boolean` — branch helper.
- `GetModelnames() As List(Of String)` — list of model names included in this job.
- `WriteAufContentToFile(TargetPath, AufContent, NCType)` + `CreateTargetAufFolder(TargetPath)` (private) — file I/O helpers for `ExportNC`.

## Surprises

1. **`Public KeepAufAsSeperateFile = True`** — comment says `' Functions.DebugMode`, suggesting this should be tied to debug mode but isn't. Always-on. Result: the AUF (intermediate) file is *always* kept on disk after NC export, not just in debug.

2. **`oEluCad`, `oBatchServer`** are constructed as fields. Same pattern as elsewhere — eager construction of heavy objects per ProfMillJob.

3. **`MasterProfMillJob`** is set by `MergeJobs` on the *source* job, never by the *target*. Means "I am part of this master" is queryable but "I have these children" isn't. Asymmetric.

4. **`Property ProfMillJob` on `ProfMillConverter` returns only the *last* optimised job** — inline comment: _"Warning, returns only the last optimized ProfMillJob! Should be rewritten to collection!"_. In dual-emit mode (Sbz140Alu + Sbz141Alu) only the last job is accessible. Q-072.

5. **Manual AUF override** is a folder-scan: any matching `.auf` in `\\jazo.local\dfs\pm\Elumatec_SBZ140\Aanpassen\` short-circuits the generated output. No audit trail. `#safety-relevant` (Q-071).

6. **`RuntimeManipulationInitialStatus = Replacement.RuntimeManipulations.Status.Inert`** — initial status defaults to "inert" (don't apply). The `AllowRuntimeManipulationsHandling = False` default means manipulations don't fire unless explicitly enabled.

7. **`_AppVersionId = 1`** default — appears to mean "old EluCad version". `_NcVersion = 0` default. Both get overwritten when reading real data.

8. **`Release-level` field is Windchill PLM-typed** (`ICenterLib.CAD.PLM.EPMDocument.ReleaseLevel`) — iCenter's job state carries the same lifecycle the CAD model carries. `SetReleaseLevel` can flip this and may trigger auto-approval (line 1089). Phase-3 follow-up.

## Open questions

- **Q-071 (new):** Manual AUF override — files in `ManualProgFolder` silently replace generated output. Audit trail? Approval workflow? `#safety-relevant`
- **Q-072 (new):** `ProfMillConverter.ProfMillJob` returns only the last optimised job. In dual-emit (Sbz140Alu + Sbz141Alu) the first machine's job is lost from this accessor. Confirm callers don't need both. `#safety-relevant`
- **Q-073 (new):** `KeepAufAsSeperateFile = True` always-on. Should this be conditional on `Functions.DebugMode` (per the inline comment) or kept always-on? `#needs-review`
- **Q-074 (new):** `SetReleaseLevel` can trigger Windchill auto-approval. Document the trigger conditions and the audit trail. `#safety-relevant`

Logged in [[../needs-review/_index]].

## Related

- [[elumatec-profmill-converter|ProfMillConverter]] — builds the ProfMillJob.
- [[elumatec-ncstructure-hierarchy|Job/Bar/Cut/Plane]] — the data it wraps.
- [[elumatec-machine-base|`Sbz14x`]] — provides the tool DB and NCX runner.
- [[elumatec-nc-program-family|NcProgram family]] — produces the final NC file via `ExportNC`.
- [[../mocs/elumatec|Elumatec MOC]]

## Coverage

`_coverage.md`: `Elumatec\ProfMillJob.vb` → `done`.
