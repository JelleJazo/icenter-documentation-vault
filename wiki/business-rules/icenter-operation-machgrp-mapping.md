---
type: business-rule
title: "iCenter-operation → MachGrpCodes mapping"
status: needs-review
module: "iCENTER/Production"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Production\\ProductionProfileCutItemsHandler.vb"
last-reviewed: ""
tags: [business-rule, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# iCenter-operation → MachGrpCodes mapping

## Rule

`ProductionProfileCutItemsHandler` knows how to write **profile cut-items** for **three** iCenter operations only, each mapped to a specific (ordered) list of machine-group codes:

| `iCenterOperationId` | MachGrpCodes (order matters) |
|---------------------:|------------------------------|
| `1` | `{"A01", "A07"}` |
| `9` | `{"S01"}` |
| `31` | `{"A07", "A01"}` |

Any other `iCenterOperationId` throws `NotImplementedException` from the constructor.

Note that **operations 1 and 31 contain the same set** of codes but in **opposite order**. The mapping is therefore not a set — order is exposed via `Public Shared Function GetImplementedOperations()` to callers that iterate `MachGrpCodes`.

## Where it lives

- File: `Production\ProductionProfileCutItemsHandler.vb`
- Symbol: `Private Shared Function GetDictionary() As Dictionary(Of Integer, String())` lines 43–50

## The code (minimal quote)

```vb
Private Shared Function GetDictionary() As Dictionary(Of Integer, String())
    Dim dict As New Dictionary(Of Integer, String())
    dict.Add(1, {"A01", "A07"})
    dict.Add(9, {"S01"})
    dict.Add(31, {"A07", "A01"})

    Return dict
End Function

Public Shared Function OperationIsImplemented(iCenterOperationId As Integer) As Boolean
    Return GetDictionary().ContainsKey(iCenterOperationId)
End Function

Public Shared Function GetImplementedOperations() As Integer()
    Return GetDictionary().Keys.ToArray()
End Function
```

## Why this is a business rule

iCenter operation IDs are JAZO-internal numeric identifiers for "what kind of machining work this part needs". The IDs map to ISAH **machine-group codes** (the codes used by ISAH's routing engine to schedule work). The mapping encoded here is a **policy decision**: which iCenter operations even *have* a profile-mill cut-items export today, and in which MachGrp(s) the part will be cut.

Adding a new iCenter operation that needs profile-mill cut-items requires editing this dictionary; same for changing the MachGrpCode set for an existing operation.

The order of MachGrpCodes is meaningful because it's passed to `ICenterObject.GetMBomMultilevel({MachGrpCode}, OrderType, True)`. Whether `GetMBomMultilevel` uses the order to *prefer* one routing over another is a Phase-3 follow-up.

## Triggers / when it fires

- Constructor lookup: `Throw New NotImplementedException` if `iCenterOperationId` isn't a key. Callers should pre-check with `OperationIsImplemented(...)`.
- In `Write()`: `GetMBomMultilevel(MachGrpCodes, OrderType, True)` — passes the array to iCenter's BOM walker.

## Effects

- Filters the multi-level BOM to only items routed through one of the listed MachGrpCodes.
- The resulting cut-items table (with `BIdentNo` enriched from EluCad) is persisted via `ICenterLib.Production.ProductionProfileCutItemsHandler.Save()`.
- Always clears the previous cut-items for the same `MachineId` first — see [[../modules/production-profile-cut-items|Q-100]].

## Edge cases / known exceptions

- **Operations 1 and 31 share the same set** (`{A01, A07}`) but order is reversed. If `GetMBomMultilevel` is order-sensitive, the two operations produce different BOMs. Q-085 / Q-106.
- **No fallback for unknown operations** — caller must pre-check `OperationIsImplemented`. The form/code that constructs this handler is the gatekeeper; if it doesn't pre-check, end users hit a `NotImplementedException`. Phase-3 follow-up to find the caller.
- **No metadata** (description, units, SME-friendly name) on the operation IDs in the source. SMEs have to know that "operation 9" means "S01 routing".

## Safety classification

- [x] Touches physical process (drives which machine cuts the part) → `#safety-relevant`
- [ ] Reversible if wrong? — Yes (edit dictionary, redeploy), but operator may have already cut the part.
- [x] Blocks production if it fails? — Yes (NotImplementedException stops the handoff).

## SME questions

- **Q-085**: confirm 1/9/31 are the only iCenter operations that have profile-mill cut-items today. Provide SME-friendly names for each.
- **Q-106 (new):** ordering of MachGrpCodes between operations 1 and 31 — is `GetMBomMultilevel` order-sensitive? If not, why two distinct operations with the same set? `#safety-relevant`

Logged in [[../needs-review/_index]].

## Related

- [[../modules/production-profile-cut-items]] — the module.
- [[../mocs/office-to-shopfloor]] — workflow hub.
- [[../modules/elumatec-cad-app|EluCadApp]] — the `BIdentNo` lookup that fills the cut-items rows.
- [[../external-systems/isah|ISAH]] — owns the `T_MachGrp` table that defines what `A01`, `A07`, `S01` mean.
