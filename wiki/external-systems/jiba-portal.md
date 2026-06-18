---
type: external-system
title: "JIBA portal"
status: stub
tags: [external-system, needs-content]
created: 2026-06-18
updated: 2026-06-18
---

# JIBA portal

JAZO-internal portal at `jiba.jazo.com` and `portal.jazo.nl`. Hosts the JIBA database (employees, assets, menus, CE-checklist XmlData, automation jobs).

> **Stub** — populate hosting, version, contact.

## Quick links

- [[../mocs/icenterlib-jiba|ICenterLib/JIBA MOC]] — 14-file folder wrapping the JIBA database.
- [[../modules/icenterlib-jiba-employee-asset|`JIBA.Employee` + `JIBA.Asset`]].
- [[../business-rules/jiba-local-email-domain|`@jazo.local` local email convention]].
- [[../business-rules/productdb-system-publisher-empid|`EmpId "0798"` service account]].
- `JibaUrl` AppParameter (in `T_AppParameter`) drives `OpenInChrome` flows.

## Tables

`T_Employee`, `T_Users`, `T_Asset`, `T_AppParameter`, `T_XmlData`, `T_AutomJob_v2`, plus menu/navigation tables.

> **Note**: A zero-byte stub also exists at `wiki/jiba-portal.md` (Obsidian auto-created at vault root). Lint H-1 — delete the root stub once this page satisfies all `[[jiba-portal]]` links.
