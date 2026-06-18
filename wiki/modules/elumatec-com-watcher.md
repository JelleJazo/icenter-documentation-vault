---
type: module
title: "Elumatec saw COM-port watcher"
status: needs-review
module: "iCENTER/Elumatec"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\FrmComWatcher.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\ClsComWatcher.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\CRs232.vb"
last-reviewed: 2026-06-18
tags: [module, entry-point, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Elumatec saw COM-port watcher

## Purpose

Listens on **serial port COM1** for line-number acknowledgements emitted by the **profile saw** (a separate physical machine from the SBZ140/141 profile mill — `MachineId = 2` in iCenter's `prodMachines` table). Updates the `LatestLineNr` column for that machine so iCenter can track which DGX row was last sawn.

Reaches the user only as one of the post-`FrmMain` modes: `Modules\Main.vb` opens `Elumatec.FrmComWatcher` after `FrmMain.Dispose()` *iff* the `ComWatcherMode` global was flipped to `True` inside `FrmMain`. See [[../architecture/entry-points]] and [[../architecture/runtime-modes]].

## Public surface

| Symbol | Kind | Used by |
|--------|------|---------|
| `FrmComWatcher.New()` | constructor | `Modules\Main.vb` `Sub Main` (line 250–251) |
| `FrmComWatcher.WindowText` | const = "iCenter COM-watcher" | window title |
| `ClsComWatcher.New(ProdMachineId)` | constructor | `FrmComWatcher.New` calls with `2` |
| `ClsComWatcher.Update(RowNr)` | sub | wired to `moRS232_CommEvent` and to the form's manual "Cut" button |
| `ClsComWatcher.Close()` | sub | `FrmComWatcher.FormClosing` |
| `Rs232` (in `CRs232.vb`) | class | third-party — Corrado Cavalli, codeworks.it (2001–2005). Implements `IDisposable`. Wraps Win32 `CreateFile`/`ReadFile`/`WriteFile` via P/Invoke. |

## Behavior in plain language

When the form opens, it eagerly constructs an `EluCadApp` (likely unnecessary — see Surprise #1), then a `ClsComWatcher` for machine ID 2. The watcher:

1. Opens **COM1** at 9600 baud, 8-N-1, 1500ms read timeout (`ClsComWatcher.New` lines 17–24). Sets DTR/RTS high. Enables event-driven RX.
2. On each `CommEvent` with `RxChar` set, reads `source.InputStreamString` and matches one of:
   - Exactly `"ID"` → arm `AcceptId = True` for the next message.
   - When armed → parse the message as `Integer`; on success call `Update(MyId)`. Disarm.
   - Single-line containing `"ID"` (the saw sometimes runs token + number on one line) → take the substring after `"ID"`, parse it.
3. `Update(RowNr)` calls `prodMachines.UpdateLatestLineNr(2, RowNr)`.

The form also exposes a manual **"Cut" button** that simulates one of these events: read a line number from a textbox, call `oWatcher.Update`, and increment the textbox.

## Surprises (read these first)

1. **`oELUCAD = New EluCadApp` is constructed twice** — once on the form, once inside `ClsComWatcher`. Neither field is used in any visible code path; almost certainly vestigial.
2. **The original feedback loop is commented out** (`ClsComWatcher.vb` lines 95–120). The Update originally did:
   - Look up `ActiveICenterBatchId` for the saw machine
   - Compute remaining quantity for the just-sawn row's `Pos`/`Box`/`Barcode` triple
   - **If this was the first cut for that Pos, print a Dymo sticker via `ClsDgxStickerPrinter`**
   - Toggle `oSawMachine.UpdateStickerPrinted(MachineId, ...)`
   Now it just records the line number. **`#needs-review`** — confirm whether this regression was intentional or whether something else now does the sticker print.
3. **Critical-level log entry on every form open:** `oApplicationLog.NewEntry("frmComWatcher used!", MsgBoxStyle.Critical) 'CB 2023-01-27: added tracking` (`FrmComWatcher.vb` line 23). The developer used a `Critical` log level as a usage tracker — strong hint that they suspected the form might be dead code and wanted hard evidence.
4. **`CRs232.vb` is third-party copy-paste.** The header comment block (lines 9–117) is the original `Rs232` class by Corrado Cavalli, ©2003. 24 revisions through April 2005, then no more changes. Win32-only P/Invoke. Implements `IDisposable`, threaded RX/TX, overlapped I/O. Unmaintained. **`#needs-review`** — if there's a serial bug in production, this is the suspect.
5. The watcher only handles **`MachineId = 2`** (hard-coded in `FrmComWatcher.New`). If the saw moves to a different machine id, this breaks silently.
6. **Single-instance assumption.** Only one `FrmComWatcher` can be open per process; it opens COM1 exclusively. If two iCenter processes try to watch the same port, one will fail silently (the `Catch` in `ClsComWatcher.New` only logs).

## Business rules surfaced here

- The saw sends `"ID"` + numeric line number for each completed cut. Format is brittle (token detection by string match, not protocol). This *is* the contract with the saw's controller. `#safety-relevant` `#needs-review` — confirm with SME that this is the actual protocol expected from the saw vendor.
- Machine ID 2 = the saw. Hard-coded.

## External systems touched

- [[../external-systems/elumatec-sbz140|Profile saw]] (separate from the SBZ140/141 mill) — serial COM1. `#safety-relevant`
- [[../external-systems/icenter-db|iCenter DB]] via `prodMachines.UpdateLatestLineNr`.

## Domain concepts

- [[../domain-concepts/_index|DGX]], [[../domain-concepts/_index|Sawn-bar tracking]], [[../domain-concepts/_index|MachineId]].

## Open questions

- **Q-006 — RESOLVED 2026-06-18**: confirmed the form is a serial-COM watcher for the saw.
- **Q-019 (new):** is the commented-out sticker-print pipeline replaced by something else, or simply turned off? If off, is it expected to stay off?
- **Q-020 (new):** the third-party `Rs232` class is unmaintained since 2005. Patch level?
- See full set in [[../needs-review/_index]].

## Coverage

`_coverage.md`:

- `Elumatec\FrmComWatcher.vb` → `done` (this note + Q-019 flag)
- `Elumatec\ClsComWatcher.vb` → `done` (this note + Q-019 flag)
- `Elumatec\CRs232.vb` → `done` (third-party, this note + Q-020 flag)
