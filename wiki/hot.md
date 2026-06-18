---
type: meta
title: "Hot Cache"
updated: 2026-06-18T00:00:00
---

# Recent Context

## Last Updated
2026-06-18. Phase 3a-5: Elumatec ProfMillJob + ProfMillConverter batch landed.

## Key Recent Facts
- Scope: three projects (iCENTER 1237 + TruTopsLib 65 + ICenterLib 723 = **2025 files**).
- Coverage now: **36 done, 6 needs-review, 1124 todo, 445 config, 414 generated**.
- 81 open SME questions (Q-001 + Q-006 resolved + Q-031 + Q-034 resolved this batch). **32 are `#safety-relevant`**.
- Elumatec subfolder coverage: **31 done / 87 todo / 5 needs-review** out of 157. **About 1/3 of Elumatec documented.**
- 7 business-rule notes now live, all `#safety-relevant`.

## ProfMillJob + ProfMillConverter summary (this batch)
- **`ProfMillConverter.Optimize()`** is the single entry point that drives the CAD→NC pipeline for one model.
- **`ConvertCut(machine, cut)`** is a 17-step pipeline (in [[modules/elumatec-profmill-converter]] table). Steps include ReplaceSpecialWorks, RemoveSawCuts, AssignTools, RotateWorksOnBottomside, DeleteDuplicates, SortWorks, SetWDepthPost, **SetMaxStepDepth**, SetWBroach, SetWHelixIntr, SetWMillDir, SetWRotationOfFlowDrillWorksOnBackside, SetWorkNos, ReplaceMill (steel-only), SetWContour, MinimizeMillTools, SuppressMultiSidedMacros (commented-out body!).
- **Q-031 resolved**: `Sbz14x.MaxStepDepth` is consumed *only* in `ProfMillConverter.SetMaxStepDepth`, and *only* as the fallback (`UseTMaxCut = False` branch). **Primary source is per-tool `oMachine.ToolDb.GetMaxCut(WToolID)` from the `.nct` tool database** — corrected [[business-rules/elu-max-step-depth]] and added new [[business-rules/elu-tool-max-cut-depth]].
- **Q-034 resolved**: `AutoReplacementMacroFile = Creo.Environment.GetProManufDir + app.config[AutoReplaceMacros]` → macro file lives in **Creo's pro-manuf install directory**, not iCenter's deployment.
- **Dual-emit discovered**: `Sbz140Alu` jobs additionally emit for `Sbz141Alu` when `AppVersion.UseFileBasedSettings`. Asymmetric (Sbz141Alu→only itself; Q-080). Documented in [[business-rules/elu-dual-emit-sbz140-sbz141]].
- **Forster thumb-hole override** (`MaxStepDepth = 6` for Rectangle features with `WY1∈(48,50)` AND `WDepth>10` AND `WW3=5`) lives in the fallback branch — currently dead. New `#dead-code`-tagged note [[business-rules/elu-forster-thumbhole-step-depth]].
- **Manual AUF override path**: `GetManualAufExists(ecwFilePath)` checks `\\jazo.local\dfs\pm\Elumatec_SBZ140\Aanpassen\` for a hand-edited `.auf` that silently replaces iCenter output. No audit trail. `#safety-relevant` (Q-071).
- **`SuppressMultiSidedMacros` body entirely commented out** — only an empty Try/Catch remains (Q-069).
- **`KeepAufAsSeperateFile = True`** always-on with comment `' Functions.DebugMode` — was supposed to be debug-only but isn't (Q-073).

## Recent Changes
- Created 2 module notes: [[modules/elumatec-profmill-converter]] (the orchestrator), [[modules/elumatec-profmill-job]] (the runtime container, ~60 methods).
- Created 3 new business-rule notes: [[business-rules/elu-tool-max-cut-depth]], [[business-rules/elu-forster-thumbhole-step-depth]], [[business-rules/elu-dual-emit-sbz140-sbz141]].
- **Corrected** [[business-rules/elu-max-step-depth]] — added a prominent banner noting the rule is fallback-only and currently inactive.
- Resolved Q-031 and Q-034. Opened Q-065..Q-081 (17 new questions, 13 `#safety-relevant`).
- Updated [[_coverage]] (+2 Elumatec done); rollup totals.
- Updated [[business-rules/_index]] and [[needs-review/_index]].

## Active Threads
- Recommended next Elumatec batches:
  1. **Per-feature Work subclasses** — Circle, Drill, SlottedHole, FreeForm, Sawcut, Group, Macro, Deburr, FreeFormPoint (and complete Rectangle). Each ~5-25 KB. Should close the abstract-surface gaps documented in [[modules/elumatec-work-base]].
  2. **Remaining replacement macros** — DoublePnotch (19 KB), AluHinge (18 KB), OpdekH, RDHS27Notch, AluSRkom, AluHUPO, ExtraLength, AluSinglePnotch + Stl-side (StlGeneral 13 KB, StlHinge, StlDoublePnotch, StlFlowDrill).
  3. **`AutoProfMillProgApproval.vb`** (25 KB) — auto-approval of profile-mill programs. `#safety-relevant`.
  4. **`Database\Profile.vb`** (54 KB) — the central profile DB entity + tool DB + offsets + fixtures.
  5. **`ClsSawList.vb`** (24 KB), **`CtrlProfMillElu.vb`** (43 KB), **`CtrlProfMillCam.vb`** (37 KB) — UI surfaces.
- Once Elumatec saturates, **SmtManufacturing** (63 `.vb`, `#safety-relevant`) and **TruTops Oseon** types in ICenterLib next.

## Notes from working tree
- Three Obsidian auto-stubs at wiki root (`jiba-portal.md`, `kardex.md`, `trutops-oseon.md`) and `.obsidian/` autoupdates remain unstaged.
