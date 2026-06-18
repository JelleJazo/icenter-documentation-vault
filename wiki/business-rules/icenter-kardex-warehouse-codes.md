---
type: business-rule
title: "Kardex warehouse codes (KD06A small-parts, KD10A large-parts)"
status: needs-review
module: "iCENTER/Kardex"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Kardex\\KardexProcessor.vb"
last-reviewed: ""
tags: [business-rule, needs-review, safety-relevant]
created: 2026-06-18
updated: 2026-06-18
---

# Kardex warehouse codes — `KD06A` (small-parts) / `KD10A` (large-parts)

## Rule

The Kardex Shuttle storage system at JAZO has at least two distinct warehouses, identified by codes:

- **`KD06A`** — the **small-parts** shuttle. Recognised by special handling of weight (Put writes `0.001` kg = 1 gram; PowerPick reweighs physically) and ProdHeaderOrdNr suffix `-06`.
- **`KD10A`** — the **large-parts** shuttle. Recognised by ProdHeaderOrdNr suffix `-10`.

These codes are **hardcoded** in `KardexProcessor.Generate` — both `Select Case` blocks for ProdHeaderOrdNr-suffixing and the weight-override rule.

The naming convention follows the broader `KD%` warehouse-code prefix family also documented at the ISAH layer ([[../modules/isah-part-and-dispatch|Q-175]]).

## Where it lives

- File: `C:\DevOps\iCenter\iCenter\iCENTER\Kardex\KardexProcessor.vb`
- Symbols affected:
  - `Generate` (ProdHeaderOrdNr suffix mutation): lines 139-156
  - `Generate` (VcALBH weight rule): lines 225-240
  - `SaveXml` (per-warehouse output sub-folder): line 289

## The code (minimal quotes)

ProdHeaderOrdNr suffix rules:

```vb
If DispatchWarehouseCode.ToUpper = "KD10A" Then
    .WriteElementString(..., orderNr & "-10", ...)
ElseIf DispatchWarehouseCode.ToUpper = "KD06A" Then
    .WriteElementString(..., orderNr & "-06", ...)
Else
    .WriteElementString(..., orderNr, ...)
End If
```

Weight rule (only for `KD06A`):

```vb
Case "VcALBH"
    Select Case DispatchWarehouseCode
        Case "KD06A"
            Select Case DirectionType
                Case TypeOfDirection.Put
                    ' Set the weight to 1 gram, the product will be weighted in PowerPick afterwards
                    .WriteElementString(..., "0.001")
                Case TypeOfDirection.Pick
                    WeightConvertedValue = FormatItemValue(...)
                    .WriteElementString(..., WeightConvertedValue)
            End Select
        Case Else
            .WriteElementString(..., asIs, ...)
    End Select
```

Per-warehouse output folder:

```vb
XmlDocument.Save(My.Settings.Properties("Isah2KardexExportPath").DefaultValue &
                 DispatchWarehouseCode & "\" & FileName)
```

So each Kardex warehouse has its own output sub-folder (e.g., `\\export-path\KD06A\file.xml` vs `\\export-path\KD10A\file.xml`) — Kardex's PowerPick watches per-warehouse and picks up its own.

## Why this is a business rule

1. **Wrong warehouse code → file lands in wrong folder → Kardex PowerPick controls the wrong physical shuttle.** Q-335 — if PowerPick is misconfigured to watch a single folder, putting `KD06A` material physically into the `KD10A` shuttle is a real risk.
2. **The 1-gram-Put trick** for `KD06A` is the **only safe way to handle very-small parts** that the Kardex pick-station scale can weigh accurately. The Kardex hardware reweighs after placement; sending "0.001" tells PowerPick "I don't know the real weight, please measure". Sending the real (computed) weight risks overriding the physical measurement with an iCenter-estimated value.
3. **The `-06` / `-10` order-number suffix** is what allows the operator (and the Kardex audit trail) to disambiguate two physical orders that share a `ProdHeaderOrdNr` but split across the two shuttles — without the suffix, the orders would collide.

## Triggers / when it fires

- Every `KardexProcessor.Generate(DispatchWarehouseCode, ...)` call where `DispatchWarehouseCode` matches `KD06A` or `KD10A`.

## Effects

- ProdHeaderOrdNr in the XML gets a `-06` or `-10` suffix per warehouse.
- For `KD06A` Put operations, weight `0.001` is sent; for Pick, the iCenter-computed weight is sent.
- Output file lands in `{Isah2KardexExportPath}\{KD06A|KD10A}\{FileName}`.

## Edge cases

- **A new Kardex warehouse** (e.g., `KD12A`) would fall through to the `Else` branch in both suffix-mutation and weight-rule paths — no suffix, no weight override. Adding one requires source edit.
- **Case-sensitivity**: comparisons use `.ToUpper` so lowercase `kd06a` is accepted; but they don't trim trailing whitespace explicitly.
- **`Unplanned` dispatch + KD10A** also gets `-10` suffix (per a different code path).

## Safety classification

- [x] Touches physical process — yes (drives which shuttle moves).
- [ ] Drives cost / pricing — no.
- [ ] Reversible if wrong? — yes (manual override on Kardex side).
- [x] Blocks production if it fails? — wrong-warehouse routing wastes operator time + possibly mis-positions material. `#safety-relevant`

## SME questions

- **Q-335 (new):** Confirm Kardex PowerPick watches per-warehouse sub-folder. If misconfigured, file → wrong shuttle.
- **Q-336 (new):** Document **all** known Kardex warehouse codes — are KD06A + KD10A the only two, or are there more (KD08A, KD12A)?
- **Q-337 (new):** Justify the 1-gram Put weight — is there a Kardex tolerance below which the hardware can't weigh?

Logged in [[../needs-review/_index]].

## Related

- [[../modules/icenter-kardex|`KardexProcessor`]] — defining module.
- [[../modules/isah-part-and-dispatch|`KD%` warehouse-prefix convention]] (Q-175) — ISAH-side counterpart.
- [[../external-systems/kardex|Kardex Shuttle]].
