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
