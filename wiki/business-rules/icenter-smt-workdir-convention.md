---
type: business-rule
title: "SmtManufacturing scratch dir = c:\\work\\ (NOT c:\\temp\\)"
status: needs-review
module: "iCENTER/SmtManufacturing"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\SmtManufacturing\\Part.vb"
last-reviewed: ""
tags: [business-rule, needs-review, safety-relevant]
created: 2026-06-18
updated: 2026-06-18
---

# SmtManufacturing scratch dir convention — `c:\work\` not `c:\temp\`

## Rule

All scratch files produced by the **iCENTER SmtManufacturing pipeline** (GEO conversions, AutoBend logs, temporary DXF/NC files) MUST be written under `c:\work\`, NOT `c:\temp\`.

The reason — preserved as an inline comment in `Part.vb`:

> `Protected Friend Const workdir As String = "c:\work\"   ''Niet c:\temp gebruiken: de CadBatchServer ruimt daar alles op!`
>
> (Don't use c:\temp: the CadBatchServer cleans everything there!)

The **CadBatchServer's housekeeping job auto-deletes everything in `c:\temp\`**, so any iCenter-side scratch file written there will disappear without warning while a job is still using it.

## Where it lives

- File: `C:\DevOps\iCenter\iCenter\iCENTER\SmtManufacturing\Part.vb` line 13
- Symbol: `Protected Friend Const Part.workdir As String = "c:\work\"`
- Comment: line 13 inline.
- Consumers: `AutoBendLogPath = workdir & "autobendlog.xml"` (line 16); plus various per-method scratch paths under `c:\work\...`.

## The code

```vb
Protected Friend Const workdir As String = "c:\work\"  ''Niet c:\temp gebruiken: de CadBatchServer ruimt daar alles op!
Public Const ManualBendFolder As String = "/temp/Bend"
Public Shared AutoBendLogPath As String = workdir & "autobendlog.xml"
```

## Why this is a business rule

The CadBatchServer (the Creo automation worker) has a janitorial process that wipes `c:\temp\`. iCenter's SMT pipeline can run **on the same host** as a CadBatchServer worker (the JAZO deployment model puts them together for performance reasons). If iCenter writes scratch to `c:\temp\`, the worker's cleanup eventually nukes it — possibly mid-job, possibly between two stages of the same conversion.

By convention, **iCenter SMT uses `c:\work\` exclusively** — outside CadBatchServer's housekeeping scope.

Anyone modifying SMT-side scratch-file paths must keep them out of `c:\temp\`. Anyone modifying CadBatchServer's cleanup must NOT extend it to `c:\work\`.

The convention is **load-bearing across two subsystems and two teams** (assuming separate ownership).

## Triggers / when it fires

- Every `Part` instantiation references `workdir` directly or via `AutoBendLogPath`.
- Per-method scratch-file path construction throughout `Part.vb` and likely `FlatPatternConverter.vb`.

## Effects

- iCENTER SMT scratch under `c:\work\` survives between job stages.
- A misroute to `c:\temp\` produces intermittent "file not found" failures.

## Edge cases

- **Headless / server deployments**: `c:\work\` must exist on every host running iCENTER SMT. No mkdir-if-missing guard visible in the constant declaration; consumer paths must defend.
- **Disk-space**: `c:\work\` has no auto-cleanup analog. Long-running iCenter installations accumulate files. Q-401 — is there a separate `c:\work\` cleanup job?
- **`ManualBendFolder = "/temp/Bend"`** is anomalous — forward-slash prefix. May resolve to `c:\temp\Bend` on Windows, or be a relative path. Q-392.
- **Cross-host coordination**: if a job is submitted on host A and processed on host B, both must have `c:\work\` writable. Q-402.

## Safety classification

- [ ] Touches physical process — indirectly (job failure → missing output → wrong/no production).
- [ ] Drives cost / pricing — no.
- [x] Reversible if wrong? — yes (rerun the job after fixing path).
- [x] Blocks production if it fails? — yes (intermittent file-not-found stops the SMT pipeline).
- `#safety-relevant` because **the wrong path is a cross-team trap** — a CadBatchServer cleanup policy change can break iCenter SMT without anyone noticing until production halts.

## SME questions

- **Q-401 (new):** Is there a separate cleanup job for `c:\work\`? Long-running installs may accumulate.
- **Q-402 (new):** Cross-host job submission — both host A (submitter) and host B (processor) need `c:\work\` writable. Confirm deployment.
- **Q-403 (new):** Document the CadBatchServer cleanup job's scope + frequency. Pinned to `c:\temp\` only?

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/smtmanufacturing|SmtManufacturing MOC]] — defining hub.
- [[../mocs/icenterlib-cadbatchserver|CadBatchServer MOC]] — the cleanup-source.
- [[../modules/pcfnet-generic-part|`GenericPart` `c:\temp\PCFNet dump`]] (Q-359) — a different subsystem that writes to c:\temp; presumably accepts cleanup.
- [[../modules/icenterlib-productdb-product|`Product` `c:\temp\dump_*.xml`]] (Q-268) — another c:\temp consumer.
