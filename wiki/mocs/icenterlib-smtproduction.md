---
type: moc
title: "ICenterLib/SmtProduction — Trumpf Oseon / PPSInterface"
status: draft
module: "ICenterLib/SmtProduction"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\SmtProduction\\"
tags: [moc, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# ICenterLib/SmtProduction

## What this hub covers

`ICenterLib\SmtProduction\` — **the Trumpf TruTops Oseon integration** for sheet-metal production. 126 source files (~282 KB). This is the **largest single integration in the codebase**: hundreds of DataService + Entity + Handler + Exporter files spanning two sub-namespaces — `Oseon` (the modern Trumpf MES) and `PPSInterface` (the legacy file-based exchange with the older PPS system).

> **JAZO refers to Oseon as "Boost" internally** (see [[icenter-boost-delete-blocked]] for the user-facing label).

## Sub-folder breakdown

```
SmtProduction\
├── DataServices\        ' top-level production-order, cut-sheet, oper-registration DataServices
├── Entities\            ' Top-level production entities + OseonAppContext
├── Enums\
├── Handlers\
├── TruTops\
│   ├── Oseon\           ' Modern Oseon API integration (DataServices + Entities)
│   ├── PPSInterface\    ' Legacy PPS file-based interface
│   │   ├── DataServices\
│   │   ├── Entities\
│   │   ├── Export\      ' FeedbackObjects (ProcessedSheetReport, PDAMessage)
│   │   └── Mappings\
│   └── Utils\           ' TruTopsConvertHandler etc.
└── ViewModels\
```

## Top files by size

| File | KB | Role |
|------|---:|------|
| `TruTops\PPSInterface\DataServices\PPSInterfaceDataService.vb` | 25 | the legacy-PPS DataService |
| `TruTops\Oseon\DataServices\CutSheetDataService.vb` | 14 | Oseon cut-sheet queries |
| `TruTops\PPSInterface\Export\FeedbackObjects\FeedbackObjectProcessedSheetReport.vb` | 11 | feedback ProcessedSheetReport entity |
| `TruTops\Oseon\DataServices\PartOnTableDataService.vb` | 10 | parts-on-table tracking |
| `DataServices\ProductionOrderDataService.vb` | 9 | top-level production-order CRUD (consumed by `IPBatch.DeleteSmtProdOrd`) |
| `TruTops\PPSInterface\Export\FeedbackObjects\FeedbackObjectPDAMessage.vb` | 9 | PDA message feedback |
| `TruTops\Oseon\DataServices\PartStatusDataService.vb` | 7 | part-status updates |
| `TruTops\Oseon\DataServices\CadCamDocumentDataService.vb` | 6 | CAD/CAM doc tracking |
| `TruTops\Oseon\DataServices\WorkplaceDataService.vb` | 6 | workplace info |
| `TruTops\Oseon\DataServices\PartBendSolutionDataService.vb` | 6 | bend-solution info |
| `TruTops\Oseon\DataServices\TTNGActionDataService.vb` | 5 | TruTops Next-Generation action calls |
| `TruTops\Oseon\Entities\PartStatusMaster.vb` | 5 | part-status master DTO |
| `DataServices\CutSheetOperRegistrationDataService.vb` | 5 | cut-sheet-operation registration |
| `TruTops\PPSInterface\Export\ProductionOrderExportHandler.vb` | 5 | PPS export handler |
| `TruTops\Utils\TruTopsConvertHandler.vb` | 5 | TruTops conversion utilities |

## Notable findings (at a glance)

1. **`OseonAppContext`** is the shared connection-context object passed to all DataServices. Consumed by [[icenterlib-icenter-batch-hierarchy|`IPBatch.DeleteSmtProdOrd`]] etc. Phase-3 follow-up.
2. **PPSInterface vs Oseon**: two parallel interfaces. PPS = the older file-exchange interface; Oseon = the modern REST/DB API. Migration in progress?
3. **`FeedbackObject*` family** — XML / file-based feedback messages JAZO sends back to PPS. Includes `ProcessedSheetReport` (sheet-by-sheet processing reports) and `PDAMessage` (shop-floor PDA-handheld events).
4. **`TTNGActionDataService`** — TTNG = TruTops Next Generation. Confirms the Oseon DataService talks to TTNG REST endpoints.
5. **`PartOnTableDataService` (10 KB)** — substantial logic for tracking which parts are physically on the laser table. `#safety-relevant`.
6. **`PartBendSolutionDataService`** — bend-solution data (sheet-bending CNC programs). `#safety-relevant`.
7. **`ProductionOrderDataService.DeleteICenterBatch`** is the destination of [[icenter-boost-delete-blocked|Boost delete]]. Owns the deletion behaviour from the Oseon side.

## Open questions

- **Q-299 (new):** Document PPSInterface vs Oseon: are both still active, or is PPSInterface deprecated? Migration plan?
- **Q-300 (new):** `OseonAppContext` — how is it constructed? Per-session, per-call?
- **Q-301 (new):** Document the FeedbackObject XML schemas — `ProcessedSheetReport`, `PDAMessage` are SME-relevant.
- **Q-302 (new):** `TTNGActionDataService` — what TTNG endpoints does it call? Versioning?
- **Q-303 (new):** `PartOnTableDataService` + `PartBendSolutionDataService` — confirm SME-level behaviour. `#safety-relevant`

Logged in [[../needs-review/_index]].

## Coverage decision

**All 126 source files marked `done` overview-level via this MOC.** Top 5 by size flagged for **Phase 4 deep-read** (business-logic sweep): PPSInterfaceDataService, CutSheetDataService, PartOnTableDataService, ProductionOrderDataService, ProcessedSheetReport.

Generated `*.Designer.vb` marked `generated`.
