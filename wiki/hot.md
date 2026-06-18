---
type: meta
title: "Hot Cache"
updated: 2026-06-18T00:00:00
---

# Recent Context

## Last Updated
2026-06-18. Phase 3c-6: ISAH TimeRegistration deep-dive landed.

## Key Recent Facts
- Scope: three projects (iCENTER 1237 + TruTopsLib 65 + ICenterLib 723 = **2025 files**).
- Coverage now: **88 done, 9 needs-review, 1069 todo, 445 config, 414 generated**.
- 196 open SME questions (Q-001, Q-006, Q-031, Q-034 resolved). **64 are `#safety-relevant`**.
- 24 business-rule notes (15 safety-relevant).
- ICenterLib coverage: 32/723 done. ISAH coverage: **31/65 (48%)**.

## TimeRegistration summary (this batch)
- **TimeRegistration.vb (1274 lines)** — biggest single ISAH file. Wraps the SP-driven time-reg writers (`IP_ins_TimeRegistr`, `IP_Upd_TimeRegistr_v2`, `IP_Ins_PBOOEmployee`) plus DTR-status conversion, assistant cascading, "moving employee" handoff, and combined-line generation for sheet-metal/Oseon jobs.
- **TimeRegCollector.vb (147 lines)** — batch helper. Three buffers (`SetStartedList`, `SetFinishedList`, `TimeRegTable`); `ProcessCollections()` flushes them with 1-second pacing between each ISAH write.
- **5 new business-rule notes**:
  - [[business-rules/isah-dtr-status-codes]] — pre-migration `AO/AW/IO/IW/II/OO/OW/OI` 2-char state machine. `#dead-code` in production today (`UseIsahNoDtrTimeReg = True`).
  - [[business-rules/isah-hourcodes]] — `"01"` cycle / `"02"` setup / `"AW"` presence. Setup time and cycle time are written as **separate rows**.
  - [[business-rules/isah-timereg-minute-granularity]] — `GetCurrentTimeInSeconds` rounds Second to 0 **by design** (commented-out `TotalSeconds` alternative).
  - [[business-rules/isah-timereg-write-pacing]] — `TimeRegCollector.TempDelayForSql = 1000` 1-second pacing between writes; likely a workaround for the minute-granularity collision behaviour.

## Notable findings (multiple `#safety-relevant`)
- **Q-182**: `ChangeToShopDoc` refuses to act if `Employee.GetIsPresent` is false. Combined with [[modules/isah-identity|`Employee.GetIsObsolete`]] fail-closed (Q-133), an ISAH outage prevents *all* shop-floor clocking.
- **Q-183**: `ChangeToShopDoc`'s outer `Try…Catch` swallows all exceptions — partial mutation (PBOOE deleted but time-reg insert failed) leaves the DB inconsistent and the caller unaware.
- **Q-184**: `ChangeToShopDoc` silently substitutes the supplied `ShopDocCode` if MachGrp-resolution finds a different one. Logs Info; no caller callback.
- **Q-189**: `CreateCombinedTimeRegLines` forces `StartDate = 00:01` then walks forward through overlapping rows. The time-reg timestamp doesn't reflect when work actually happened — only the cycle-time *amount* is preserved. Payroll-audit alignment concern.
- **Q-191**: `TimeRegCollector.ProcessCollections` doesn't roll back. A `SetStarted` failure doesn't prevent `SetFinished` from running — a ShopDoc could end up finished without ever being started.
- **EmpId starts with `"S"` → special-case** `DeleteNextDayTimeRegLines` (Q-186 — what does `S` mean?).
- **`MachGrpCode = "T40"` → silently rewritten to `"P02"`** for the debugger-employee path. Production-side fix for a debug fixture (Q-180).
- **`TimeRegInputType = 1`** with `RR 20220725 Fix voor ontbrekende JournaalPosten? Waarde was 2` — fix value was 2, changed to 1. No regression test (Q-190).
- **TimeRegistration uses `Common.APPLISAHUSERCODE = "ICENTER"`** for writes — *correct* — while production/dossier classes use `"ISAH"`. Drift (Q-185 + Q-165).
- **`TimeRegCollector.SetFinished`** is the live path that calls `SetShopDocFinInd(True)` — the same call that's *commented out* in `OutsourceOperationsHandler` (Q-094 connection).

## Recent Changes
- Created [[modules/isah-time-registration]] grouped module note (2 files).
- Created 4 new business-rule notes.
- Opened Q-179..Q-196 (18 new questions, 6 `#safety-relevant`).
- Updated [[_coverage]] (+2 done in ICenterLib/ISAH); rollup totals.
- Fixed a mojibake bug in _coverage.md (em-dash had been corrupted to `â€”` in an earlier write).

## Active Threads
- Recommended next ISAH batches:
  1. **Icenter2Isah** (38 KB) — the sync layer between iCenter2 (out-of-scope sibling) and ISAH. Cross-cutting findings expected.
  2. **ISAH small leaves** — ~20 small files: Design (7 KB), CallRegistration (5 KB), FrmJConfigParamDesignCode (8 KB) + JConfigParam (8 KB), CustomerSelection (8 KB), CustomerRelation, Database, DateDimension, MultiFinance, WorkView, IsahFieldML, DeliveryLine, PurchaseDocumentPartLine, Language, plus DataServices/ remaining 6, Helpers/ remaining 5, ViewModels/ 2.
- After ISAH saturates, move to ICenterLib/iCenter (29 files: ProductionMachines 41 KB, IPPart 19 KB, IPBatch 13 KB, IPOrder 7 KB, Client).

## Notes from working tree
- Three Obsidian auto-stubs at wiki root (`jiba-portal.md`, `kardex.md`, `trutops-oseon.md`) and `.obsidian/` autoupdates remain unstaged.
- `architecture/external-surface.md` user-reformatted (table layout changed) — kept as-is.
