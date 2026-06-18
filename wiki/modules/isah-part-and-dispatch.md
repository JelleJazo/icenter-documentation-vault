---
type: module
title: "ISAH Part + dispatch — Part / PartDispatch / PartDataService / PartSelection / PartVendor"
status: done
module: "ICenterLib/ISAH"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\Part.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\PartDispatch.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\PartDispatchCollectorDataService.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\DataServices\\PartDispatchCollectorDataService.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\DataServices\\PartDataService.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\PartSelection.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\PartVendor.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# ISAH Part + dispatch

## Purpose

Seven classes that together model the **ISAH Part** entity (T_Part — the master article record) and its dispatch / warehouse-issue workflow:

```
Part (T_Part)                                                — 136-column master article record
   ├── PartSelection (T_PartSelection)                       — many-to-many: part ↔ selection codes
   ├── PartVendor    (IP_sel_PartVendorRS SP)                — vendor records for a part
   └── PartDispatch  (T_PartDispatch via IP_Ins_PartDispatch)— per-issue warehouse dispatch

PartDataService.GetByDesignCode                              — read T_Part by DesignCode (data-service style)
PartDispatchCollectorDataService (×2 — root and DataServices)— assemble per-dossier dispatch DataTables
```

Part is the **most-referenced entity** in the ISAH wrapper. Every BOM walk, every workflow that calls `Part.GetPrintCode(partCode)` or `Common.IsPcfAdminDossier`, every `ISAH.MachGrp.GetDeptCode` chain eventually touches Part.

## `Part` (49 KB, ~1000 lines)

Largest non-TimeRegistration ISAH file. ~25 public methods, predominantly **dynamic field accessors**.

### Public surface (selected)

| Symbol | Returns | Notes |
|--------|---------|-------|
| `New(PartCode)` | constructor | PartCode is the natural primary key |
| `Shared GetPartField(PartCode, ReturnValue)` | Object | **DYNAMIC ACCESSOR** — builds `SELECT <ReturnValue> FROM T_Part WHERE PartCode = @PartCode` |
| `GetPartField(ReturnValue)` | Object | instance overload — same |
| `GetDesignCode` / `GetDescription` / `GetDensity` / `GetPrintCode` / `GetPartGroupCode` / `GetCalcQty` | typed accessors | thin wrappers over `GetPartField` |
| `Shared GetCoatingPrimers()` | DataTable | `WHERE P.PartCode LIKE 'VPR-%' AND P.PartCode <> 'VPR-INVULLEN'` |
| `Shared GetPrintCode(PartCode)` | `Enums.ISAH.PrintCode` | nullsafe — returns `NietPrinten` if absent |
| `Shared GetOrdCode(PartCode)` | `Enums.ISAH.OrdCode` | thin wrapper |
| `Shared GetCoatingSystemPartsOpers(PartCode, SelectedColor)` | DataSet | dual-resultset: T_BillOfMat + T_BillOfOper join; embeds `T_PartSelection.SelectionCode IN ('024', '025')` and `'018'` (dark primer flag) |
| `CopyOperationsToProdHeader(Opers, ...)` | sub | copy a subset of operations from this Part's BOO into a ProductionHeader |
| `IP_sel_BOORecordSet()` | DataTable | full BOO record-set for this Part |
| `SetDispatchInfo(ByRef WarehouseCode, ByRef LocationCode)` | sub | reads the part's default `DispatchWarehouseCode` / `DispatchLocationCode` into out-parameters |
| `GetLastUpdatedOn()` | String | for optimistic concurrency |
| `IP_Sel_PartRecord()` | DataTable | full ISAH record fetch — SP `IP_sel_PartRecord` |
| `CopyTo(PartCode)` | `Part` | clone this Part to a new PartCode |
| `IP_cpy_PartsList(partcode1, ...)` | sub | bulk-copy variant |
| `UpdateDescription(value)` | sub | mutate description |
| `IP_upd_Part(...)` | sub | full-record update (similar to PBOO's 30-parameter inserts) |
| `GetPartExtraInfo(ByRef DesignCode, ...)` | sub | populate many out-parameters at once |
| `GetPartSurchargeCode()` | String | |
| `GetGeneratePD()` | Boolean | "should this part auto-generate a Purchase Document?" |
| `ContainsSelection(SelectionCode)` | Boolean | delegates to PartSelection |
| `GetPartVendorDT()` | DataTable | delegates to PartVendor |
| `GetPartBasicViewModel()` | `ViewModels.PartBasicViewModel` | view-model wrapper |

### Critical detail — dynamic `GetPartField`

```vb
Public Shared Function GetPartField(ByVal PartCode As String, ByVal ReturnValue As String) As Object
    Dim sSQL As String = "SELECT " & ReturnValue & " FROM T_Part WHERE PartCode = @PartCode"
    ' ...
End Function
```

**`ReturnValue` is interpolated directly into the SQL** with no validation. This is the same pattern flagged in [[icenterlib-common|Q-114]] for `Common.GetSqlInString` and [[icenterlib-common|Q-109]] for `Common.GetTableData`. Safe today because `ReturnValue` is always a hardcoded column name from caller code (`"DesignCode"`, `"Description"`, `"weight"`, etc.) — but the **pattern is widely copied** across ICenterLib and one stray user-input callsite would be a SQL-injection vector.

This pattern is the *primary read path* for Part — every per-field accessor (`GetDesignCode`, `GetDescription`, `GetDensity`, etc.) wraps `GetPartField`. Means: one Part read = one SQL round-trip per field. A form that displays 10 Part fields runs 10 queries. No caching. Q-170.

### Hardcoded coating sentinels

`GetCoatingPrimers()` filters on `PartCode LIKE 'VPR-%'` (prefix convention for "verfprimer" = paint primer) and explicitly excludes the placeholder `'VPR-INVULLEN'` ("VPR-FILLIN" — a manual-entry sentinel). Q-171: document the `VPR-` prefix and the `INVULLEN` sentinel.

`GetCoatingSystemPartsOpers(PartCode, SelectedColor)` embeds three hardcoded `SelectionCode`s in its SQL:

- **`'018'`** — "UseDarkPrimer" indicator: presence/absence of this selection on the chosen colour-part determines whether the coating system uses dark primer.
- **`'024'`** and **`'025'`** — paint-related selection categories (Phase-4 follow-up to identify exactly).

Documented as [[../business-rules/isah-coating-selection-codes]].

## `PartDispatch` (8 KB)

```vb
Public Class PartDispatch
    Public Enum ProcessStatus
        NietVerwerken      = 1    ' "don't process"
        KlaarVoorVerwerken = 2    ' "ready to process"
        Verwerkt           = 3    ' "processed"
    End Enum

    Public Sub IP_Ins_PartDispatch(<35 parameters>)
End Class
```

Single behaviour: insert into `T_PartDispatch` via SP `IP_Ins_PartDispatch`. **35 parameters** including `WarehouseCode`, `LocationCode`, `PartCode`, `InvtPartCode`, `InvtLineNr`, `FromProdHeaderDossierCode`, `ProdHeaderDossierCode`, `ProdBOMLineNr`, `CostCenterCode`, `LotNr`, `CertificateCode`, `DispatchUser`, `DispatchProcessedUser`, dimensions (Length/Width/Height), `ProcessStatus`, `PartDispatchType`, `PartDispatchOnType`, dates, `CompletedInd`, `RevisionNr`, `InventoryStatusCode`, `InvtCreDate`, `TransServObjectCode`, `TargetServObjectCode`, `ReplacedABSLineNr`, `MultiLevelReplacementInd`, `PurDocCode`, `PDPartLineNr`, `PicklistNr`. Output parameters: `@new_PartDispatchCode` and `@LastUpdatedOn`.

Notable hard-codes:
- **`@LogProgramCode = 2210000`** — magic ISAH log-program code (cf. Q-151 family).
- **`@LogMessType = 4`** — magic log-message type.
- The `IP_Ins_PartDispatch` SP is the only ISAH dispatch *writer* in iCenter; reads go through the dispatch collector data services.

The three-state `ProcessStatus` enum is documented as [[../business-rules/isah-partdispatch-process-status]].

## `PartDispatchCollectorDataService` — duplicated in two namespaces

The same class name lives in **two files**:

| Path | Namespace | Lines | Has the extra check? |
|------|-----------|-------|---------------------|
| `ISAH\PartDispatchCollectorDataService.vb` | `ISAH` | 167 | **yes** — `countMaxNotPicked` check (`PickedQty < InvtQty AND JobDoneInd = False`) with "calc mismatch" log entry on disagreement |
| `ISAH\DataServices\PartDispatchCollectorDataService.vb` | `ISAH.DataServices` | 151 | **no** — only counts `MaxJobDoneInd = False` |

Both share the same query, the same hard-coded filters, and the same SP calls (`JIP_Get_Pro008`, `JIP_Prc_PRO008`). The `DataServices` variant is a trimmed copy that drops the divergence-detection logic. **Two namespaces, one logical class — code duplication smell** (Q-168, `#needs-review`). Phase-3 follow-up to identify which callers use which.

### Hardcoded query filters

The query in `GetBillOfOperationByDossierCode` (both versions) hard-codes:

- `PBOO.MachGrpCode='M38'` — picking is only for machine-group M38.
- `PBOO.FinishedInd = 0` — only unfinished operations.
- `PH.ProdStatusCode = '40'` — only ProductionHeaders in status 40 (per [[icenterlib-common|`WorkViewDefaultFromStatusCode = "40"`]] convention).
- `WH.WarehouseCode LIKE 'KD%'` — only warehouses with `KD` prefix.
- `P.DispatchWarehouseCode <> 'ALG'` — exclude the `ALG` ("general") warehouse.

**The function accepts `@MachGrpCode` as a parameter but ignores it** — `sqlcmd.Parameters.AddWithValue("@MachGrpCode", MachGrpCode)` is added on line 94 of the root file but the SQL never references `@MachGrpCode`. The parameter is **dead code**. Q-169 (`#needs-review`).

The hardcoded `MachGrpCode='M38'` and `WarehouseCode LIKE 'KD%'` are documented as [[../business-rules/isah-partdispatch-collector-filters]].

### `CalculateJobDone` differences

Both implementations group dispatch rows by `LineNr` and compute `MaxJobDoneInd` per line. The root version additionally:

- Computes `MaxPickedQty` and `MaxInvtQty` per group.
- Counts rows where `MaxPickedQty < MaxInvtQty AND MaxJobDoneInd = False`.
- If the two counts disagree, emits an Info-level log: `"<DossierCode> ... calc mismatch: <a> <> <b>"` and uses the **picked-vs-invt** count to decide `JobDoneInd`, *not* the original `MaxJobDoneInd` count.

The DataServices version just uses `MaxJobDoneInd`. So **the two versions can produce different `JobDoneInd` values for the same input**. Whichever caller uses which version sees a different shop-floor "is this dispatch line done?" answer. `#safety-relevant` if any caller compares the two. Q-172.

## `PartDataService.GetByDesignCode` (4 KB)

Single method. Runs a `SELECT` over T_Part with **all 136 columns explicitly listed** (the entire T_Part schema), filtered by `DesignCode = @DesignCode`. Returns a `DataTable`. Pure read-only data service style.

The explicit column list (vs `SELECT *`) is defensive — if ISAH adds a column, the data-service still returns the previously-expected shape. But the trade-off: any **renamed** column silently disappears from the result.

## `PartSelection` (3 KB)

Two-key wrapper around `T_PartSelection`:

```vb
Public Class PartSelection
    Public ReadOnly Property PartCode      As String
    Public ReadOnly Property SelectionCode As String

    Public Sub New(PartCode, SelectionCode)
    Public Function Exists()                           As Boolean
    Public Shared Function GetRecord(PartCode, SelectionCode) As DataTable
    Public Shared Function GetRecords(PartCode)        As DataTable
End Class
```

`GetRecords(PartCode)` returns all selection-codes on the part; `GetRecord(PartCode, SelectionCode)` filters that result via a `DataView.RowFilter` on `SelectionCode` (client-side filter rather than re-querying). Carries a giant commented-out direct-SqlCommand variant — the live code uses `DataHandler.GenericQuery`.

## `PartVendor` (1 KB)

Trivial. Single method `GetRecords()` runs SP `IP_sel_PartVendorRS` and returns its result. The reverse-direction wrapper to read vendor information for a Part.

## Surprises

1. **Dynamic-SQL `GetPartField`** is the *standard read pattern* for Part. Every field accessor builds its own SQL string with the column name interpolated. Q-170: even though every caller passes a literal, the pattern means N field reads → N round-trips.
2. **Two `PartDispatchCollectorDataService` classes** with the same name in different namespaces, slightly different semantics. Q-168.
3. **Dead `@MachGrpCode` parameter** in the collector query — bound but never referenced in the SQL. Hardcoded `'M38'` is the real filter. Q-169.
4. **Divergent `JobDoneInd` logic** between root and DataServices variants — picked-vs-invt check exists only on the root version. Q-172 (`#safety-relevant`).
5. **`VPR-INVULLEN`** is a magic sentinel PartCode in the primer catalogue (`INVULLEN` = "fill in" in Dutch). Phase-4 candidate.
6. **`GetCoatingSystemPartsOpers`** returns a `DataSet` (two resultsets in one round-trip) rather than two separate `DataTable`s. Only place in the wider ISAH wrapper that uses `DataSet`. Phase-3 to confirm.
7. **`PartDispatch.IP_Ins_PartDispatch`** has 35 parameters but no overload — every caller must supply every parameter (often as `Nothing`/zero/empty). Refactor candidate (parameter object).
8. **`PartDispatch.ProcessStatus`** is Dutch-named (`NietVerwerken / KlaarVoorVerwerken / Verwerkt`). Localised enum — uncommon for code.
9. **`PartSelection.GetRecord` filters client-side** via `DataView.RowFilter` against the already-fetched `GetRecords` table. Means: every `GetRecord` call fetches *all* selections for the part then filters in-process. Fine for small T_PartSelection rows; less great if a part has thousands.

## Business rules surfaced here

- [[../business-rules/isah-partdispatch-process-status|`PartDispatch.ProcessStatus` 3-state enum]] — NietVerwerken (1) / KlaarVoorVerwerken (2) / Verwerkt (3).
- [[../business-rules/isah-partdispatch-collector-filters|Dispatch-collector hardcoded filters]] — `MachGrpCode='M38'`, `WarehouseCode LIKE 'KD%'`, `DispatchWarehouseCode <> 'ALG'`, `ProdStatusCode='40'`, `FinishedInd=0`.
- [[../business-rules/isah-coating-selection-codes|Coating-system selection codes]] — `'018'` = "use dark primer", `'024'` / `'025'` = paint categories.
- `VPR-` prefix convention for primer parts (with `VPR-INVULLEN` sentinel).
- `'M38'` machine-group code = dispatch-collector target (the only MachGrp dispatch ever touches).
- `'KD%'` warehouse-code prefix = dispatch-eligible warehouses.

## Open questions

- **Q-168 (new):** Two `PartDispatchCollectorDataService` classes (root + DataServices namespace) with the same name and ~95% identical code. Pick one, delete the other.
- **Q-169 (new):** `@MachGrpCode` parameter in the dispatch-collector query is bound but never referenced; the hardcoded `'M38'` is the real filter. Remove the dead parameter and document the M38 assumption. `#needs-review`
- **Q-170 (new):** `Part.GetPartField(name)` is the primary read path — one round-trip per field. Cache T_Part rows per PartCode for the call duration? `#needs-review`
- **Q-171 (new):** Document the `VPR-` PartCode prefix convention and the `VPR-INVULLEN` sentinel.
- **Q-172 (new):** Root and DataServices versions of `PartDispatchCollectorDataService.CalculateJobDone` can produce different `JobDoneInd` for the same input. Confirm callers don't compare the two. `#safety-relevant`
- **Q-173 (new):** `PartDispatch.IP_Ins_PartDispatch` has 35 parameters — refactor to parameter object?
- **Q-174 (new):** `PartDataService.GetByDesignCode` explicitly lists 136 columns. Document the policy: do we want explicit-column data services or just `SELECT *`?
- **Q-175 (new):** Document the `'M38'` MachGrp role (dispatch-collector target) and the `'KD%'` warehouse-prefix convention.

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/icenterlib-isah]] — parent MOC.
- [[isah-dossier|DossierDetail.GetPartCode]] — uses `Part.GetPrintCode` indirectly.
- [[isah-production-hierarchy|BillOfMat / BillOfOper]] — both keyed by `Part`.
- [[isah-shop-and-pur-doc|ShopDoc]] — `PartDispatchCollectorDataService` joins through ShopDoc to find unfinished operations.
- [[isah-lookups|`Selection`]] — `PartSelection.SelectionCode` is a `T_Selection` code.

## Coverage

`_coverage.md`:
- `ICenterLib\ISAH\Part.vb` → `done` (overview; the 1000-line file's per-method enumeration deferred)
- `ICenterLib\ISAH\PartDispatch.vb` → `done`
- `ICenterLib\ISAH\PartDispatchCollectorDataService.vb` → `done`
- `ICenterLib\ISAH\DataServices\PartDispatchCollectorDataService.vb` → `done`
- `ICenterLib\ISAH\DataServices\PartDataService.vb` → `done`
- `ICenterLib\ISAH\PartSelection.vb` → `done`
- `ICenterLib\ISAH\PartVendor.vb` → `done`
