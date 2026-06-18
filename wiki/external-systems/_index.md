---
type: moc
title: "External Systems — Index"
status: draft
tags: [moc, external-system]
created: 2026-06-18
updated: 2026-06-18
---

# External Systems

Every system iCenter integrates with: ERP DB, MES, CAD/PLM, web apps, machines, printers, mail. Sources confirmed from `app.config`, `Modules\Main.vb`, project references, and the `Imports` block of `iCenter.vbproj`.

> **Every external system is a trust boundary.** Note authentication, protocol, retry behavior, and failure mode. Anything that can stop a machine if iCenter misbehaves gets `#safety-relevant`.

See [[../architecture/external-surface]] for the boundary diagram and Phase-2 mapping.

## Pages (stubs to populate during Phase 3)

### Business systems
- [[isah|ISAH]] — ERP DB + API. `oISAH`, `ClsISAH` in `ICenterLib`.
- [[icenter-db|iCenter DB]] — own SQL DB. `oICENTER`, `ClsICenter`.
- [[icenter2-db|iCenter2 DB]] — sibling-app DB, `iCenter2NewestDataModel`. Migration only.
- [[jiba-portal|JIBA portal]] — `jiba.jazo.com` web app.
- [[webclock|WebClock]] — time-registration web app.
- [[xwiki|XWiki — wiki.jazo.nl]] — internal wiki.

### CAD / PLM / MES
- [[creo|PTC Creo]] — CAD authoring. `ICenterLib.CAD.Creo`. `#safety-relevant`
- [[windchill|PTC Windchill]] — PLM. `ICenterLib.CAD.PLM`.
- [[solid-edge|Siemens Solid Edge]] — CAD authoring; importer EXE.
- [[trutops-oseon|TruTops Oseon / Trumpf]] — sheet-metal MES. `SmtPart*DataService`. `#safety-relevant`
- [[elfsquad|Elfsquad]] — product configurator. `IsahCustomisingElfsquadDataService`.
- [[jconfigurator|jConfigurator (ASMX SOAP)]] — `portal.jazo.nl/jconfigurator`.
- [[boost-pps|Boost PPS]] — production planning. Import timeout `BoostPPSImportTimeout=1800`.
- [[sola-data-connector|Sola data connector]] — sibling project, `AppHandler` global.

### Physical machines / hardware
- [[elumatec-sbz140|Elumatec SBZ140]] — CNC profile mill. `Elumatec\` folder. `#safety-relevant`
- [[elumatec-dg|Elumatec DG (Door+Gate)]] — CNC profile mill variant. `#safety-relevant`
- [[kardex|Kardex Shuttle]] — vertical-lift storage. `Isah2KardexExportPath`. `#safety-relevant` (storage drive)
- [[dymo-label|Dymo LabelWriter]] — label printer. `MyLabelWriter`, `MyDymoPrinter`.

### Utility / desktop integrations
- [[outlook|MS Outlook (COM)]] — `ProgIdOutlook="Outlook.Application"`.
- [[jmail-launcher|jMailLauncher]] — sibling project, mail helper EXE.
- [[pdf-xchange|PDF-XChange Viewer]] — ActiveX + EXE. License key in `app.config`. `#needs-review`
- [[ghostscript|GhostScript]] — PDF print. EXE under DFS share.

### Identity / infrastructure
- [[active-directory|Active Directory]] — `System.DirectoryServices.AccountManagement`.
- [[dfs-share|`\\jazo.local\dfs\` (DFS share)]] — file-system trust boundary. Not a "system" per se, but every absolute path roots here. `#needs-review`
