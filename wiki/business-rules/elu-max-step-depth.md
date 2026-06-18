---
type: business-rule
title: "Max step depth per material (Elumatec)"
status: needs-review
module: "iCENTER/Elumatec/Machine"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\Machine\\Sbz140Alu.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\Machine\\Sbz140Stl.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\Machine\\Sbz140Rvs.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\Machine\\Sbz141Alu.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\app.config"
last-reviewed: ""
tags: [business-rule, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Max step depth per material (Elumatec SBZ140 / 141)

## Rule

Every Elumatec machine variant exposes a fixed **maximum cutting step depth** that depends on the workpiece material. Aluminium machines step up to **6 mm** per pass; steel and stainless steel machines step up to **1.6 mm**.

## Where it lives

- Settings keys: `EluMaxStepDepthALU` and `EluMaxStepDepthSTL` in `app.config` (lines 257 and 259).
- Per-machine property: `MaxStepDepth As Double` on each `Sbz14*` class:

| Machine variant | Property reads | Default value |
|-----------------|----------------|---------------|
| `Sbz140Alu` (line 43–47) | `EluMaxStepDepthALU` | `6` (mm) |
| `Sbz141Alu` (line 49–53) | `EluMaxStepDepthALU` | `6` (mm) |
| `Sbz140Stl` (line 18–22) | `EluMaxStepDepthSTL` | `1.6` (mm) |
| `Sbz140Rvs` (line 18–22) | `EluMaxStepDepthSTL` | `1.6` (mm) |

## The code (minimal quote)

`Sbz140Alu.vb` line 43–47:

```vb
Public Overrides ReadOnly Property MaxStepDepth As Double
    Get
        Return Double.Parse(My.Settings.Properties("EluMaxStepDepthALU").DefaultValue)
    End Get
End Property
```

(The steel variants do the same with the `STL` key.)

`app.config` line 257–260:

```xml
<setting name="EluMaxStepDepthSTL" serializeAs="String"><value>1.6</value></setting>
<setting name="EluMaxStepDepthALU" serializeAs="String"><value>6</value></setting>
```

## Why this is a business rule

`MaxStepDepth` is the **per-pass cut depth** the post-processor / NC-generation pipeline assumes when slicing a deep feature (slot, pocket, deburr) into multiple passes. Two safety consequences:

1. **Too deep** → tool overload, tool breakage, possible part ejection, machine stop. Steel and stainless steel are slower-yielding and harder, hence the much lower 1.6 mm.
2. **Too shallow** → unnecessary passes, longer cycle time, but no safety hit.

Because the property is `ReadOnly` and read from `My.Settings.Properties(...).DefaultValue`, the value can only be changed by editing `app.config` and redeploying — not by a user setting in-app. That's a feature (you can't change it from the shop floor) but also a hazard (silent edit during deploy).

## Triggers / when it fires

- Read at NC-program generation time by anything in `Works\` / `Works\Replacements\` that calculates pass count for a deep feature. Phase-3 follow-up: enumerate every callsite of `Sbz14x.MaxStepDepth` (current grep finds the four property declarations only; callsites likely live in `Work.vb` / `AluGeneral.vb` / `StlGeneral.vb` / `ProfMillConverter.vb`).
- Indirectly via the abstract `Sbz14x.MaxStepDepth` MustOverride property — any new machine variant must supply a value.

## Effects

- Determines pass count → determines toolpath length → determines NC program size and the time the machine spends on that feature.
- Determines per-pass mechanical load on the tool and spindle. **Direct safety impact.**

## Edge cases / known exceptions

- **`Sbz140Stl` and `Sbz140Rvs` read the *same* `EluMaxStepDepthSTL` setting** even though stainless steel typically tolerates *less* step depth than mild steel. This is `#needs-review` — is there a planned `EluMaxStepDepthRVS` that was never added? Or do JAZO's stainless-steel workpieces actually take the same step depth as the steel ones?
- The two-flavour split (ALU vs STL) assumes a binary material model. If the factory ever runs a third material (e.g. brass), the property must be added.
- `Double.Parse` is called with **no `IFormatProvider`** — depends on the current thread culture. With Dutch Windows defaults this would parse `"6"` correctly but `"1.6"` as `16` (Dutch uses `,` as decimal). The current code happens to work because the `app.config` value is `"1.6"` and `Double.Parse` accepts both `.` and `,` if the candidate succeeds. **`#needs-review`** — confirm this isn't culture-fragile on locales other than `nl-NL` / `en-US`.

## Safety classification

- [x] Touches physical process / setpoint / interlock / safety → `#safety-relevant`
- [x] Reversible if wrong? — Yes (edit `app.config`, redeploy), but only after a tool breaks or a workpiece is scrapped.
- [x] Blocks production if it fails? — Yes (tool change at minimum; possible cell stop).

## SME questions

- **Q-030 (new):** Are 1.6 mm (STL) and 6 mm (ALU) the values actually in use today, or have they been overridden in a deployed `app.config`?
- **Q-027:** Confirm `Sbz140Stl` and `Sbz140Rvs` really should share the same `EluMaxStepDepthSTL`. Is a separate `EluMaxStepDepthRVS` needed?
- **Q-031 (new):** Enumerate every callsite that *uses* `Sbz14x.MaxStepDepth` (Phase-3 follow-up; likely in `Works\Replacements\AluGeneral.vb` / `StlGeneral.vb` / `ProfMillConverter.vb`).

Logged in [[../needs-review/_index]].

## Related

- [[../modules/elumatec-machine-base|`Sbz14x` family]]
- [[../mocs/elumatec|Elumatec subsystem MOC]]
- [[../external-systems/elumatec-sbz140|Elumatec SBZ140 / SBZ141]]
- [[elu-large-rectangle-classification]] — companion threshold rule
