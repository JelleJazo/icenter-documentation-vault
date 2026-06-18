---
type: meta
title: "Hot Cache"
updated: 2026-06-18T00:00:00
---

# Recent Context

## Last Updated
2026-06-18. Phase 3a-3: Elumatec Works/Replacements pipeline batch landed and pushed.

## Key Recent Facts
- Scope: three projects (iCENTER 1237 + TruTopsLib 65 + ICenterLib 723 = **2025 files**).
- Coverage now: **14 done, 6 needs-review, 1146 todo, 445 config, 414 generated**.
- 49 open SME questions (Q-001 + Q-006 resolved; Q-019..Q-049 opened across the Elumatec batches). 22 are `#safety-relevant`.
- **Elumatec subsystem MOC + Works sub-MOC** — [[mocs/elumatec]] and [[mocs/elumatec-works]].
- **Critical distinction discovered**: there are **two separate "large rectangle" rules** that look similar but differ in scope and effect:
  - `Rectangle.SetWBroach` (all profiles, `app.config` 260×20, sets `WBroach=1` flag)
  - `LargeRectangle.ApplyTo` (only door-needle profiles 100116/100285/100103, hard-coded 200×20, *replaces* Rectangle with FreeForm polyline)
  The existing [[business-rules/elu-large-rectangle-classification]] note has been corrected to flag the distinction; new [[business-rules/elu-largerect-freeform-replacement]] covers the second rule. Q-044 asks whether they should be aligned.
- **Flow-drill replacement** documented end-to-end ([[modules/elumatec-replacement-flowdrill]] + [[business-rules/elu-flowdrill-replacement]]). Ø9.3 mm or deep holes get rewritten to macros `EC00601` (with countersink) or `EC00054` (without). Non-countersunk Ø9.3 holes are *silently deactivated* (Q-041, `#safety-relevant`). Heavy profile-specific recovery for misrecognition (100142, 100381).
- **Macro `.ncd` file** at `Profiles\Resources\AutoReplaceMacros.ncd` (relative path) is part of the deployment surface — referenced by `WorksReplacement.GetWorksByMacro` and used by Flowdrill / AluGeneral / others. Macro IDs `EC00601`, `EC00054` confirmed in use. `#safety-relevant`. Q-034 asks about path resolution.
- **`AluGeneral.vb`** (119 KB) overview only documented ([[modules/elumatec-replacement-alu-general]]). It's a giant `Select Case BIdentNo` accumulating 5+ years of production-floor exceptions. Each `Case` is a candidate business rule (Q-046 to enumerate ~20).
- **Work base class** has 25 abstract methods. `SplitSteps(MaxStepDepth As Double)` is the consumer of `Sbz14x.MaxStepDepth` — partial answer to Q-031.
- 11 profile IDs documented in the Works MOC table (100103/104/105/114/115/116/142/143/180/285/292/293/294/381/999). Each is a Phase-4 domain-concept candidate.

## Recent Changes
- Created [[mocs/elumatec-works]] (sub-MOC, catalogues 40 files).
- Created 5 module notes: [[modules/elumatec-work-base]], [[modules/elumatec-works-replacement-base]], [[modules/elumatec-replacement-flowdrill]], [[modules/elumatec-replacement-large-rectangle]], [[modules/elumatec-replacement-alu-general]].
- Created 2 business-rule notes: [[business-rules/elu-flowdrill-replacement]], [[business-rules/elu-largerect-freeform-replacement]].
- Corrected [[business-rules/elu-large-rectangle-classification]] to flag the two-rule distinction.
- Opened Q-034..Q-049 (16 new questions, 9 `#safety-relevant`).
- Updated [[_coverage]] (+4 Elumatec done, +1 needs-review); [[index]] and rollup.

## Active Threads
- Recommended next Elumatec batches (Phase 3a continued):
  1. **NC structure + emission** — `NcStructure\Cut.vb` (40 KB), `Bar.vb` (20 KB), `Job.vb`, `Plane.vb`; `AufSerializer\NcProgramAuf.vb` (27 KB), `NcProgramEluXml.vb`.
  2. **`ProfMillJob` (70 KB) + `ProfMillConverter` (67 KB)** — runtime job representation and the converter that drives `SplitSteps` (closing Q-031).
  3. **Per-feature subclasses** — Circle, Drill, Rectangle (complete it), SlottedHole, FreeForm, Sawcut. Each likely 1 small note.
  4. **Remaining replacement macros** — DoublePnotch, AluHinge, OpdekH, RDHS27Notch, AluSRkom, AluHUPO, ExtraLength, AluSinglePnotch, plus the Stl-side variants.
  5. **`AutoProfMillProgApproval.vb`** — auto-approval of profile-mill programs. `#safety-relevant`.
  6. **`Database\Profile.vb`** (54 KB) + tool DB + offsets + fixtures.
- Once Elumatec saturates, **SmtManufacturing** (63 `.vb`, `#safety-relevant`) and the **TruTops Oseon** types in ICenterLib are next.

## Notes from working tree
- Three Obsidian auto-stubs at wiki root (`jiba-portal.md`, `kardex.md`, `trutops-oseon.md`) and `.obsidian/` autoupdates remain unstaged.
