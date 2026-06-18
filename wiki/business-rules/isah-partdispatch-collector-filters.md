---
type: business-rule
title: "Dispatch-collector hardcoded filters (M38, KD%, ALG, status 40)"
status: needs-review
module: "ICenterLib/ISAH"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\PartDispatchCollectorDataService.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\DataServices\\PartDispatchCollectorDataService.vb"
last-reviewed: ""
tags: [business-rule, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Dispatch-collector hardcoded filters

## Rule

The `PartDispatchCollectorDataService.GetBillOfOperationByDossierCode` query — the one that gathers dispatchable operations for the per-dossier pick view — applies **five hardcoded filters** that restrict the visible workflow:

| Filter | Hardcoded value | Meaning |
|--------|----------------|---------|
| `PBOO.MachGrpCode` | `'M38'` | only operations on **machine-group M38** are dispatch-collected |
| `WH.WarehouseCode LIKE` | `'KD%'` | only warehouses whose code starts with **KD** are eligible |
| `P.DispatchWarehouseCode <>` | `'ALG'` | parts whose default dispatch warehouse is **`ALG` (general)** are excluded |
| `PBOO.FinishedInd =` | `0` | only **unfinished** operations |
| `PH.ProdStatusCode =` | `'40'` | only ProductionHeaders in status **`'40'`** (the [[icenter-status-code-default-range\|default work-view start status]]) |

Operators searching for a dispatch outside these filters see *nothing*. The function accepts an `@MachGrpCode` parameter but **never references it in the SQL** — Q-169 marks it as dead.

## Where it lives

- File 1: `ICenterLib\ISAH\PartDispatchCollectorDataService.vb` lines 76–92
- File 2: `ICenterLib\ISAH\DataServices\PartDispatchCollectorDataService.vb` lines 60–76
- Both versions share the same hardcoded filters.

## The code (minimal quote)

```sql
SELECT ...
FROM T_DossierDetail DD
INNER JOIN T_ProdHeadDosDetLink PHDDL ON ...
INNER JOIN T_ProdBillOfOper PBOO ON ...
INNER JOIN T_ProdBillOfMat PBOM ON ...
INNER JOIN JZ_ProdRefNr PRN ON ...
INNER JOIN T_ProductionHeader PH ON ...
INNER JOIN T_ShopDoc SD ON ...
INNER JOIN T_Part P ON PBOM.SubPartCode = P.PartCode
INNER JOIN T_Warehouse WH ON P.DispatchWarehouseCode = WH.WarehouseCode
WHERE DD.DossierCode = @DossierCode
  AND DD.DetailCode <> '000'
  AND PBOO.MachGrpCode = 'M38'           -- hardcoded
  AND PBOO.FinishedInd = 0               -- hardcoded
  AND PH.ProdStatusCode = '40'           -- hardcoded
  AND WH.WarehouseCode LIKE 'KD%'        -- hardcoded
  AND P.DispatchWarehouseCode <> 'ALG'   -- hardcoded
GROUP BY ...
```

## Why this is a business rule

`M38` is **the** dispatch-collector target machine group in JAZO's current configuration. Whichever workstation / pick-station / KD-prefix warehouse system is wired to this DataService, it sees only operations routed through M38. If JAZO ever needs a second dispatch-collector for a different machine group, this code requires editing.

The `KD%` warehouse-code prefix and the `ALG` exclusion encode JAZO's warehouse layout: warehouses prefixed `KD` are dispatch-eligible (likely *kardex* / picking warehouses); `ALG` ("algemeen" = general) is the catch-all warehouse not subject to dispatch tracking.

`ProdStatusCode = '40'` aligns with [[icenter-status-code-default-range|`Common.WorkViewDefaultFromStatusCode = "40"`]] — production headers at status 40 are the actively-running ones eligible for dispatch.

## Triggers / when it fires

- Anywhere `PartDispatchCollectorDataService.GetDataByDossierCode(DossierCode, MachGrpCode, DispatchType)` is called. Phase-3 follow-up to enumerate callers — likely the dispatch UI in iCENTER `Forms/ShopProcess/`.

## Effects

- Returns only the matching operations.
- Side effects when `DispatchType = 1`: `ProcessDispatchDataByShopDocCode` runs the ISAH SP `JIP_Prc_PRO008` for each row before the pick-status calculation — i.e. selecting "dispatch type 1" *triggers* an ISAH stored-procedure for every row of the result.

## Edge cases / known exceptions

- **Dead `@MachGrpCode` parameter.** Caller passes a value; query ignores it; the hardcoded `'M38'` always wins. Misleading API. Q-169.
- **Status code `'40'` only.** Production headers at other statuses (50 = finished? 30 = preparing? — see Q-160) are invisible to dispatch.
- **`KD%` is `LIKE` not `=`.** Adding a sixth or seventh KD-prefixed warehouse (e.g. `KDXX01`) needs no code change. Same for renaming — `KD01` → `KDA1` is fine. `KD` → `K1D` breaks.
- **Two divergent `CalculateJobDone` implementations** consume this query — see [[../modules/isah-part-and-dispatch|Q-172]]. Same hardcoded filters, different downstream computation.

## Safety classification

- [x] Drives shop-floor pick visibility → `#safety-relevant`
- [ ] Reversible if wrong? — Yes (source edit) but operators won't see the affected dispatches in the meantime.
- [x] Blocks production if it fails? — Yes (pickers won't see work).

## SME questions

- **Q-169** (cross-listed): remove the dead `@MachGrpCode` parameter and document the M38 assumption.
- **Q-175** (cross-listed): document the M38 / KD-prefix / ALG warehouse conventions.
- **Q-178 (new):** Multi-dispatch-target support — if JAZO ever needs a second dispatch-collector for a different machine group, what's the refactor path?

Logged in [[../needs-review/_index]].

## Related

- [[../modules/isah-part-and-dispatch]] — defining module (and Q-168 about the duplicated implementation).
- [[icenter-status-code-default-range]] — sibling rule about status code `'40'`.
- [[../mocs/icenterlib-isah]] — parent MOC.
