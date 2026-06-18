---
type: module
title: "LargeRectangle replacement — Elumatec"
status: done
module: "iCENTER/Elumatec/Works/Replacements"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\Works\\Replacements\\LargeRectangle.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `LargeRectangle.vb` — door-needle / opdekje rectangles → FreeForm

## Purpose

For three specific profiles (`BIdentNo` in `{"100116", "100285", "100103"}` — *deurnaaldprofiel* + opdekje variants), replaces `Rectangle` features above a minimum size with **`FreeForm`** polylines. Two variants:

- **Sharp-cornered rectangle** (`oRectangle.WW3 = ToolDiameter / 2`) → simple FreeForm polyline (5 relative points).
- **Rounded-corner rectangle** → extended FreeForm with arc segments at each corner (10 relative points including arcs).

> **Do not confuse this with [[../business-rules/elu-large-rectangle-classification|the `EluLargeRectangle*` SetWBroach rule]].** They are *different* rules using *different* thresholds on the *same* features:
> | Rule | Where | Profile scope | Length threshold | Width threshold | Effect |
> |------|-------|---------------|-----------------:|----------------:|--------|
> | `Rectangle.SetWBroach` | `Works\Rectangle.vb` | **all** profiles | `EluLargeRectangleMinLength` (260) | `EluLargeRectangleMinWidth` (20) | sets `WBroach = 1` |
> | `LargeRectangle.ApplyTo` | this file | only 100116 / 100285 / 100103 | hard-coded `> 200` | hard-coded `> 20` | replaces Rectangle with FreeForm |
> Both fire for the door-needle profiles; only the broach rule fires for other profiles.

## Public surface

| Symbol | Value |
|--------|-------|
| `Description` | `"Voorplaat deurnaald en opdekje"` ("Door-needle front plate and opdekje") |
| `Name` | `"Voorplaat deurnaald en opdekje"` (same string) |
| `ApplyTo(oCut, BIdentNo)` | mutates `oCut.Works` |

## Behavior in plain language

1. Guard: if `BIdentNo NOT IN {"100116", "100285", "100103"}` → `Exit Sub`. (Note: the commented line `''Or BIdentNo = "100103" Or BIdentNo = "100112"` shows previous iterations of the allowlist; the live code includes 100103 but excludes 100112.)
2. Read the profile from the `.epd` DB.
3. For each Rectangle in `oCut.Works` where `GetWorkWidth > 20 AND WW1 > 200`:
   a. If the Rectangle can be simply rotated bottom-to-top (`SimpleRotationPossible`), do so.
   b. `AssignTool(oMachine.ToolDb, oProfile, False, False)`. If still no tool: `WActive = 0`, comment `"Nog vervangen door vrije vorm"` ("Still to be replaced by free form"), continue.
   c. Look up the picked tool's `TDiameter`.
   d. **Branch on corner type** (`WW3 = ToolDiameter / 2` means sharp corners ≤ half the tool diameter, otherwise rounded):
      - **Sharp**: synthesise a 5-point `FreeForm` polyline that traces the rectangle's perimeter, then add a final "exit" point at `(5, -(width - tool)/2)` to leave the pocket. Active. Comment `"Vervanging ivm grote rechthoec"` ("Replacement due to large rectangle").
      - **Rounded**: synthesise a 10-point `FreeForm` with arc segments (`AddRelativePoint` with the 4-arg overload supplies the arc centre). Comment `"Via EluCad doorsturen ivm foute NCW export in iCenter"` ("Send via EluCad due to wrong NCW export in iCenter"). Final exit at `(10, -(width - tool)/2)`.
   e. Add the new FreeForm to `AddWorks`. Mark the original Rectangle for deletion and set its `WActive = 0`.
4. Apply mutations (delete originals, add new FreeForms).

## Surprises

1. **Hard-coded `> 200` length threshold doesn't match `EluLargeRectangleMinLength = 260` in `app.config`.** Same idea, *different* numbers, *different* code path. The rectangle could trigger one rule but not the other depending on its dimensions. The author may have intended these to be the same — or they may genuinely be independent decisions. **`#needs-review`** (Q-044).
2. **The rounded-corner branch's comment** _"Via EluCad doorsturen ivm foute NCW export in iCenter"_ says iCenter's own NCW export of `FreeFormPoint`s is buggy, so the rounded-corner FreeForms must round-trip through EluCad first. **`WActive = 1`** is set (after a *much* longer commented-out earlier version that had `WActive = 0`) — the "buggy" path is now believed fixed enough to ship. `#needs-review`.
3. **The full prior implementation is retained as a commented block** (lines 141–175) — same algorithm but with `WW3 = Work.Direction.Left` (left-handed milling direction) instead of `Center`. CB+SL annotated `"2015-08-28 CB+SL: vrije vorm blijkt altijd gelijk te zijn aan de rechthoek. Dus rechthoek verwijderen"` ("freeform turns out to always equal the rectangle; therefore remove the rectangle"). Useful historical context.
4. **`oRectangle.SimpleRotationPossible(oProfile)`** is called but its semantics aren't visible from this file alone — Phase-3 follow-up on `Rectangle.vb`.
5. **Tool diameter is read again from `dtTools.Rows.Find(WToolID).Item("TDiameter")`** even though the tool was just assigned via `AssignTool`. The redundancy is fine but slightly confusing.
6. **The five/ten "relative points"** in the FreeForm construction trace the rectangle's tool-centre path — they are offset *inward* by `ToolDiameter / 2` from the rectangle's true perimeter. This is correct only if the tool radius equals the desired final corner radius (sharp-corner case). The rounded-corner branch handles non-equal radii via arcs.
7. **The exit-point hack** (`AddRelativePoint(5, ...)` for sharp, `AddRelativePoint(10, ...)` for rounded) is the conventional Elumatec way to "lift the tool" — the difference 5 vs 10 is unexplained in the source. `#needs-review`.

## Business rules surfaced here

- [[../business-rules/elu-largerect-freeform-replacement|Large rectangle → FreeForm for door-needle profiles]] — companion to (but *distinct from*) [[../business-rules/elu-large-rectangle-classification]].

## Open questions

- **Q-044 (new):** Two different "large rectangle" thresholds exist — `>200×>20` here (hard-coded) and `≥260×≥20` in `Rectangle.SetWBroach` (from `app.config`). Are they meant to be the same rule? `#safety-relevant`
- **Q-045 (new):** `AddRelativePoint(5/10, ...)` exit-point — what does the 5 vs 10 mm difference mean? `#needs-review`

Logged in [[../needs-review/_index]].

## Related

- [[elumatec-works-replacement-base]] — base class.
- [[../business-rules/elu-large-rectangle-classification|`SetWBroach` rule]] — the *other* large-rectangle rule.
- [[../business-rules/elu-largerect-freeform-replacement|FreeForm replacement rule]] — companion rule note.
- [[../mocs/elumatec-works]] — replacement catalogue.

## Coverage

`_coverage.md`: `Elumatec\Works\Replacements\LargeRectangle.vb` → `done`.
