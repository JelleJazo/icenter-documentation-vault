---
type: meta
title: "Hot Cache"
updated: 2026-06-18T00:00:00
---

# Recent Context

## Last Updated
2026-06-18. Phase 1 inventory underway.

## Key Recent Facts
- Authoritative scope per [[../CLAUDE]] is `C:\DevOps\iCenter\iCenter\iCENTER\` — a single VB.NET WinForms project (`iCenter.vbproj`) with ~30 sub-folders (CAD, CAM, Sales, WorkPreparation, Production, Engineering, Kardex, Elumatec, SmtManufacturing, UniLink, …).
- Codebase is **not** under git at any walked level — inventory must be built with `Get-ChildItem -Recurse` / ripgrep, not `git ls-files`.
- Earlier scaffold mentioned the `iCenter2` sibling folder; that has been superseded by the CLAUDE.md update. `iCenter2` is **out of scope** unless the user re-opens that question.
- Mission has two non-negotiable goals: (1) full coverage of every source file, (2) every business rule surfaced with path+symbol and flagged for SME review.
- Code touches physical production processes — anything process/setpoint/interlock/safety-relevant gets #needs-review and #safety-relevant.

## Recent Changes
- Realigned [[index]], [[overview]], [[_coverage]], [[meta/conventions]] to the new scope (`iCenter\iCenter\iCENTER`).
- Vault layout unchanged: still uses [[_templates|six templates]], `_coverage.md`, sub-indexes per note type.

## Active Threads
- Phase 1 inventory in progress: enumerate every source file under `iCENTER\` (excluding `bin\`, `obj\`, `.vs\`, `packages\`, `Web References\`, `My Project\`).
- Open question for SME: are the legacy `*.Designer.vb` partials hand-maintained or VS-generated? Default assumption: generated → `status: generated`.
