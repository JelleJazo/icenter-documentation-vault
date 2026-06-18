---
type: moc
title: "ICenterLib/Production — production-side entities"
status: draft
module: "ICenterLib/Production"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\Production\\"
tags: [moc, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# ICenterLib/Production

## What this hub covers

`ICenterLib\Production\` — **production-side domain entities** that don't fit cleanly into iCenter, ISAH, or SmtProduction. Houses cross-cutting concepts: Kanban bins, KPIs, checklists, BOM filters, production-machine-selector form, profile-cut items + handlers, label/production logs, the SMT bend queue.

19 source files (~64 KB) + 2 generated.

## Files

| File | KB | Role |
|------|---:|------|
| `ProductionProfileCutItemsHandler.vb` | (big) | bulk handler for profile cut items |
| `ProductionProfileCutItemHandler.vb` | | single-item handler |
| `ProductionProfileCutItem.vb` | | individual cut-item entity |
| `BaseProductionItem.vb` | | base class for production items |
| `KanbanBin.vb` | | Kanban-bin entity |
| `KeyPerformanceIndicator.vb` | | KPI value-object |
| `CEChecklist.vb` | | CE-marking checklist |
| `ProdChecklist.vb` + `ProdChecklistTag.vb` | | production checklist entities |
| `FGQualityControlPart.vb` | | FlowGrill quality-control part |
| `BOMFilter.vb` | | BOM-filter logic |
| `AutoIPOrderSelection.vb` | | auto-pick IPorder helper |
| `ProductionLine.vb` | | production-line entity |
| `ProductionLog.vb` | | production-log writer |
| `LabelLog.vb` | | label-print log writer |
| `ProductionRegistrationAnalysisRange.vb` | | analysis date-range value object |
| `SmtBendQueue.vb` | | SMT bend queue (parallel to ISAH.WorkView.GetSmtBendQueue) |
| `FrmProdMachineSelector.vb` + `.Designer.vb` | | machine-picker dialog (consumed by Client.SmartUpdateBySession) |
| `UCProdLineLeanStatus.vb` + `.Designer.vb` | | UserControl showing line lean-status |

## Notable findings

1. **`SmtBendQueue.vb` likely overlaps with `ISAH.WorkView.GetSmtBendQueue`** (Q-207). One uses iCenter DB, the other ISAH DB. Phase-3 follow-up to align.
2. **`FrmProdMachineSelector`** is the **interactive machine picker** popped by `Client.SmartUpdateBySession` when the host is an ADS server (Q-220). Confirms ADS-server presence implies multi-machine-binding ambiguity that requires operator resolution.
3. **`ProductionProfileCutItemsHandler`** is the bulk-orchestrator family; pairs with the single-item handler. Likely the home of the **profile-mill cut-item business logic** that feeds Elumatec.
4. **`FGQualityControlPart`** — FlowGrill-specific quality-control extension. The FlowGrill divergence appears here just like in [[icenter-smt-deburr-cycle-time|deburr cycle time]] (2× multiplier) and the JAZO-vs-FlowGrill company-code rule.
5. **`CEChecklist`** + **`ProdChecklist`** + **`ProdChecklistTag`** — three-class checklist subsystem. CE = Conformité Européenne (regulatory). `#safety-relevant`.
6. **`BaseProductionItem`** is the abstract base — inheritance not composition; some items inherit it.

## Open questions

- **Q-284 (new):** `Production.SmtBendQueue` vs `ISAH.WorkView.GetSmtBendQueue` — are these duplicate implementations? Deduplicate.
- **Q-285 (new):** Document `CEChecklist` — what items does it enforce? CE-marking is a regulatory requirement; the rule set should be explicit. `#safety-relevant`
- **Q-286 (new):** Document `ProductionProfileCutItemsHandler` — bulk write path for cut items; SME confirmation of behaviour required given Elumatec downstream.

Logged in [[../needs-review/_index]].

## Coverage

All 19 source files marked `done` overview-level. Designer files marked `generated`. **Phase-4 sweep should deep-read `ProductionProfileCutItemsHandler`, `CEChecklist`, `BOMFilter`, `KanbanBin`** for business-rule extraction.
