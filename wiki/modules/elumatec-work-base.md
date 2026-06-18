---
type: module
title: "Work — abstract base for all Elumatec feature classes"
status: done
module: "iCENTER/Elumatec/Works"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\Works\\Work.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant]
created: 2026-06-18
updated: 2026-06-18
---

# `Elumatec\Works\Work.vb` — abstract feature base

## Purpose

`Work` is the abstract base class for **every machinable feature** on an Elumatec workpiece: drilled holes, slots, rectangles, free-form pockets, saw cuts, deburr operations, macro references, groups. ~92 KB, ~25 abstract members, ~30 concrete properties. Every concrete subclass under `Works\` inherits from it.

## Public surface

### Common properties (set by feature subclasses or replacement macros)

| Property | Type | Meaning |
|----------|------|---------|
| `WNo` | Integer | sequence number within the cut |
| `WType` | String | feature type code (text serialisation) |
| `WSide` | Integer | which side of the bar — see `Sides` enum below |
| `WPriority` | Integer (default 0) | ordering hint within a cut |
| `WX1`, `WY1` | Double | primary position (mm) |
| `WW1` | Double | primary dimension — diameter / length / width depending on feature |
| `WHeight`, `WDepth` | Double | height and depth (mm) |
| `WFeed` | Double (default 1) | feed rate multiplier |
| `WDTSecIntr`, `WDTSecExtr` | Double | dwell-time inside / outside (sec) |
| `WDepthSec` | Integer (default 1) | depth-section count |
| `WDepthPost`, `WDepthPostAutoSet` | Double / Boolean | depth post-processing |
| `WDepthTab` | String | depth table (encoded text) |
| `WName` | String | feature name |
| `WToolID` | String | tool ID from the machine's Tool DB |
| `WActive` | Integer (default 1) | 0 = suppressed |
| `WRotation` | Integer (default 0) | rotation in degrees |
| `WComment` | String | free-text annotation (used by replacement macros to leave a trail) |
| `WHelixIntr` | Integer (default 0) | helix intrusion flag |
| `Replaced` | Boolean | set `True` by replacement macros — prevents re-application |
| `SuppressedByConverter` | Boolean | set `True` to skip during NC export |
| `WBroachDefined` | Boolean | broach flag was explicitly set |
| `WParent` | String | parent reference (group / macro source) |
| `SkipSplitSteps` | Boolean | if `True`, `SplitSteps(MaxStepDepth)` is a no-op for this Work |
| `Plane` | `Plane` | which `NcStructure\Plane` this Work belongs to |
| `ParentGroup` | `Group` (default `Nothing`) | parent group reference |
| `Cut` | `Cut` | back-reference to the owning `Cut` |
| `RuntimeManipulations` | `Replacement.RuntimeManipulations` | per-Work list of runtime manipulations (mutated by replacements at runtime) |

### Enums

```vb
Public Enum Sides
    Top = 1, Front = 2, Rear = 3, Left = 4, Right = 5,
    Bottom = 6, Free = 7, None = 0
End Enum

' Dutch parallel — same numeric codes
Public Enum SidesNl
    Boven=1, Voor=2, Achter=3, Links=4, Rechts=5,
    Onder=6, Vrij=7, Geen=0
End Enum

Public Enum Direction
    Center=0, Right=1, Left=2
End Enum
```

The English/Dutch enums share the same underlying integer codes — they're identifier aliases for the same values. Code that consumes `WSide` can safely use either.

### Constants

| Const | Value | Use |
|-------|-------|-----|
| `HeaderText` | `":WORK"` | ECW/text-file serialisation header |
| `ScaleFactor` | `100` | display / persistence scale |
| `Precision` | `1` (private) | display rounding |

### `MustOverride` surface (every subclass must implement)

```vb
MustOverride Property SortIndex As Integer
MustOverride ReadOnly Property PropertiesTable As DataTable

Public MustOverride Function GetTableNameText()    As String
Public MustOverride Function GetXmlNode(oXMLDoc)   As XmlElement
Public MustOverride Function GetEcwText()          As String          ' text serialisation
Public MustOverride Function GetNcwText(X, Y, Z)   As String          ' NCW (CAM) format
Public MustOverride Function GetWorkWidth()        As Double
Public MustOverride Function IdenticalTo(oWork)    As Boolean
Public MustOverride Function GetToolPathLength(oMachine) As Double
Public MustOverride Function GetAsDxfEntities(color) As List(Of DxfEntity)
Public MustOverride Function Includes(value)       As Boolean

Public MustOverride Sub SetWBroach(oTools)
Public MustOverride Sub AssignTool(oTools, oProfile, ignoreSides, ignoreIntrusionLength)
Public MustOverride Sub SetWContour(oMachine)
Public MustOverride Sub SplitSteps(MaxStepDepth As Double)
Public MustOverride Sub SetWHelixIntr(Value)
Public MustOverride Sub SetWMillDir(Value)
```

> **`SplitSteps(MaxStepDepth)` is the consumer of [[../business-rules/elu-max-step-depth|`Sbz14x.MaxStepDepth`]]** (Q-031 partial answer). Each feature subclass decides how to chunk a deep cut into multiple passes given the per-material limit. The actual call sites that pass `MaxStepDepth` are in `ProfMillConverter` (Phase-3 follow-up).

> **`SetWBroach(oTools)`** is the entry point for the broach-decision logic. `Rectangle.SetWBroach` is documented in [[../business-rules/elu-large-rectangle-classification]]; the other feature subclasses have their own implementations.

> **`AssignTool(oTools, oProfile, ignoreSides, ignoreIntrusionLength)`** matches the feature against the machine's tool DB to pick a real tool ID. Failure (no tool found) typically sets `WActive = 0` or `WToolID = "-"` — the feature is silently disabled.

### Computed properties

| Property | Returns |
|----------|---------|
| `MaterialStartDepth` | depth at which the cut enters material (computed from `GetDepthTable`) |
| `MaterialStepCount` | number of in-material steps |
| `MaterialStepSum` | sum of in-material steps |
| `GetYSign(Side)` | `+1` for Top, `-1` for Bottom, `+1` otherwise |

`GetDepthTable` is itself an abstract method; each subclass exposes its depth table as a `DataTable` with at least columns `D` (depth) and `M` (in-material flag 0/1).

## Behavior in plain language

A `Work` is an in-memory description of one feature on one bar — position, dimensions, tool, depth-sectioning, and a side selector. The rendering / NC-export pipeline calls:

1. Replacement macros (`WorksReplacement.ApplyTo`) — may rewrite `Replaced`, swap the Work for a different subclass, or add `RuntimeManipulations`.
2. `AssignTool(...)` — pick a tool from the machine's Tool DB.
3. `SetWBroach(...)` — decide whether to broach or contour (where applicable).
4. `SetWContour(...)` — fine-grained contour selection.
5. `SplitSteps(MaxStepDepth)` — chunk into per-pass depth slices respecting the safety threshold.
6. `GetNcwText(...)` / `GetXmlNode(...)` — emit serialised output for the post-processor.

`Cut` (its parent) owns a `List(Of Work)`. Replacements iterate `oCut.Works`, decide which features to alter, and mutate the list. Most replacements use a `DeleteWorks` + `AddWorks` + `ReplaceDict` pattern to defer mutations until after the iteration loop.

## Surprises

1. **The Dutch and English `Sides` enums share integer values** — defensible (so either can be used as an alias) but creates a documentation hazard: a `WSide = 1` could mean Top or Boven; readers need to know the codes coincide.
2. **Most `MustOverride` methods don't have a default base implementation** — there's no graceful fallback. Adding a new feature subclass means implementing 15+ methods. Skipping any of them is a compile error, which is good, but the duplication across subclasses is large.
3. **`RuntimeManipulations` is mutable per Work** — replacements can stack manipulations. The interaction between manipulations and the abstract methods isn't obvious from the base class alone; needs a Phase-3 follow-up note on `RuntimeManipulation*` classes.
4. **`Replaced` is a sticky flag** — once set, replacements check `MyWork.Replaced` and skip. But nothing un-sets it. If replacement ordering is wrong, an earlier replacement can "lock" a Work against a later, more correct, one.
5. **XML serialisation uses `ICenterLib.Common.ApplicationCulture`** (line 181 onwards) — explicit culture pinning. The replacement code generally doesn't pin culture explicitly when calling `Double.Parse`, however (see [[../business-rules/elu-max-step-depth]] culture note).

## Open questions

- See [[../mocs/elumatec-works|Works MOC]] Q-034..Q-038 + earlier Elumatec questions.
- Phase-3 follow-up: per-feature subclass notes (Circle, Drill, Line, Rectangle, SlottedHole, FreeForm, Sawcut, Deburr, Macro, Group). Of these only `Rectangle` is partially documented (via the business-rule note).

## Coverage

`_coverage.md`:
- `Elumatec\Works\Work.vb` → `done` (abstract surface fully documented; per-subclass notes are separate work)
