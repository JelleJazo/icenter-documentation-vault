---
type: meta
title: "Operation Log"
status: active
created: 2026-06-18
updated: 2026-06-18
tags: [meta, log]
---

# Operation Log

Append-only chronological record. **Newest entries at the top.** Never edit past entries.

---

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
