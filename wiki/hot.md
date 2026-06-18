---
type: meta
title: "Hot Cache"
updated: 2026-06-18T00:00:00
---

# Recent Context

## Last Updated
2026-06-18. Phase 3a: first Elumatec batch landed.

## Key Recent Facts
- Scope: three projects (iCENTER 1237 + TruTopsLib 65 + ICenterLib 723 = **2025 files**).
- Coverage now: **10 done, 5 needs-review, 1151 todo, 445 config, 414 generated**.
- 33 open SME questions (Q-001 resolved, Q-006 resolved, Q-019..Q-033 added in this batch). 11 are `#safety-relevant`.
- **Elumatec subsystem hub** written at [[mocs/elumatec]]. Catalogues the 140 `.vb` files across 8 sub-folders (root, AufSerializer, Database, DXF, Machine, NcStructure, Optimizer, Works/+/Replacements).
- **Q-006 resolved**: `FrmComWatcher` is a **serial COM-port watcher for the saw machine** (MachineId 2, COM1, 9600 8-N-1). Listens for `"ID"` + line-number tokens; updates `prodMachines.LatestLineNr`. The original sticker-printing pipeline is **commented out** (Q-019 — vestigial?). The underlying `CRs232` class is third-party copy-paste (Corrado Cavalli ©2003), unmaintained since 2005 (Q-020).
- **Two safety-relevant business rules written**: [[business-rules/elu-max-step-depth]] (`EluMaxStepDepth*` 1.6mm STL / 6mm ALU) and [[business-rules/elu-large-rectangle-classification]] (`EluLargeRectangle*` 260×20mm AND-thresholds → broach).
- **Sbz14x machine family** documented in [[modules/elumatec-machine-base]]: abstract base + four concrete variants (Sbz140Alu / Sbz140Stl / Sbz140Rvs / Sbz141Alu). Each registers an ordered list of `WorksReplacement` macros. Sbz140Alu registered macros: Flowdrill, DoublePnotch, AluSinglePnotch, LargeRectangle, OpdekH, RDHS27Notch, AluHinge, AluSRkom, AluHUPO, ExtraLength, AluGeneral. The `MachineXFeedRate = 1000` is literally labelled `'Guess` in source (Q-021).
- **EluCadApp.vb** is 128 KB — only the overview note written ([[modules/elumatec-cad-app]]); per-cluster sub-notes pending.
- **NC export pipeline** (`Sbz14x.CreateNCX`): spawns external `NcxExePath` post-processor 3–4× with same output file (Q-028 — does it append or overwrite?). Path comes from `AppVersion`, not `app.config` — installed EluCad version silently changes NC output.

## Recent Changes
- Created [[mocs/elumatec]] (subsystem MOC, 140 files catalogued).
- Created 3 module notes: [[modules/elumatec-com-watcher]], [[modules/elumatec-cad-app]], [[modules/elumatec-machine-base]].
- Created 2 business-rule notes: [[business-rules/elu-max-step-depth]], [[business-rules/elu-large-rectangle-classification]].
- Resolved Q-006, opened Q-019..Q-033 (15 new questions).
- Updated [[_coverage]] (5 Elumatec files now `done`, 4 `needs-review`); refreshed rollup.
- Updated [[business-rules/_index]] and [[mocs/_index]] to reference the new pages.

## Active Threads
- Recommended next Elumatec batches (Phase 3a continued):
  1. **Works / Replacements pipeline** — `Works\Work.vb` (92 KB base) + per-feature subclasses + the 11 Alu replacement macros + Stl equivalents. **Highest safety value.**
  2. **NC structure + emission** — `NcStructure\Cut.vb`, `Bar.vb`, `Job.vb`, `Plane.vb`; `AufSerializer\NcProgramAuf.vb`, `NcProgramEluXml.vb`.
  3. **`ProfMillJob` + `ProfMillConverter`** — runtime job representation and conversion (137 KB combined).
  4. **`AutoProfMillProgApproval.vb`** — auto-approval of profile-mill programs. `#safety-relevant`.
  5. **`Database\Profile.vb`** + tool DB + offsets + fixtures.
- After Elumatec saturates, move to **SmtManufacturing** (63 `.vb`, `#safety-relevant`) and the **TruTops Oseon** types in ICenterLib.

## Notes from working tree
- Two empty 1-line files at wiki root (`jiba-portal.md`, `kardex.md`) — Obsidian auto-stubs from clicking unresolved wikilinks. Left unstaged; user should either delete or fill in proper locations under `external-systems/`.
- `wiki/.obsidian/` got auto-modified by Obsidian during the session; left unstaged.
