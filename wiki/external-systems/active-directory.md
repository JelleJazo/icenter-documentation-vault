---
type: external-system
title: "Active Directory (JAZO domain)"
status: stub
tags: [external-system, needs-content]
created: 2026-06-18
updated: 2026-06-18
---

# Active Directory

JAZO's Windows Active Directory. iCenter queries it via `System.DirectoryServices.AccountManagement`.

> **Stub** — populate domain controllers, OU layout, sync to JIBA `T_Users`.

## Quick links

- `JAZO\` prefix stripped from usernames in [[../modules/icenterlib-jiba-employee-asset|`JIBA.Employee.GetUserInfoByUsername`]].
- [[../business-rules/jiba-local-email-domain|`@jazo.local` local email]] — the AD-domain-derived mail suffix.
