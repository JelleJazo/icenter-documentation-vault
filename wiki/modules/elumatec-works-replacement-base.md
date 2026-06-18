---
type: module
title: "WorksReplacement — abstract base for Elumatec replacement macros"
status: done
module: "iCENTER/Elumatec/Works/Replacements"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\Works\\Replacements\\WorksReplacement.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant]
created: 2026-06-18
updated: 2026-06-18
---

# `Elumatec\Works\Replacements\WorksReplacement.vb` — replacement base

## Purpose

Abstract base for every macro that rewrites the contents of a `Cut`. Each concrete replacement is constructed with a target `Sbz14x` machine, looks up the machine's tool DB at construction, and exposes a single `ApplyTo(oCut, BIdentNo)` method that mutates the cut in place.

55 lines total — a very thin contract. Almost all of the *logic* lives in the 19 concrete subclasses (see [[../mocs/elumatec-works|Works MOC]]).

## Public surface

```vb
Public MustInherit Class WorksReplacement
    Public MustOverride ReadOnly Property Description As String
    Public MustOverride ReadOnly Property Name        As String

    Protected Property dtTools  As DataTable
    Protected Property oMachine As Machine.Sbz14x

    Public Sub New(oMachine As Machine.Sbz14x)
        Me.oMachine = oMachine
        dtTools = oMachine.GetTools
    End Sub

    Public MustOverride Sub ApplyTo(oCut As Cut, BIdentNo As String)

    Protected Function GetWorksByMacro(oCut, EcMacro, Optional VarZ=0, Optional VarY=0) As List(Of Work)
    Protected Function GetGroupByMacro(oCut, EcMacro, VarZ, VarY, AutoDetermineWSide) As Group
End Class
```

## Behavior in plain language

- **Construction** is eager: the machine's tool DB is read into `dtTools` immediately (via `oMachine.GetTools`). All replacements share the *same* tool-DB DataTable for a given machine instance.
- **`ApplyTo(oCut, BIdentNo)`** is the only behavioural method. Implementations typically:
  1. Read the profile from the `.epd` DB (`Dim oProfile As New Profile(BIdentNo) : oProfile.ReadFromDatabase(EluCadApp.ProfileDbFile)`).
  2. Maintain three local mutation buffers — `DeleteWorks As List(Of Work)`, `AddWorks As List(Of Work)`, `ReplaceDict As Dictionary(Of Work, Work)`.
  3. Iterate `oCut.Works`, dispatch on `BIdentNo` and feature kind, fill the buffers.
  4. Apply the buffers after the iteration loop.
- **Macro lookup** — `GetWorksByMacro` / `GetGroupByMacro` instantiate a `Macro(oCut, EcMacro)` and call its `ReadFromFile(ProfMillConverter.AutoReplacementMacroFile, VarZ, VarY)`. The macro file is read once per call (no shared cache visible in this base class).

## Surprises

1. **All exceptions are swallowed** — `GetWorksByMacro` catches everything and returns an empty list; `GetGroupByMacro` catches everything and returns `Nothing`. The caller can't tell macro-file errors apart from "macro not found". **`#needs-review`** — silent failures in a `#safety-relevant` path.
2. **No retry on macro-file I/O** — by contrast, `Sbz14x.ReadToolDatabase` retries `IOException` 5 times because of parallel post-processor access (see [[elumatec-machine-base]]). Macro-file reads have no such resilience.
3. **`dtTools` is captured once at construction.** If the tool DB is updated mid-session and the same `WorksReplacement` instance is reused, the cached tool table is stale. Whether instances are reused depends on the call pattern in `ProfMillConverter` — a Phase-3 follow-up.
4. **The `Name` and `Description` properties exist** but no base behaviour uses them. They're for UI / logging only.
5. **`BIdentNo` is passed `As String`** — profile IDs like `"100142"` are stringly-typed throughout. Comparing as `BIdentNo = "100142"` is brittle (leading/trailing whitespace is not stripped).

## Business rules surfaced here

- The macro-file path comes from `ProfMillConverter.AutoReplacementMacroFile`, which threads through to `app.config` key `AutoReplaceMacros = "Profiles\Resources\AutoReplaceMacros.ncd"` (relative path — see Q-034 in [[../mocs/elumatec-works]]). The `.ncd` file is part of the deployable surface. Macro IDs (e.g. `EC00601`, `EC00054`) are referenced from concrete replacements; the actual macro definitions live in this external file. `#safety-relevant`

## Open questions

- **Q-039 (new):** `GetWorksByMacro` / `GetGroupByMacro` swallow exceptions silently. Switch to logged-then-rethrown? `#safety-relevant`
- **Q-040 (new):** is `WorksReplacement.dtTools` ever stale across a long session (multi-batch CAD batchserver run)? `#safety-relevant`

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/elumatec-works|Elumatec Works MOC]] — catalogues all 19 concrete replacements.
- [[elumatec-replacement-flowdrill]] — exemplar replacement (Flowdrill).
- [[elumatec-replacement-large-rectangle]] — exemplar replacement (LargeRectangle).
- [[elumatec-replacement-alu-general]] — the catch-all (overview only).
- [[elumatec-machine-base]] — registers the per-machine replacement lists.

## Coverage

`_coverage.md`: `Elumatec\Works\Replacements\WorksReplacement.vb` → `done`.
