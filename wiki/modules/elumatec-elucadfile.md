---
type: module
title: "EluCadFile — .ecw text-format parser"
status: done
module: "iCENTER/Elumatec/NcStructure"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\NcStructure\\EluCadFile.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `EluCadFile.vb` — `.ecw` text-format parser

## Purpose

Hand-rolled line-based parser for **EluCad's text-format work-drawing files (`.ecw`)**. Reads the file (or string / array of lines) and builds the in-memory [[elumatec-ncstructure-hierarchy|`Job` → `Bar` → `Cut` → `Work` + `Plane`]] tree. Pairs with the `GetEcwText()` writers on `Job` / `Bar` / `Cut` / `Plane` for round-tripping.

## Public surface

```vb
Public Class EluCadFile
    Public Property Filepath As String
    Public Property Jobs As List(Of Job)

    Public Shared Function ReadFromFile(Filepath As String) As EluCadFile
    Public Shared Function ReadFromString(Value As String) As EluCadFile
    Public Shared Function ReadFromArray(Value As String()) As EluCadFile

    Public Sub SetEdittedManually(Value As Boolean)
End Class
```

Private enum `Section` (File / Job / Bar / Cut / Work / GroupDefinition) and private static helpers `GetDoubleValue(s)` + `EvaluateExpression(expr)`.

## Behavior in plain language

The parser is a **single-pass state machine** that walks every line of the input. Each line is normalised to `Key = Value` (or just `Key` for section markers). Section transitions are triggered by header tokens:

| Token | Effect | Next section |
|-------|--------|--------------|
| `:JOB` | new `Job`, appended to `Jobs` | Job |
| `:BAR` | new `Bar` (deferred — not added until first Work) | Bar |
| `:CUT` | new `Cut`, parent = Cut (sic — also tracked as `ParentSection`) | Cut |
| `:WORK` | finalise pending Cut (`Job.AddCut(MyBar.PartCode, MyCut, ...)`); start collecting Work properties | Work |
| `:WGROUP <n>` | start a Group definition with key `<n>` (stored in `GroupDefinition` dictionary) | GroupDefinition |
| *(empty line in Work)* | finalise the current Work via `Works.Work.CreateFromDictionary(MyWorkDict)` and append it to either `MyCut.Works` or the current group | back to File |

Per-section parsing:

- **Job**: dispatches on `CNCDRIVER`, `ORDER`, `INFO`, `VAR0`..`VAR9`, `JACTIVE` (sets the matching `Job` property).
- **Bar**: dispatches on `BIDENTNO`, `BWIDTH`, `BHEIGHT`, `BFILEDB`, `BLENGTH`, `BACTIVE`. `BIDENTNO` also triggers `MyBar.CalcPartCode()` to derive `PartCode` from `BIdentNo`. `BFILEDB` is treated as a *filename only* and joined to `EluCadApp.ProfileDbFile`'s parent folder — so a `BFileDB = "Foo.epd"` becomes the full path.
- **Cut**: dispatches on `CNO`, `CSTATION`, `CCOMNO`, `CDESCRIPTION`, `CLENGTH`, the four `CANGLE*` corners, `CCOPIES`, `CROTATION`, `CMIRROR`, `CCORL`, `CCORR`, `COFFSETX/Y/Z`, `CACTIVE`, `CCHOPANGLELEFT/RIGHT`.
- **Work**: builds a `MyWorkDict : Dictionary(String, String)` of all keys read. On empty line:
  - If `WActive = 0` and `SuppressedByConverter` is missing, set `SuppressedByConverter = "true"` (back-compat hack — older serialisations didn't have the flag).
  - `Works.Work.CreateFromDictionary(MyWorkDict)` instantiates the right Work subclass.
  - **Special case for `FreeForm`**: `FF.GetAsFreeFormPoint(MyWorkDict)` checks whether this Work block is actually a *point* of an existing FreeForm; if so, the point is appended to the FreeForm in progress and the Work itself is discarded.
  - **`Group` Works** receive their members from `GroupDefinition(WGroup)` — populated earlier by `:WGROUP` blocks.
  - In `GroupDefinition` parent section, Works are appended to the group rather than to a Cut.

## Surprises

1. **Empty-line termination of Work blocks** (line 244). A stray blank line inside any other section will silently fall through and not break anything obvious — but inside a Work block it commits whatever's been collected so far. Brittle to manual-edited `.ecw` files.

2. **`BIDENTNO` triggers `CalcPartCode`** which calls back into `EluCadApp.GetProfileInfoByBIdentNo(...)` (line 159). Parser has a hidden dependency on the profile DB being loaded.

3. **`GetDoubleValue(s)`** (line 312):
   ```vb
   If IsNumeric(s) Then
       Return Double.Parse(s, Common.ApplicationCulture)
   Else
       Try
           Return EvaluateExpression(Value)
       Catch ex As Exception
           Return 0  ''Ignore for now
       End Try
   End If
   ```
   `EvaluateExpression` uses `DataTable.Compute(expression, "")`, which supports `+`, `-`, `*`, `/`, `%`, `()`, plus boolean operators, plus DataTable aggregate syntax. **So any numeric ECW cell can be an arithmetic expression.** No way to control or discover the grammar without reading the .NET docs. Q-053. `#safety-relevant`.

4. **Parse errors return `0`** instead of throwing. A malformed expression silently becomes 0. Catastrophic for safety-critical dimensions (e.g. `WDepth = "2*x"` where x is unbound → 0 → cut won't enter material).

5. **Bar deferred-append**: `MyBar = New Bar()` at the `:BAR` line, but the Bar isn't added to a Job until the first `:WORK` triggers `MyJob.AddCut(MyBar.PartCode, MyCut, MyBar.GetXmlNode(TempBarXmlDoc))` (line 98). A Bar with no Cuts is never registered.

6. **`ParentSection`** is tracked separately from `MySection` — used to distinguish "Work inside a Cut" from "Work inside a GroupDefinition". Easy to miss; the dispatch in the Work-finalisation branch reads it (line 271 onwards).

7. **`MyWorkDict` is rebuilt per Work** but `MyFreeForm` *persists* across Work blocks within the same Cut/Group (line 280, 293). FreeForm-point Works look up `MyFreeForm` — if FreeForm parsing got interrupted, points get silently appended to the wrong FreeForm.

8. **No reverse-mapping** to verify that what we just parsed matches `Job.GetEcwText()`. No round-trip test visible. Likely no test exists at all (file has no unit-test pair in scope).

## Business rules surfaced here

- The empty-line-terminates-Work convention is part of the ECW format spec — third-party-driven (Elumatec). `#needs-review` whether iCenter must always emit a trailing blank line on writes (Q-054).
- `BActive = 0` Work blocks gain a synthetic `SuppressedByConverter = "true"` on read — preserves intent for round-tripping but changes the semantic.

## Open questions

- **Q-053** (from MOC): document the `DataTable.Compute` expression grammar permitted in ECW numeric cells. `#safety-relevant`
- **Q-054** (from MOC): empty-line-terminates-Work — confirm with SME. `#needs-review`
- **Q-060 (new):** parse errors silently become `0`. Is there a logging/alert path that should fire instead? `#safety-relevant`
- **Q-061 (new):** the FreeForm-point continuation logic relies on `MyFreeForm` persisting between Work blocks. What happens if a `:WORK` of a different type appears between FreeForm points?

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/elumatec-ncpipeline|NC pipeline MOC]]
- [[elumatec-ncstructure-hierarchy|`Job`/`Bar`/`Cut`/`Plane` hierarchy]] — the model this parser builds.
- [[../modules/elumatec-cad-app|`EluCadApp`]] — owns `ProfileDbFile` referenced by `BFILEDB` resolution.

## Coverage

`_coverage.md`: `Elumatec\NcStructure\EluCadFile.vb` → `done`.
