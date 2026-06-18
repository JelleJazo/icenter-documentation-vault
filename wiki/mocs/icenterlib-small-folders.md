---
type: moc
title: "ICenterLib — small folders sweep"
status: draft
module: "ICenterLib/*"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\"
tags: [moc, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# ICenterLib — small folders sweep

> Single MOC covering the 15+ ICenterLib sub-folders with ≤10 source files each. Each gets a short paragraph + file inventory. Anything substantial gets promoted to a dedicated MOC later.

## `DataServices/` (10 files, 21 KB)

Modern `DataService` classes hosting cross-cutting queries:
- `BomFilterDataService`, `DocumentDataService`, `EnvironmentDataService`, `JobDataService`, `LabelDataService`, `MarkingPlaceDataService`, `ProductionLineDataService`, `ProductionRegistrationDataService`, `TimeRegistrationDataService`, `UserDataService`.

Pattern: each wraps a SP or inline SQL via `DataHandler.GenericQuery`. Pairs with the namespace-mate `Handlers/` and `Entities/` folders in callers (e.g., SmtProduction).

## `Elfsquad/` (5 files + 2 generated, 37 KB)

JAZO's bridge to **Elfsquad** (external visual configurator SaaS). Entities + DataService for Elfsquad config IDs, deployment management, and configuration linking. Pairs with [[isah-sub-services|`IsahCustomisingElfsquadDataService`]] (HTTP client). `#external-system`.

## `JMail/` (6 files, 16 KB)

JAZO-internal mail helper:
- `JMail.vb` (the SMTP client wrapper)
- `MailMessage.vb`, `MailAttachment.vb`, `MailRecipient.vb` (entities)
- `MailTemplate.vb` (template wrapper)
- `MailReply.vb` (reply parsing)

## `UserControls/` (15 source + 11 generated, 60 KB)

Shared WinForms UserControls:
- `FrmWebView.vb` — Chromium/WebView2-host (consumed by Servicedesk)
- `FrmUserSelection.vb` — generic user-picker dialog
- `FrmDataGridFilter.vb` — grid-filter UI
- Various smaller controls (toolbar buttons, status indicators, etc.)

## `CAD/` (116 files, 577 KB) — **deferred**

The largest folder. Creo/Windchill/CAD plumbing. **Deferred to Phase 3f** — needs its own MOC. **Files left as `todo`**.

## `SmtProduction/` (126 files, 282 KB) — **deferred**

The Trumpf Oseon integration. **Deferred to Phase 3f** — needs its own MOC. **Files left as `todo`**.

## `PCFNet/` (39 source + 3 generated, 464 KB) — **deferred**

Sheet-metal calculator + control-definition library. **Deferred to Phase 3f**. **Files left as `todo`**.

## Tiny folders

| Folder | Files | Role |
|--------|------:|------|
| `Comparer/` | 1 | generic IComparer |
| `Connections/` | 1 | already done — [[../modules/icenterlib-connections]] |
| `CrystalReport/` | 5 | Crystal Reports wrappers |
| `Debug/` | 1 | debug helper |
| `DistributedLock/` | 1 | distributed-lock primitive (used by CadBatchServer) |
| `Enums/` | 2 | shared enums — `Enums.Application.*` |
| `GUI/` | 3 | GUI helpers |
| `Helpers/` | 1 | misc helper |
| `LaserWork/` | 2 + 1 gen | LaserWork app wrapper |
| `Metabase/` | 4 | Metabase reporting |
| `ModelDefinition/` | 3 | CAD model-definition entities |
| `Prodex/` | 6 | Prodex CAD-data exchange |
| `SmartForms/` | 1 | SmartForms integration |
| `SmtCadCam/` | 4 | SMT CAD/CAM helpers |
| `STEP3D/` | 5 | STEP3D viewer integration |
| `Ticketing/` | 5 | ticketing helpers |
| `TimeRegistration/` | 2 | iCenter-side time-reg lookup helpers |
| `Zabbix/` | 1 | Zabbix monitoring client |
| `My Project/` | 1 source + 3 gen | VB project file (config) |
| `obj/` | 2 | build output (config) |
| `Web References/` | 1 | SOAP web reference (config) |

## Coverage

| Folder | Status |
|--------|--------|
| DataServices, Elfsquad, JMail, UserControls, Comparer, CrystalReport, Debug, DistributedLock, Enums, GUI, Helpers, LaserWork, Metabase, ModelDefinition, Prodex, SmartForms, SmtCadCam, STEP3D, Ticketing, TimeRegistration, Zabbix | `done` overview-level via this MOC |
| `CAD/`, `SmtProduction/`, `PCFNet/` | `todo` — deferred to Phase 3f |
| Generated files | `generated` |
| `My Project/`, `obj/`, `Web References/` | `config` |

## Open questions

- **Q-292 (new):** SMT/CAM divide between `SmtCadCam/` (4 files) and `SmtProduction/` (126 files) — what scope does each cover?
- **Q-293 (new):** `Ticketing/` folder — separate from `Servicedesk` (Zammad). Documented as Phase-4 follow-up.
- **Q-294 (new):** `Metabase/` — Metabase BI tool integration. Document the metabase URL + datasets used.

Logged in [[../needs-review/_index]].
