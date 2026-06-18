---
type: meta
title: "Hot Cache"
updated: 2026-06-18T00:00:00
---

# Recent Context

## Last Updated
2026-06-18. Phase 3a-4: Elumatec NC structure + emission batch landed.

## Key Recent Facts
- Scope: three projects (iCENTER 1237 + TruTopsLib 65 + ICenterLib 723 = **2025 files**).
- Coverage now: **34 done, 6 needs-review, 1126 todo, 445 config, 414 generated**.
- 64 open SME questions (Q-001 + Q-006 resolved). 27 are `#safety-relevant`.
- **Elumatec subsystem now has three sub-MOCs**: [[mocs/elumatec]] (top), [[mocs/elumatec-works]] (Works/Replacements pipeline), [[mocs/elumatec-ncpipeline]] (NC structure + emission).
- Elumatec subfolder coverage: **29 done / 89 todo / 5 needs-review** out of 157. Roughly 1/3 documented.

## NC pipeline summary (this batch)
- **In-memory hierarchy**: `Job` (collection of Bars by PartCode) → `Bar` (one profile bar) → `Cut` (one piece) → `Work` (one feature). `Plane` is the bar-face coordinate-system; `PlaneCollection` holds custom planes only (standard sides 1-6 implicit; `GetPlaneByWSide(WSide → WSide-7)` returns Nothing for sides 1-6 — Q-051).
- **`.ecw` parser** (`EluCadFile.ReadFromArray`): hand-rolled state machine, German-keyword section headers, empty-line-terminates-Work. Numeric cells accept arithmetic expressions via `DataTable.Compute` (Q-053). Parse errors silently → 0 (Q-060).
- **AUF format** (`NcProgramAuf` + Programm/Kontur/ZeileAuftrag/ZeileProgramm/ZeileKontur/ZeileTTab): older, section-block German text. Markers `[BEGIN_AUFTRAG]`, `[BEGIN_PROGRAMM]`, `[BEGIN_KONTUR]`. Parsed by a 7-flag state machine. `vbCrLf`-only line splits (Q-064).
- **EluXml format** (`NcProgramEluXml` + EluXmlProgram/ProgramDetail/Job/JobItem/JobSubItem): newer, XML. Auto-detected via `<?xml` prefix. Implements `ComputeCycleTime` (the AUF path may not — Q-062). `GetNCStringWithIaNr` is a stub that returns input unchanged (Q-050, `#safety-relevant`).
- **`UseSbzCalculatedDuration = True`** global routes scheduling estimates to `NcProgramEluXml.ComputeCycleTime` — confirms the EluXml path is load-bearing for production planning.

## Recent Changes
- Created [[mocs/elumatec-ncpipeline]] (sub-MOC for 20 files).
- Created 3 module notes: [[modules/elumatec-ncstructure-hierarchy]] (Job/Bar/Cut/Plane), [[modules/elumatec-elucadfile]] (.ecw parser), [[modules/elumatec-nc-program-family]] (AUF + EluXml serialisers).
- Opened Q-050..Q-064 (15 new questions, 7 `#safety-relevant`). Most notable: Q-050 (EluXml IA-number stub), Q-053 (ECW arithmetic-expression grammar), Q-057 (Plane translation expressions), Q-060 (silent parse-error → 0), Q-062 (AUF cycle-time path).
- Updated [[_coverage]] (+20 done in Elumatec/NcStructure + AufSerializer); rollup totals.
- Updated [[mocs/_index]] to list the new sub-MOC.

## Active Threads
- Recommended next Elumatec batches (Phase 3a continued):
  1. **`ProfMillJob` (70 KB) + `ProfMillConverter` (67 KB)** — runtime job orchestration; bridges the NcStructure model and the WorksReplacement pipeline. This will close Q-031 (callsites of `Sbz14x.MaxStepDepth`) and likely surface more business rules.
  2. **Per-feature Work subclasses** — Circle, Drill, SlottedHole, FreeForm, Sawcut, Group, Macro, Deburr, FreeFormPoint. Each ~5-25 KB. Complete `Rectangle.vb` too.
  3. **Remaining replacement macros** — DoublePnotch, AluHinge, OpdekH, RDHS27Notch, AluSRkom, AluHUPO, ExtraLength, AluSinglePnotch + Stl-side (StlGeneral, StlHinge, StlDoublePnotch, StlFlowDrill).
  4. **`AutoProfMillProgApproval.vb`** — auto-approval. `#safety-relevant`.
  5. **`Database\Profile.vb`** (54 KB) + tool DB + offsets + fixtures.
- Once Elumatec saturates, **SmtManufacturing** (63 `.vb`, `#safety-relevant`) and **TruTops Oseon** types in ICenterLib next.

## Notes from working tree
- Three Obsidian auto-stubs at wiki root (`jiba-portal.md`, `kardex.md`, `trutops-oseon.md`) and `.obsidian/` autoupdates remain unstaged.
