---
type: business-rule
title: "SMT deburr speed = 0.225 m² / min"
status: needs-review
module: "ICenterLib/(root)"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\AppSettings.vb"
last-reviewed: ""
tags: [business-rule, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# SMT (sheet-metal) deburr speed = 0.225 m² / min

## Rule

iCenter assumes the sheet-metal deburring machine processes **0.225 square metres per minute** (equivalently `0.225 / 60` m²/sec). This constant is used in cycle-time / capacity calculations for sheet-metal jobs.

The constant is a **compile-time literal** in `AppSettings.vb` — not configurable at runtime via `T_ApplicationSettings`. Changing it requires a recompile of ICenterLib.

## Where it lives

- File: `ICenterLib\AppSettings.vb` line 45
- Symbol: `Public Const SmtDeburrSpeed As Double = 0.225 / 60`

## The code (minimal quote)

```vb
Public Const SmtDeburrSpeed As Double = 0.225 / 60  ''m2/sec
```

The inline comment confirms the unit is **m²/sec**. The literal `0.225` is therefore m²/minute (= 13.5 m²/hour).

## Why this is a business rule

Deburr cycle time directly feeds production planning. If a sheet metal part has area `A m²`, the assumed deburr time is `A / SmtDeburrSpeed = A * 60 / 0.225 sec`. For a 1 m² panel that's ~267 seconds (4 min 27 sec).

If the constant doesn't match the **actual** deburr machine throughput at JAZO, then:
- **Under-estimate** → schedules are tight; jobs run late; the deburr station becomes a bottleneck.
- **Over-estimate** → capacity is left on the floor; jobs are flagged as "late" when they aren't.

The value 0.225 m²/min was presumably measured once and hard-coded. Machine wear, blade replacement schedules, or material changes (alu vs steel) might all shift the actual throughput.

## Triggers / when it fires

- Wherever cycle-time for an SMT-deburr operation is calculated. Phase-3 follow-up: callsites via Grep on `SmtDeburrSpeed`.
- Expected to be in `ICenterLib.SmtProduction.*` and/or `iCENTER\SmtManufacturing\*`.

## Effects

- Schedules: produces an estimated cycle time per SMT-deburr operation.
- Cost / capacity reports: shop-floor capacity for the deburr station is derived from this number.

## Edge cases / known exceptions

- **Single rate, all materials.** No material-dependent variant (cf. [[elu-max-step-depth|`EluMaxStepDepth*`]] which is material-aware). Aluminium vs steel vs stainless might deburr at different rates.
- **Single rate, all parts shapes.** Edge-length matters more than area for some deburr designs; this constant assumes the area-rate model.
- **Compile-time inlined.** If a consuming assembly was built against an older ICenterLib value, recompiling ICenterLib alone won't update the consumer's inlined copy.

## Safety classification

- [ ] Touches physical process — no (it's a planning estimate, not a setpoint).
- [x] Production-planning input → `#safety-relevant` (mis-estimated capacity can leave orders late or stations idle).
- [x] Reversible if wrong? — Yes (recompile + redeploy), but accumulated planning errors are harder to undo.
- [ ] Blocks production if it fails? — No.

## SME questions

- **Q-113** (from MOC): SME-confirm `0.225 m²/min` matches actual JAZO deburr throughput today. When was it measured?
- **Q-130 (new):** Should this be material-aware (alu vs steel vs stainless)?
- **Q-131 (new):** Should this be runtime-tunable via `T_ApplicationSettings` instead of compile-time?

Logged in [[../needs-review/_index]].

## Related

- [[../modules/icenterlib-appsettings]] — defining module.
- [[elu-max-step-depth]] — sibling example of a material-aware process constant.
