---
type: moc
title: "External Systems — Index"
status: stub
tags: [moc, external-system]
created: 2026-06-18
updated: 2026-06-18
---

# External Systems

Every system iCenter integrates with: machines, PLCs, file drops, databases, web services, mail, identity providers.

> **Every external system is a trust boundary.** Note authentication, protocol, retry behavior, and failure mode. Anything that can stop a machine if iCenter misbehaves gets `#safety-relevant`.

## Pages

_(Populated during Phase 2/3. Use the [[../_templates/external-system|external-system template]].)_

### Suggested initial entries (verify by reading code)
- Trumpf machine integration — see `JAZO.ICenter.Sheetmetal.Trumpf`
- TruTops nesting — see `JAZO.SheetMetal.TruTops`
- SQL database — see `JAZO.iCenter.Persistence`, `DatabaseMigration`
- Identity / auth — see `JAZO.iCenter.Identity`
- Email — see `jMail`, `jMailLauncher`
- Installer / runtime services — see `JAZO.iCenter.Installer`, `AppLauncher`
