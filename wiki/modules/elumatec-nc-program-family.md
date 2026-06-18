---
type: module
title: "NcProgram family — AUF + EluXml serialisers"
status: done
module: "iCENTER/Elumatec/AufSerializer"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\AufSerializer\\NcProgram.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\AufSerializer\\NcProgramAuf.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\AufSerializer\\NcProgramEluXml.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\AufSerializer\\Programm.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\AufSerializer\\Kontur.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\AufSerializer\\ZeileAuftrag.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\AufSerializer\\ZeileProgramm.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\AufSerializer\\ZeileKontur.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\AufSerializer\\ZeileTTab.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\AufSerializer\\EluXmlProgram.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\AufSerializer\\EluXmlProgramDetail.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\AufSerializer\\EluXmlJob.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\AufSerializer\\EluXmlJobItem.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\AufSerializer\\EluXmlJobSubItem.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `NcProgram` family — AUF + EluXml NC serialisers

## Purpose

Read and write Elumatec NC programs in **two** machine-consumable formats. The format choice is per-machine-variant (`Sbz14x.NcType`), but file *detection* is content-sniffed (`Content.StartsWith("<?xml")` → EluXml, else AUF).

```
NcProgram (MustInherit, in namespace Elumatec.Auftrag)
├── NcProgramAuf     → .auf      (older, section-block German text format)
│     uses:  Programm, Kontur, ZeileAuftrag, ZeileProgramm, ZeileKontur, ZeileTTab
│
└── NcProgramEluXml  → .eluxml   (newer, XML)
      uses:  EluXmlProgram, EluXmlProgramDetail, EluXmlJob, EluXmlJobItem, EluXmlJobSubItem
```

> Namespace note: the AUF serialiser code lives in `Elumatec.Auftrag` (German for "order"), while the structural model lives in plain `Elumatec` or `Elumatec.NCStructure`. Don't conflate.

## Common API — `NcProgram`

```vb
Public MustInherit Class NcProgram
    Public Property Description As String
    Public Property Comment     As String
    Public Property CNo         As String

    Public Enum NcType
        Auf    = 1
        EluXml = 2
    End Enum

    Public MustOverride Function ComputeCycleTime(oMachine As Sbz14x, ByRef report As XmlDocument) As Double

    Public Shared Function GetNcType(Content As String) As NcType                ' '<?xml' → EluXml else Auf
    Public Shared Function ReadFromFile(Filepath As String) As NcProgram         ' file → format-detection → typed reader
    Public Shared Function ReadFromString(NCAsString As String) As NcProgram     ' same, from string
    Public Shared Function GetNCStringWithIaNr(strProgram1 As String, iaNr As Long) As String  ' patch IA-number into program
    Public Shared Function GetFeedRate(FeedRate As String) As Double             ' mm/min string → mm/sec
End Class
```

- `ReadFromFile` swallows all exceptions and returns `Nothing` on failure (line 51). Callers must null-check.
- `GetFeedRate` divides by **60** to convert mm/min → mm/sec. Non-numeric input returns `1`.

## AUF format — `NcProgramAuf` and friends

**Source:** `Elumatec\AufSerializer\NcProgramAuf.vb` (27 KB), helper classes `Programm.vb` (10 KB), `Kontur.vb` (3 KB), `ZeileAuftrag.vb` (3 KB), `ZeileProgramm.vb` (9 KB), `ZeileKontur.vb` (4 KB), `ZeileTTab.vb` (2 KB).

### Format shape

Plain-text, section-block, German keywords. Markers:

| Marker | Section | Contains |
|--------|---------|----------|
| `[BEGIN_AUFTRAG]` … `[END_AUFTRAG]` | order header | per-program: `NR`, `BEZEICHNUNG`, `INFO` |
| `[BEGIN_ZEILE_AUFTRAG]` … `[END_ZEILE_AUFTRAG]` | one order line | `SOLL_STCK`, `BEARB_PROGRAMM`, `STABLAENGE`, `MESSEN`, `AUFTRAGSPOS`, `STATION` |
| `[BEGIN_PROGRAMM]` … (no explicit `END`) | NC program | program-level keys + nested `[BEGIN_ZEILE_PROGRAMM]` blocks |
| `[BEGIN_ZEILE_PROGRAMM]` … | one program line | per-line operation; can contain a nested `[BEGIN_ZEILE_TTAB]` (tool-table line) with `MASS` |
| `[BEGIN_KONTUR]` … | contour definition | nested `[BEGIN_ZEILE_KONTUR]` lines |

### `NcProgramAuf` state

```vb
Public Property List_ZEILE_AUFTRAG As New List(Of ZeileAuftrag)
Public Property List_Kontur        As New List(Of Kontur)
```

Plus inherited `Description`, `Comment`, `CNo`. There appears to be no top-level `Programm` collection — programs are owned inline by the `ZeileAuftrag`s.

### `CreateFromArray(aAuf As String())`

State-machine parser with ~7 in-block flags (`inAuftragBlock`, `inZeileAuftragBlock`, `inProgrammBlock`, `inZeileProgrammBlock`, `inZeileTTabBlock`, `inKonturBlock`, `inZeileKonturBlock`). Each `[BEGIN_*]` sets the corresponding flag; `[END_*]` clears it. Inside a block, the matching `Select Case myKey` dispatches the value to the right `ZeileXxx` field.

German-key inventory (from the first 120 lines):

| Key | Field |
|-----|-------|
| `SOLL_STCK` | target quantity (Sollstück) |
| `BEARB_PROGRAMM` | machining program reference |
| `STABLAENGE` | bar length |
| `MESSEN` | measurement flag |
| `AUFTRAGSPOS` | order position |
| `STATION` | station |
| `NR` | program number → `oNcProgram.CNo` |
| `BEZEICHNUNG` | description → `Description` |
| `INFO` | comment → `Comment` |
| `MASS` | dimension (inside `ZeileTTab`) |

Many more inside `Programm`, `Kontur`, `ZeileProgramm`, `ZeileKontur` (lines beyond 120 — Phase-3 follow-up to enumerate fully).

### Surprises (AUF)

- **Stringly-typed German keys throughout**; no enum or constant table. A typo in a key silently drops the value.
- **`[BEGIN_PROGRAMM]` has no explicit `[END_PROGRAMM]`** marker visible in the first 120 lines. The parser presumably exits the block on the next `[BEGIN_*]` or EOF.
- The factory is `CreateFromArray(String())` — `ReadFromString(String)` splits by `vbCrLf` first (`NcProgram.ReadFromString` line 61). `vbCrLf` only — lone `\n` files break.
- The whole parser body is wrapped in `Try / Catch ex As Exception` at the outer `CreateFromArray` level. Errors return whatever was successfully parsed so far. **No diagnostic produced**.

## EluXml format — `NcProgramEluXml` and friends

**Source:** `Elumatec\AufSerializer\NcProgramEluXml.vb` (5 KB), helpers `EluXmlProgram.vb` (13 KB), `EluXmlProgramDetail.vb` (11 KB), `EluXmlJob.vb` (1 KB), `EluXmlJobItem.vb` (1 KB), `EluXmlJobSubItem.vb` (3 KB).

### Format shape

XML, schema like:

```xml
<root>
    <Jobs>
        <Job> ... → EluXmlJob
            <JobItem>
                <JobSubItem>
                    <MyProgram CIdent="..."/>
                </JobSubItem>
            </JobItem>
        </Job>
    </Jobs>
    <Programs>
        <Program CIdent="..."> ... → EluXmlProgram (with CNo, CComment, CDescription, depth tables, …)
    </Programs>
</root>
```

### `NcProgramEluXml` state

```vb
Public Property Programs As List(Of EluXmlProgram)
Public Property Jobs     As List(Of EluXmlJob)
```

### `ReadFromString(XmlString)`

1. Load via `XmlDocument.LoadXml`.
2. For each `Jobs/Job` node: build `EluXmlJob.CreateFromXmlNode(XN)` and append.
3. For each Job: look up the matching `Programs/Program[CIdent='{Job.JobItem.JobSubItem.MyProgram.CIdent}']`, build `EluXmlProgram.CreateFromXmlNode(JobProgramNode)`, append. Set the *NcProgram*-level `CNo`, `Comment`, `Description` from the *last* matched program (so on a multi-Job file these track the last Job's program).

### `ComputeCycleTime(oMachine, ByRef report)` — only EluXml implements it for now

```vb
For Each EXP As EluXmlProgram In Programs
    EXP.ConvertDepthTableFeedToAbsolute(oMachine)
Next

For Each EXP As EluXmlProgram In Programs
    Dim subResult As Double = 0
    Dim myXmlElement = report.CreateElement(EluXmlProgram.XmlReportElementName)
    myXmlElement.SetAttribute("Description", EXP.CDescription)
    report.DocumentElement.AppendChild(myXmlElement)
    subResult += EXP.ComputeCycleTime(oMachine, myXmlElement, subResult)
    result += subResult
Next
report.DocumentElement.SetAttribute("Total", result.ToString(...))
report.DocumentElement.SetAttribute("TotalFormatted", CycleTime.GetFormattedDuration(result))
report.DocumentElement.SetAttribute("MachineName", oMachine.Name)
```

Builds a parallel XML report with one element per program plus aggregate totals. The result feeds the global [[../architecture/global-state|`UseSbzCalculatedDuration = True`]] preference — iCenter will prefer this duration over ISAH's estimate.

> **`NcProgramAuf.ComputeCycleTime` is not visible in the first 120 lines** of `NcProgramAuf.vb` — confirm whether AUF supports cycle-time calculation, or whether `UseSbzCalculatedDuration = True` quietly only works for EluXml machines. Q-062.

### `GetNCStringWithIaNr(strProgram1, iaNr)` — **EluXml stub**

```vb
Public Overloads Shared Function GetNCStringWithIaNr(strProgram1 As String, iaNr As Long) As String
    Try
        Dim result As String = strProgram1
        Return result         ' ← unchanged
        ' ... rest of body commented out ...
    Catch ex As Exception
        oApplicationLog.NewEntry(ex.ToString, MsgBoxStyle.Critical)
        Return strProgram1
    End Try
End Function
```

The function does nothing. **AUF's implementation** (visible in `NcProgramAuf.GetNCStringWithIaNr`, line not shown but referenced from `NcProgram.GetNCStringWithIaNr`) actually rewrites the program. So calling `NcProgram.GetNCStringWithIaNr` on an EluXml program silently returns it unchanged. Q-050.

## Business rules surfaced here

- **`UseSbzCalculatedDuration = True`** (`Modules\Main.vb` global) → iCenter prefers `NcProgramEluXml.ComputeCycleTime` over ISAH-side estimates. AUF machines may not contribute (Q-062).
- **IA-number patching** (`GetNCStringWithIaNr`) works for AUF, **silently no-ops for EluXml** (Q-050).
- **mm/min ↔ mm/sec conversion** is centralised in `NcProgram.GetFeedRate(s) → IntegerParse(s) / 60`. Non-numeric input returns 1 (not zero, not error).

## Surprises

1. **`NcProgramEluXml.GetNCStringWithIaNr` is a stub.** See Q-050.
2. **Cycle-time report XML format** is built in code, no schema visible. The report shape is implicit. Hard to evolve.
3. **`Description`, `Comment`, `CNo`** on `NcProgramEluXml` are set from the *last* matched program in a multi-Job file. Means: a multi-Job EluXml's top-level metadata reflects only one of the Jobs.
4. **AUF parser is one giant function** (`CreateFromArray`, ~700+ lines if pattern continues), heavy in-block state. State-machine refactor would help.
5. **Helper classes `Programm` / `Kontur` / `ZeileXxx`** are POCOs (Plain Old VB Objects) — their public surface is just properties. The parser stuffs into them; presumably a `GetText()` method on each emits AUF-format text for the inverse direction. Phase-3 follow-up to enumerate exact fields.
6. **No round-trip tests visible.** Same as `EluCadFile` — no unit-test pair in scope.

## Open questions

- **Q-050** (from MOC): `NcProgramEluXml.GetNCStringWithIaNr` is a stub. Is appending IA-numbers to EluXml programs silently broken? `#safety-relevant`
- **Q-052** (from MOC): `Job.cncdriver = "1.1elu"`. Does the post-processor branch on this?
- **Q-062 (new):** `NcProgramAuf.ComputeCycleTime` implementation not seen yet; confirm AUF machines actually contribute to `UseSbzCalculatedDuration` reporting. `#safety-relevant`
- **Q-063 (new):** `NcProgramEluXml.ReadFromString` populates top-level `Description` / `Comment` / `CNo` from the *last* matched program. Confirm intentional. `#needs-review`
- **Q-064 (new):** AUF parser splits input by `vbCrLf` only. UNIX-line-ending `.auf` files would be one giant line. Confirm Elumatec only emits CRLF. `#needs-review`

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/elumatec-ncpipeline|NC pipeline MOC]]
- [[elumatec-ncstructure-hierarchy|Job/Bar/Cut/Plane hierarchy]] — the in-memory model these serialisers ultimately encode.
- [[elumatec-machine-base|`Sbz14x`]] — `NcType` per machine variant decides which serialiser to use, and `CreateNCX` runs the external post-processor against AUF/EluXml inputs.

## Coverage

`_coverage.md`:
- `Elumatec\AufSerializer\NcProgram.vb` → `done`
- `Elumatec\AufSerializer\NcProgramAuf.vb` → `done` (covers state-machine shape; per-section helper detail deferred)
- `Elumatec\AufSerializer\NcProgramEluXml.vb` → `done`
- `Elumatec\AufSerializer\Programm.vb` → `done` (POCO)
- `Elumatec\AufSerializer\Kontur.vb` → `done` (POCO)
- `Elumatec\AufSerializer\ZeileAuftrag.vb` → `done` (POCO)
- `Elumatec\AufSerializer\ZeileProgramm.vb` → `done` (POCO)
- `Elumatec\AufSerializer\ZeileKontur.vb` → `done` (POCO)
- `Elumatec\AufSerializer\ZeileTTab.vb` → `done` (POCO)
- `Elumatec\AufSerializer\EluXmlProgram.vb` → `done` (referenced from MOC; deep cycle-time logic deferred)
- `Elumatec\AufSerializer\EluXmlProgramDetail.vb` → `done`
- `Elumatec\AufSerializer\EluXmlJob.vb` → `done`
- `Elumatec\AufSerializer\EluXmlJobItem.vb` → `done`
- `Elumatec\AufSerializer\EluXmlJobSubItem.vb` → `done`
