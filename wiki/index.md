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

> **Source codebase:** `C:\DevOps\iCenter\iCenter\iCENTER\` — single VB.NET WinForms project (`iCenter.vbproj`) with ~30 sub-folders covering CAD/CAM, Sales, WorkPreparation, Production, Engineering, and external-system integrations.
> **Mission:** Full file coverage + business-logic surfacing. See [[../CLAUDE|standing instructions]].

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

| Sub-folder | Files | Done | Todo | Needs-review |
|------------|------:|-----:|-----:|-------------:|
| **iCENTER (all)** | **1237** | 5 | 532 | 1 |

> 532 todo entries are mostly `.vb` source. 341 are `generated` (`*.Designer.vb` + `*.resx`) and 358 are `config` (assets, project metadata, signing keys). 5 root/`Modules` files documented in Phase 2 (see [[architecture/_index]]); `FrmMain.vb` flagged `needs-review` pending its Phase-3 split. Full per-folder breakdown lives in [[_coverage]].
