---
type: business-rule
title: "Track-operation pattern (TR\\d\\d)"
status: needs-review
module: "ICenterLib/ISAH"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\MachGrp.vb"
last-reviewed: ""
tags: [business-rule, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Track-operation pattern — `TR\d\d`

## Rule

A `MachGrpCode` matching the regex **`TR\d\d`** (e.g. `TR01`, `TR02`, … `TR99`) is classified as a **"track operation"** by `MachGrp.IsTrackOperation`. The classification is a per-instance read-only property — any consumer can ask "is this machine-group a track-op?" without a database round-trip.

Phase-3 follow-up is required to enumerate what behaviour is gated on `IsTrackOperation = True` (Q-145). The naming "track" likely refers to **operation-tracking** — a sub-class of operations that participate in iCenter's operation-tracking workflow (timestamped, traced).

## Where it lives

- File: `ICenterLib\ISAH\MachGrp.vb` lines 8, 103–107
- Symbols: `TrackOperationRegex As Regex = New Regex("TR\d\d")` and `IsTrackOperation As Boolean`

## The code (minimal quote)

```vb
Public ReadOnly Property TrackOperationRegex As Regex = New Regex("TR\d\d")

Public ReadOnly Property IsTrackOperation As Boolean
    Get
        Return TrackOperationRegex.Match(MachGrpCode).Success
    End Get
End Property
```

## Why this is a business rule

The `TR` prefix is a **JAZO convention** for machine groups participating in operation tracking. Renaming machine groups to/from `TR##` codes silently flips them in or out of the tracked-operations workflow.

The classification doesn't drive any visible behaviour in `MachGrp.vb` itself — it's a flag for downstream consumers. Until Q-145 is answered, the rule is documented as "this is how `TR##`-coded machine groups are identified" without claiming to know the consequences.

## Triggers / when it fires

- On-demand: each call to `IsTrackOperation` runs the regex match.
- Inputs: the instance's `MachGrpCode` property.

## Effects

Defined by callers. Phase-3 follow-up via `grep IsTrackOperation` to enumerate.

## Edge cases / known exceptions

- **Exactly two digits.** `TR1` (one digit) doesn't match; `TR123` (three digits) matches `TR12` as a prefix because `Regex.Match` finds a match *anywhere* — but the consumer code uses `.Success`, so `TR123` would also report true. Whether that's intentional depends on whether `TR###` codes exist; probably not. Q-147 — should the regex be anchored (`^TR\d\d$`)?
- **Case sensitive by default.** `tr01` does not match. `Regex.Match` is case-sensitive unless `RegexOptions.IgnoreCase` is passed. The convention is uppercase TR, so probably fine.
- **No `^` or `$` anchors.** Any string *containing* `TR\d\d` matches — even `MYTR01XYZ`. MachGrpCodes are typically short ID strings so collisions are unlikely.

## Safety classification

- [ ] Touches physical process — depends on what consumers do with the flag.
- [x] Classifies machine groups for downstream behaviour → `#needs-review`
- [ ] Reversible if wrong? — Yes (rename or re-tag).
- [ ] Blocks production if it fails? — Unknown until Q-145 resolves.

## SME questions

- **Q-145** (from MOC): document downstream consequences of `IsTrackOperation = True`. What does a "track op" trigger?
- **Q-147 (new):** Should `TrackOperationRegex` be anchored (`^TR\d\d$`) to reject substring matches?

Logged in [[../needs-review/_index]].

## Related

- [[../modules/isah-machgrp]] — defining module.
- [[../mocs/icenterlib-isah]] — parent MOC.
