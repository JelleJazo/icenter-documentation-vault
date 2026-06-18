---
type: module
title: "OperationSubstitutionHandler — swap machine-group ops"
status: done
module: "iCENTER/WorkPreparation"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\WorkPreparation\\OperationSubstitutionHandler.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `OperationSubstitutionHandler.vb` — swap one machine-group op for another

## Purpose

In-memory rewrite of a `ClsProdObjects` (production-objects bundle) to **replace every reference to one machine-group operation with another**. Used when werkvoorbereiding needs to reroute a job to a different machine group — e.g. when a machine is down or capacity-balancing.

Takes four immutable inputs: `FromOperation`, `ToOperation`, `KeepSetupTime`, `KeepCycleTime`. Mutates the production-objects' DataTables: `DtObjects`, `dtSurfTreatmentPart`, `dtSurfTreatmentOper`, `dtOperTotal`, and `DtMachGrp` (the last is only modified if the target operation is missing).

## Public surface

```vb
Public ReadOnly Property FromOperation   As String
Public ReadOnly Property ToOperation     As String
Public ReadOnly Property KeepSetupTime   As Boolean
Public ReadOnly Property KeepCycleTime   As Boolean

Public Sub New(fromOperation, toOperation, keepSetupTime, keepCycleTime)
Public Sub Execute(ProdObjects As ClsProdObjects)
```

`Execute` runs five private subs in fixed order:

1. `VerifyInMachGrps(ProdObjects)` — ensure `ToOperation` exists in `ProdObjects.DtMachGrp`; pull the matching row from `oICENTER.GetMachGrps(True)` if missing.
2. `VerifyColumnsInParts(ProdObjects)` — add 5 columns to `DtObjects` if missing: `MachSetupTime_<To>`, `MonoMachCycleTime_<To>`, `TotalMachTime_<To>`, `MachCycleTime_<To>`, and the boolean `<To>` flag column.
3. `SubstituteInParts(ProdObjects)` — for each row where `<From>=True`: flip the booleans and copy/zero the timing columns based on `KeepSetupTime`/`KeepCycleTime`.
4. `SubstituteInSurfTreatmentPart(ProdObjects)` — flip MachGrpCode in surface-treatment-part rows. **(Iterates `dtSurfTreatmentOper`!)** — Q-095.
5. `SubstituteInSurfTreatmentOper(ProdObjects)` — flip MachGrpCode in surface-treatment-oper rows. **(Iterates `dtSurfTreatmentPart`!)** — Q-095.
6. `SubstituteInOperTotals(ProdObjects)` — flip MachGrpCode in `dtOperTotal` rows.

## Behavior in plain language

For each part with `FromOperation = True`:
- Flip `<From>` to `False` and `<To>` to `True`.
- If `KeepSetupTime` AND `KeepCycleTime`: copy four timing columns from `_<From>` to `_<To>`; otherwise zero the `_<To>` columns.
- **Always zero** the `_<From>` columns regardless of keep-flags (lines 81–84).

This means: even if you ask to "keep" the times, the `_<From>` columns get zeroed — preserving correctness of "this part's `FromOperation` time is now zero because it's been substituted".

## Surprises

1. **The two surface-treatment subs have swapped iteration sources** (lines 88–100). `SubstituteInSurfTreatmentPart` iterates `ProdObjects.dtSurfTreatmentOper`, and `SubstituteInSurfTreatmentOper` iterates `ProdObjects.dtSurfTreatmentPart`. Either:
   - **(a)** Both tables happen to have the same `MachGrpCode` column so the swap is harmless, OR
   - **(b)** This is a bug — each sub updates the wrong table.
   The names suggest (b). **Q-095, `#needs-review`**.
2. **Hard-coded `KeepCycleTime` logic** only triggers the copy when both flags are true (lines 69–73). If `KeepSetupTime=True` and `KeepCycleTime=False`, the code zeroes everything anyway. The `KeepCycleTime` flag therefore acts as the gating flag, not the setup-time one. Q-097.
3. **String concatenation in column names** (`"MachSetupTime_" & ToOperation`) — if `ToOperation` ever contains characters illegal in a DataTable column name, the `Columns.Add` will throw.
4. **No transaction.** If `oICENTER.GetMachGrps(True)` fails mid-Execute, `ProdObjects.DtMachGrp` may have a partially-added row.
5. The class is **stateless** (all inputs are immutable ReadOnly properties) and the input `ProdObjects` is mutated in place — so the same handler can be `Execute`'d against many objects sequentially without re-construction.

## Business rules surfaced here

- The four-column timing model (`MachSetupTime_X`, `MonoMachCycleTime_X`, `TotalMachTime_X`, `MachCycleTime_X`) is iCenter's standard per-operation timing schema. Any machine-group code is dynamically wide-column'd into this schema.
- Substitution is a *full replacement* — there's no "split the time between From and To" mode.

## External systems touched

- [[../external-systems/icenter-db|iCenter DB]] via `oICENTER.GetMachGrps(True)`.
- Reads + mutates `ClsProdObjects` (in-memory; the actual ProdObjects class lives in iCENTER's `Classes/` — phase-3 follow-up).

## Open questions

- **Q-095 (new):** the two surface-treatment subs use swapped table sources (`*Part` iterates `dtSurfTreatmentOper` and vice versa). Bug or harmless? `#safety-relevant`
- **Q-097 (new):** `KeepSetupTime` flag is read but only acts when `KeepCycleTime` is *also* true. Is this intentional? `#needs-review`

Logged in [[../needs-review/_index]].

## Coverage

`_coverage.md`: `WorkPreparation\OperationSubstitutionHandler.vb` → `done`.
