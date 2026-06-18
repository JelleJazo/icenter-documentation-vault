---
type: module
title: "Flowdrill replacement — Elumatec"
status: done
module: "iCENTER/Elumatec/Works/Replacements"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\Works\\Replacements\\Flowdrill.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `Flowdrill.vb` — circles / drilled holes → flow-drill macro

## Purpose

Replaces `Circle` and `Drill` features that meet specific size criteria with a **flow-drill macro** (`EC00601` with countersink, `EC00054` without). Flow-drilling is a heat-forming process that creates a threaded bushing without removing material — used in JAZO's aluminium profiles to make tapped fixings without separate inserts.

Registered as the **first** replacement in `Sbz140Alu.New` (and `Sbz141Alu.New`) — meaning it gets first crack at every circle/drill before the more generic replacements (`DoublePnotch`, `AluHinge`, …, `AluGeneral`).

## Public surface

| Symbol | Returns |
|--------|---------|
| `Description` | `"Bepaalde cirkels vervangen door vloeiboorgat"` (Dutch — "Replace certain circles with flow-drill holes") |
| `Name` | `"Vloeiboor 9.3"` (Dutch — "Flow-drill 9.3") |
| `ApplyTo(oCut, BIdentNo)` | mutates `oCut.Works` in place |

## Behavior in plain language

For each `Work` in the cut:

1. **Match candidate** if `(TypeOf Work Is Circle Or TypeOf Work Is Drill)` AND **any** of:
   - `Work.WW1 = 9.31` (legacy generic detection — diameter ~9.31 mm)
   - `Work.WDepth - Work.MaterialStartDepth > 6` (UDF-based detection: any deep hole)

2. **Profile-specific corrections** for two BIdentNo values:
   - **`BIdentNo = "100142"` (P-profile)**: if the drill was incorrectly recognised from the **rear** side, rotate it back to the front (`Work.RotateFromRearToFront(oProfile)`). Then, if a matching front-side flow-drill exists at the same X, clone its parameters to fix the rear-side gap. If no match is found and `Work.WDepth > 59`, synthesise a Ø8.5 mm hole as "Auto herstel naar 8.5" ("Auto recovery to 8.5").
   - **`BIdentNo ∈ {"100142", "100381"}`**: same recovery pattern but for **bottom→top** rotation (`Work.RotateFromBottomToTop`).

   Each branch has a `DepthAccepted` gate: depth must be > 10 mm, OR the depth table must match one of two specific patterns (`(0)=2, (1)>10, (2).M=1` or `(1)=2.1, (2)>10, (3).M=1`). The `DepthAccepted` heuristic is a defensive check against mis-recognition.

3. **Countersink detection** — search the same side for a `Deburr` feature at the same `(WX1, WY1)`. If found, `HoleIsCountersunk = True`.

4. **Macro selection**:
   - If countersunk: macro `EC00601` (flow-drill with integrated countersink). Controlled by `UseFlowDrillWithCountersink = True` (hard-coded local in `ApplyTo`).
   - If countersunk but `UseFlowDrillWithCountersink = False`: macro `EC00054` (flow-drill without countersink — code path for if the integrated tool is unstable).
   - If **not** countersunk: deactivate the hole — `Work.WToolID = "-" ; Work.WActive = 0` and emit no flow-drill. _The comment says "Wordt nog niet ondersteund" — "not yet supported"_.

5. **Cleanup** — if the macro was applied and a `Deburr` exists at the same (WX1, WY1) on the opposite side, delete it (the flow-drill macro already handles end-side countersinking).

6. **Apply mutations** — swap `ReplaceDict` entries into `oCut.Works`, remove `DeleteWorks`, add `AddWorks`.

## Code annotations worth noting

The file is dense with dated `'CB <date>:` comments documenting individual production bugs and fixes:

| Date | Comment | Implication |
|------|---------|-------------|
| 2021-07-01 | _"tevens verdwijnt het gat aan de achterzijde wat er wel hoort te zitten"_ | mis-recognition of rear-side holes was a real production issue |
| 2021-07-02 | _"soms wordt een gat vanaf achterzijde herkend"_ | same |
| 2022-04-05 | _"sinds EluCad4.1: verkeerd herkende rechthoek"_ | EluCad 4.1 changed feature recognition; iCenter compensates |
| 2022-06-16 | _"added detection on WSide Bottom for these BIdentNo's"_ | bottom-side recovery added |
| 2022-07-08 | _"om een of andere reden gaat de herkenning toch nog niet goed"_ | fallback Ø8.5 mm hole synthesis added — author was not sure why mis-recognition persisted |

→ **This file is a textbook example of compensating glue code**. Every numerical and string constant traces back to a real production incident. Treat any change here with extreme care. `#safety-relevant`.

## Business rules surfaced here

- [[../business-rules/elu-flowdrill-replacement|Flow-drill replacement rule]] — circles/drills of Ø9.31 mm OR deeper than 6 mm in material become flow-drill macros, with profile-specific recovery for misrecognition.

## Surprises

1. **`UseFlowDrillWithCountersink = True`** is a hard-coded local boolean, not an `app.config` toggle. The comment _"Uitgezet omdat het direct verzinken met de vloeiboor niet stabiel is"_ documents a past period when it was off ("turned off because integrated countersinking with the flow-drill is unstable"). The fact that it's local-only means flipping it requires a code change + deploy.
2. **`DeleteCoincidentCountersink = False`** is similarly hard-coded local. The disabled branch would delete a same-side `Deburr` after applying the flow-drill macro. Currently *enabled only* on the opposite side; this is intentional per the inline comment.
3. **The non-countersunk path silently deactivates the hole** (`WActive = 0`) and emits *nothing*. If a profile shipped with a non-countersunk Ø9.3 mm hole that *should* have been flow-drilled, the bar is sent to the machine missing the hole. **`#safety-relevant` / `#needs-review`**.
4. **Equality tests use `=` on `Double`** — `Work.WW1 = 9.31`, `Work.WY1 = -47 Or Work.WY1 = -46.5`. With floating-point comparison this is brittle to upstream changes in recognition precision.
5. **`Magic depth values`** — `> 59`, `= 2`, `= 2.1`, `> 10` in the recovery branches. These come from observation of EluCad's actual depth tables; no SME-friendly names. Each is a candidate for a named constant.

## Open questions

- **Q-041 (new):** Why does the non-countersunk Ø9.3 mm hole get silently deactivated instead of emitted as an ordinary drill? Is there a downstream operator manual-add path? `#safety-relevant`
- **Q-042 (new):** The hard-coded magic depths (`59`, `2`, `2.1`, `10`) in the recovery branches — what feature recognition logic do they correspond to? Document each in a domain-concept note. `#needs-review`
- **Q-043 (new):** `UseFlowDrillWithCountersink = True` was once `False`. Confirm with SME that integrated countersinking is currently stable enough to keep enabled. `#safety-relevant`
- Q-035 (from Works MOC) — confirm what BIdentNo `100381` represents.

Logged in [[../needs-review/_index]].

## Related

- [[elumatec-works-replacement-base]] — base class.
- [[../mocs/elumatec-works]] — replacement catalogue.
- [[../business-rules/elu-flowdrill-replacement]] — companion rule note.

## Coverage

`_coverage.md`: `Elumatec\Works\Replacements\Flowdrill.vb` → `done`.
