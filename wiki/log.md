---
type: meta
title: "Operation Log"
status: active
created: 2026-06-18
updated: 2026-06-18
tags: [meta, log]
---

# Operation Log

## 2026-06-18 — Phase 3c-2: ICenterLib/ISAH foundation entities

- Opened ICenterLib's ISAH folder (65 files). Created [[mocs/icenterlib-isah]] sub-MOC with size + role catalogue + recommended doc roadmap.
- Wrote 4 module notes covering 10 ISAH entities:
  - [[modules/isah-lookups]] — Selection (universal `T_Selection` accessor via `IP_sel_SelectionRecord`) + Setting (single-method stub).
  - [[modules/isah-identity]] — Employee + User + Customer + Vendor + Company. Hard-coded JAZO/`041` and FlowGrill/`042`. Employee has dead DTR-status SQL branches (Q-132). User is pure static; `GetJobDescription` reads `MemoTypeCode='JEM10' AND LangCode='NL'` (Q-134).
  - [[modules/isah-shop-and-pur-doc]] — ShopDoc + PurDoc. **Key finding**: ShopDoc state actually lives on `T_ProdBillOfOper` (Q-136). `SetShopDocStartedInd(started)` ignores its argument (Q-142).
  - [[modules/isah-machgrp]] — MachGrp + the `TR\d\d` track-operation regex.
- Wrote 2 new business-rule notes:
  - [[business-rules/isah-company-codes]] — JAZO `041/JAZO` vs FlowGrill `042/FLOWGRIL`.
  - [[business-rules/isah-track-operation-pattern]] — `TR\d\d` regex.
- Opened Q-132..Q-147 (16 new questions, 1 `#safety-relevant`: Q-133 — `Employee.GetIsObsolete` fail-closed on ISAH outage).
- Coverage delta: +8 done (Selection, Setting, Employee, User, Company, ShopDoc, PurDoc, MachGrp), +2 needs-review (Customer, Vendor — small, deferred). Totals: 67 done / 1090 todo / 445 config / 414 generated / 9 needs-review of 2025.
- **Next:** ISAH dossier hierarchy (DossierMain, DossierDetail) then ISAH production hierarchy (ProductionHeader, PBOO, PBOM, PBOS, BillOfOper, BillOfMat).

## 2026-06-18 — Phase 3c-1: ICenterLib survey + foundation files

- User chose ICenterLib as the next subsystem (after stopping Elumatec at 31/157 done).
- Created [[mocs/icenterlib]] top-MOC catalogueing 35 top-level folders + the 723-file scope. Lays out a per-priority roadmap (ISAH > iCenter > SmtProduction > CAD > PCFNet > the rest).
- Wrote 3 root-file module notes:
  - [[modules/icenterlib-common]] — static helpers + magic constants (533 lines). Defines IA/IAK/PRN part-code prefixes, `GuestEmpId="0000"`, 40-49 work-view status range, terminal-server/RAS/ADS hostname branching, `IsSharedWindowsAccount → only "PVS"`. Found `GetTableData` server-side `EXEC` pattern (Q-109).
  - [[modules/icenterlib-connections]] — 15 connection factories. **All hard-coded credentials** including ISAH `sa` (Q-107, Q-108, Q-118, Q-119). `ConnectICenter2Development` connects to raw dev IP `10.11.70.32` as `sander-h`. `UseIsahTestDb` is process-mutable (Q-110).
  - [[modules/icenterlib-appsettings]] — `T_ApplicationSettings` reader via `SIP_GetAppSetting` SP. `IsInsideMaintenanceWindow` reads `My.Settings` (Q-112). `SmtDeburrSpeed = 0.225 m²/sec` constant lives here (Q-122).
- Wrote 4 new business-rule notes:
  - [[business-rules/icenter-part-code-prefixes]] — IA / IAK / PRN.
  - [[business-rules/icenter-status-code-default-range]] — default work-view 40-49.
  - [[business-rules/icenterlib-maintenance-window]] — default 01:00-04:00 if `My.Settings` unset.
  - [[business-rules/smt-deburr-speed]] — `0.225 m²/min` hard-coded.
- Opened 25 new Q-107..Q-131 (11 `#safety-relevant`). Most consequential:
  - **Q-107**: hard-coded DB credentials throughout `Connections.vb` — `p2yeXeC7` shared across iCenter/JIBA/Windchill/ProductDb/Isah/TruTops Oseon; ISAH `sa` with literal password; Kardex creds.
  - **Q-109**: `Common.GetTableData(TableName)` uses `EXEC('SELECT * FROM ' + @TableName)`.
  - **Q-118**: `ConnectICenter2Development` hard-codes a developer's IP + Windows-username + plaintext password.
  - **Q-119**: ISAH `sa` login used by `ConnectIsahSA`; document who needs it.
  - **Q-114**: `Common.GetSqlInString` string-concats values — apostrophe in any value breaks SQL.
- Coverage delta: +3 done (Common.vb, Connections.vb, AppSettings.vb). Totals: 59 done / 1100 todo / 445 config / 414 generated / 7 needs-review of 2025.
- **Next:** ISAH subsystem deep-dive (65 files; the most-referenced ICenterLib folder from iCENTER).


Append-only chronological record. **Newest entries at the top.** Never edit past entries.

---

## 2026-06-18 — Phase 3b-1: Office → shop-floor handoff (Sales / Engineering / WorkPreparation / Production)

- User stopped Elumatec deep-dive at 31/157 files ("is in-depth enough"). Pivoted to office-to-shop-floor workflow.
- Created [[mocs/office-to-shopfloor]] sub-MOC covering 20 files across 4 folders.
- Wrote 7 module notes:
  - [[modules/sales-customer-team]] — hard-coded sales team codes 031/032/033.
  - [[modules/workprep-outsource-operations]] — the big outsourcing pipeline (OutsourceOperationsHandler + FrmOutsourceOperations). Per-vendor STEP+PDF packaging, PurOrd creation, IsahDoc drop, ShopDoc/IPpart state updates.
  - [[modules/workprep-operation-substitution]] — swap machine-group operations. **Likely bug Q-095**: swapped surface-treatment table sources.
  - [[modules/workprep-ipbatch-collector]] — single SQL query against ISAH `T_ProdBillOfOper` (joins six tables).
  - [[modules/production-profile-cut-items]] — per-machine cut-items handler. Only operations 1/9/31 implemented. **Likely bug Q-101**: `MachineId = Math.Max(iPPartId, 0)` copy-paste.
  - [[modules/production-profile-milling-import]] — UniLink CSV import pipeline (5 files). The iCenter→UniLink counterpart to the Elumatec NC pipeline.
  - [[modules/engineering-overview]] — 9 Engineering forms incl. `FrmDrwCheck` (PDF-XChange ActiveX viewer), `FrmDesignCodeTool` (WebView2 to tekeningnummers.jazo.com), `ModelCopies/CopyLocalizer` family.
- Wrote 3 new business-rule notes:
  - [[business-rules/sales-team-codes]] — `{031, 032, 033}`.
  - [[business-rules/outsource-ext-oper-part-code]] — `UITBESTEDING01` hard-coded.
  - [[business-rules/icenter-operation-machgrp-mapping]] — `1→A01,A07`, `9→S01`, `31→A07,A01`.
- Opened 25 new questions Q-082..Q-106 (11 `#safety-relevant`). Most notable:
  - **Q-095** swapped surface-treatment tables in OperationSubstitutionHandler.
  - **Q-100, Q-101** copy-paste / unconditional-clear in ProductionProfileCutItemsHandler.
  - **Q-094** `SetShopDocFinInd(True)` commented out — is ShopDoc ever marked finished?
  - **Q-090** brittle JavaScript injection against `tekeningnummers.jazo.com`.
- Coverage delta: +20 done (1 Sales + 4 WorkPreparation + 6 Production + 8 Engineering + 1 needs-review). Totals: 56 done / 1103 todo / 445 config / 414 generated / 7 needs-review of 2025.
- **Next:** user-chosen (CadBatchserver, Classes/, SmtManufacturing, or companion projects).

## 2026-06-18 — Phase 3a-5: Elumatec ProfMillJob + ProfMillConverter batch

- Wrote 2 module notes:
  - [[modules/elumatec-profmill-converter]] — the orchestrator. Documented `Optimize()` (entry point, validates + picks machine + dual-emits + invokes ConvertCut) and the 17-step `ConvertCut` pipeline (each step with one-line role + flagged subtleties from inline CB-dated comments).
  - [[modules/elumatec-profmill-job]] — the runtime container. Overview-only for 1352 lines / ~60 methods grouped into 12 behavioural clusters: construction, NC export (4), import (2), NCX post-processing (6), manual AUF override (2), cycle time (4), production-machine timing (1), runtime manipulations (4), navigation (5), job merging (1), status flags (8), misc (5).
- **Q-031 resolved**: `Sbz14x.MaxStepDepth` is consumed only in `ProfMillConverter.SetMaxStepDepth` (line 907), and only as the fallback when `UseTMaxCut = False` (currently hard-coded to `True`). Primary source in production is per-tool `oMachine.ToolDb.GetMaxCut(WToolID)` from the `.nct` tool DB.
- **Q-034 resolved**: `AutoReplacementMacroFile = ICenterLib.CAD.Creo.Environment.GetProManufDir + app.config[AutoReplaceMacros]`. Macro file lives in **Creo's pro-manuf install directory**, not in iCenter's deployment tree.
- Wrote 3 new business-rule notes:
  - [[business-rules/elu-tool-max-cut-depth]] — the live primary step-depth rule.
  - [[business-rules/elu-forster-thumbhole-step-depth]] — the dead-code Forster override on the fallback branch. `#dead-code` tagged.
  - [[business-rules/elu-dual-emit-sbz140-sbz141]] — Sbz140Alu also emits for Sbz141Alu when `AppVersion.UseFileBasedSettings`. Asymmetric (Q-080).
- **Corrected** [[business-rules/elu-max-step-depth]] — added a prominent banner stating the rule is fallback-only and currently inactive in production.
- Opened 17 new Q-065..Q-081 (13 `#safety-relevant`). Notable:
  - Q-071 — manual AUF override silently replaces generated output, no audit trail. `#safety-relevant`
  - Q-075 — `GetMaxCut = 0` → `SplitSteps` skipped → silent single-pass full-depth cut. `#safety-relevant`
  - Q-074 — `SetReleaseLevel` can trigger Windchill auto-approval; document trigger conditions. `#safety-relevant`
  - Q-068 — `SetWMillDir = -1` now applies to aluminium (per CB 2023-02-28). `#safety-relevant`
- Coverage delta: +2 done (ProfMillConverter, ProfMillJob). Totals: 36 done / 1124 todo / 445 config / 414 generated / 6 needs-review of 2025.
- **Next:** per-feature Work subclasses (Circle, Drill, SlottedHole, FreeForm, Sawcut, Group, Macro, Deburr, FreeFormPoint) + remaining replacement macros (DoublePnotch, AluHinge, OpdekH, ...).

## 2026-06-18 — Phase 3a-4: Elumatec NC structure + emission batch

- Created [[mocs/elumatec-ncpipeline]] cataloguing 20 files across `Elumatec\NcStructure\` (6 files) and `Elumatec\AufSerializer\` (14 files).
- Wrote 3 module notes:
  - [[modules/elumatec-ncstructure-hierarchy]] — `Job`/`Bar`/`Cut`/`Plane`/`PlaneCollection` in-memory model. Documented the public surface of all five classes, the `Bar.AddCut` plane-transfer invariant (Q-055), the `PlaneCollection.GetPlaneByWSide → WSide-7` mapping (Q-051 — only handles custom planes), and that `Cut` implements `ICloneable` (used by replacements).
  - [[modules/elumatec-elucadfile]] — the `.ecw` text-format parser. Hand-rolled state machine. Numeric cells accept arithmetic expressions via `DataTable.Compute` (Q-053). Parse errors silently → 0 (Q-060). Empty-line terminates Work blocks (Q-054).
  - [[modules/elumatec-nc-program-family]] — the AUF + EluXml serialiser families (14 helper files + 2 main + 1 abstract). Documented format detection (`<?xml` prefix), the AUF state-machine markers (`[BEGIN_AUFTRAG]` / `[BEGIN_PROGRAMM]` / `[BEGIN_KONTUR]`), and the German key inventory. **Critical finding: `NcProgramEluXml.GetNCStringWithIaNr` is a stub that returns input unchanged** (Q-050, `#safety-relevant`). EluXml `ComputeCycleTime` is what feeds the `UseSbzCalculatedDuration = True` global; AUF equivalent not yet seen (Q-062).
- Opened 15 new Q-050..Q-064 (7 `#safety-relevant`).
- Coverage delta: +20 done (Job, Bar, Cut, Plane, PlaneCollection, EluCadFile, NcProgram, NcProgramAuf, NcProgramEluXml, Programm, Kontur, ZeileAuftrag, ZeileProgramm, ZeileKontur, ZeileTTab, EluXmlProgram, EluXmlProgramDetail, EluXmlJob, EluXmlJobItem, EluXmlJobSubItem). Totals: 34 done / 1126 todo / 445 config / 414 generated / 6 needs-review of 2025.
- **Next:** `ProfMillJob.vb` (70 KB) + `ProfMillConverter.vb` (67 KB) — the runtime orchestration that bridges WorksReplacement and the NC structure built here.

## 2026-06-18 — Phase 3a-3: Elumatec Works/Replacements pipeline batch

- Created [[mocs/elumatec-works]] catalogueing all 40 `Works\` files (13 feature classes + 19 replacement macros + base classes + UI helpers).
- Wrote 5 module notes:
  - [[modules/elumatec-work-base]] — `Work.vb` abstract base (92 KB); documented the ~25 `MustOverride` surface, `Sides` / `SidesNl` / `Direction` enums (English and Dutch share integer codes), the `Replaced` sticky flag, and the `RuntimeManipulations` hook.
  - [[modules/elumatec-works-replacement-base]] — `WorksReplacement` thin abstract base (only 55 lines). Established the macro-file dependency (`AutoReplaceMacros.ncd`) and the `DeleteWorks`/`AddWorks`/`ReplaceDict` mutation pattern that all replacements follow.
  - [[modules/elumatec-replacement-flowdrill]] — dense walkthrough of `Flowdrill.vb`. Heavy profile-specific recovery for BIdentNo 100142/100381 misrecognition. Documented the `UseFlowDrillWithCountersink` hard-coded local + the silent deactivation path for non-countersunk Ø9.3 holes.
  - [[modules/elumatec-replacement-large-rectangle]] — `LargeRectangle.vb`. Profile-restricted (100116/100285/100103). Hard-coded `>200×>20` thresholds. Two FreeForm shape variants (sharp vs rounded corners).
  - [[modules/elumatec-replacement-alu-general]] — overview only for the 119 KB catch-all. Documented the structure (one big `Select Case BIdentNo`) and the first ~120 lines of branches; per-branch notes deferred to Q-046 (~20 case branches estimated).
- Wrote 2 new business-rule notes:
  - [[business-rules/elu-flowdrill-replacement]] — Ø9.3 mm / deep holes → flow-drill macros `EC00601`/`EC00054`. Non-countersunk holes silently deactivated.
  - [[business-rules/elu-largerect-freeform-replacement]] — door-needle large rectangles → FreeForm polylines.
- **Corrected** [[business-rules/elu-large-rectangle-classification]] — flagged the two-rules distinction (SetWBroach all-profile vs LargeRectangle profile-restricted). Q-044 asks whether the two thresholds should be aligned.
- Opened 16 new Q-034..Q-049 (9 `#safety-relevant`). Notable: silent exception swallowing in macro-file reads (Q-039), silent deactivation of non-countersunk flow-drill holes (Q-041), two divergent "large rectangle" thresholds (Q-044), hard-coded magic geometry in AluGeneral (Q-047/Q-048).
- Coverage delta: +4 done (Work.vb, WorksReplacement.vb, Flowdrill.vb, LargeRectangle.vb), +1 needs-review (AluGeneral.vb). Totals: 14 done / 1146 todo / 445 config / 414 generated / 6 needs-review of 2025.
- Pushed all earlier commits (Phase 1 through Phase 3a-2) to origin/main before this batch.
- **Next:** NC structure + emission (`NcStructure\*`, `AufSerializer\*`), then `ProfMillJob` + `ProfMillConverter`.

## 2026-06-18 — Phase 3a: first Elumatec batch

- Built the Elumatec subsystem hub at [[mocs/elumatec]] — catalogues all 140 `.vb` files across 8 sub-folders and the recommended Phase-3 ordering.
- Wrote 3 module notes:
  - [[modules/elumatec-com-watcher]] — `FrmComWatcher` + `ClsComWatcher` + `CRs232`. **Resolves Q-006**: it's a serial COM-port watcher for the saw (MachineId 2, COM1, 9600 8-N-1). Most of its original behaviour is commented out (Q-019).
  - [[modules/elumatec-cad-app]] — overview-only for the 128 KB `EluCadApp.vb` god-class. Marked `needs-review` until per-cluster sub-notes land.
  - [[modules/elumatec-machine-base]] — abstract `Sbz14x` + 4 concrete variants (Sbz140Alu/Stl/Rvs, Sbz141Alu). Documented the NC-program generation pipeline (`CreateNCX` spawns external `NcxExePath` 3–4× per file) and the per-variant constants (one explicitly labelled `'Guess`).
- Wrote 2 business-rule notes (both `#safety-relevant`):
  - [[business-rules/elu-max-step-depth]] — `EluMaxStepDepth*` 1.6mm STL / 6mm ALU. Same setting reused for Sbz140Stl and Sbz140Rvs (Q-027). `Double.Parse` without culture (Q-030 / culture-fragility note).
  - [[business-rules/elu-large-rectangle-classification]] — `EluLargeRectangle*` 260×20mm `AND`-thresholds. Q-033 questions whether `AND` should be `OR`.
- Opened 15 new SME questions Q-019..Q-033 (11 `#safety-relevant`). Resolved Q-006.
- Coverage delta: +5 done (FrmComWatcher.vb, ClsComWatcher.vb, CRs232.vb, Sbz14x.vb, Sbz140Alu.vb), +4 needs-review (EluCadApp.vb, Sbz140Stl/Rvs, Sbz141Alu). Totals: 10 done / 1151 todo / 445 config / 414 generated / 5 needs-review of 2025.
- **Next:** continue Elumatec — `Works\` + `Works\Replacements\` pipeline (highest safety value), then NC structure + emission, then ProfMillJob/Converter.

## 2026-06-18 — Scope widened to TruTopsLib + ICenterLib

- User direction: widen scope to include `TruTopsLib` and `ICenterLib` (resolving [[needs-review/_index|Q-001]]).
- TruTopsLib located at `C:\DevOps\iCenter\iCenter\TruTopsLib\` — 65 files, mixed `.vb` + `.cs` (the project ships both `.csproj` and `.vbproj`).
- ICenterLib **not** at the path the `iCenter.vbproj` / `iCENTER.sln` reference. User pointed me at `C:\Users\jelle-r\source\repos\JIBA\iCenter And Tools\ICenterLib\ICenterLib\` — 723 files across 35 top-level folders that mirror iCENTER's structure. Build-path discrepancy logged as new **Q-018**.
- Inventoried both projects via `Get-ChildItem -Recurse` and spliced per-project sections into [[_coverage]]. Combined totals: 2025 files / 5 done / 1160 todo / 445 config / 414 generated / 1 needs-review.
- Updated [[../CLAUDE]] to list all three in-scope roots.
- Refreshed [[overview]], [[index]], [[hot]], [[meta/conventions]], [[architecture/_index]], [[architecture/project-references]], [[modules/_index]] to reflect the wider scope.
- **Next:** Phase 3 — Elumatec subsystem deep dive (140 `.vb` files, `#safety-relevant`).

## 2026-06-18 — Phase 2 architecture pass complete

- Read `iCenter.vbproj` (3399 lines), `app.config` (313 lines), `ApplicationEvents.vb`, `packages.config`, `Modules\Main.vb` (266 lines), and the top 200 lines of `FrmMain.vb` (god-form, 15 216 total).
- Wrote 6 architecture pages: [[architecture/entry-points]], [[architecture/runtime-modes]], [[architecture/global-state]], [[architecture/build-and-deploy]], [[architecture/project-references]], [[architecture/external-surface]] + refreshed [[architecture/_index]].
- Rewrote 6 sub-index pages ([[external-systems/_index]], [[business-rules/_index]], [[domain-concepts/_index]], [[modules/_index]], [[mocs/_index]], [[needs-review/_index]]) to reflect the actual iCENTER folder layout — old scaffolds referenced non-existent `JAZO.iCenter.*` projects.
- Wrote first concrete module note: [[modules/main-module]].
- Surfaced **17 SME questions** (Q-001 through Q-017) in [[needs-review/_index]], of which 4 are `#safety-relevant`.
- Marked 5 files `done` in [[_coverage]]: `app.config`, `ApplicationEvents.vb`, `iCenter.vbproj`, `packages.config`, `Modules\Main.vb`. `FrmMain.vb` flagged `needs-review` pending its Phase-3 split. Roll-up totals now 5 done / 532 todo / 341 generated / 358 config / 1 needs-review (still 1237 total).
- Updated [[overview]], [[hot]], [[meta/conventions]], and [[_templates/module]] + [[_templates/business-rule]] to remove leftover `iCenter2` / `.cs` placeholders.
- **Next:** Phase 3 — deep module docs. Recommended order by impact: `Elumatec/`, `SmtManufacturing/`, `CadBatchserver/`, `Classes/` (sub-MOC), then `FrmMain.vb` split.

## 2026-06-18 — Phase 1 inventory complete

- Scope corrected: `C:\DevOps\iCenter2\` was a guess in the scaffold. Per the updated [[../CLAUDE|standing instructions]], the real target is `C:\DevOps\iCenter\iCenter\iCENTER\` — a single VB.NET WinForms project (`iCenter.vbproj`), not a multi-project solution. The `iCenter2` sibling is **out of scope**.
- Codebase is **not** git-tracked at any walked level. Used `Get-ChildItem -Recurse` (PowerShell) instead of `git ls-files` for the inventory.
- Enumerated 1237 files across 27 top-level sub-folders, excluding `bin`, `obj`, `.vs`, `packages`, `My Project`, `Web References`.
- Filled [[_coverage]] with one table per sub-folder. Heuristic statuses applied: `generated` for `*.Designer.vb` + `*.resx`; `config` for assets, `.vbproj*`, `app.config`, `packages.config`, `.pfx`, `.snk`; `todo` for the remaining 535 `.vb` source files.
- Realigned [[index]], [[overview]], [[hot]], [[meta/conventions]] to the new scope.
- **Next:** Phase 2 — architecture pass. Read `iCenter.vbproj` for dependencies/refs, walk `FrmMain.vb` + `ApplicationEvents.vb` as entry points, sketch top-level data flow and external-system surface (Elumatec, SmtManufacturing, Kardex, UniLink, SolaDataConnector, etc.).

## 2026-06-18 — Vault scaffolded

- Created wiki/ folder structure (architecture, modules, business-rules, domain-concepts, external-systems, mocs, needs-review, meta, _templates)
- Initialized index, log, hot, overview, _coverage
- Wrote 6 note templates (architecture, module, business-rule, domain-concept, external-system, moc)
- Configured .obsidian/ with vault-colors snippet
- Source codebase identified: `C:\DevOps\iCenter2\` (git-tracked)
- **Next:** Phase 1 — inventory via `git ls-files` in iCenter2, populate `_coverage.md` with one row per source file (status=todo)
