---
type: module
title: "AluGeneral replacement — Elumatec catch-all"
status: needs-review
module: "iCENTER/Elumatec/Works/Replacements"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\Works\\Replacements\\AluGeneral.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `AluGeneral.vb` — catch-all aluminium replacements

> **Status: overview only.** 119 KB / ~3 000 lines / dozens of profile-specific branches. This page describes the *shape* and the *most surprising bits*; individual rules will become their own business-rule notes during Phase 4.

## Purpose

The **last** replacement registered in `Sbz140Alu.New` / `Sbz141Alu.New`, so it runs after all the specific replacements (Flowdrill, DoublePnotch, LargeRectangle, …) have had their say. One giant `Select Case BIdentNo` with profile-specific rewrites that didn't merit a dedicated class.

In practice this file holds 5+ years of accumulated production-floor exceptions. Every `Case "100xxx"` block is a "this profile needed a special tweak" story.

## Public surface

```vb
Public Class AluGeneral
    Inherits WorksReplacement
    Public Sub New(oMachine As Machine.Sbz14x)
    Public Overrides Sub ApplyTo(oCut As Cut, BIdentNo As String)
End Class
```

Just the `ApplyTo` override and a private helper (`IsStandardDoorBoltHole`) plus likely many other private helpers (Phase 3 follow-up — enumerate `Private Function` / `Private Sub` declarations).

## Structure

`ApplyTo` consists of:

1. Setup buffers (`DeleteWorks`, `NewWorks`, `ReplaceDict`, `oProfile`).
2. A massive **`Select Case BIdentNo`** dispatch. Each case has multiple sub-rules. Confirmed branches from reading the first ~120 lines:

| `BIdentNo` | Sub-rules sketched |
|------------|--------------------|
| `"100142"` (P-profile) | Disable auto-drill conversion on circles at `Y = -47` or `Y = -46.5` (drill breaks easily due to a nokje / nub). |
| `"100180"` (Koker 40×20×2) | Deactivate "ontwateringsgaten" (drainage holes) at hard-coded `X = 15` or `X = (CLength − 15)`, `Y = -20`, `Depth = 2`, `WW1 = 10`, on Top or Bottom. |
| `"100114"`, `"100105"`, `"100143"`, `"100292"`, `"100293"`, `"100294"`, `"100115"`, `"100104"` (door planks + door frame NS hinge holes) | Find pairs of `StandardDoorBoltHole`s; pair them, deactivate, mark `Replaced + SuppressedByConverter`. |
| Within `"100143"` / `"100114"` | "Scharnier-/duimgaten" (hinge / thumb holes) — search for Rectangles with `Width ∈ [8.5, 8.54]` and `WW3 = 1.5`; if `Length = 51.5` directly replace; otherwise correct misrecognised rectangles by splitting them in two with adjusted `WX1`. |

…and many more cases (3 000 lines total). The pattern is consistent: dispatch on profile → identify feature geometry → mutate.

3. Apply buffers (`DeleteWorks` / `NewWorks` / `ReplaceDict`) to `oCut.Works`.

## Surprises

1. **Negative tests pile up.** Many branches have a `If MyWork.Replaced Then Continue For` skip — meaning `AluGeneral` runs against features that earlier replacements have already touched, and relies on the `Replaced` sticky-flag (see [[elumatec-work-base]]) to avoid re-rewriting them.
2. **Hard-coded geometric constants are everywhere.** `Y = -47`, `X = 15`, `Y = -20`, `Depth = 2`, `WW1 = 10`, `Width ∈ [8.5, 8.54]`, `WW3 = 1.5`, `Length = 51.5`. Each maps to a specific physical feature on a specific profile. None are named.
3. **Recovery from EluCad mis-recognition** — the `100114` hinge-rectangle correction block (lines 91–119) explicitly splits a misrecognised rectangle in two and adds `WComment = "Toegevoegd door iCenter"` ("Added by iCenter"). This is forensic glue — the comment is intentionally distinctive so production can identify auto-corrections.
4. **`IsStandardDoorBoltHole(work, cut, otherWork)`** is the only named pattern-detection helper visible. Almost everything else is inline.
5. The first ~120 lines are a tiny fraction of the file. Expect: many more cases, possibly with cross-profile branches, and at least a handful of private helpers.

## Business rules

Everything in this file is a business rule. Phase 4 needs a dedicated `business-rules\alu-general-<BIdentNo>.md` per branch — easily 20+ notes once enumerated. Recommended approach:

1. First, enumerate every `Case "1000xx"` in the file (`grep -n 'Case "1' AluGeneral.vb`).
2. For each `BIdentNo`, write one business-rule note with the SME-friendly profile name, the geometric inputs, and the effect.
3. Cross-link from [[../mocs/elumatec-works|the Works MOC]] BIdentNo table.

## Open questions

- **Q-046 (new):** Enumerate every `Case "1*"` branch in `AluGeneral` and create one business-rule note per. Estimated ~20 cases. `#safety-relevant`
- **Q-047 (new):** The `Y = -47` / `Y = -46.5` "nokje breekt" magic numbers — confirm the breakage failure mode (tool, drill, or workpiece?) and document. `#safety-relevant`
- **Q-048 (new):** Hard-coded `xValue = 15` and `yValue = -20` for `100180` Koker drainage holes — confirm with SME these match the live profile drawings. `#safety-relevant`

Logged in [[../needs-review/_index]].

## Related

- [[elumatec-works-replacement-base]] — base class.
- [[../mocs/elumatec-works|Elumatec Works MOC]] — replacement catalogue.
- [[elumatec-work-base|Work base class]] — `Replaced` sticky-flag semantics.

## Coverage

`_coverage.md`: `Elumatec\Works\Replacements\AluGeneral.vb` → `needs-review` (overview only; per-branch notes pending).
