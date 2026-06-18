---
type: meta
title: "Hot Cache"
updated: 2026-06-18T00:00:00
---

# Recent Context

## Last Updated
2026-06-18. Scope widened to three projects; starting Elumatec deep dive.

## Key Recent Facts
- Scope now spans **three** projects per [[../CLAUDE]]:
  - `C:\DevOps\iCenter\iCenter\iCENTER\` — 1237 files (WinForms shell).
  - `C:\DevOps\iCenter\iCenter\TruTopsLib\` — 65 files (Trumpf parser, mixed `.vb`/`.cs`).
  - `C:\Users\jelle-r\source\repos\JIBA\iCenter And Tools\ICenterLib\ICenterLib\` — 723 files (shared library; 35 top-level folders mirroring iCENTER's structure).
- Combined inventory: **2025 files**. Status: 5 done, 1160 todo, 445 config, 414 generated, 1 needs-review.
- **Q-018** (new): the `<ProjectReference>` to ICenterLib inside `iCenter.vbproj` (`..\..\ICenterLib\...`) and `iCENTER.sln` (`..\ICenterLib\...`) both point to a path that **doesn't exist** on this machine. The real source is under the user's personal `source\repos\` tree. CI / multi-developer behavior unknown.
- **Q-001** (resolved): both companion projects are in scope.
- Build: .NET Framework 4.8, x86, WinExe, `Option Strict Off`. ClickOnce publish to `Z:\iCenter\BetaRelease\`. Assembly signing **disabled since 2023-07-18** (expired cert).
- Entry point: `iCenter.Main` (custom `Sub Main`) at `Modules\Main.vb:198`. Dispatches on `-m <mode>`.
- Global state: `Modules\Main.vb` is a god-module — ~80 `Public` fields acting as the DI container.
- External surface: 23 systems mapped (see [[architecture/external-surface]]). Safety-relevant: Elumatec SBZ140/DG, TruTops Oseon, PTC Creo, Kardex.
- 18 open SME questions (Q-001…Q-018, Q-001 resolved). 4 are `#safety-relevant`.

## Recent Changes
- Inventoried TruTopsLib (65 files) and ICenterLib (723 files); spliced into [[_coverage]] with per-project sections.
- Updated [[../CLAUDE]] to list all three in-scope roots (the user did this conversationally; reflected in CLAUDE.md by edit).
- Updated [[overview]], [[index]], [[meta/conventions]], [[architecture/_index]], [[architecture/project-references]] to reflect the wider scope.
- Resolved [[needs-review/_index|Q-001]]; opened [[needs-review/_index|Q-018]] about the ICenterLib build-path discrepancy.
- [[modules/_index]] now lists "Companion projects" with sub-area pointers for Phase 3 prioritisation.

## Active Threads
- Starting Elumatec deep dive: 140 `.vb` files across 8 subfolders in `iCENTER\Elumatec\`. `#safety-relevant` (drives a CNC profile mill).
- Plan: build an Elumatec subsystem MOC, then write per-file module notes starting with the entry forms (`FrmProfileView`, `FrmComWatcher`, `CtrlProfMillElu`, `CtrlProfMillCam`), then NC export (`NCStructure/`, `AufSerializer/`), then runtime manipulations (`Works/Replacements/`), then machine + database + DXF helpers.
- Will document `ICenterLib.CAD.Creo` and `ICenterLib.CAD.PLM` types only when they appear in Elumatec call paths (avoid unscoped sprawl).
