---
type: business-rule
title: "ProductionHeader dossier-code format (PD\\d{8})"
status: needs-review
module: "ICenterLib/ISAH"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\ProductionHeader.vb"
last-reviewed: ""
tags: [business-rule, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# ProductionHeader dossier-code format — `PD\d{8}`

## Rule

Every ISAH `ProdHeaderDossierCode` is **exactly 10 characters**: a fixed prefix `PD` followed by 8 digits. Validated by `ProductionHeader.IsProdHeaderDossierCode(value)` via the regex `PD\d{8}`. Examples: `PD00012345`, `PD99999999`.

## Where it lives

- File: `ICenterLib\ISAH\ProductionHeader.vb` lines 10–12

## The code (minimal quote)

```vb
Public Shared Function IsProdHeaderDossierCode(ByVal Value As String) As Boolean
    Return Regex.IsMatch(Value, "PD\d{8}")
End Function
```

## Why this is a business rule

iCenter uses this format check to **discriminate between possible identifiers** — operators paste in OrdNr / QuotNr / ProdHeaderDossierCode / ShopDocCode / IPpart numbers, and several search-style features need to know which kind of identifier they got. The `PD<8 digits>` shape is the unambiguous signature of a production-header.

Format consequences:
- **Maximum 100 million ProductionHeaders** in JAZO's ISAH life (8 digits). Currently nowhere near.
- Any rename of the prefix (e.g. `PROD-` instead of `PD`) breaks this validator and every consumer that pattern-matches against ISAH dossier codes.
- A future second JAZO entity (cf. [[isah-company-codes|JAZO vs FlowGrill]]) using a different prefix would not be recognised. Q-166.

## Triggers / when it fires

- Anywhere code needs to verify "is this a production-header dossier code?" before proceeding. Phase-3 follow-up via Grep on `IsProdHeaderDossierCode` to enumerate callers.

## Effects

- Returns Boolean. Caller decides what to do.

## Edge cases / known exceptions

- **Regex unanchored.** `Regex.IsMatch(value, "PD\d{8}")` returns true for *any* string *containing* `PD<8 digits>` — e.g. `xPD12345678y` matches. Q-167 — should be `^PD\d{8}$` to enforce exactly 10 chars.
- **Case sensitive.** `pd12345678` does *not* match. The convention is uppercase PD.
- **Allows leading zeros.** `PD00000001` matches and is presumably the first ProductionHeader in ISAH's life.

## Safety classification

- [ ] Touches physical process — no.
- [x] Drives identifier classification → `#needs-review` (mis-classification could silently route to the wrong lookup).
- [ ] Reversible if wrong? — Yes.
- [ ] Blocks production if it fails? — No.

## SME questions

- **Q-166 (new):** A future second JAZO entity (cf. [[isah-company-codes|JAZO vs FlowGrill]]) using a different prefix would not be recognised. Confirm `PD` is universal across all current and future entities.
- **Q-167 (new):** Regex `PD\d{8}` is unanchored. Should be `^PD\d{8}$` to reject substrings.

Logged in [[../needs-review/_index]].

## Related

- [[../modules/isah-production-hierarchy]] — defining module.
- [[../mocs/icenterlib-isah]] — parent MOC.
- [[isah-track-operation-pattern]] — sister rule (regex on MachGrpCode).
