---
type: external-system
title: "\\\\jazo.local\\dfs\\ — DFS share"
status: stub
tags: [external-system, needs-content, needs-review, safety-relevant]
created: 2026-06-18
updated: 2026-06-18
---

# `\\jazo.local\dfs\` — DFS share

Windows DFS namespace serving as the canonical file-system root for almost every absolute path iCenter touches. Not a "system" per se, but a **single point of failure** the application implicitly trusts.

> **Stub** — populate DFS namespace map, replication topology, backup policy.

## Quick links

- [[../modules/icenter-kardex|Kardex XSLT path]] — `\\JAZO.LOCAL\DFS\JIBA.NET\XSLT\PartDispatch\Isah2KardexOrder.xslt` (Q-323 `#safety-relevant`).
- Many `AppSettings` UNC paths root here.
- [[../mocs/icenterlib-smtproduction|TruTops file paths]] — likewise.

## Why `#safety-relevant`

Renaming the DFS namespace, breaking replication, or losing the share silently disables: Kardex order flow, TruTops template DXFs, GhostScript EXE invocation, JIBA portal cross-references, and more. Q-323 is the highest-priority DFS-related question.
