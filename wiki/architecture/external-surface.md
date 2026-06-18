---
type: architecture
title: "External surface — systems iCenter integrates with"
status: draft
module: "iCENTER/(root)"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\app.config"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Modules\\Main.vb"
last-reviewed: ""
tags: [architecture, external-system, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# External surface

## What this view shows

A boundary diagram (in table form) of everything iCenter talks to. Every row maps to a [[../external-systems/_index|external-systems/]] note. Sources are `app.config`, `Modules\Main.vb` field initializers, and the [[project-references|project & DLL references]].

## Systems

| System | Kind | Direction | Configured at | Talked to via |
|--------|------|-----------|---------------|---------------|
| [[../external-systems/isah|ISAH]] | ERP DB + API | bidirectional | `IsahDocRoot`, `IsahReportBaseFolder` (app.config) | `oISAH` (`ClsISAH`, in `ICenterLib`) |
| [[../external-systems/jiba-portal|JIBA portal]] | Web app | read | `https://jiba.jazo.com/...` (multiple `app.config` keys) | `oJIBA` (`ClsJIBA`, in `ICenterLib`) + WebView2 |
| [[../external-systems/kardex|Kardex Shuttle]] | Vertical-lift storage | write (file drop) + read (web UI) | `Isah2KardexExportPath`, `KardexInterfaceUrl` | XML drop + WebView2 |
| [[../external-systems/elumatec-sbz140|Elumatec SBZ140]] | CNC profile mill | write (NC programs), read (DB) | `profileDB`, `EluDxfDir`, `EluMillNrDbPath`, `IA_nr_progs` | `Elumatec\` folder (NC export, COM watcher) `#safety-relevant` |
| [[../external-systems/elumatec-dg|Elumatec Door+Gate (DG)]] | CNC profile mill (variant) | write/read | `EluDxfDirDgx`, `EluEdpDirDgx` | `Elumatec\` folder `#safety-relevant` |
| [[../external-systems/trutops-oseon|TruTops Oseon / Trumpf]] | Sheet-metal MES | bidirectional | (services, no URL in app.config) | `SmtPartDataService`, `SmtCadCamDocumentDataService`, `SmtPartStatusDataService` via `ICenterLib.SmtProduction.TruTops.Oseon` `#safety-relevant` |
| [[../external-systems/creo|PTC Creo]] | CAD authoring | drive (model gen, publish) | (registry / Creo install) | `ICenterLib.CAD.Creo`, `CreoViewControl`, `CadBatchserver/JobModelGeneratorCreo`, `JobPublishCreo` `#safety-relevant` |
| [[../external-systems/windchill|PTC Windchill]] | PLM | read (parts, docs) | (Windchill server URL via `WCUser`) | `ICenterLib.CAD.PLM` + `WindchillUserAuthenticationWatcher` (sibling project) |
| [[../external-systems/solid-edge|Siemens Solid Edge]] | CAD authoring | import | `SolidEdgeImporterPath` → `\\jazo.local\dfs\applications\iCenter\SolidEdgeImporter\SolidEdgeImporter.application` | external EXE (ClickOnce launched) |
| [[../external-systems/boost-pps|Boost PPS]] | Production planning | import (XLSX/XML) | `BoostPPSImportTimeout=1800` | `JobAutoManufacturing`? `#needs-review` |
| [[../external-systems/elfsquad|Elfsquad]] | Product configurator | read | (via ISAH bridge) | `IsahCustomisingElfsquadDataService` |
| [[../external-systems/jconfigurator|jConfigurator (ASMX)]] | SOAP web service | call | `https://portal.jazo.nl/jconfigurator/jconfigurator.asmx` | generated SOAP proxy in `Web References\` (excluded from inventory) |
| [[../external-systems/xwiki|XWiki — wiki.jazo.nl]] | Internal wiki | read/link | `http://wiki.jazo.nl:8080/xwiki` | hyperlinks + WebView2 |
| [[../external-systems/webclock|WebClock]] | Time-registration web app | hosted | `https://webclock.jazo.com/contentpages/kardex/icenterinterface.aspx` | WebView2 |
| [[../external-systems/jmail-launcher|jMailLauncher]] | Email helper EXE | spawn | `EmailEBTVTemplate`, `EmailPurOrdEmailTemplate` | external process (sibling project) |
| [[../external-systems/outlook|MS Outlook (COM)]] | Local email client | drive | `ProgIdOutlook="Outlook.Application"` | late-bound COM Automation |
| [[../external-systems/pdf-xchange|PDF-XChange]] | PDF viewer/editor | embed + spawn | `PDFXLicensekey`, `PDFXLicensecode` (in app.config) | ActiveX (`AxInterop.PDFXCviewAxLib`) + exec `PDFXCview.exe` |
| [[../external-systems/ghostscript|GhostScript]] | PDF→print | spawn | `\\jazo.local\dfs\applications\iCenter\Resources\GhostScript\gswin32c.exe`, `GSPrint\gsprint64.exe` | external EXE |
| [[../external-systems/sola-data-connector|Sola data connector]] | (internal) | bidirectional | (none — sibling project) | `SolaDataConnector.AppHandler` |
| [[../external-systems/icenter-db|iCenter DB]] | own SQL DB | bidirectional | (connection string in `ICenterLib`) | `ClsICenter` (in `ICenterLib`) |
| [[../external-systems/icenter2-db|iCenter2 DB]] | SQL DB of sibling app | read | label `"iCenter2NewestDataModel"` | `DataMigration.DataBaseConnection` |
| [[../external-systems/active-directory|Active Directory]] | identity | read | (machine context) | `System.DirectoryServices.AccountManagement` |
| [[../external-systems/dymo-label|Dymo LabelWriter]] | Label printer | drive | `MyDymoPrinter` | `Classes\StickersAndLabels\DymoLabelTest`, `MyLabelWriter` global |

> **Stub notes have not yet been created** for each row — Phase 2 here is the *map*; Phase 3 fills the individual `external-systems/*.md` pages.

## How data flows on startup

1. Process launches with optional `-m <mode>` flag.
2. `Module Main` field initializers eagerly construct `oICENTER`, `oISAH`, `oJIBA`, `oProductDb`, `prodMachines`, `SolaDataConnectorAppHandler`, etc.
3. `Sub Main` → `Init()` hits ISAH (`RefreshIsahSortOrder`), iCenter DB (`GetGenericNames`, `GetAllGenericMachGrpLines`), Oseon (`GetDefaultOseonAppContext`), and Elfsquad (`IsahCustomisingElfsquadDataService`).
4. `FrmMain` (interactive) constructor opens an `ICenterLib.ICenter.Client` and calls `SmartUpdateBySession()` — close the form if it returns false. The "smart update" likely reaches a server. `#needs-review`.

## Key surprises

- **PDF-XChange license key in plain text** in `app.config` (lines 100–105). `#needs-review` (vendor-license disclosure / source-control hygiene).
- **XWiki creds `pvs / pvs` hard-coded** (`app.config` lines 208–213) — a separate machine identity for wiki access. `#needs-review`.
- **`loadFromRemoteSources enabled="true"`** — required to load DLLs from the DFS UNC share; lowers CAS protection (`app.config` line 42).
- **No service-bus / queue.** All integration is direct: DB → DB, file drop → file drop, HTTP call → HTTP call. Failures propagate immediately to the user.

## Open questions

- **Q-015:** Is the `pvs/pvs` XWiki account read-only or also write-capable? `#needs-review`
- **Q-016:** Is the embedded PDF-XChange license valid for redistribution? Confirm with vendor.
- **Q-017:** For each `#safety-relevant` row, document the failure mode in its `external-systems/*.md` note.

## Related

- [[_index]]
- [[../external-systems/_index|external-systems/]] (will host per-system notes)
- [[global-state]] (singletons live in `Modules\Main.vb`)
