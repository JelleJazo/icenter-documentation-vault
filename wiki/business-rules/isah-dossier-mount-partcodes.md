---
type: business-rule
title: "Mount-detail part-codes (DossierMain.GetMontDetailCode)"
status: needs-review
module: "ICenterLib/ISAH"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\DossierMain.vb"
last-reviewed: ""
tags: [business-rule, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Mount-detail part-codes — hard-coded set of 4

## Rule

`DossierMain.GetMontDetailCode()` returns all `T_DossierDetail` rows for this dossier whose `PartCode` is one of **four fixed values**:

| PartCode | Likely meaning |
|----------|----------------|
| `MONTAGE TP` | mounting TP (turn-key partial?) |
| `090` | (numeric code — unknown meaning) |
| `PLAN EXT MONT` | planning external mounting |
| `CSA00011` | (article code — unknown meaning) |

Anything outside this set is *not* returned by this method. The query is hard-coded — adding a fifth mount-detail part-code requires a source change.

Two further plan-line constants live on `DossierMain` at class level:

- `PlanIntMonPartCode = "PLAN INT MONT"` — internal mounting plan line
- `PlanTekWvBPartCode = "PLAN TEK WVB"` — drawing-WvB (werkvoorbereiding) plan line

These are exposed as `Public Const` so other code can reference them by name (rather than retyping the string).

## Where it lives

- File: `ICenterLib\ISAH\DossierMain.vb`
- Symbol: `Public Function GetMontDetailCode() As DataTable` (lines 437–466)
- Plan-line constants: lines 7–8

## The code (minimal quote)

```vb
Public Const PlanIntMonPartCode As String = "PLAN INT MONT"
Public Const PlanTekWvBPartCode As String = "PLAN TEK WVB"

' ...

Public Function GetMontDetailCode() As DataTable
    ...
    sSQL = "SELECT DM.OrdNr,
                   DD.DetailCode,
                   DD.DossierCode,
                   DD.DetailSubCode,
                   DD.PartCode,
                   DD.Description
            FROM dbo.T_DossierMain DM
            INNER JOIN T_DossierDetail DD ON DM.DossierCode = DD.DossierCode
            WHERE PartCode IN ('MONTAGE TP','090','PLAN EXT MONT','CSA00011')
              AND DM.DossierCode = @DossierCode"
    ...
End Function
```

## Why this is a business rule

iCenter's mounting workflow (`MONTAGE` in factory speak) attaches to specific JAZO-internal part codes that mark a dossier-detail line as a mounting line. The four codes in `GetMontDetailCode` are the **closed set** iCenter currently recognises.

Adding a new mount-related part-code in ISAH without also adding it here means the new code is invisible to iCenter's mount handling — the dossier-detail line will exist but iCenter's mounting-aware features won't pick it up.

The two plan-line constants (`PLAN INT MONT`, `PLAN TEK WVB`) are referenced elsewhere as `DossierMain.PlanIntMonPartCode` etc. (Phase-3 follow-up on callsites). They represent different *kinds* of plan-line distinct from mount-detail lines.

## Triggers / when it fires

- `GetMontDetailCode()` runs whenever a caller asks for the mounting lines of a dossier. Phase-3 follow-up via Grep to enumerate callsites.

## Effects

- Returns the matching rows (or empty DataTable). Downstream code branches on whether the dossier has any.

## Edge cases / known exceptions

- **Exactly these four part codes.** Add `PartCode IN` member → invisible. Remove one → invisible. Both require source edits.
- **String concat in SQL** — these are literals, not user input. Safe.
- **No `Common.IsPcfAdmin`-style permission check** — any user able to read the dossier sees its mount detail.

## Safety classification

- [ ] Touches physical process — no.
- [x] Affects workflow surfacing → `#needs-review` (the operator-visible "mount detail" view depends on this).
- [ ] Reversible if wrong? — Yes (source change + redeploy).
- [ ] Blocks production if it fails? — No (silent — features just don't fire on un-listed codes).

## SME questions

- **Q-156 (new):** Provide SME-friendly meanings for the four part codes (`MONTAGE TP`, `090`, `PLAN EXT MONT`, `CSA00011`). Is the set complete today?
- **Q-157 (new):** Why are `PlanIntMonPartCode` and `PlanTekWvBPartCode` exposed as constants while the four mount-detail codes are inlined into the SQL? Should they be unified into a shared registry?

Logged in [[../needs-review/_index]].

## Related

- [[../modules/isah-dossier]] — defining module.
- [[../mocs/icenterlib-isah]] — parent MOC.
- [[../mocs/office-to-shopfloor]] — mounting workflows touch the office-to-shop-floor handoff.
