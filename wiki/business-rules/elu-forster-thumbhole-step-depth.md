---
type: business-rule
title: "Forster profile thumb-hole step-depth override (Elumatec, currently dead)"
status: needs-review
module: "iCENTER/Elumatec"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\ProfMillConverter.vb"
last-reviewed: ""
tags: [business-rule, safety-relevant, needs-review, dead-code]
created: 2026-06-18
updated: 2026-06-18
---

# Forster profile thumb-hole step-depth override (currently dead)

## Rule

When iCenter machines a `Rectangle` feature on a **Forster profile** at a **thumb-hole** location, the maximum cut step depth is forced to **6 mm** regardless of the per-tool TMaxCut. The thumb-hole is identified by **geometric signature**:

- `WY1 > 48 AND WY1 < 50` (Y-position within a 2 mm band centred ~49 mm)
- `WDepth > 10` (deep cut)
- `WW3 = 5` (corner radius / classifier = 5 mm)

> **⚠️ Currently dead code.** The override lives inside the `Else` branch of `ProfMillConverter.SetMaxStepDepth` — the branch that only runs when `UseTMaxCut = False`. The local `UseTMaxCut` is currently hard-coded to `True` (line 910), so this override never fires in production. Q-065 asks SME whether the rule should be moved into the live `UseTMaxCut = True` branch (which would require checking it before the per-tool path).

## Where it lives

- File: `Elumatec\ProfMillConverter.vb`
- Symbol: `Private Sub SetMaxStepDepth(...)` lines 917–922 (inside the `Else` branch)

## The code (minimal quote)

```vb
Else
    ''Uitzonderingen:
    ''- Duimgaten in Forster profiel
    If TypeOf oWork Is Rectangle AndAlso oWork.WY1 > 48 _
       AndAlso oWork.WY1 < 50 AndAlso oWork.WDepth > 10 _
       AndAlso CType(oWork, Rectangle).WW3 = 5 Then
        MaxStepDepth = 6
    Else
        MaxStepDepth = oMachine.MaxStepDepth
    End If
End If
```

## Why this is a business rule

"Duimgaten in Forster profiel" — **thumb holes in Forster profile**. Forster is a steel-profile vendor (the JAZO factory uses Forster steel profiles in some product lines). The thumb hole is a specific feature that, when machined on the Stl/Rvs SBZ140 variants, was empirically found to need a different step depth than the material-wide `EluMaxStepDepthSTL = 1.6 mm`. The hard-coded `6 mm` matches the aluminium step depth — meaning whoever wrote this decided the steel thumb-hole was tool-limited (not material-limited) and could be cut more aggressively.

If the override is currently dead but the underlying assumption (steel thumb-hole tolerates 6 mm steps) is correct, the per-tool TMaxCut path must also reflect that fact. Otherwise: silent regression to the per-tool value (which may be conservative).

## Triggers / when it fires

Today: **never**. The branch is unreachable in production.

If `UseTMaxCut` were ever `False`:
- Inputs: a `Rectangle` Work on a Cut whose profile is a steel one (the branch isn't profile-checked but the geometry signature is a Forster-specific thumb-hole).
- Trigger: `WY1 > 48 AND WY1 < 50 AND WDepth > 10 AND WW3 = 5`.

## Effects

If active: bypasses the per-tool TMaxCut and forces `MaxStepDepth = 6` for the matching Rectangle. `Work.SplitSteps(6)` then chunks it accordingly.

## Edge cases / known exceptions

- **No profile check.** Any profile with a Rectangle in the same geometric band would also match. Acceptable today because the branch is unreachable.
- **Geometric signature is brittle.** A future re-recognition that shifts `WY1` by 0.1 mm to 49.95 mm still matches, but at 50.00 mm doesn't. Likewise `WW3 = 5` is a strict double-equality.
- **No SME-friendly profile ID** in the comment — only "Forster profiel". Identifying which `BIdentNo`s this matches would let the rule become profile-aware. Q-078.

## Safety classification

- [x] Touches physical process → `#safety-relevant` (if ever activated)
- [x] Currently dead → `#dead-code`
- [ ] Reversible if wrong? — Same as TMaxCut: scrap on over-cut.
- [x] Blocks production if it fails? — Yes (tool breakage on under-conservative step).

## SME questions

- **Q-065** (open): is `UseTMaxCut = False` ever set in production, making this rule active?
- **Q-078 (new):** which `BIdentNo`s correspond to "Forster profiel" thumb-holes? Replace the geometric signature with a profile-ID check. `#safety-relevant`
- **Q-079 (new):** if dead, remove the override or migrate it into the per-tool path so the steel thumb-hole's intended step depth is preserved? `#safety-relevant`

Logged in [[../needs-review/_index]].

## Related

- [[../modules/elumatec-profmill-converter|`ProfMillConverter.SetMaxStepDepth`]] — the callsite
- [[elu-max-step-depth]] — the fallback `EluMaxStepDepth*` rule (same branch)
- [[elu-tool-max-cut-depth]] — the live primary rule
