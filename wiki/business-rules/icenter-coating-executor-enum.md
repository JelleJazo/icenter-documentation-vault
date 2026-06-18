---
type: business-rule
title: "Coating.Executor — 5 surface-treatment execution paths"
status: needs-review
module: "iCENTER/Classes/Coating"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Classes\\Coating\\Coating.vb"
last-reviewed: ""
tags: [business-rule, needs-review, safety-relevant]
created: 2026-06-18
updated: 2026-06-18
---

# `Coating.Executor` — 5 surface-treatment execution paths

## Rule

A coating job at JAZO is executed via **one of five paths**, encoded as integer values:

| Value | Name | Meaning |
|------:|------|---------|
| 0 | `None` | no coating (sentinel) |
| 1 | `InternalPowderCoat` | JAZO's own powder-coat line |
| 2 | `ExternalCoating` | sent to a coating partner |
| 3 | `ExternalOther` | sent to a different external partner (plating / anodizing?) |
| 4 | `InternalLacquer` | JAZO's own paint/lacquer line |
| 5 | `Galvanize` | galvanizing (likely EBTV — external) |

## Where it lives

- File: `C:\DevOps\iCenter\iCenter\iCENTER\Classes\Coating\Coating.vb` lines 3-10
- Symbol: `Public Enum Coating.Executor`

## The code

```vb
Public Class Coating
    Public Enum Executor
        None                = 0
        InternalPowderCoat  = 1
        ExternalCoating     = 2
        ExternalOther       = 3
        InternalLacquer     = 4
        Galvanize           = 5
    End Enum
    ...
End Class
```

## Why this is a business rule

The Executor value drives:

- **Where the part physically moves**: internal stays at JAZO; external 2/3/5 triggers an outbound dispatch (and an inbound on return).
- **What process records exist**: only internal paths populate `T_CoatingLayerThickness` (since external partners don't feed iCenter back, presumably).
- **Which cost center bears the time**: internal hours register in JAZO time-reg; external work bills as a purchase.
- **Whether quality data is captured**: internal lines capture layer-thickness measurements ([[../modules/isah-sub-services|`UpdateProdLeadTimeHandler` hardcoded for M38 + CoatingLayerThickness query]]).

A misrouted Executor → wrong physical movement, wrong cost attribution, missing or fabricated quality data. **The 5 codes are likely the foundation of the entire JAZO surface-treatment audit trail.**

## Triggers / when it fires

- Anywhere `Coating.Executor` is read/written. Phase-4 follow-up to enumerate full consumer list.

## Effects

- **Internal paths (1, 4)** → JAZO machines clock the time, `T_CoatingLayerThickness` gets entries.
- **External paths (2, 3, 5)** → outbound shipment created; inbound expected on return; external invoice triggered.
- **`None (0)`** → no coating step in the BOO at all.

## Edge cases

- A new external coating partner can be added as a JAZO process change but would re-use Executor=2 or =3 — the enum doesn't model individual partners; that's done at the vendor / part-code level downstream.
- `InternalLacquer (4)` is distinct from `InternalPowderCoat (1)` — JAZO operates **two distinct internal lines**. SME should confirm.
- **`Galvanize (5)`** vs **`ExternalCoating (2)`** — both external, but galvanizing is structurally distinct (zinc-coat vs paint/powder), needs different prep/handling. Probably routed to specific vendor (EBTV per [[icenter-coating-dept-codes]]).
- `Executor` is independent of the `CoatingSystem` field (e.g., "RAL 9006 powder-coat") — the system is the *spec*, the executor is the *destination*.

## Safety classification

- [x] Touches physical process — yes (drives material routing).
- [x] Drives cost / pricing — yes (internal vs external cost center).
- [ ] Reversible if wrong? — yes, in principle (rebook), but **physical material may already have been processed** by the wrong path.
- [ ] Blocks production if it fails? — depends on the misroute.
- `#safety-relevant` because the value is the **routing decision** for the surface-treatment step.

## SME questions

- **Q-348 (new):** Confirm the 5 Executor values are exhaustive — no need for `InternalAnodize` or `ExternalGalvanize` distinctions.
- **Q-349 (new):** Enumerate every reader and writer of `Coating.Executor`. Are there enums-by-string anywhere that should use this enum?
- **Q-350 (new):** What sets the Executor in the first place — a part-master attribute, a config-rule, an operator choice?

Logged in [[../needs-review/_index]].

## Related

- [[../modules/icenter-coating|`iCENTER\Classes\Coating\`]] — defining module.
- [[icenter-coating-dept-codes|Coating dept codes (JCOA/JALU/JSTL/EBTV)]].
- [[../modules/isah-sub-services|`UpdateProdLeadTimeHandler M38`]] — coating-side lead-time tracker.
