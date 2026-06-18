---
type: business-rule
title: "Default surcharge code EX10 (Icenter2Isah)"
status: needs-review
module: "ICenterLib/ISAH"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\Icenter2Isah.vb"
last-reviewed: ""
tags: [business-rule, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Default surcharge code — `EX10`

## Rule

When iCenter's sync layer (`Icenter2Isah`) is given a job to push to ISAH without an explicit `SurChargeCode`, it falls back to the constant **`"EX10"`** as the default. Every iCenter-generated purchase document / part-line that doesn't carry a specific surcharge code is tagged `EX10` in ISAH.

## Where it lives

- File: `ICenterLib\ISAH\Icenter2Isah.vb` line 25
- Symbol: `Public Const DefaultSurChargeCode As String = "EX10"`

## The code (minimal quote)

```vb
Public Const DefaultSurChargeCode As String = "EX10"
```

## Why this is a business rule

`EX10` is a JAZO-internal **surcharge code** that drives cost-accounting and price-uplift rules on ISAH side. The corresponding row in `T_Surcharge` defines the per-cost-component multipliers (part / oper / tool / ext-oper / total surcharge percentages and amounts) iCenter-originated lines inherit.

If `EX10` ever gets renamed or deleted in ISAH, every iCenter→ISAH sync that doesn't supply an explicit code starts failing the SP call (or worse, writes lines with an unrecognised code).

## Triggers / when it fires

- Anywhere `Icenter2Isah.SIP_Get_PartsAndOpers(..., SurChargeCode)` is called with `Nothing` / empty as the SurChargeCode parameter. Phase-3 follow-up to enumerate callers.

## Effects

- `T_PurchaseDocument` / `T_ProdBillOfSurcharge` rows carry `SurChargeCode = "EX10"`.
- Downstream ISAH cost-rollup uses the `EX10` row from `T_Surcharge` to compute totals.

## Edge cases / known exceptions

- **`Public Const`**: inlined at compile time. Caller assemblies built against an older value would keep using it.
- No validation that `EX10` actually exists in `T_Surcharge` at call time — bad rename → silent failures.

## Safety classification

- [ ] Touches physical process — no.
- [x] Drives cost / pricing → `#needs-review` (financial-correctness concern).
- [ ] Reversible if wrong? — Yes (correct the rows post-hoc).
- [ ] Blocks production if it fails? — Indirectly via cost reporting.

## SME questions

- **Q-209 (new):** Document the `EX10` surcharge row in `T_Surcharge` — what multipliers does it apply? Is it likely to ever be renamed?

Logged in [[../needs-review/_index]].

## Related

- [[../modules/isah-icenter-to-isah|`Icenter2Isah`]] — defining module.
- [[isah-production-hierarchy|`PBOS`]] — the per-job surcharge lines.
- [[../mocs/icenterlib-isah]] — parent MOC.
