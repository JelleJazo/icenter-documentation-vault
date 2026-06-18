---
type: moc
title: "Elumatec NC structure + emission — Map of Content"
status: draft
tags: [moc, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Elumatec NC structure + emission

## What this hub covers

The **output side** of the Elumatec pipeline: how iCenter takes the in-memory job description (already rewritten by [[elumatec-works|Works/Replacements]]) and serialises it to the NC program file that the SBZ140/141 machine actually consumes. Also the inverse: reading existing `.ecw` (EluCad work-drawing) and `.auf` / `.eluxml` files back into the in-memory model.

Three layers, in order of execution:

```
1. In-memory hierarchy        Job ─→ Bar ─→ Cut ─→ Works (List<Work>) + Planes (PlaneCollection)
                              (NcStructure/*)

2. ECW parser                 EluCadFile.ReadFromFile(path)  ← parses .ecw text files
                              (NcStructure/EluCadFile.vb)    → produces the in-memory hierarchy

3. NC program serialisers     NcProgram (abstract)
                              ├── NcProgramAuf      → .auf    (older, section-block text)
                              └── NcProgramEluXml   → .eluxml (newer, XML)
                              (AufSerializer/*)
```

Connection to the rest of Elumatec:

- The in-memory hierarchy is the *output* of [[elumatec-works|Works/Replacements]] — replacements mutate `Cut.Works`.
- `Sbz14x.CreateNCX` ([[elumatec-machine-base]]) spawns Elumatec's external post-processor `NcxExePath` against `.ncw` files. The serialiser pipeline in this sub-MOC produces the *inputs* to those external runs (NCW / AUF / EluXml).
- The format choice (`NcType.Auf` vs `NcType.EluXml`) is a per-machine-variant property: SBZ140/141 ALU + STL + RVS all default to AUF; future EluXml machines would set `NcType = EluXml`. Detection at read time uses `Content.StartsWith("<?xml")`.

## Scope

`iCENTER\Elumatec\NcStructure\` (6 files, ~80 KB) and `iCENTER\Elumatec\AufSerializer\` (14 files, ~120 KB). Despite the name, `AufSerializer\` hosts **both** the AUF and the EluXml serialisers — only the older one matches the folder name.

## Files

### `NcStructure\` — in-memory model + ECW parser

| File | KB | Documented in |
|------|---:|---------------|
| `Cut.vb` | 40 | [[../modules/elumatec-ncstructure-hierarchy]] |
| `Bar.vb` | 20 | [[../modules/elumatec-ncstructure-hierarchy]] |
| `Job.vb` | 14 | [[../modules/elumatec-ncstructure-hierarchy]] |
| `Plane.vb` | 4 | [[../modules/elumatec-ncstructure-hierarchy]] |
| `PlaneCollection.vb` | 2 | [[../modules/elumatec-ncstructure-hierarchy]] |
| `EluCadFile.vb` | 14 | [[../modules/elumatec-elucadfile]] |

### `AufSerializer\` — NC program serialisers (AUF + EluXml)

| File | KB | Role | Documented in |
|------|---:|------|---------------|
| `NcProgram.vb` | 4 | abstract base + factory (auto-detects format) | [[../modules/elumatec-nc-program-family]] |
| `NcProgramAuf.vb` | 27 | `.auf` reader/writer (section-block text format) | [[../modules/elumatec-nc-program-family]] |
| `NcProgramEluXml.vb` | 5 | `.eluxml` reader/writer (XML format) | [[../modules/elumatec-nc-program-family]] |
| `Programm.vb` | 10 | AUF "Programm" block | [[../modules/elumatec-nc-program-family]] |
| `Kontur.vb` | 3 | AUF "Kontur" block | [[../modules/elumatec-nc-program-family]] |
| `ZeileAuftrag.vb` | 3 | AUF order-line | [[../modules/elumatec-nc-program-family]] |
| `ZeileProgramm.vb` | 9 | AUF program-line | [[../modules/elumatec-nc-program-family]] |
| `ZeileKontur.vb` | 4 | AUF contour-line | [[../modules/elumatec-nc-program-family]] |
| `ZeileTTab.vb` | 2 | AUF tool-table-line | [[../modules/elumatec-nc-program-family]] |
| `EluXmlProgram.vb` | 13 | EluXml `<Program>` element | [[../modules/elumatec-nc-program-family]] |
| `EluXmlProgramDetail.vb` | 11 | EluXml `<Program>` detail | [[../modules/elumatec-nc-program-family]] |
| `EluXmlJob.vb` | 1 | EluXml `<Job>` | [[../modules/elumatec-nc-program-family]] |
| `EluXmlJobItem.vb` | 1 | EluXml job item | [[../modules/elumatec-nc-program-family]] |
| `EluXmlJobSubItem.vb` | 3 | EluXml job sub-item | [[../modules/elumatec-nc-program-family]] |

## Business rules surfaced in this batch

- **Cycle-time calculation feeds back to ISAH planning.** `NcProgramEluXml.ComputeCycleTime` iterates `EluXmlProgram`s, sums their times, and returns a `Double`. The global [[../architecture/global-state|`UseSbzCalculatedDuration = True`]] in `Modules\Main.vb` switches iCenter to prefer this calculated duration over ISAH's. **`#safety-relevant`** (drives production planning).
- **`Bar.BSuppl = "JAZO"`** hard-coded in NCW output when `NcwExportProfile.IncludeBSuppl = True`. Embeds the company name into the bar header. Cosmetic, but a hard-coded string nonetheless.
- **`Job.cncdriver = "1.1elu"`** hard-coded for ECW output; `"1.1"` (no `elu`) for NCW output. Driver-version marker. `#needs-review` — does the post-processor key off this value?

## Surprises worth flagging

1. **`NcProgramEluXml.GetNCStringWithIaNr` is a stub.** Returns the input string unchanged. The real implementation is commented-out (lines 65–83). Means: appending IA-numbers (likely order codes) to NC programs **works for AUF but is silently a no-op for EluXml**. `#needs-review` — Q-050.

2. **`EluCadFile.GetDoubleValue` falls back to `DataTable.Compute(expression)`** for non-numeric strings. So a depth-table cell or property may be an arithmetic expression (e.g. `2*Var0+5`). Powerful but undocumented; the only hint is `'CB 2024-08-19: replaced .Replace("/", "").Trim by EvaluateExpression`.

3. **Empty-line terminates a Work block** in the ECW parser (`EluCadFile.vb` line 244). Brittle: a stray empty line inside `[BEGIN_AUFTRAG]` block would split it.

4. **`PlaneCollection.GetPlaneByWSide(WSide)` indexes `WSide - 7`** (line 58). The six standard side enum values (Top=1, Front=2, …, Bottom=6) map to **negative indices** and return `Nothing`. The method only retrieves *custom* planes for WSide ≥ 7 (Free + user-added). Misleading API. `#needs-review` — Q-051.

5. **`oEluCad As New EluCadApp` re-constructed** in `Bar.GetEcwText`, `Bar.GetNcwText`, `Bar.CalcPartCode`, `Job` (via private field), `ClsComWatcher`, `FrmComWatcher`, etc. Each construction re-initialises the profile-DB DataTable schema. Already flagged as a pattern in the [[elumatec-com-watcher|COM watcher note]].

6. **Stringly-typed German section names** in the AUF parser (`[BEGIN_AUFTRAG]`, `BEZEICHNUNG`, `SOLL_STCK`, `BEARB_PROGRAMM`, `STABLAENGE`). Direct copy from the Elumatec file-format spec. Domain glossary for non-German readers needed.

7. **`Cut` implements `ICloneable`** — replacements use this when they need an "original" copy of a Work (see [[elumatec-replacement-flowdrill|Flowdrill]] `OriginalWork = Work.Clone`). Deep-vs-shallow semantics of `Cut.Clone()` not visible from the file header; Phase-3 follow-up.

## Open questions

- **Q-050:** `NcProgramEluXml.GetNCStringWithIaNr` is a stub (returns input unchanged). Is appending IA-numbers to EluXml programs silently broken? `#safety-relevant`
- **Q-051:** `PlaneCollection.GetPlaneByWSide(WSide) → WSide - 7` only handles custom planes (WSide ≥ 7). Standard sides 1-6 return `Nothing`. Intentional or bug? `#needs-review`
- **Q-052:** `Job.cncdriver = "1.1elu"` (ECW) vs `"1.1"` (NCW). Does the Elumatec post-processor branch on this value? `#needs-review`
- **Q-053:** `EluCadFile.GetDoubleValue` accepts arithmetic expressions via `DataTable.Compute`. Confirm intent — and document the expression grammar SMEs can rely on. `#safety-relevant`
- **Q-054:** Empty-line-terminates-Work semantics in the ECW parser. Confirm that intentional and that the Elumatec writer always emits a trailing blank line. `#needs-review`

Logged in [[../needs-review/_index]].

## Related

- [[elumatec|Elumatec subsystem MOC]]
- [[elumatec-works|Works MOC]] — produces the in-memory hierarchy consumed here
- [[../modules/elumatec-machine-base]] — runs the external post-processor against the output files
- [[../modules/elumatec-cad-app]] — owns the profile DB used at every serialisation step
