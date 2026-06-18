---
type: moc
title: "Architecture — Map of Content"
status: draft
tags: [moc, architecture]
created: 2026-06-18
updated: 2026-06-18
---

# Architecture

Cross-cutting structural views of the `iCENTER` project. Each note here is a *view* of the codebase; per-folder module notes live in [[../modules/_index|modules/]].

> **Anchor:** [[../overview|overview]] — one-paragraph "what is iCenter".

## Views

- [[entry-points]] — `Sub Main()` in `Modules\Main.vb` and the command-line mode switch it dispatches on.
- [[runtime-modes]] — interactive (default), `cadbatchserver`, `updateicenter`, plus the conditional `BatchServer` and `ComWatcher` post-main forms.
- [[global-state]] — the god-module `Modules\Main.vb`: ~80 public globals (singletons, caches, font handles, hard-coded constants) shared across the whole app.
- [[build-and-deploy]] — `.NET Framework 4.8`, `x86`, WinExe, ClickOnce publish to `Z:\iCenter\BetaRelease\`, assembly signing currently disabled (expired cert, 2023-07-18).
- [[project-references]] — companion projects (`ICenterLib`, `TruTopsLib`) compiled-in, plus DLLs sourced from `\\jazo.local\dfs\Applications\Development\Dot Net DLLs\`.
- [[external-surface]] — every external system reached at startup or referenced from `app.config`. Detailed notes live in [[../external-systems/_index|external-systems/]].

## Open questions

- [[project-references|`ICenterLib` and `TruTopsLib`]] are now in scope (Q-001 resolved 2026-06-18). New **Q-018** open about ICenterLib's mismatched build-reference path.
- Should the disabled ClickOnce signing pipeline (lines 3383–3391 of `iCenter.vbproj`) be documented? Likely a release-engineering / SME concern.

## Related

- [[../_coverage]] — Phase 1 file inventory.
- [[../../CLAUDE]] — standing instructions.
