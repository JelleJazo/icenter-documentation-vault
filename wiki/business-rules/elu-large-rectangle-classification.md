---
type: business-rule
title: "Large-rectangle classification → WBroach = 1"
status: needs-review
module: "iCENTER/Elumatec/Works"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\Works\\Rectangle.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\app.config"
last-reviewed: ""
tags: [business-rule, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Large-rectangle classification → broach in one pass

> **⚠️ Important distinction:** there are **two separate "large rectangle" rules** in the Elumatec subsystem that look similar but do different things.
>
> | This page | Companion |
> |-----------|-----------|
> | `Rectangle.SetWBroach` | `LargeRectangle.ApplyTo` |
> | **all profiles** | only profiles 100116 / 100285 / 100103 |
> | thresholds in `app.config` (`260 × 20`) | hard-coded (`> 200 × > 20`) |
> | sets `WBroach = 1` flag on the same Rectangle | **replaces** Rectangle with a FreeForm polyline |
> | [[elu-large-rectangle-classification|this rule]] | [[elu-largerect-freeform-replacement]] |
>
> A rectangle on a door-needle profile that is 250 × 21 mm fires *only* the FreeForm-replacement rule (not the broach classifier). A rectangle on any other profile that is 300 × 21 mm fires *only* the broach classifier.

## Rule

A rectangular pocket on an Elumatec profile-mill workpiece is classified as **"large"** (and machined as a single broach instead of contoured) when its **length ≥ 260 mm AND its width ≥ 20 mm**. The threshold is encoded in `app.config` settings `EluLargeRectangleMinLength` and `EluLargeRectangleMinWidth`. Applies to **all** profiles.

## Where it lives

- File: `Elumatec\Works\Rectangle.vb`
- Symbol: `Rectangle.SetWBroach(oTools As DataTable)` (line 60–94)

## The code (minimal quote)

`Works\Rectangle.vb` line 80–93:

```vb
''Zeer grote rechthoeken altijd volledig ruimen     ' "Always broach very large rectangles"
If Me.WBroach = 0 Then
    ''Lengte >   en breedte > 
    Try
        Dim MinLength As Decimal = Decimal.Parse(My.Settings.Properties("EluLargeRectangleMinLength").DefaultValue)
        Dim MinWidth  As Decimal = Decimal.Parse(My.Settings.Properties("EluLargeRectangleMinWidth").DefaultValue)
        If Me.WW1 >= MinLength And Me.GetWorkWidth >= MinWidth Then
            Me.WBroach = 1
        End If
    Catch ex As Exception
        oApplicationLog.NewEntry(ex.ToString, MsgBoxStyle.Critical)
    End Try
End If
```

`app.config` line 263–266:

```xml
<setting name="EluLargeRectangleMinLength" serializeAs="String"><value>260</value></setting>
<setting name="EluLargeRectangleMinWidth"  serializeAs="String"><value>20</value></setting>
```

## Why this is a business rule

`WBroach` is a binary flag on the rectangle feature:

- `WBroach = 0` → contour the rectangle with a tool path that follows the rectangle's outline. Small/medium rectangles, or rectangles with a tool ≥ 60 % of the smallest side, take this path.
- `WBroach = 1` → broach it (one-shot machining of the whole rectangle volume).

The earlier branch of `SetWBroach` (lines 60–78) already sets `WBroach = 1` for a few standard cases: no tool ID, tool diameter < 60 % of the smallest rectangle side, or the depth-post is auto-set. The block quoted above adds a **safety override**: even if the earlier branch decided to contour, if the rectangle is bigger than the threshold it gets broached anyway.

The rule encodes a process choice: large pockets, when contoured, would generate very long tool paths with risk of tool wear, residual webs, or unfinished corners; the factory prefers to broach them in one operation.

## Triggers / when it fires

- Called during NC-program generation, for each `Rectangle` feature on the workpiece.
- Inputs:
  - `Me.WW1` — rectangle's primary dimension (likely length along bar axis).
  - `Me.GetWorkWidth` — rectangle's secondary dimension.
- The other call paths in `SetWBroach` consult the tool DB (`oTools.Rows.Find(WToolID)`); this large-rect override does not.

## Effects

- Sets `WBroach = 1` on the in-memory rectangle. Downstream NC-export code uses `WBroach` to decide which toolpath template to emit, so this flips the machining strategy for the feature.

## Edge cases / known exceptions

- **Hard-coded `AND`** — both length and width must clear their thresholds. A 1 000 × 15 mm pocket would stay contoured. SME-confirm this matches intent. `#needs-review`
- **Order of evaluation**: the block runs *only if* the prior branch chose `WBroach = 0`. If the tool-diameter rule already chose to broach, this override is a no-op. Documenting just so it's clear the rule cannot *cancel* a broach decision, only force one.
- **`Decimal.Parse` without `IFormatProvider`** — same culture concern as in [[elu-max-step-depth|MaxStepDepth]]. Likely fine on Dutch / en-US.
- **Any parse exception is logged at `Critical`** but otherwise swallowed — meaning if the setting were ever malformed, the rule would silently revert to whatever the prior branch decided. `#needs-review`

## Safety classification

- [x] Touches physical process / setpoint / interlock / safety → `#safety-relevant`
- [ ] Reversible if wrong? — Yes (edit `app.config`, redeploy), but lossy: contoured rectangles that should have been broached can leave residual material.
- [ ] Blocks production if it fails? — No (graceful fallback to contour) but produces wrong output.

## SME questions

- **Q-032 (new):** Confirm 260 × 20 mm thresholds are correct for *all four* Elumatec variants (ALU, STL, RVS, SBZ141). The rule isn't material-aware.
- **Q-033 (new):** Should the rule be `OR` instead of `AND`? A 1 m × 15 mm slot is "long" enough to suffer the same contouring problems as a 260 × 25 mm pocket.

Logged in [[../needs-review/_index]].

## Related

- [[../modules/elumatec-machine-base|`Sbz14x` family]] (Rectangle is called from the machine-specific pipeline)
- [[elu-max-step-depth]] — companion threshold rule
- [[elu-largerect-freeform-replacement]] — the *other* large-rectangle rule (profile-restricted, FreeForm replacement)
- [[../mocs/elumatec|Elumatec subsystem MOC]]
- [[../mocs/elumatec-works|Works MOC]]
