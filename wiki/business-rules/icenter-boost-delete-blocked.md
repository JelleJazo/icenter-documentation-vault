---
type: business-rule
title: "Boost SMT order delete blocked once parts are completed"
status: needs-review
module: "ICenterLib/iCenter"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\IPBatch.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\IPOrder.vb"
last-reviewed: ""
tags: [business-rule, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Boost (SMT/Oseon) order delete blocked once parts have started

## Rule

A user-initiated delete of an iCenter→Boost (Oseon) batch is **rejected** if any IPpartLine within the batch has `Completed = 1`. The Dutch exception message is shown:

> "**{MachGrpCode}: Opdrachten uit Boost verwijderen is niet mogelijk. Productie is reeds gestart.**"
>
> (Cannot remove orders from Boost. Production has already started.)

The rule applies per-IPbatch. At the IPorder level, the delete fans out across the SMT MachGrps `{"P44", "P46"}` and accumulates exceptions for any that fail.

## Where it lives

- File: `ICenterLib\iCenter\IPBatch.vb` lines 328-341
- Symbol: `IPBatch.DeleteSmtProdOrd(oseonAppContext As SmtProduction.Entities.OseonAppContext)`
- Fan-out: `ICenterLib\iCenter\IPOrder.vb` lines 60-81 — `IPOrder.DeleteSmtProdOrd` iterates `GetIPBatches({"P44", "P46"})`.

## The code (minimal quote)

```vb
Public Function DeleteSmtProdOrd(oseonAppContext) As Boolean
    Dim POI As New SmtProduction.DataServices.ProductionOrderDataService(oseonAppContext)
    Dim flag As Boolean = False

    If GetCompletedPartsCount() > 0 Then
        flag = False
        Throw New Exception(String.Format(
          "{0}: Opdrachten uit Boost verwijderen is niet mogelijk. Productie is reeds gestart.",
          MachGrpCode))
    Else
        POI.DeleteICenterBatch(Id)
        flag = True
    End If

    Return flag
End Function
```

## Why this is a business rule

Boost (the JAZO-internal name for the SMT Oseon production-order system) tracks **physical work in progress**. Deleting an order whose parts have already been cut/bent/processed would create a phantom — material exists but no longer has an order to consume it. Rule enforces "if any part is completed, the order is sticky".

The threshold is **`Completed > 0`**, not `Completed = total`. Even one completed part locks the whole batch. This is conservative.

## Triggers / when it fires

- When the user / an admin action calls `IPBatch.DeleteSmtProdOrd` or `IPOrder.DeleteSmtProdOrd`.
- Hard-coded entry points need Phase-3 follow-up (likely a context-menu or admin-screen action somewhere in `iCENTER\SmtManufacturing` or similar).

## Effects

- **If blocked**: Dutch `Exception` propagates to the UI; nothing changes in Oseon or iCenter.
- **If allowed**: `ProductionOrderDataService.DeleteICenterBatch(Id)` deletes the order from Oseon (likely also cascades in iCenter — to be verified). Returns `True`.

## Edge cases / known exceptions

- **`GetCompletedPartsCount` reads `T_IPpartLines.Completed = 1`** via `DataView` filter over `GetIPParts(False)`. If `GetIPParts` returns `Nothing` (DB error), `GetCompletedPartsCount` throws NullReferenceException — propagated up; user sees a cryptic stack rather than the Dutch business message.
- **Empty batch (no parts at all)**: `GetCompletedPartsCount = 0`, delete is allowed.
- **`{P44, P46}` is hardcoded** in IPOrder.DeleteSmtProdOrd (Q-228). Adding a new SMT MachGrp = source change.
- **Fan-out partial-failure**: `IPOrder.DeleteSmtProdOrd` collects exceptions per-batch and throws an aggregate `MySystem.ExceptionList`. Some batches may have been deleted before another batch's rejection.

## Safety classification

- [x] Touches physical process — yes (controls what's in the SMT production queue).
- [ ] Drives cost / pricing — indirectly.
- [ ] Reversible if wrong? — **No**: a successful delete removes the order from Oseon; re-creating it requires re-syncing from iCenter.
- [x] Blocks production if it fails? — yes (a blocked delete keeps the order alive in Oseon).

## SME questions

- **Q-249 (new):** Confirm `{P44, P46}` is the complete set of SMT MachGrps. Should there be a config-driven list?
- **Q-250 (new):** What happens to the cascaded iCenter records when `ProductionOrderDataService.DeleteICenterBatch` succeeds? Is `T_IPbatchLines` row removed or just flagged `IbDisabled=1`?
- **Q-251 (new):** When IPOrder.DeleteSmtProdOrd fans out across P44+P46 and one fails after the other succeeded, can the user retry safely?

Logged in [[../needs-review/_index]].

## Related

- [[../modules/icenterlib-icenter-batch-hierarchy|`IPBatch` + `IPOrder`]] — defining modules.
- [[../external-systems/oseon|Oseon]] — destination system.
- [[../mocs/icenterlib-icenter]] — parent MOC.
