---
type: external-system
title: "PTC Creo"
status: stub
tags: [external-system, needs-content, safety-relevant]
created: 2026-06-18
updated: 2026-06-18
---

# PTC Creo

PTC's parametric-CAD application. JAZO's **primary CAD authoring tool**. Driven from iCenter via Creo Trail files + Pro/Program parameters + an in-process API exposed under `ICenterLib.CAD.Creo`.

> **Stub** — populate version, license server, install paths.

## Quick links

- [[../mocs/icenterlib-cad|ICenterLib/CAD MOC]] — 116-file Creo+geometry library.
- [[../mocs/icenterlib-cadbatchserver|CadBatchServer MOC]] — headless Creo automation.
- `CAD.Creo.Environment.GetProManufDir` — resolves Creo's pro-manuf directory (used by TruTopsLib MigrationFix template).
- [[windchill|PTC Windchill]] — companion PLM.

## Q-relevant

- Q-384 — what happens when Creo isn't installed? (TruTops `MigrationFix.TEMPLATEDXF` depends on `GetProManufDir`.)
