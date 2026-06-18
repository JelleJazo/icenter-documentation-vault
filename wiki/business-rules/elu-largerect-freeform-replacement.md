---
type: business-rule
title: "Large rectangle → FreeForm (door-needle profiles)"
status: needs-review
module: "iCENTER/Elumatec/Works/Replacements"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\Works\\Replacements\\LargeRectangle.vb"
last-reviewed: ""
tags: [business-rule, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Large rectangle → FreeForm replacement (door-needle profiles)

## Rule

For three specific aluminium profiles only — `BIdentNo` in `{"100116", "100285", "100103"}` (the door-needle front-plate "deurnaaldprofiel" and the "opdekje" variants) — every `Rectangle` feature with **length > 200 mm AND width > 20 mm** is replaced by a **`FreeForm`** polyline that traces the same outline. Two shape variants:

- If `WW3 = ToolDiameter / 2` (sharp corners): a 5-point FreeForm.
- Otherwise (rounded corners): a 10-point FreeForm with arc segments at each corner.

Other profiles, or rectangles below the threshold, are **not** affected by this rule. (They may still be subject to the [[elu-large-rectangle-classification|SetWBroach rule]] for the `WBroach` flag — that's a different mechanism.)

## Where it lives

- File: `Elumatec\Works\Replacements\LargeRectangle.vb`
- Symbol: `LargeRectangle.ApplyTo(oCut As Cut, BIdentNo As String)` (lines 30–193)

## The code (minimal quote)

`LargeRectangle.vb` lines 39–49:

```vb
If Not (BIdentNo = "100116" Or BIdentNo = "100285" Or BIdentNo = "100103") Then  'Alleen nog maar voor deurnaaldprofiel en opdekje
    Exit Sub
End If

Dim oProfile As New Profile(BIdentNo)
oProfile.ReadFromDatabase(Elumatec.EluCadApp.ProfileDbFile)

For Each Work As Work In oCut.Works
    If TypeOf Work Is Rectangle Then
        Dim oRectangle As Rectangle = CType(Work, Rectangle)
        If oRectangle.GetWorkWidth > 20 And oRectangle.WW1 > 200 Then
            ''Rechthoek groter dan de minimale afmetingen, dus vervangen
```

## Why this is a business rule

The door-needle ("deurnaald") front plate sits in a specific assembly position where a contoured (rectangle) cut leaves residual material or has poor surface finish. The FreeForm path runs the tool around the rectangle's outline as a controlled-direction polyline — with arcs in the rounded-corner case — giving a smoother finish and reliable corner machining.

Choosing not to apply this for other profiles is intentional: the FreeForm-via-EluCad round-trip carries its own export bug (see comment in source: _"Via EluCad doorsturen ivm foute NCW export in iCenter"_), and applying it universally would slow down production unnecessarily.

## Triggers / when it fires

- Runs once per `Cut` once per program, when registered into the machine's `WorksReplaceList` — currently only Sbz140Alu / Sbz141Alu.
- Inputs: rectangle features on profiles 100116 / 100285 / 100103, length > 200 mm AND width > 20 mm. Tool assignment must succeed.

## Effects

- Original Rectangle: `WActive = 0`, added to `DeleteWorks` (removed after the loop).
- New FreeForm Work added to `AddWorks`. Its `Replaced = True` and `WComment` includes either `"Vervanging ivm grote rechthoec"` (sic, sharp branch) or `"Via EluCad doorsturen ivm foute NCW export in iCenter"` (rounded branch).
- Downstream: NC export emits a FreeForm contour rather than a Rectangle pocket — different toolpath, different cycle time, different finish.

## Edge cases / known exceptions

- **Profile allowlist is exclusive** — adding a profile means a code change to `LargeRectangle.ApplyTo`. There is no `app.config` setting for it.
- **Hard-coded `>200 × >20` thresholds.** Do *not* match `EluLargeRectangleMinLength = 260` / `EluLargeRectangleMinWidth = 20` in `app.config`. See sibling rule [[elu-large-rectangle-classification]] and **Q-044** about whether these should be aligned.
- **Tool-assignment failure** (`WToolID = ""`) ⇒ rectangle keeps `WActive = 0` with comment `"Nog vervangen door vrije vorm"` ("Still to be replaced by free form"). The Rectangle isn't deleted, no FreeForm is added — silent partial failure. `#needs-review`
- **Rounded-corner FreeForm path** had `WActive = 0` in an earlier iteration (see commented block lines 141–175) because the NCW export was buggy. Currently `WActive = 1` — the path is presumed fixed but the legacy comment remains. `#needs-review`
- **The 5-point sharp branch's final point** is `(5, -(width - tool)/2)`; the 10-point rounded branch's final is `(10, ...)`. The 5 vs 10 mm exit-distance is unexplained (Q-045).
- **`SimpleRotationPossible(oProfile)`** — semantics undefined here; lives in `Rectangle.vb`. Affects which way the FreeForm is oriented.

## Safety classification

- [x] Touches physical process → `#safety-relevant`
- [ ] Reversible if wrong? — Once cut, no.
- [x] Blocks production if it fails? — If tool-assign silently fails, the bar ships with a deactivated rectangle (i.e. *no* pocket). Operator must catch this manually.

## SME questions

- **Q-044**: align the two large-rectangle thresholds (200/20 here vs 260/20 in app.config)?
- **Q-045**: 5 vs 10 mm exit-point distance — what does the difference mean?
- **Q-049 (new)**: tool-assign silent failure (`WToolID = ""`) — should this raise an alert instead of producing a deactivated rectangle? `#safety-relevant`

Logged in [[../needs-review/_index]].

## Related

- [[../modules/elumatec-replacement-large-rectangle]] — module walkthrough.
- [[elu-large-rectangle-classification]] — companion rule using the `app.config` thresholds, all profiles.
- [[../mocs/elumatec-works]] — replacement catalogue.
