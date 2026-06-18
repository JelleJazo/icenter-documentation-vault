---
type: business-rule
title: "Flow-drill replacement (Elumatec)"
status: needs-review
module: "iCENTER/Elumatec/Works/Replacements"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\Works\\Replacements\\Flowdrill.vb"
last-reviewed: ""
tags: [business-rule, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Flow-drill replacement

## Rule

When iCenter's NC pipeline sees a circle or drilled hole on an aluminium profile that meets the flow-drill criteria, the feature is **replaced** with one of two macro substitutions:

- **Ø 9.3–9.31 mm circle/drill** OR **any circle/drill where (`WDepth - MaterialStartDepth`) > 6 mm**
- AND a `Deburr` exists at the same `(WX1, WY1)` on the same side (i.e. the design intends a countersink)
- → macro `EC00601` (flow-drill **with** integrated countersink, default) or `EC00054` (flow-drill **without**, only if `UseFlowDrillWithCountersink = False`)

If the same hole meets the size criteria but **lacks** the same-side `Deburr`, the hole is **silently deactivated** (`WActive = 0`, `WToolID = "-"`) and no flow-drill is emitted. The original comment is *"Wordt nog niet ondersteund"* ("not yet supported"). This is a partial-functionality gap — **see [[../needs-review/_index|Q-041]]**.

In addition, two profiles (`BIdentNo` 100142 P-profile and 100381) get **side-recovery branches**: if the recogniser put the flow-drill on the rear (or bottom) side and a matching front- (or top-) side hole is present, the rear/bottom is rotated and the gap is patched with a cloned or synthesised hole.

## Where it lives

- File: `Elumatec\Works\Replacements\Flowdrill.vb`
- Symbol: `Flowdrill.ApplyTo(oCut As Cut, BIdentNo As String)` (lines 30–258)

## The code (minimal quote)

`Flowdrill.vb` lines 41–48 — the match condition:

```vb
For Each Work As Work In oCut.Works
    ReplaceWork = False
    If (TypeOf Work Is Circle Or TypeOf Work Is Drill) And Work.WW1 = 9.31 Then    'Oude manier in generiek
        ReplaceWork = True
    ElseIf (TypeOf Work Is Circle Or TypeOf Work Is Drill) AndAlso _
           Work.WDepth - Work.MaterialStartDepth > 6 Then  'Nieuwe manier in generiek met User Defined Feature
        ReplaceWork = True
    End If
```

Lines 177–188 — macro pick / silent deactivation:

```vb
If HoleIsCountersunk Then
    If UseFlowDrillWithCountersink Then
        ReplacementWorks = GetWorksByMacro(oCut, "EC00601")
    Else
        ReplacementWorks = GetWorksByMacro(oCut, "EC00054")
    End If
Else
    ''Wordt nog niet ondersteund
    ''ReplacementWorks = GetWorksByMacro(oCut, "EC00054")
    Work.WToolID = "-"
    Work.WActive = 0
End If
```

The macro IDs `EC00601` / `EC00054` are looked up in the macro database file pointed at by `ProfMillConverter.AutoReplacementMacroFile` (→ `app.config` key `AutoReplaceMacros = Profiles\Resources\AutoReplaceMacros.ncd`).

## Why this is a business rule

Flow-drilling is a **physical-process choice** specific to JAZO's aluminium fabrication: instead of drilling a hole and adding a separate threaded insert, the wall material is heated and displaced into a tapped bushing using a flow-drill bit. It saves a part and an assembly step, and produces a stronger fastening for thin-wall extrusions. The rule encodes which standard JAZO holes use this process.

Consequences if wrong:
- **False negative** (hole *should* be flow-drilled but isn't): operator sees a plain hole, has to thread it manually or insert a separate nut. Cost: assembly time, possible weakening of joint.
- **False positive** (hole *shouldn't* be flow-drilled but is): material is locally heated and displaced. Reversal requires re-fabricating the bar. Cost: scrap.
- **Silent-deactivate path** (criteria met but no Deburr): bar is sent to the machine missing the hole entirely. Cost: operator has to drill it by hand, or the bar is scrap if not noticed.

## Triggers / when it fires

- Runs once per `Cut`, once per profile-mill program, as the **first** registered replacement in `Sbz140Alu.New` / `Sbz141Alu.New`.
- Inputs: every `Work` of type `Circle` or `Drill` in the cut. Side (`WSide`), position (`WX1, WY1`), primary dimension (`WW1`), depth (`WDepth`), profile (`BIdentNo`), and the presence of same-side `Deburr` features.
- External dependency: the `AutoReplaceMacros.ncd` macro file must contain macros `EC00601` and `EC00054`. If either is missing, `GetWorksByMacro` silently returns an empty list (see [[../modules/elumatec-works-replacement-base|Q-039]]) and the replacement does nothing — the hole reverts to the prior `Replaced = False` state and falls through to subsequent replacements.

## Effects

- Per-feature mutation of `oCut.Works`:
  - `ReplaceDict` swaps: original Circle/Drill → flow-drill Drill from the macro.
  - `AddWorks`: synthesised holes from the side-recovery branches.
  - `DeleteWorks`: countersinks on the opposite side that the flow-drill macro already handles.
- Downstream: the post-processor emits the macro's tool path (a specific tool / RPM / feed combination optimised for flow-drilling), not a regular drilling cycle.

## Edge cases / known exceptions

- **`Work.WW1 = 9.31`** is a literal `Double` equality check. With FP precision this is fragile to upstream changes.
- **`Work.WDepth - Work.MaterialStartDepth > 6`** is the newer "any deep hole" path. The 6 mm threshold is hard-coded — no `app.config` link.
- **Profile-specific recovery** for BIdentNo 100142 / 100381:
  - Rear-side detection: rotates from rear→front, then patches the rear-side gap with a cloned matching-front hole *or* synthesises an Ø 8.5 mm hole if `WDepth > 59`. The Ø 8.5 hole gets `WDepthTab = "2 1.00 1 1.00"` and comment `"Auto herstel naar 8.5"`.
  - Bottom-side detection: rotates from bottom→top with similar recovery.
- **Hard-coded magic depth values** in the recovery `DepthAccepted` checks: `> 10`, `= 2`, `= 2.1`, `> 10`. They map to specific patterns produced by EluCad's recogniser. Each is undocumented (Q-042).
- **`UseFlowDrillWithCountersink` is a hard-coded local** (`True`). Was `False` at some point with comment _"direct verzinken met de vloeiboor niet stabiel"_ ("direct countersinking with the flow-drill is unstable"). No `app.config` toggle. Confirm with SME (Q-043).

## Safety classification

- [x] Touches physical process → `#safety-relevant`
- [ ] Reversible if wrong? — Scrap-only: a flow-drilled hole can't be un-flow-drilled.
- [x] Blocks production if it fails? — Yes (operator intervention required to drill missing holes; possible bar scrap).

## SME questions

- **Q-035**: confirm BIdentNo `100381`.
- **Q-041**: silently-deactivated non-countersunk Ø 9.3 holes — intentional?
- **Q-042**: magic depth values (`>10`, `=2`, `=2.1`, `>59`, etc.) — document each.
- **Q-043**: `UseFlowDrillWithCountersink = True` — confirm stability today.

Logged in [[../needs-review/_index]].

## Related

- [[../modules/elumatec-replacement-flowdrill]] — module-level walkthrough.
- [[../modules/elumatec-works-replacement-base]] — replacement contract.
- [[../mocs/elumatec-works]] — replacement catalogue.
