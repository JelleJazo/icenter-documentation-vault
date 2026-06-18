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
