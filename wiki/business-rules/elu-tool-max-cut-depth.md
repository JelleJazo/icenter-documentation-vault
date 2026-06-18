---
type: business-rule
title: "Per-tool max cut depth (Elumatec, primary rule)"
status: needs-review
module: "iCENTER/Elumatec"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\ProfMillConverter.vb"
last-reviewed: ""
tags: [business-rule, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Per-tool max cut depth (the *primary* step-depth rule)

## Rule

In `ProfMillConverter.SetMaxStepDepth` (the only callsite that decides how a Work is sliced into per-pass steps), the **maximum cut depth per Work** is taken from the **per-tool `TMaxCut` value** stored in the machine's **tool database**. The Work's `WToolID` looks up its tool row; `oMachine.ToolDb.GetMaxCut(WToolID)` returns the maximum cut depth for that specific tool. `Work.SplitSteps(MaxStepDepth)` then chunks the cut into per-pass slices.

This is the rule that fires in production today. The companion [[elu-max-step-depth|`EluMaxStepDepth*` rule]] is the fallback branch and is **inactive** while `UseTMaxCut = True` (which it currently always is).

## Where it lives

- File: `Elumatec\ProfMillConverter.vb`
- Symbol: `Private Sub SetMaxStepDepth(ByRef oMachine As Machine.Sbz14x, ByRef oCut As Cut)` (line 907–928)
- Sub-symbol: `oMachine.ToolDb.GetMaxCut(oWork.WToolID)` (line 914)

## The code (minimal quote)

`ProfMillConverter.vb` lines 907–928:

```vb
Private Sub SetMaxStepDepth(ByRef oMachine As Machine.Sbz14x, ByRef oCut As Cut)
    Dim MaxStepDepth As Double
    Dim UseTMaxCut As Boolean = True 'CB\MW: aangezet

    For Each oWork As Work In oCut.Works
        If UseTMaxCut Then
            MaxStepDepth = oMachine.ToolDb.GetMaxCut(oWork.WToolID)         ' ← THE PRIMARY PATH
        Else
            ''Uitzonderingen:
            ''- Duimgaten in Forster profiel
            If TypeOf oWork Is Rectangle AndAlso oWork.WY1 > 48 _
               AndAlso oWork.WY1 < 50 AndAlso oWork.WDepth > 10 _
               AndAlso CType(oWork, Rectangle).WW3 = 5 Then
                MaxStepDepth = 6
            Else
                MaxStepDepth = oMachine.MaxStepDepth                         ' ← fallback path; see elu-max-step-depth
            End If
        End If

        If Not oWork.Replaced And Not oWork.SkipSplitSteps And MaxStepDepth > 0 Then
            oWork.SplitSteps(MaxStepDepth)
        End If
    Next
End Sub
```

The `'CB\MW: aangezet` comment (Dutch: "turned on") shows two engineers' initials and an intentional enable of the TMaxCut path.

## Why this is a business rule

Per-tool TMaxCut encodes a *tool-vendor-driven* limit: each tool entry in the **tool database file** (`SBZ140_ALU.nct` etc., e.g. `\\jazo.local\dfs\pm\Elumatec_SBZ140\Directories\database\SBZ140_ALU.nct`) carries its own `TMaxCut` field set by whoever maintains the tool DB. The limit accounts for:

- the specific cutter's geometry (flute count, helix angle, coating)
- the machine + spindle combo at JAZO
- the materials this tool is approved to cut on this machine

This is the *correct* place for step-depth limits — vendor-specific tools should not be limited to a single material-wide threshold. Editing TMaxCut for a tool requires write access to the tool DB file (typically Elumatec's tool-database utility), not iCenter source.

Consequences if wrong:
- **TMaxCut too high** for the tool → tool overload, tool breakage, possible workpiece ejection. Same failure mode as the [[elu-max-step-depth|`EluMaxStepDepth*` rule]] but per-tool.
- **TMaxCut too low** → unnecessary passes, longer cycle time.
- **`GetMaxCut` returns 0 for an unknown WToolID** → the `If ... MaxStepDepth > 0 Then` guard skips `SplitSteps` — the Work is sent as one single pass at full depth. **`#safety-relevant`** — Q-075.

## Triggers / when it fires

- Once per Work in every Cut, during `ProfMillConverter.ConvertCut` step 8 ([[../modules/elumatec-profmill-converter|see pipeline table]]).
- Skipped for Works with `Replaced = True` or `SkipSplitSteps = True` — these are macro-output features that the replacement-macro author already split into steps.

## Effects

- `Work.SplitSteps(MaxStepDepth)` mutates the Work's depth table — the abstract method is implemented per feature subclass. Outputs a multi-step depth profile that the post-processor turns into a multi-pass tool path.

## Edge cases / known exceptions

- **`GetMaxCut(WToolID) = 0`** (e.g. unknown or missing tool entry) → guard `MaxStepDepth > 0` skips `SplitSteps`. **Silent single-pass at full depth.** Q-075.
- **`Replaced` works skipped** — relies on the replacement-macro author having pre-split steps.
- **`SkipSplitSteps` flag** on a Work also skips splitting. Set by which code path? Phase-3 follow-up.
- **Tool DB I/O retried 5× with 1 s wait** in `Sbz14x.ReadToolDatabase` (called by every concrete machine constructor) — handles `IOException` from parallel post-processor access. If all 5 retries fail, the tool DB is unavailable and `GetMaxCut` will throw or return 0 (Phase-3 follow-up to confirm).

## Safety classification

- [x] Touches physical process → `#safety-relevant`
- [ ] Reversible if wrong? — Scrap-only for over-cut, costly only for under-cut.
- [x] Blocks production if it fails? — Yes (tool breakage stops the cell; missing tool entry produces a one-shot full-depth cut).

## SME questions

- **Q-075 (new)**: `GetMaxCut(WToolID) = 0` → `SplitSteps` skipped → single-pass full-depth cut. Should this fail loudly instead? `#safety-relevant`
- **Q-076 (new)**: How is the tool DB (`*.nct`) maintained? Confirm change-control around `TMaxCut` edits. `#safety-relevant`
- **Q-077 (new)**: `SkipSplitSteps = True` on a Work — which code paths set it, and is the contract "the upstream code guarantees correct stepping"? `#safety-relevant`

Logged in [[../needs-review/_index]].

## Related

- [[../modules/elumatec-profmill-converter|`ProfMillConverter.SetMaxStepDepth`]] — the callsite
- [[elu-max-step-depth]] — the (currently inactive) fallback rule
- [[elu-forster-thumbhole-step-depth]] — sibling override on the same fallback branch
- [[../modules/elumatec-machine-base|`Sbz14x`]] — provides `ToolDb` and `MaxStepDepth`
- [[../external-systems/elumatec-sbz140|Elumatec SBZ140 / SBZ141]]
