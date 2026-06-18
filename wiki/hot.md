---
type: meta
title: "Hot Cache"
updated: 2026-06-18T00:00:00
---

# Recent Context

## Last Updated
2026-06-18. Phase 2 architecture pass complete.

## Key Recent Facts
- Authoritative scope: `C:\DevOps\iCenter\iCenter\iCENTER\` — single VB.NET WinForms project (`iCenter.vbproj`), 1237 files, ~707 `.vb` source files.
- Build: .NET Framework 4.8, x86, WinExe, `Option Strict Off`. ClickOnce publish to `Z:\iCenter\BetaRelease\`. Assembly signing **disabled since 2023-07-18** (expired cert).
- Entry point: `iCenter.Main` (custom `Sub Main`) at `Modules\Main.vb:198`. Reads `-m <mode>`; dispatches to interactive `FrmMain`, `CadBatchserver.FrmCadBatchServer`, or `Functions.UpdateIcenter()`. Post-main can open `Batchserver.FrmBatchServer` or `Elumatec.FrmComWatcher`.
- Global state: `Modules\Main.vb` is a god-module — ~80 `Public` fields act as the DI container. Includes singletons (`oICENTER`, `oISAH`, `oJIBA`, Oseon services, …), constants that act as business rules, hard-coded paths, and at least one hard-coded secret (`sPDFOwnerPassword`).
- Cross-project deps **out of scope** but compiled in: `ICenterLib` (`C:\DevOps\ICenterLib\…`) and `TruTopsLib` (`..\TruTopsLib\…`). Most data-access lives there. **Q-001** open with SME.
- External surface: 23 systems mapped (see [[architecture/external-surface]]). Safety-relevant: Elumatec SBZ140/DG, TruTops Oseon, PTC Creo, Kardex.
- Hard-coded thresholds spotted in `app.config` (e.g. `EluMaxStepDepthSTL=1.6`, `EluMaxStepDepthALU=6`, `SmtMaxValueOutline=2980`, `CoatingPickTimeWarning=16`) — all `#safety-relevant` candidates pending SME confirm.
- 17 open SME questions logged as **Q-001 through Q-017** in [[needs-review/_index]].

## Recent Changes
- Wrote Phase 2 architecture pages: [[architecture/entry-points]], [[architecture/runtime-modes]], [[architecture/global-state]], [[architecture/build-and-deploy]], [[architecture/project-references]], [[architecture/external-surface]], plus refreshed [[architecture/_index]].
- Rewrote every `_index.md` (external-systems, business-rules, domain-concepts, modules, mocs, needs-review) to reflect the actual iCENTER layout.
- Refreshed [[overview]] with the real picture.
- Wrote first concrete module note: [[modules/main-module]].
- Marked 5 root/`Modules` files `done` in [[_coverage]] (`app.config`, `ApplicationEvents.vb`, `iCenter.vbproj`, `packages.config`, `Modules\Main.vb`); `FrmMain.vb` → `needs-review` (15 216-line god-form, Phase-3 split needed).
- Fixed [[_templates/module]] + [[_templates/business-rule]] to use new path scope and `.vb` instead of `.cs`.

## Active Threads
- Phase 3 (deep module docs) is the next batch. Largest-impact folders to start with:
  1. `Elumatec/` (140 `.vb`, `#safety-relevant`)
  2. `SmtManufacturing/` (63 `.vb`, `#safety-relevant`)
  3. `CadBatchserver/` (27 `.vb`, headless mode)
  4. `Classes/` (101 `.vb`) — needs a sub-MOC due to size
  5. `FrmMain.vb` — split into multiple notes by tab/feature area
- Phase 4 business-logic sweep can begin in parallel once the watchlist in [[business-rules/_index]] turns into actual pages.
- Open question for the user: should the wiki also cover `ICenterLib` / `TruTopsLib`? See **Q-001**.
