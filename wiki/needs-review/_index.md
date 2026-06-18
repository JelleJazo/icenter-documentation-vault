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
| **Q-106** | 2026-06-18 | [[../business-rules/icenter-operation-machgrp-mapping]] | Ordering of MachGrpCodes between operations 1 and 31 — is `GetMBomMultilevel` order-sensitive? | safety-relevant |
| **Q-105** | 2026-06-18 | [[../modules/engineering-overview]] | `frmGenericStatus`'s commented-out PDF-XChange viewer activation — deliberately disabled or forgotten? | low |
| **Q-104** | 2026-06-18 | [[../modules/engineering-overview]] | Order-number convention: position 2 = `"0"` means quote, otherwise order. Confirm with SME. | low |
| **Q-103** | 2026-06-18 | [[../modules/production-profile-milling-import]] | `PMMExportHandler.ProcessFile` wipes all `<PMMEXPORT3D>` attributes before setting `name`. Should others be preserved? | medium |
| **Q-102** | 2026-06-18 | [[../modules/production-profile-milling-import]] | Document SME workflow for the per-generic `AutoAddIaNr` flag. | medium |
| **Q-101** | 2026-06-18 | [[../modules/production-profile-cut-items]] | `MachineId = Math.Max(iPPartId, 0)` looks like a copy-paste error — `MachineId` itself isn't normalised. | safety-relevant |
| **Q-100** | 2026-06-18 | [[../modules/production-profile-cut-items]] | `Clear(MachineId)` runs unconditionally; a model-lookup failure wipes the machine's previous cut items. | safety-relevant |
| **Q-099** | 2026-06-18 | [[../modules/workprep-ipbatch-collector]] | Query filter requires a `JZ_ProdRefNr` row — confirm rule "only jobs with assigned ref-nr are outsourceable". | medium |
| **Q-098** | 2026-06-18 | [[../modules/workprep-ipbatch-collector]] | Empty `SearchValue` returns no rows. Confirm intentional. | medium |
| **Q-097** | 2026-06-18 | [[../modules/workprep-operation-substitution]] | `KeepSetupTime` flag only acts when `KeepCycleTime` is also true. Intentional? | medium |
| **Q-096** | 2026-06-18 | [[../modules/workprep-outsource-operations]] | `WriteExchangeFile` uses default encoding (Windows-1252 on Dutch box). Vendors expecting UTF-8 may see mojibake. | medium |
| **Q-095** | 2026-06-18 | [[../modules/workprep-operation-substitution]] | Two surface-treatment subs use swapped table sources. Bug or harmless? | safety-relevant |
| **Q-094** | 2026-06-18 | [[../modules/workprep-outsource-operations]] | `SetShopDocFinInd(True)` commented out (line 405). Is the ShopDoc ever marked finished, or has it moved elsewhere? | safety-relevant |
| **Q-093** | 2026-06-18 | [[../modules/workprep-outsource-operations]] | `FrmOutsourceOperations.GetLazyMachGrpFilterIn` — what determines the list? User-editable or hard-coded? | medium |
| **Q-092** | 2026-06-18 | [[../modules/sales-customer-team]] | `DTFilter` declared at class scope and shadowed in `New()`. Remove the field? | low |
| **Q-091** | 2026-06-18 | [[../modules/sales-customer-team]] | What does the "apply" action actually call on ISAH? | low |
| **Q-090** | 2026-06-18 | [[../modules/engineering-overview]] | `FrmDesignCodeTool` brittle JavaScript injection against tekeningnummers.jazo.com. Document contract. | medium |
| **Q-089** | 2026-06-18 | [[../modules/engineering-overview]] | `UitsparingVoorplaatMeerpslAlu` registered-out in CopyLocalizer. Confirm truly dead code. | low |
| **Q-088** | 2026-06-18 | [[../modules/engineering-overview]] | `BatchServerMode` temporarily flipped to `True` during ModelCopies searches — why? | medium |
| **Q-087** | 2026-06-18 | [[../modules/workprep-outsource-operations]] | `WaitForPurDocFolder` polls every 500ms for 20s. What happens during ISAH peak load? | medium |
| **Q-086** | 2026-06-18 | [[../modules/workprep-outsource-operations]] | CSV exchange-file schema (8 columns) is hard-coded. Vendors changing format → silent breakage. | medium |
| **Q-085** | 2026-06-18 | [[../business-rules/icenter-operation-machgrp-mapping]] | Confirm iCenter operation IDs 1/9/31 are the only ones. Provide SME-friendly names. | safety-relevant |
| **Q-084** | 2026-06-18 | [[../modules/workprep-outsource-operations]] | Only `ProfileId = 1` is implemented for outsourcing. What other ProfileIds exist? | safety-relevant |
| **Q-083** | 2026-06-18 | [[../business-rules/outsource-ext-oper-part-code]] | Is there ever a need for a second outsource part code (other than `UITBESTEDING01`)? | safety-relevant |
| **Q-082** | 2026-06-18 | [[../business-rules/sales-team-codes]] | Confirm `031/032/033` are the only sales teams. Surface their SME-friendly names. | medium |
| **Q-081** | 2026-06-18 | [[../business-rules/elu-dual-emit-sbz140-sbz141]] | Confirm `ExportNC` per-machine output filepaths don't collide in dual-emit. | safety-relevant |
| **Q-080** | 2026-06-18 | [[../business-rules/elu-dual-emit-sbz140-sbz141]] | Dual-emit is asymmetric: Sbz140Alu→both, Sbz141Alu→itself only. Intentional? | safety-relevant |
| **Q-079** | 2026-06-18 | [[../business-rules/elu-forster-thumbhole-step-depth]] | If Forster override is dead in production, remove it or migrate it into the per-tool path? | safety-relevant |
| **Q-078** | 2026-06-18 | [[../business-rules/elu-forster-thumbhole-step-depth]] | Which `BIdentNo`s correspond to "Forster profiel" thumb-holes? Replace geometric signature with profile-ID check. | safety-relevant |
| **Q-077** | 2026-06-18 | [[../business-rules/elu-tool-max-cut-depth]] | `SkipSplitSteps = True` on a Work — which code paths set it, and what's the upstream guarantee? | safety-relevant |
| **Q-076** | 2026-06-18 | [[../business-rules/elu-tool-max-cut-depth]] | How is the tool DB (`*.nct`) maintained? Confirm change-control around `TMaxCut` edits. | safety-relevant |
| **Q-075** | 2026-06-18 | [[../business-rules/elu-tool-max-cut-depth]] | `GetMaxCut(WToolID) = 0` → `SplitSteps` skipped → single-pass full-depth cut. Fail loudly instead? | safety-relevant |
| **Q-074** | 2026-06-18 | [[../modules/elumatec-profmill-job]] | `SetReleaseLevel` can trigger Windchill auto-approval. Document trigger conditions and audit trail. | safety-relevant |
| **Q-073** | 2026-06-18 | [[../modules/elumatec-profmill-job]] | `KeepAufAsSeperateFile = True` always-on. Should this be conditional on `Functions.DebugMode` (per inline comment)? | low |
| **Q-072** | 2026-06-18 | [[../modules/elumatec-profmill-job]] | `ProfMillConverter.ProfMillJob` returns only the *last* optimised job; dual-emit loses the first. Confirm callers don't need both. | safety-relevant |
| **Q-071** | 2026-06-18 | [[../modules/elumatec-profmill-job]] | Manual AUF override — files in `ManualProgFolder` silently replace generated output. Audit trail? | safety-relevant |
| **Q-070** | 2026-06-18 | [[../modules/elumatec-profmill-converter]] | `SetWHelixIntr` reads `DefaultWHelixIntr` from iCenter DB `AppSettings` — document as business rule. | safety-relevant |
| **Q-069** | 2026-06-18 | [[../modules/elumatec-profmill-converter]] | `SuppressMultiSidedMacros` body commented out; only empty Try/Catch remains. Remove or restore? | medium |
| **Q-068** | 2026-06-18 | [[../modules/elumatec-profmill-converter]] | `SetWMillDir = -1` now applies to aluminium per CB 2023-02-28. Confirm matches current factory practice. | safety-relevant |
| **Q-067** | 2026-06-18 | [[../modules/elumatec-profmill-converter]] | `MyCut.CCopies = 1` "tijdelijke fix" for EluXml signature mismatch. Still needed? | medium |
| **Q-066** | 2026-06-18 | [[../modules/elumatec-profmill-converter]] | `AppVersion.UseFileBasedSettings` gates dual-emit. What determines this flag? | safety-relevant |
| **Q-065** | 2026-06-18 | [[../modules/elumatec-profmill-converter]] | Is `UseTMaxCut = False` ever set in production? If never, the Forster override is dead. | safety-relevant |
| **Q-064** | 2026-06-18 | [[../modules/elumatec-nc-program-family]] | AUF parser splits input by `vbCrLf` only. UNIX-line-ending `.auf` files would be one giant line. Confirm Elumatec only emits CRLF. | low |
| **Q-063** | 2026-06-18 | [[../modules/elumatec-nc-program-family]] | `NcProgramEluXml.ReadFromString` sets top-level `Description`/`Comment`/`CNo` from the *last* matched program in a multi-Job file. Intentional? | medium |
| **Q-062** | 2026-06-18 | [[../modules/elumatec-nc-program-family]] | `NcProgramAuf.ComputeCycleTime` implementation not yet seen; confirm AUF machines contribute to `UseSbzCalculatedDuration` reporting. | safety-relevant |
| **Q-061** | 2026-06-18 | [[../modules/elumatec-elucadfile]] | FreeForm-point continuation relies on `MyFreeForm` persisting between Work blocks. What happens if a `:WORK` of another type appears mid-FreeForm? | medium |
| **Q-060** | 2026-06-18 | [[../modules/elumatec-elucadfile]] | ECW parse errors silently become `0`. Is there a logging/alert path that should fire instead? | safety-relevant |
| **Q-059** | 2026-06-18 | [[../modules/elumatec-ncstructure-hierarchy]] | `Bar.GetNcwText` performs I/O (`OffsetFile.Read`, `FixtureCollection.ReadAll`) per bar. Cache for large multi-bar jobs? | low |
| **Q-058** | 2026-06-18 | [[../modules/elumatec-ncstructure-hierarchy]] | `Job` carries a private `oELUCAD = New EluCadApp` field; constructed on every Job. Safe to share/pool? | low |
| **Q-057** | 2026-06-18 | [[../modules/elumatec-ncstructure-hierarchy]] | `Plane.GetFromXmlNode` evaluates `WPTransX/Y/Z` as expressions via `Works.Work.GetCalcContext()`. Document the expression grammar. | safety-relevant |
| **Q-056** | 2026-06-18 | [[../modules/elumatec-ncstructure-hierarchy]] | `Cut.StatusCode.MissingProeManufData` references Pro/E (pre-Creo). Still relevant after Creo migration? | low |
| **Q-055** | 2026-06-18 | [[../modules/elumatec-ncstructure-hierarchy]] | `Bar.AddCut` moves the Cut's Planes onto the Bar. Confirm invariant that Cut.Planes is always `Nothing` after add. | medium |
| **Q-054** | 2026-06-18 | [[../mocs/elumatec-ncpipeline]] | Empty-line-terminates-Work semantics in ECW parser. Confirm intentional, and that the Elumatec writer always emits a trailing blank line. | medium |
| **Q-053** | 2026-06-18 | [[../mocs/elumatec-ncpipeline]] | `EluCadFile.GetDoubleValue` accepts arithmetic expressions via `DataTable.Compute`. Document the expression grammar SMEs can rely on. | safety-relevant |
| **Q-052** | 2026-06-18 | [[../mocs/elumatec-ncpipeline]] | `Job.cncdriver = "1.1elu"` (ECW) vs `"1.1"` (NCW). Does the Elumatec post-processor branch on this value? | medium |
| **Q-051** | 2026-06-18 | [[../mocs/elumatec-ncpipeline]] | `PlaneCollection.GetPlaneByWSide(WSide → WSide-7)` only handles custom planes (WSide ≥ 7). Standard sides 1-6 return `Nothing`. Intentional or bug? | medium |
| **Q-050** | 2026-06-18 | [[../mocs/elumatec-ncpipeline]] | `NcProgramEluXml.GetNCStringWithIaNr` is a stub (returns input unchanged). Is appending IA-numbers to EluXml programs silently broken? | safety-relevant |
| **Q-049** | 2026-06-18 | [[../business-rules/elu-largerect-freeform-replacement]] | LargeRectangle tool-assign silent failure (`WToolID = ""`) — should this alert instead of producing a deactivated rectangle? | safety-relevant |
| **Q-048** | 2026-06-18 | [[../modules/elumatec-replacement-alu-general]] | Hard-coded `xValue=15`, `yValue=-20` for `100180` Koker drainage holes — match live profile drawings? | safety-relevant |
| **Q-047** | 2026-06-18 | [[../modules/elumatec-replacement-alu-general]] | `Y=-47` / `Y=-46.5` "nokje breekt" magic — confirm the breakage mode (tool/drill/workpiece?) and document. | safety-relevant |
| **Q-046** | 2026-06-18 | [[../modules/elumatec-replacement-alu-general]] | Enumerate every `Case "1*"` branch in `AluGeneral.vb` (~20 cases) and create one business-rule note per. | safety-relevant |
| **Q-045** | 2026-06-18 | [[../modules/elumatec-replacement-large-rectangle]] | `AddRelativePoint(5/10, ...)` exit-point in LargeRectangle — what does the 5 vs 10 mm difference mean? | low |
| **Q-044** | 2026-06-18 | [[../modules/elumatec-replacement-large-rectangle]] | Two different "large rectangle" thresholds: `>200×>20` (hard-coded) here vs `≥260×≥20` (`app.config`) in `Rectangle.SetWBroach`. Aligned intentionally? | safety-relevant |
| **Q-043** | 2026-06-18 | [[../modules/elumatec-replacement-flowdrill]] | `UseFlowDrillWithCountersink = True` was once `False`. Confirm integrated countersinking is stable enough today. | safety-relevant |
| **Q-042** | 2026-06-18 | [[../modules/elumatec-replacement-flowdrill]] | Magic depth values in Flowdrill recovery (`>59`, `=2`, `=2.1`, `>10`) — what feature-recognition patterns do they correspond to? | medium |
| **Q-041** | 2026-06-18 | [[../modules/elumatec-replacement-flowdrill]] | Non-countersunk Ø9.3 hole silently deactivated — intentional, or is there a downstream manual-add path? | safety-relevant |
| **Q-040** | 2026-06-18 | [[../modules/elumatec-works-replacement-base]] | `WorksReplacement.dtTools` cached at construction — ever stale across a long batchserver session? | safety-relevant |
| **Q-039** | 2026-06-18 | [[../modules/elumatec-works-replacement-base]] | `GetWorksByMacro` / `GetGroupByMacro` swallow exceptions silently — switch to log-and-rethrow? | safety-relevant |
| **Q-038** | 2026-06-18 | [[../mocs/elumatec-works]] | `FrmTestDrwProfile.vb` looks dev-only — confirm `#dead-code` candidate. | low |
| **Q-037** | 2026-06-18 | [[../mocs/elumatec-works]] | Stl-side machines register a much smaller replacement list than Alu. Intentional, or under-implemented? | medium |
| **Q-036** | 2026-06-18 | [[../mocs/elumatec-works]] | Confirm replacement registration order in `Sbz140Alu.New` is intentional (AluGeneral last). | safety-relevant |
| **Q-035** | 2026-06-18 | [[../mocs/elumatec-works]] | What does `BIdentNo = "100381"` represent? Paired with 100142 in Flowdrill rear-side recovery. | low |
| **Q-034** | 2026-06-18 | [[../modules/elumatec-profmill-converter]] | _(resolved 2026-06-18)_ — `AutoReplacementMacroFile = Creo.Environment.GetProManufDir + app.config[AutoReplaceMacros]`. The macro file lives in **Creo's pro-manuf directory**, not in iCenter's deployment. Confirmed via `ProfMillConverter.AutoReplacementMacroFile` (line 1350). | safety-relevant (resolved) |
| **Q-033** | 2026-06-18 | [[../business-rules/elu-large-rectangle-classification]] | Should the 260×20 mm rule be `OR` instead of `AND`? Long thin slots would currently stay contoured. | safety-relevant |
| **Q-032** | 2026-06-18 | [[../business-rules/elu-large-rectangle-classification]] | Confirm 260×20 mm thresholds are correct for all four machine variants (ALU, STL, RVS, SBZ141). | safety-relevant |
| **Q-031** | 2026-06-18 | [[../modules/elumatec-profmill-converter]] | _(resolved 2026-06-18)_ — Only callsite is `ProfMillConverter.SetMaxStepDepth` (line 907), which uses `MaxStepDepth` only as a fallback when `UseTMaxCut = False`. The primary source is per-tool `oMachine.ToolDb.GetMaxCut(WToolID)` — documented in [[../business-rules/elu-tool-max-cut-depth]]. | medium (resolved) |
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
| Q-034 | [[../mocs/elumatec-works]] | AutoReplaceMacros.ncd relative path |
| Q-036 | [[../mocs/elumatec-works]] | replacement registration order |
| Q-039 | [[../modules/elumatec-works-replacement-base]] | silent macro-file exception swallowing |
| Q-040 | [[../modules/elumatec-works-replacement-base]] | dtTools cache staleness |
| Q-041 | [[../modules/elumatec-replacement-flowdrill]] | silent-deactivate non-countersunk flow-drill |
| Q-043 | [[../modules/elumatec-replacement-flowdrill]] | UseFlowDrillWithCountersink hard-coded `True` |
| Q-044 | [[../modules/elumatec-replacement-large-rectangle]] | two large-rect thresholds |
| Q-046 | [[../modules/elumatec-replacement-alu-general]] | enumerate every `Case` in AluGeneral |
| Q-047 | [[../modules/elumatec-replacement-alu-general]] | `Y=-47/-46.5` "nokje breekt" magic |
| Q-048 | [[../modules/elumatec-replacement-alu-general]] | `100180` Koker drainage-hole magic numbers |
| Q-049 | [[../business-rules/elu-largerect-freeform-replacement]] | LargeRectangle silent tool-assign failure |
| Q-050 | [[../mocs/elumatec-ncpipeline]] | EluXml IA-number patcher is a stub |
| Q-053 | [[../mocs/elumatec-ncpipeline]] | ECW arithmetic-expression grammar |
| Q-057 | [[../modules/elumatec-ncstructure-hierarchy]] | Plane translation expression grammar |
| Q-060 | [[../modules/elumatec-elucadfile]] | ECW parse errors silently → 0 |
| Q-062 | [[../modules/elumatec-nc-program-family]] | AUF cycle-time path not yet verified |
| Q-065 | [[../modules/elumatec-profmill-converter]] | Forster override dead since `UseTMaxCut=True` |
| Q-066 | [[../modules/elumatec-profmill-converter]] | `UseFileBasedSettings` dual-emit gate |
| Q-068 | [[../modules/elumatec-profmill-converter]] | `SetWMillDir = -1` now applies to aluminium |
| Q-070 | [[../modules/elumatec-profmill-converter]] | `DefaultWHelixIntr` business rule |
| Q-071 | [[../modules/elumatec-profmill-job]] | Manual AUF override / audit trail |
| Q-072 | [[../modules/elumatec-profmill-job]] | `ProfMillConverter.ProfMillJob` loses dual-emit first job |
| Q-074 | [[../modules/elumatec-profmill-job]] | `SetReleaseLevel` auto-approval |
| Q-075 | [[../business-rules/elu-tool-max-cut-depth]] | `GetMaxCut = 0` → silent single-pass full-depth |
| Q-076 | [[../business-rules/elu-tool-max-cut-depth]] | Tool DB change-control around TMaxCut |
| Q-077 | [[../business-rules/elu-tool-max-cut-depth]] | `SkipSplitSteps` upstream guarantee |
| Q-078 | [[../business-rules/elu-forster-thumbhole-step-depth]] | Identify Forster BIdentNos |
| Q-079 | [[../business-rules/elu-forster-thumbhole-step-depth]] | Forster override: remove or migrate |
| Q-080 | [[../business-rules/elu-dual-emit-sbz140-sbz141]] | Dual-emit asymmetry |
| Q-081 | [[../business-rules/elu-dual-emit-sbz140-sbz141]] | Dual-emit per-machine filepath collision check |
| Q-083 | [[../business-rules/outsource-ext-oper-part-code]] | `UITBESTEDING01` hard-coded ext-operation part code |
| Q-084 | [[../modules/workprep-outsource-operations]] | Only `ProfileId = 1` outsourcing implemented |
| Q-085 | [[../business-rules/icenter-operation-machgrp-mapping]] | iCenter operation IDs (1/9/31 only) |
| Q-094 | [[../modules/workprep-outsource-operations]] | `SetShopDocFinInd(True)` commented out |
| Q-095 | [[../modules/workprep-operation-substitution]] | Swapped surface-treatment table sources (bug?) |
| Q-100 | [[../modules/production-profile-cut-items]] | Unconditional `Clear(MachineId)` |
| Q-101 | [[../modules/production-profile-cut-items]] | `MachineId = Math.Max(iPPartId, 0)` copy-paste |
| Q-106 | [[../business-rules/icenter-operation-machgrp-mapping]] | Operation 1 vs 31 MachGrpCode ordering |
