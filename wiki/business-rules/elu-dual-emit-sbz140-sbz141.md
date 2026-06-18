---
type: business-rule
title: "Dual NC emission for Sbz140Alu → also Sbz141Alu"
status: needs-review
module: "iCENTER/Elumatec"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\ProfMillConverter.vb"
last-reviewed: ""
tags: [business-rule, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Dual NC emission — Sbz140Alu also emits for Sbz141Alu

## Rule

When `ProfMillConverter.Optimize()` picks a target machine and the chosen machine is the **`Sbz140Alu`** (aluminium SBZ140) AND the current EluCad app version reports `UseFileBasedSettings = True`, iCenter **additionally** emits an NC program for the **`Sbz141Alu`** (aluminium SBZ141). The whole `ConvertCut` pipeline runs twice — once per machine — from independent fresh copies of the raw Cut.

Aluminium SBZ140 jobs therefore produce NC output that *both* machines can consume. Either physical machine can pick up the work without re-engineering. **Other machine variants (Sbz140Stl, Sbz140Rvs, Sbz141Alu when chosen first) do not get this dual-emit treatment.**

## Where it lives

- File: `Elumatec\ProfMillConverter.vb`
- Symbol: `Public Function Optimize() As Boolean` lines 75–80

## The code (minimal quote)

```vb
Dim oMachine As Machine.Sbz14x = Machine.Sbz14x.GetSbzMachineByProfileMachineId(oProfile.ProfMillMachineId)
...
TargetMachines = New List(Of Machine.Sbz14x) From {oMachine}
If TypeOf oMachine Is Machine.Sbz140Alu Then
    If oEluCad.AppVersion.UseFileBasedSettings Then
        TargetMachines.Add(New Machine.Sbz141Alu)
    End If
End If
```

## Why this is a business rule

The two aluminium machines (Sbz140Alu and Sbz141Alu) are interchangeable for the alu-side product mix — they have the same `EluMaxStepDepthALU = 6 mm`, the same registered `WorksReplacement` lists in their constructors, and similar tool capabilities. The factory keeps both running so work can flow to whichever has capacity. Producing NC for both up-front means:

- **Operational flexibility**: an operator on either machine can pick up the same job; no re-engineering on machine-change.
- **Failover**: if one machine goes down, work can move without delay.
- **Implicit cost**: dual cycle-time calculation, dual tool-DB lookups, dual disk I/O for the NC files.

The `UseFileBasedSettings` gate likely means: only emit for both machines if the deployment is using file-based EluCad settings (probably newer EluCad installs where both machines share the same registry/file backbone). Otherwise emit only for the chosen machine.

## Triggers / when it fires

- Once per `Optimize()` call, before the per-machine `ConvertCut` loop.
- Inputs: chosen target machine (`oMachine`) and the EluCad app version's `UseFileBasedSettings` flag.

## Effects

- `TargetMachines` list grows from 1 to 2.
- The per-machine `ConvertCut` loop runs twice. Each iteration:
  - Reads a fresh `Cut` from the model XML.
  - Sets `CStation = oProfile.GetPreferredSbzStation(myMachine)` (per-machine preferred station).
  - Applies the EluXml `CCopies = 1` hack if the machine emits EluXml.
  - Runs `ConvertCut(myMachine, MyCut)` (the 17-step pipeline).
- Each successful iteration produces its own NC output file via `ExportNC` (later in the pipeline).
- Per-machine exceptions are aggregated into a `MySystem.ExceptionList`. If any machine fails, the whole `Optimize()` throws the list. Success is "every machine succeeded".

## Edge cases / known exceptions

- **`UseFileBasedSettings = False`** → dual-emit disabled. Sbz140Alu jobs go to Sbz140Alu only.
- **Chosen machine is `Sbz141Alu` directly** → no dual-emit. Sbz141Alu jobs go to Sbz141Alu only. **Asymmetric.** Q-080 — is this intentional?
- **`ProfMillConverter.ProfMillJob` accessor returns only the *last* optimised job** (see [[../modules/elumatec-profmill-job|ProfMillJob]] surprise #4). After dual-emit, only the *second* machine's job (Sbz141Alu) is reachable via that accessor. Q-072.
- **Per-machine output filenames** must differ — `ExportNC(filepath, ...)` is called per machine; the filepath presumably encodes the machine name. Phase-3 follow-up to confirm.

## Safety classification

- [x] Touches physical process → `#safety-relevant`
- [ ] Reversible if wrong? — No. A bar machined on the wrong machine is scrap.
- [x] Blocks production if it fails? — Half-failure (one machine emits, other doesn't) leaves the floor with an asymmetric setup.

## SME questions

- **Q-066** (open): what determines `AppVersion.UseFileBasedSettings`, and how do operators know whether dual-emit is on?
- **Q-080 (new):** asymmetric handling — Sbz140Alu emits for both, Sbz141Alu emits only for itself. Intentional? `#safety-relevant`
- **Q-081 (new):** confirm `ExportNC` per-machine output filepaths don't collide. `#safety-relevant`

Logged in [[../needs-review/_index]].

## Related

- [[../modules/elumatec-profmill-converter|`ProfMillConverter.Optimize`]] — where the dual-emit decision lives
- [[../modules/elumatec-machine-base|`Sbz14x` family]] — Sbz140Alu and Sbz141Alu definitions
- [[../mocs/elumatec|Elumatec MOC]]
