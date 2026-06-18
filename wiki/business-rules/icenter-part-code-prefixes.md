---
type: business-rule
title: "iCenter part-code prefixes (IA, IAK, PRN)"
status: needs-review
module: "ICenterLib/(root)"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\Common.vb"
last-reviewed: ""
tags: [business-rule, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# iCenter part-code prefixes (IA / IAK / PRN)

## Rule

iCenter assigns identifiers to "production" entities using three fixed string prefixes:

| Prefix | Constant | Meaning |
|--------|----------|---------|
| `IA` | `Common.IPPARTPREFIX` | iCenter Production part (= "IA-nummer" in factory speak) |
| `IAK` | `Common.IPPARTCOPYPREFIX` | Copy of an iCenter Production part (`K` = *kopie*) |
| `PRN` | `Common.IPPRNPREFIX` | Production reference number |

These prefixes are concatenated with numeric IDs to form the displayed part / batch / reference identifiers. Example: `OutsourceOperationsHandler` constructs the sticker filename via `IPPARTPREFIX & IPpartId & ".pdf"` → `IA12345.pdf` for IPPartId 12345.

## Where it lives

- File: `ICenterLib\Common.vb` lines 29–31
- Symbols: `Public Const IPPARTPREFIX As String = "IA"`, `IPPARTCOPYPREFIX = "IAK"`, `IPPRNPREFIX = "PRN"`

## The code (minimal quote)

```vb
Public Const APPLISAHUSERCODE As String = "ICENTER"
Public Const IPPARTPREFIX     As String = "IA"
Public Const IPPARTCOPYPREFIX As String = "IAK"
Public Const IPPRNPREFIX      As String = "PRN"
```

## Why this is a business rule

The "IA" prefix is **factory shorthand for an iCenter Production part** and appears on:
- physical stickers (printed via `JPLT_PartIdentSticker` → `.pdf` filename per Q-094 area)
- ISAH purchase-order references (see `OutsourceOperationsHandler.LabelFilename` pattern)
- on-floor operator interfaces (label scans, kanban bins)

Changing a prefix breaks:
- every printed sticker layout that uses the prefix as a visual identifier
- every external system that scans these stickers (Kardex picker, saw COM-watcher, outsourcing vendor exchange files)
- every report / dashboard that filters on "IA*" prefixes

The `IAK` prefix is used by part-copy workflows (Phase-3 follow-up — which code path makes copies and assigns `IAK*` IDs).

## Triggers / when it fires

- Wherever iCenter renders a user-visible IPpart identifier.
- `OutsourceOperationsHandler.ProcessDataByVendIdProfileId` line 338: `DRV.Item("LabelFilename") = Common.IPPARTPREFIX & DRV.Item("IPpartId") & ".pdf"`.
- Phase-3 follow-up: full enumeration of callsites via Grep on `IPPARTPREFIX` / `IPPARTCOPYPREFIX` / `IPPRNPREFIX`.

## Effects

- Determines the textual prefix on every IPpart/IPbatch sticker, label, and reference number.
- Drives ISAH-side reference numbers exposed to vendors and operators.

## Edge cases / known exceptions

- The constants are **`Public Const`** (compile-time inlined). Callers that referenced `Common.IPPARTPREFIX` from an external assembly would inline `"IA"` at compile time. Changing the constant in ICenterLib without recompiling the consuming assembly leaves the consumer with the old value.
- No length validation. If `IPpartId` is a 7-digit number, the concatenated sticker filename is `IA<7 digits>.pdf` — still well under filename-length limits.

## Safety classification

- [ ] Touches physical process — no.
- [x] Touches business identifiers / labels that operators rely on → `#needs-review`
- [ ] Reversible if wrong? — Yes (code change) but disruptive (sticker reprints).
- [ ] Blocks production if it fails? — No.

## SME questions

- **Q-125 (new):** Confirm `IA / IAK / PRN` are the only prefixes the factory recognises. Are there other workflows that should be added?
- **Q-126 (new):** Document the `IAK` (copy) workflow — when is a part copied and who triggers it?

Logged in [[../needs-review/_index]].

## Related

- [[../modules/icenterlib-common]] — defining module.
- [[../modules/workprep-outsource-operations]] — biggest visible consumer (sticker filenames).
- [[../mocs/office-to-shopfloor]] — context for IPpart workflow.
