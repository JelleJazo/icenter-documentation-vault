---
type: external-system
title: "Siemens Solid Edge"
status: stub
tags: [external-system, needs-content]
created: 2026-06-18
updated: 2026-06-18
---

# Siemens Solid Edge

Alternative CAD authoring tool (alongside [[creo|PTC Creo]]). iCenter consumes Solid Edge output via the `Part.ImportSEDxf(filepath)` entry and the `ReplaceSolidEdgeBendLines` path in `FlatPatternConverter`.

> **Stub** — populate version, importer-EXE location, usage scope (subset of products?).

## Quick links

- [[../mocs/smtmanufacturing|SmtManufacturing MOC]] — `Part.ImportSEDxf` consumer.
- `TC_SE_BEND_DATA` Trumpf XData app-id (Q-394) — Solid-Edge-specific bend annotation.
- `Enums.Application.CadSource` — the dispatch enum (Creo vs Solid Edge vs …).
