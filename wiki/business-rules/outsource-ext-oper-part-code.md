---
type: business-rule
title: "Outsource ext-operation part code (UITBESTEDING01)"
status: needs-review
module: "iCENTER/WorkPreparation"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\WorkPreparation\\OutsourceOperationsHandler.vb"
last-reviewed: ""
tags: [business-rule, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Outsource ext-operation part code = `UITBESTEDING01`

## Rule

When iCenter creates an **ISAH purchase order for outsourced operations**, the *ext-operation part code* it passes to `myPurOrd.CreatePurOrdByExtOperParts` is the literal string `"UITBESTEDING01"` (Dutch: "outsourcing 01"). This is the only such code emitted by iCenter today.

## Where it lives

- File: `WorkPreparation\OutsourceOperationsHandler.vb`
- Symbol: `Private Sub ProcessDataByVendIdProfileId(VendId, ProfileId)` line 311

## The code (minimal quote)

```vb
Dim myPurOrd As New PurOrd()
Dim AutoSendEmail As Boolean = False
Dim extOperPartCode As String = "UITBESTEDING01"

Dim ExistingPurDocCode As Integer = 0
If PurDoc IsNot Nothing Then
    ExistingPurDocCode = PurDoc.PurDocCode
End If

Dim PurDocCode As Integer = myPurOrd.CreatePurOrdByExtOperParts(
    extOperPartCode, lExtOperPart, VendId,
    AutoSendEmail, False, ExistingPurDocCode, DelDate)
```

## Why this is a business rule

`UITBESTEDING01` is a JAZO-internal **part-code convention** that tags a purchase-order line as "this is an outsourced operation, not a physical part purchase". ISAH presumably has a corresponding `T_Part` (or `JZ_*`) entry with this code that carries the right cost-account, VAT, and routing flags for outsourced work.

If JAZO ever introduces a second outsource flow (e.g. `UITBESTEDING02` for a different type of outsourced operation), it would require a code change here — there's no per-vendor or per-machine-group lookup for the part code.

## Triggers / when it fires

- Once per `(VendId, ProfileId)` combination in `OutsourceOperationsHandler.ProcessData()`, which itself runs once per vendor batch the user processes through `FrmOutsourceOperations`.
- Currently only `ProfileId = 1` is implemented — see [[../modules/workprep-outsource-operations]] Q-084.

## Effects

- Creates an ISAH purchase document keyed by `UITBESTEDING01`. The resulting `PurDoc` is held on the handler and used to (a) resolve the `IsahDoc\Purchase\<PurOrdNr>\01\` drop folder, (b) read back the `PurOrdNr` used as the `Reference` prefix for the per-vendor folders.

## Edge cases / known exceptions

- **`AutoSendEmail = False`** is also hard-coded (line 310). Even if `UITBESTEDING01` had email-sending configured in ISAH, this call wouldn't trigger it.
- The existing-PurDoc path (`ExistingPurDocCode`) lets the user append to an in-progress purchase doc instead of creating a new one each time `ProcessData` runs.

## Safety classification

- [x] Touches business process (purchase order, financial commitment) → `#safety-relevant`
- [ ] Reversible if wrong? — Yes, via ISAH PurDoc edit.
- [ ] Blocks production if it fails? — Halts outsourcing only.

## SME questions

- **Q-083**: is there ever a need for a second outsource part-code (e.g. for a different vendor agreement)?
- **Q-094** (from [[../modules/workprep-outsource-operations]]): the `SetShopDocFinInd(True)` is commented out — what currently marks a ShopDoc as finished after outsourcing?

Logged in [[../needs-review/_index]].

## Related

- [[../modules/workprep-outsource-operations]] — the full pipeline.
- [[../mocs/office-to-shopfloor]] — workflow hub.
- [[../external-systems/isah|ISAH]] — `PurOrd`, `PurDoc`, `T_Part` host the code.
