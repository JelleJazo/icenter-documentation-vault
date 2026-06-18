---
type: moc
title: "ICenterLib/CadBatchServer — Creo automation job framework"
status: draft
module: "ICenterLib/CadBatchServer"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\CadBatchServer\\"
tags: [moc, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# ICenterLib/CadBatchServer

## What this hub covers

`ICenterLib\CadBatchServer\` — the **shared library for the CadBatchserver job framework**. CadBatchserver is JAZO's headless background processor that runs Creo / Windchill / publishing jobs out-of-band from the interactive iCenter UI. This folder hosts the **job-parameter DTOs, dataservices, status-tracking entities, distributed locks, and Modelgenerator-specific subsystems** that both the CadBatchserver workers and the iCenter clients use.

17 source files (~63 KB), no generated.

## Files

| File | Role |
|------|------|
| `JobParameters.vb` | job-parameter DTO |
| `JobDataService.vb` | job CRUD via DataService |
| `JobAlreadyExistsException.vb` | typed exception |
| `JobToolbox.vb` | misc job helpers |
| `CadBatchserverDataService.vb` | CadBatchserver-level DataService |
| `CadBatchserverStatus.vb` | per-server status DTO |
| `CadBatchserverStatusCollection.vb` | collection wrapper |
| `CadBatchserverStatusDataService.vb` | server-status CRUD |
| `DistributedLockCreoPublish.vb` | distributed-lock for Creo publish jobs |
| `PublishJobInstructions.vb` | publish-job instructions DTO |
| `PublishMonitor.vb` | publish-job monitor |
| `PublishWatchDirProcessor.vb` | watch-dir → publish-job dispatcher |
| `ModelgeneratorDataService.vb` | Modelgenerator DataService |
| `Modelgenerator\ModelgeneratorTask.vb` | model-generator task DTO |
| `Modelgenerator\ModelgeneratorInstructions.vb` | model-generator instructions |
| `Modelgenerator\DuplicateInstruction.vb` | duplicate-instruction handler |
| `Modelgenerator\CadInputParameters.vb` | CAD-input parameters for Modelgenerator |

## Notable findings

1. **`DistributedLockCreoPublish`** — distributed lock specifically for Creo-publish operations. Suggests **multiple CadBatchserver workers** can target the same product but only one may publish at a time. Likely uses `DistributedLock\` folder for the lock primitive.
2. **`PublishWatchDirProcessor`** — file-system-watcher pattern: a directory is watched, new files trigger publish jobs. Phase-3 follow-up to identify the watch-dir path config.
3. **The Modelgenerator subsystem** is a **CAD-input-parameter → CAD-model generator**, distinct from the publish flow. Likely produces parametric Creo models from input parameters.
4. **`JobAlreadyExistsException`** — typed exception used for "idempotent submit" — caller catches it to know the job was already in flight.

## Open questions

- **Q-289 (new):** Document the CadBatchserver architecture — how many workers, how do they coordinate via DistributedLock, what triggers a job vs a Modelgenerator task.
- **Q-290 (new):** Document the `PublishWatchDirProcessor`'s watched directory path — likely a network share.
- **Q-291 (new):** Document the Modelgenerator vs Publish split — these are two distinct job pipelines.

Logged in [[../needs-review/_index]].

## Coverage

All 17 source files marked `done` overview-level. Phase-4 sweep should deep-read `PublishWatchDirProcessor`, `DistributedLockCreoPublish`, `ModelgeneratorTask` for business-rule extraction.
