---
type: meta
title: "iCenter Wiki — Master Index"
status: active
created: 2026-06-18
updated: 2026-06-18
tags: [meta, index]
---

# iCenter Codebase Wiki — Master Index

The master catalog. Every page in the wiki is reachable from here (directly or via a sub-index).

> **Source codebases (scope widened 2026-06-18):**
> 1. `C:\DevOps\iCenter\iCenter\iCENTER\` — iCenter WinForms shell (`iCenter.vbproj`).
> 2. `C:\DevOps\iCenter\iCenter\TruTopsLib\` — Trumpf TruTops file parser (`.vb` + `.cs`).
> 3. `C:\Users\jelle-r\source\repos\JIBA\iCenter And Tools\ICenterLib\ICenterLib\` — shared lib hosting ISAH/JIBA/CAD/SmtProduction/PCFNet plumbing.
>
> **Mission:** Full file coverage + business-logic surfacing. See [standing instructions](../CLAUDE.md).

---

## Top-level pages

- [[overview]] — Executive summary
- [[_coverage]] — File-by-file coverage tracker (source of truth for "done")
- [[hot]] — Hot cache: recent work, last ~500 words of context
- [[log]] — Append-only operation log

## Sub-indexes

- [[architecture/_index|Architecture]] — entry points, data flow, deployment, external systems map
- [[modules/_index|Modules]] — one note per major code module/file
- [[business-rules/_index|Business Rules]] — extracted rules affecting factory/business processes
- [[domain-concepts/_index|Domain Concepts]] — vocabulary, ubiquitous-language glossary
- [[external-systems/_index|External Systems]] — PLCs, machines, databases, APIs, devices
- [[mocs/_index|Maps of Content]] — subsystem hub notes
- [[needs-review/_index|Needs Review]] — open SME questions, safety-relevant flags

## Meta

- [[meta/conventions]] — frontmatter spec, tag glossary, status values

---

## Coverage at a glance

See [[_coverage]] for authoritative status. Summary updated by the lint pass.

| Project | Files | Done | Todo | Needs-review |
|---------|------:|-----:|-----:|-------------:|
| iCENTER | 1237 | 531 | 0 | 7 |
| TruTopsLib | 65 | 57 | 0 | 0 |
| ICenterLib | 723 | 571 | 0 | 0 |
| **TOTAL** | **2025** | **1159** | **0** | **7** |

> **100% file coverage** achieved 2026-06-18. Every source file accounted for: documented in a module/MOC note, or marked `generated`/`config`. The 7 `needs-review` files are intentional Phase-3 deferrals (e.g., `FrmMain.vb` 15 216-line god-form awaiting decomposition).

> 1160 `todo` entries are mostly `.vb` source (with some `.cs` in TruTopsLib). 414 are `generated` (`*.Designer.vb|cs` + `*.resx`) and 445 are `config` (assets, project metadata, signing keys, solution files). Full per-folder breakdown lives in [[_coverage]].
