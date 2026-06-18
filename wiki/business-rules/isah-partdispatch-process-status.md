---
type: business-rule
title: "PartDispatch ProcessStatus (1=Niet, 2=Klaar, 3=Verwerkt)"
status: needs-review
module: "ICenterLib/ISAH"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\PartDispatch.vb"
last-reviewed: ""
tags: [business-rule, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `PartDispatch.ProcessStatus` — 3-state enum

## Rule

Every row in `T_PartDispatch` carries a `ProcessStatus` integer that controls whether warehouse staff and downstream ISAH workflows act on the dispatch line. iCenter defines exactly three values:

| Code | Enum value | Dutch meaning | Effect |
|-----:|------------|---------------|--------|
| `1` | `NietVerwerken` | "don't process" | dispatch line is recorded but skipped by downstream processing |
| `2` | `KlaarVoorVerwerken` | "ready to process" | the active value when iCenter wants the dispatch picked / processed |
| `3` | `Verwerkt` | "processed" | line has been handled — terminal state |

Any caller of `PartDispatch.IP_Ins_PartDispatch` must supply one of these three values for the `@ProcessStatus` parameter.

## Where it lives

- File: `ICenterLib\ISAH\PartDispatch.vb` lines 6–10

## The code (minimal quote)

```vb
Public Class PartDispatch
    Public Enum ProcessStatus
        NietVerwerken      = 1
        KlaarVoorVerwerken = 2
        Verwerkt           = 3
    End Enum
    ' ...
End Class
```

## Why this is a business rule

`T_PartDispatch.ProcessStatus` drives shop-floor warehouse picking — whether a dispatch row is *visible* to the picker, *picked*, or *historical*. Misclassification means either:

- **NietVerwerken set when the line should be picked** → the part never reaches the shop floor (silent under-supply).
- **Verwerkt set before the pick actually happened** → ISAH's reporting shows complete while the part is missing.
- **KlaarVoorVerwerken set after Verwerkt** → the line re-appears in the pick queue, risk of double-pick.

The transitions between states are not encoded in this enum — iCenter callers (and ISAH's own picking workflows) are responsible for moving rows through the state machine.

## Triggers / when it fires

- `PartDispatch.IP_Ins_PartDispatch(..., @ProcessStatus, ...)` — the only writer in iCenter passes whatever the caller supplied. Phase-3 follow-up via Grep on `ProcessStatus.` to enumerate which values each caller emits.

## Effects

- Persists to `T_PartDispatch.ProcessStatus` column.
- Consumed by ISAH-side workflows (picking queue, inventory reservation, etc.) outside iCenter's scope.

## Edge cases / known exceptions

- **Three-state enum with no `Unknown`.** A caller that passes `0` or `4` bypasses the enum and ends up in ISAH with an undefined state. The SP signature accepts `Integer`, not the enum.
- **Dutch enum names.** Same convention as elsewhere in iCenter (e.g. `Sides` / `SidesNl` in [[../modules/elumatec-work-base|Elumatec Work]]); search-friendly for Dutch operators, less so for English readers.
- **No transition validation.** Nothing in this class enforces `KlaarVoorVerwerken → Verwerkt` direction; `Verwerkt → KlaarVoorVerwerken` re-opening is a SQL write away.

## Safety classification

- [ ] Touches physical process — indirectly (controls whether the picker fetches the part).
- [x] Drives shop-floor workflow visibility → `#needs-review`
- [ ] Reversible if wrong? — Yes (re-set the column).
- [ ] Blocks production if it fails? — Misset values silently break picking workflow.

## SME questions

- **Q-176 (new):** Document the transitions: who moves a row from 2 → 3? Is the 3 → 2 re-open path ever used legitimately?
- **Q-177 (new):** Add `Unknown = 0` to the enum and validate at the writer to reject unknown values?

Logged in [[../needs-review/_index]].

## Related

- [[../modules/isah-part-and-dispatch]] — defining module.
- [[../mocs/icenterlib-isah]] — parent MOC.
- [[isah-partdispatch-collector-filters]] — sibling rule about the dispatch-collector query filters.
