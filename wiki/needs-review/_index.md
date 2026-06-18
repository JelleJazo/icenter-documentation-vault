---
type: moc
title: "Needs Review — Index"
status: active
tags: [moc, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Needs Review — SME Queue

Every open question for the SME lives here. Anything with `#safety-relevant` is highest priority.

## How it works

When documenting any code path where intent is unclear:
1. In the page, set `status: needs-review` and add `#needs-review` (and `#safety-relevant` if applicable).
2. Write the *specific* open question in a `## Open question` section on that page.
3. Add a one-line entry below pointing to the page. Use the running ID (`Q-NNN`).

## Open questions

_(Append as you go. Newest at the top.)_

| ID | Date | Page | Question | Severity / tag |
|----|------|------|----------|----------------|
| **Q-018** | 2026-06-18 | [[../architecture/project-references]] | The `ICenterLib` project-reference path in `iCenter.vbproj` / `iCENTER.sln` resolves to a non-existent directory; the actual source is under the user's personal `source\repos\` tree. Where is the canonical / build-server copy? | release-engineering |
| **Q-017** | 2026-06-18 | [[../architecture/external-surface]] | For each `#safety-relevant` external system, what is the failure mode if it goes down? | safety-relevant |
| **Q-016** | 2026-06-18 | [[../architecture/external-surface]] | Is the embedded PDF-XChange license valid for redistribution? Confirm with vendor. | compliance |
| **Q-015** | 2026-06-18 | [[../architecture/external-surface]] | Is the `pvs/pvs` XWiki account read-only or write-capable? | security |
| **Q-014** | 2026-06-18 | [[../architecture/project-references]] | `System.Xml.dll` referenced from v4.6.2 reference-assemblies while target is `v4.8`. Drift or intentional? | low |
| **Q-013** | 2026-06-18 | [[../architecture/project-references]] | DFS-hosted house DLLs are loaded from a live share. Is the per-version folder treated as immutable, or do operators update in place? | release-engineering |
| **Q-012** | 2026-06-18 | [[../architecture/build-and-deploy]] | Confirm `Functions.UpdateIcenter()` is the live update mechanism; document its source/destination paths and signing model. | release-engineering |
| **Q-011** | 2026-06-18 | [[../architecture/build-and-deploy]] | Is `Z:\iCenter\BetaRelease\` the production publish target or only Beta? Where does production release go? | release-engineering |
| **Q-010** | 2026-06-18 | [[../architecture/build-and-deploy]] | Has Authenticode signing been re-enabled since 2023-07-18 in any branch / pipeline outside `iCenter.vbproj`? | compliance |
| **Q-009** | 2026-06-18 | [[../architecture/global-state]] | Confirm hard-coded constant lists (kick-off ops, lean-mode machine groups, dept codes) still match current factory configuration. | safety-relevant |
| **Q-008** | 2026-06-18 | [[../architecture/global-state]] | Confirm `iCenter2NewestDataModel` DB is still hit at runtime; if so, document its schema scope. | data-lifecycle |
| **Q-007** | 2026-06-18 | [[../architecture/global-state]] | Replace `sPDFOwnerPassword` literal with a secret store? Confirm it controls real PDF protection. | security / safety-relevant |
| **Q-006** | 2026-06-18 | [[../architecture/runtime-modes]] | Is `Elumatec.FrmComWatcher` the COM-port (serial) hand-shake to SBZ140 or a generic COM watcher? | safety-relevant |
| **Q-005** | 2026-06-18 | [[../architecture/runtime-modes]] | Where in `FrmMain` does `ComWatcherMode = True` / `BatchServerMode = True` get set? | medium |
| **Q-004** | 2026-06-18 | [[../architecture/entry-points]] | Is `Functions.UpdateIcenter()` (the `-m updateicenter` path) still in use after the 2023-07 cert expiry? | medium |
| **Q-003** | 2026-06-18 | [[../architecture/entry-points]] | What conditions inside `FrmMain` flip `BatchServerMode` / `ComWatcherMode`? | medium |
| **Q-002** | 2026-06-18 | [[../architecture/entry-points]] | Who launches `-m cadbatchserver`? Scheduled task? Service wrapper? User `JZPUBLISH`? | medium |
| **Q-001** | 2026-06-18 | [[../architecture/_index]] | _(resolved 2026-06-18)_ — Both `ICenterLib` and `TruTopsLib` widened **into scope** per user direction. ICenterLib source located at `C:\Users\jelle-r\source\repos\JIBA\iCenter And Tools\ICenterLib\ICenterLib` — see new **Q-018** about the build-path discrepancy. | scope (resolved) |
| **Q-000** | 2026-06-18 | [[../overview]] | _(superseded)_ — Was the older `C:\DevOps\iCenter` codebase in scope, or only `iCenter2`? Resolved: per updated CLAUDE.md, scope is `C:\DevOps\iCenter\iCenter\iCENTER` only; `iCenter2` is out. | scope (resolved) |

## Safety-relevant queue

| ID | Page | One-line |
|----|------|----------|
| Q-006 | [[../architecture/runtime-modes]] | `Elumatec.FrmComWatcher` — possible serial driver |
| Q-007 | [[../architecture/global-state]] | Hard-coded PDF owner password |
| Q-009 | [[../architecture/global-state]] | Hard-coded workflow constant lists |
| Q-017 | [[../architecture/external-surface]] | Per-system failure-mode docs missing |
