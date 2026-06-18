---
type: external-system
title: "Elumatec SBZ140 / SBZ141"
status: stub
tags: [external-system, needs-content, safety-relevant]
created: 2026-06-18
updated: 2026-06-18
---

# Elumatec SBZ140 / SBZ141

CNC **profile-milling machines** for aluminium and steel extrusions. JAZO's primary profile-mill family. iCenter drives them via the `Elumatec\` folder (NC pipeline → AUF / EluXml → controller).

> **Stub** — populate firmware versions, drive type, post-processor specifics.

## Quick links

- [[../mocs/elumatec|Elumatec MOC]] — top-level CAD→NC pipeline overview.
- [[../mocs/elumatec-ncpipeline|NC pipeline]] — `Job`/`Bar`/`Cut`/`Plane` hierarchy + emitters.
- [[../mocs/elumatec-works|Works/Replacements pipeline]] — per-feature handlers + 19 replacement macros.
- [[../modules/elumatec-elucadfile|EluCadFile `.ecw` parser]].
- [[../business-rules/elu-dual-emit-sbz140-sbz141|Dual-emit SBZ140 + SBZ141 rule]].
- [[../business-rules/elu-max-step-depth|Max step depth]] / [[../business-rules/elu-tool-max-cut-depth|Max cut depth]].

> **Note**: A zero-byte stub also exists at `wiki/elumatec-sbz140.md` (Obsidian auto-created at vault root). Lint H-1 — delete the root stub once this page satisfies all `[[elumatec-sbz140]]` links.
