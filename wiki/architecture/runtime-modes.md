---
type: architecture
title: "Runtime modes"
status: draft
module: "iCENTER/Modules"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Modules\\Main.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Batchserver"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\CadBatchserver"
last-reviewed: ""
tags: [architecture, entry-point, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Runtime modes

## What this view shows

The four distinct run-time personalities `iCenter.exe` can take. Each one selects a different top form and a different set of background workers.

## The four modes

| Mode | Selected by | Top form | Folder |
|------|-------------|----------|--------|
| **Interactive** (default) | no `-m` flag | `FrmMain` (god-form, ~15 200 lines) | root + `Forms/` |
| **CAD batch server** | `-m cadbatchserver` | `CadBatchserver.FrmCadBatchServer` | `CadBatchserver/` |
| **Self-update** | `-m updateicenter` | (none — runs `Functions.UpdateIcenter()` then exits) | `ICenterLib` external |
| **Post-main generic batch** | `BatchServerMode = True` (set inside `FrmMain`) | `Batchserver.FrmBatchServer` | `Batchserver/` |
| **Post-main COM watcher** | `ComWatcherMode = True` (set inside `FrmMain`) | `Elumatec.FrmComWatcher` | `Elumatec/` |

## CAD batch server — what it runs

`CadBatchserver\FrmCadBatchServer.vb` is the host. Per the `<Compile Include>` block in `iCenter.vbproj` (lines 291–324), this folder ships a collection of `Job*.vb` classes that the form schedules:

- `JobAutoManufacturing.vb`
- `JobArchive.vb`
- `JobCreateProdOrd.vb` — create production orders
- `JobCreatePurOrdDocs.vb` — create purchase-order documents
- `JobEngOrdFinNotification.vb` — engineering-order-finished notification
- `JobGenerateAndReleaseModel.vb`
- `JobGenericModelBackup.vb`
- `JobGeo2Dxf.vb` — convert geometry → DXF
- `JobModelGeneratorCreo.vb` — drive Creo model generator
- `JobPartDispatch.vb` / `JobSmtPartDispatch.vb` — part dispatch
- `JobProductionRegistration.vb`
- `JobPublishCreo.vb` — Creo publish
- `JobRebootMonitor.vb` — server-reboot monitor
- `JobSendEmail.vb`
- `JobSmtOperSync.vb` — sheet-metal operation sync
- plus `BatchServerWatch.vb`, `CadBatchserverTools.vb`, `Job.vb` (base), `ServerReboot.vb`, `Modelgenerator\ModelgeneratorTask.vb`, `Publisher\Trailfile.vb`, and the dialog `FormRunJob`.

Each `Job*.vb` is a candidate for a dedicated [[../business-rules/_index|business-rule]] note during Phase 4. Anything that **drives a machine, publishes CAD output, sends email, or creates an ISAH order** is `#safety-relevant` until SME confirms otherwise.

## Generic post-main batch — `Batchserver/`

A separate, smaller batch host:

- `Batchserver\FrmBatchServer.vb` + `BatchserverToolkit.vb`

Only launches if `FrmMain` flips the `BatchServerMode` global before disposing. Use case unknown — see [[entry-points|Q-003]].

## COM watcher — `Elumatec\FrmComWatcher.vb`

Launches after `FrmMain` closes if `ComWatcherMode = True`. Lives under the Elumatec folder, so almost certainly watches the **serial / COM port** to a [[../external-systems/elumatec-sbz140|Elumatec SBZ140]] profile-mill machine. **`#safety-relevant`, `#needs-review`** until SME confirms what events it acts on and whether it can send commands back.

## Self-update — `Functions.UpdateIcenter()`

`Functions` is defined in `ICenterLib` (project reference). Outside this wiki's scope; the call exists at `Modules\Main.vb` line 223. Likely a hand-rolled equivalent of ClickOnce update — since the real ClickOnce signing target is disabled (`iCenter.vbproj` lines 3383–3391, expired cert), this may be the actual update mechanism in production. **#needs-review**.

## Key decisions / surprises

- The `-m cadbatchserver` mode uses `ShowDialog()` — it's a modal Windows Form even when run "headlessly". On a server with no interactive session this would block; the assumption is that there's *some* user session (likely `JZPUBLISH`, see commented-out check at line 226 of `Modules\Main.vb`).
- The 5 modes are **not mutually exclusive at the process level**: a single launch can do interactive → generic batch sequentially, by toggling `BatchServerMode` mid-session.

## Open questions

- **Q-002 / Q-003 / Q-004** mirrored from [[entry-points]].
- **Q-005:** When and where in `FrmMain` does `ComWatcherMode = True` get set? Same for `BatchServerMode`. (Grep `BatchServerMode = True` and `ComWatcherMode = True` during Phase 3.)
- **Q-006:** Is `Elumatec.FrmComWatcher` the COM-port (serial) hand-shake to the SBZ140, or a generic "Component Object Model" watcher? Folder context strongly suggests serial — confirm with SME. `#safety-relevant`.

## Related

- [[_index]]
- [[entry-points]]
- [[../external-systems/elumatec-sbz140|Elumatec SBZ140]] (stub)
