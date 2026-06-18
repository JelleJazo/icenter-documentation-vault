---
type: moc
title: "Needs Review — Index"
status: active
tags: [moc, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Needs Review — SME Queue

Every open question for the SME lives here. Anything with `#safety-relevant` is highest priority.

## How it works

When documenting any code path where intent is unclear:
1. In the page, set `status: needs-review` and add `#needs-review` (and `#safety-relevant` if applicable).
2. Write the *specific* open question in a `## Open question` section on that page.
3. Add a one-line entry below pointing to the page. Use the running ID (`Q-NNN`).

## Open questions

_(Append as you go. Newest at the top.)_

| ID | Date | Page | Question | Severity / tag |
|----|------|------|----------|----------------|
| **Q-033** | 2026-06-18 | [[../business-rules/elu-large-rectangle-classification]] | Should the 260×20 mm rule be `OR` instead of `AND`? Long thin slots would currently stay contoured. | safety-relevant |
| **Q-032** | 2026-06-18 | [[../business-rules/elu-large-rectangle-classification]] | Confirm 260×20 mm thresholds are correct for all four machine variants (ALU, STL, RVS, SBZ141). | safety-relevant |
| **Q-031** | 2026-06-18 | [[../business-rules/elu-max-step-depth]] | Enumerate every callsite of `Sbz14x.MaxStepDepth` (likely in `Works\Replacements\AluGeneral.vb` / `StlGeneral.vb` / `ProfMillConverter.vb`). | medium |
| **Q-030** | 2026-06-18 | [[../business-rules/elu-max-step-depth]] | Are 1.6 mm (STL) and 6 mm (ALU) the values actually in use today, or have they been overridden in deployed `app.config`? | safety-relevant |
| **Q-029** | 2026-06-18 | [[../modules/elumatec-machine-base]] | `WorksReplaceList` ordering: confirm with SME the order in each `Sbz*.New` is intentional and load-bearing. | safety-relevant |
| **Q-028** | 2026-06-18 | [[../modules/elumatec-machine-base]] | `CreateNCX` Auf path runs the post-processor 3–4× with the same `OutputFile`. Confirm post-proc *appends* and isn't *overwriting* between calls. | safety-relevant |
| **Q-027** | 2026-06-18 | [[../modules/elumatec-machine-base]] | Difference between `Sbz140Stl` and `Sbz140Rvs`? Both read `EluMaxStepDepthSTL`; what distinguishes them at the machine? | medium |
| **Q-026** | 2026-06-18 | [[../modules/elumatec-cad-app]] | `OverwriteClientEluCadRegistry` switch silently overwrites EluCad settings registry from `SourceRegFile`. When is this safe? What if EluCad is mid-edit? | medium |
| **Q-025** | 2026-06-18 | [[../modules/elumatec-cad-app]] | The `dgxdata.dgx` → `dgxdata.elu` rename handshake — what happens on rename failure? Is there a timeout? | safety-relevant |
| **Q-024** | 2026-06-18 | [[../modules/elumatec-cad-app]] | Confirm `UnknownBIdentNo = "100999"` is a reserved sentinel (no real profile has that ID). | low |
| **Q-023** | 2026-06-18 | [[../modules/elumatec-machine-base]] | `Sbz140Alu.GoHomeDuration = 8 + CycleTimeToolChange` — confirm. | medium |
| **Q-022** | 2026-06-18 | [[../modules/elumatec-machine-base]] | `Sbz14x.PartRotationDuration = 30 sec` constant — calibration source? | safety-relevant |
| **Q-021** | 2026-06-18 | [[../modules/elumatec-machine-base]] | `Sbz140Alu.MachineXFeedRate = 1000 'Guess` (literal comment in source). Confirm all four variants. | safety-relevant |
| **Q-020** | 2026-06-18 | [[../modules/elumatec-com-watcher]] | `CRs232` is third-party copy-paste (Corrado Cavalli, ©2003) unmaintained since 2005. Patch level? | release-engineering |
| **Q-019** | 2026-06-18 | [[../modules/elumatec-com-watcher]] | `ClsComWatcher.Update` body is mostly commented out (lines 95–120) and a `Critical` log was added in 2023-01 as a "is anybody using this?" probe. Vestigial or live? | safety-relevant |
| **Q-018** | 2026-06-18 | [[../architecture/project-references]] | The `ICenterLib` project-reference path in `iCenter.vbproj` / `iCENTER.sln` resolves to a non-existent directory; the actual source is under the user's personal `source\repos\` tree. Where is the canonical / build-server copy? | release-engineering |
| **Q-017** | 2026-06-18 | [[../architecture/external-surface]] | For each `#safety-relevant` external system, what is the failure mode if it goes down? | safety-relevant |
| **Q-016** | 2026-06-18 | [[../architecture/external-surface]] | Is the embedded PDF-XChange license valid for redistribution? Confirm with vendor. | compliance |
| **Q-015** | 2026-06-18 | [[../architecture/external-surface]] | Is the `pvs/pvs` XWiki account read-only or write-capable? | security |
| **Q-014** | 2026-06-18 | [[../architecture/project-references]] | `System.Xml.dll` referenced from v4.6.2 reference-assemblies while target is `v4.8`. Drift or intentional? | low |
| **Q-013** | 2026-06-18 | [[../architecture/project-references]] | DFS-hosted house DLLs are loaded from a live share. Is the per-version folder treated as immutable, or do operators update in place? | release-engineering |
| **Q-012** | 2026-06-18 | [[../architecture/build-and-deploy]] | Confirm `Functions.UpdateIcenter()` is the live update mechanism; document its source/destination paths and signing model. | release-engineering |
| **Q-011** | 2026-06-18 | [[../architecture/build-and-deploy]] | Is `Z:\iCenter\BetaRelease\` the production publish target or only Beta? Where does production release go? | release-engineering |
| **Q-010** | 2026-06-18 | [[../architecture/build-and-deploy]] | Has Authenticode signing been re-enabled since 2023-07-18 in any branch / pipeline outside `iCenter.vbproj`? | compliance |
| **Q-009** | 2026-06-18 | [[../architecture/global-state]] | Confirm hard-coded constant lists (kick-off ops, lean-mode machine groups, dept codes) still match current factory configuration. | safety-relevant |
| **Q-008** | 2026-06-18 | [[../architecture/global-state]] | Confirm `iCenter2NewestDataModel` DB is still hit at runtime; if so, document its schema scope. | data-lifecycle |
| **Q-007** | 2026-06-18 | [[../architecture/global-state]] | Replace `sPDFOwnerPassword` literal with a secret store? Confirm it controls real PDF protection. | security / safety-relevant |
| **Q-006** | 2026-06-18 | [[../modules/elumatec-com-watcher]] | _(resolved 2026-06-18)_ — `FrmComWatcher` confirmed as a serial COM-port watcher for the **saw** (MachineId 2, COM1, 9600 8-N-1). Most original behaviour is commented out — see new Q-019. | safety-relevant (resolved) |
| **Q-005** | 2026-06-18 | [[../architecture/runtime-modes]] | Where in `FrmMain` does `ComWatcherMode = True` / `BatchServerMode = True` get set? | medium |
| **Q-004** | 2026-06-18 | [[../architecture/entry-points]] | Is `Functions.UpdateIcenter()` (the `-m updateicenter` path) still in use after the 2023-07 cert expiry? | medium |
| **Q-003** | 2026-06-18 | [[../architecture/entry-points]] | What conditions inside `FrmMain` flip `BatchServerMode` / `ComWatcherMode`? | medium |
| **Q-002** | 2026-06-18 | [[../architecture/entry-points]] | Who launches `-m cadbatchserver`? Scheduled task? Service wrapper? User `JZPUBLISH`? | medium |
| **Q-001** | 2026-06-18 | [[../architecture/_index]] | _(resolved 2026-06-18)_ — Both `ICenterLib` and `TruTopsLib` widened **into scope** per user direction. ICenterLib source located at `C:\Users\jelle-r\source\repos\JIBA\iCenter And Tools\ICenterLib\ICenterLib` — see new **Q-018** about the build-path discrepancy. | scope (resolved) |
| **Q-000** | 2026-06-18 | [[../overview]] | _(superseded)_ — Was the older `C:\DevOps\iCenter` codebase in scope, or only `iCenter2`? Resolved: per updated CLAUDE.md, scope is `C:\DevOps\iCenter\iCenter\iCENTER` only; `iCenter2` is out. | scope (resolved) |

## Safety-relevant queue

| ID | Page | One-line |
|----|------|----------|
| Q-007 | [[../architecture/global-state]] | Hard-coded PDF owner password |
| Q-009 | [[../architecture/global-state]] | Hard-coded workflow constant lists |
| Q-017 | [[../architecture/external-surface]] | Per-system failure-mode docs missing |
| Q-019 | [[../modules/elumatec-com-watcher]] | COM watcher's behaviour mostly commented out — vestigial? |
| Q-021 | [[../modules/elumatec-machine-base]] | `MachineXFeedRate = 1000 'Guess` |
| Q-022 | [[../modules/elumatec-machine-base]] | `PartRotationDuration = 30 sec` — calibration source? |
| Q-025 | [[../modules/elumatec-cad-app]] | DGX rename handshake failure mode? |
| Q-028 | [[../modules/elumatec-machine-base]] | Post-processor invoked 3–4× with same output file |
| Q-029 | [[../modules/elumatec-machine-base]] | `WorksReplaceList` ordering — load-bearing? |
| Q-030 | [[../business-rules/elu-max-step-depth]] | 1.6 / 6 mm step depths — still current? |
| Q-032 | [[../business-rules/elu-large-rectangle-classification]] | 260×20 mm large-rect thresholds correct for all 4 variants? |
| Q-033 | [[../business-rules/elu-large-rectangle-classification]] | `AND` vs `OR` for large-rect classifier |
