---
type: external-system
title: "Kardex Shuttle"
status: stub
tags: [external-system, needs-content, safety-relevant]
created: 2026-06-18
updated: 2026-06-18
---

# Kardex Shuttle

**Vertical-lift storage system**. Multiple physical shuttles (KD06A small-parts, KD10A large-parts, plus KD05A coating-side per [[../modules/icenter-coating|Coating]]) operated by Kardex's own PowerPick controller. iCenter generates per-warehouse XML order files that PowerPick consumes via watched folder.

> **Stub** — populate model, controller version, PowerPick watch-folder paths, network connectivity.

## Quick links

- [[../modules/icenter-kardex|`KardexProcessor` + `FrmKardexInterface`]] — defining iCenter module.
- [[../business-rules/icenter-kardex-warehouse-codes|KD06A / KD10A warehouse codes]].
- `Isah2KardexExportPath` AppSettings — destination folder for XML orders.
- `\\JAZO.LOCAL\DFS\JIBA.NET\XSLT\PartDispatch\Isah2KardexOrder.xslt` (Q-323 `#safety-relevant`) — XSLT transform stylesheet.

> **Note**: A zero-byte stub also exists at `wiki/kardex.md` (Obsidian auto-created at vault root). Lint H-1 — delete the root stub once this page satisfies all `[[kardex]]` links.
