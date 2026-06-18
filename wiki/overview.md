---
type: architecture
title: "iCenter — Executive Overview"
status: draft
module: ""
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER"
  - "C:\\DevOps\\iCenter\\iCenter\\TruTopsLib"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib"
last-reviewed: ""
tags: [overview, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# iCenter — Executive Overview

> **Status:** draft. Filled during Phase 2 (architecture pass). Refined as Phase 3 module notes land.

## What iCenter is

A long-running VB.NET Windows-Forms application that sits at the centre of JAZO Zevenaar's manufacturing IT stack. It connects the **ISAH ERP** (orders, BOMs, time registration) to **CAD/PLM** (Creo, Windchill, Solid Edge), to **sheet-metal MES** (Trumpf TruTops Oseon), and to physical CNC profile-mill machines (Elumatec SBZ140 and Door+Gate), plus a Kardex vertical-lift storage system, label printers, and an internal portal (JIBA). Roles range from engineers and work preparators (*werkvoorbereiders*) to shop-floor operators using lean-mode kiosk views.

It is one executable (`iCenter.exe`, .NET 4.8, x86, WinExe) that can run in [several modes](architecture/runtime-modes.md): the default interactive UI, a CAD batch worker (`-m cadbatchserver`), a self-updater (`-m updateicenter`), plus generic/COM-port post-main workers gated by globals.

> Scope widened on 2026-06-18 to include the two companion VB.NET projects (`ICenterLib` and `TruTopsLib`) per [[../CLAUDE|standing instructions]]. ICenterLib (723 files, 35 top-level folders) hosts most of the data-access and PLM/CAD plumbing; TruTopsLib (65 files, mixed `.vb` + `.cs`) parses Trumpf TruTops file formats. Total inventory: 2025 files. See [[needs-review/_index]] Q-001 (resolved) and Q-018 (new — build-path discrepancy on ICenterLib).

## What it controls

- **Elumatec SBZ140** profile mill (aluminum + steel) — NC programs, offsets, license management, possibly serial-COM monitoring. `#safety-relevant` See [[external-systems/elumatec-sbz140]].
- **Elumatec Door+Gate (DG)** profile mill variant. `#safety-relevant`
- **Trumpf sheet-metal machines via TruTops Oseon** — part/document/status data services drive the MES; iCenter writes parts and reads status. `#safety-relevant` See [[external-systems/trutops-oseon]].
- **PTC Creo** — model generation, publish, view embed. `#safety-relevant` if iCenter changes CAD files.
- **Kardex Shuttle** — XML drops at `\\JAZO.LOCAL\DFS\PM\Kardex Shuttle\XmlXchange\In\Put\` + a web UI; storage retrievals. `#safety-relevant` (drives lift movements via Kardex's own interface).
- **Dymo label printers** + thermal stickers — kanban bin, IPpart, ord-ref labels.
- **PDF print pipeline** via GhostScript and PDF-XChange Viewer (ActiveX + EXE).
- **Email** via MS Outlook COM and the sibling `jMailLauncher` EXE.

## Top-level architecture

See [[architecture/_index]] for the views. Quick map:

```
                     +----------------+
                     |  iCenter.exe   |
                     |  (.NET 4.8 x86)|
                     +-------+--------+
                             |
       +---------------------+---------------------+
       |                                           |
  Modules\Main.vb (Sub Main)                  FrmMain (15 200 lines)
       |                                           |
  globals + singletons:                       all interactive UI:
  oICENTER, oISAH, oJIBA,                     treeview, search, print,
  Oseon services, prodMachines,               coating, work prep,
  Sola, Elfsquad ...                          production, sales, ...
       |                                           |
       +------> External systems  <----------------+
                (see architecture/external-surface)
```

Single process, no service-bus. All integration is direct: DB→DB, file drop→file drop, HTTP→HTTP, COM→COM. Failures propagate to the user via `MsgBox` + `oApplicationLog`.

## Key modules (Phase 3 will populate these)

- [[modules/main-module|Modules\Main.vb]] — entry, globals, mode dispatch.
- [[modules/elumatec|Elumatec/]] — profile-mill subsystem. `#safety-relevant`
- [[modules/smt-manufacturing|SmtManufacturing/]] — sheet-metal subsystem. `#safety-relevant`
- [[modules/cad-batchserver|CadBatchserver/]] — headless CAD job runner.
- [[modules/work-preparation|WorkPreparation/]] — werkvoorbereiding.
- [[modules/production|Production/]] — shop-floor.

Full list: [[modules/_index]] / [[_coverage]].

## Open questions

See [[needs-review/_index]] for the full queue. Phase 2 surfaced **17 open questions** (Q-001 through Q-017), including:

- Scope of companion projects (Q-001).
- Hard-coded secrets in source (Q-007, Q-016).
- Disabled signing pipeline since 2023-07-18 (Q-010).
- Whether the various `#safety-relevant` integrations have failure-mode docs (Q-017).
- The exact behavior of `Elumatec.FrmComWatcher` — likely a serial-COM driver (Q-006). `#safety-relevant`
