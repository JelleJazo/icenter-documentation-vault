---
type: meta
title: "Hot Cache"
updated: 2026-06-18T00:00:00
---

# Recent Context

## Last Updated
2026-06-18. Phase 3c-1: ICenterLib survey + foundation files landed.

## Key Recent Facts
- Scope: three projects (iCENTER 1237 + TruTopsLib 65 + ICenterLib 723 = **2025 files**).
- Coverage now: **59 done, 7 needs-review, 1100 todo, 445 config, 414 generated**.
- 131 open SME questions (Q-001, Q-006, Q-031, Q-034 resolved). **53 are `#safety-relevant`**.
- 14 business-rule notes (10 safety-relevant).
- ICenterLib opened: top-MOC + 3 root-file deep-dives (Common, Connections, AppSettings). 3/723 done.

## ICenterLib summary (this batch)
- **35 top-level folders catalogued** in [[mocs/icenterlib]]. Biggest: SmtProduction (128), CAD (127), Resources (75), ISAH (65), PCFNet (45), UserControls (34), ProductDb (31), iCenter (29), DataHandler (23), Production (23), MySystem (22), CadBatchServer (17), JIBA (14).
- **[[modules/icenterlib-common]]** — 533-line static-helper class. Defines IA/IAK/PRN part-code prefixes, `GuestEmpId = "0000"`, 40-49 work-view status range, terminal-server/RAS/ADS hostname branching, `IsSharedWindowsAccount` recognises only `"PVS"`. **Common.GetTableData uses `EXEC('SELECT * FROM ' + @TableName)`** (Q-109, `#safety-relevant`).
- **[[modules/icenterlib-connections]]** — 15 connection factories. **All hard-coded creds in source**: iCenter / JIBA / Windchill / ProductDb / Isah / TruTops Oseon all share password `p2yeXeC7`; ISAH `sa` login uses `koyTRedgh&*(kl:[`; Kardex uses `Kardex/K@rdex951`; ZeroCode service has API key + base64 password as constants. **Q-107, Q-108, Q-118, Q-119** all `#safety-relevant`. `ConnectICenter2Development` connects to raw IP `10.11.70.32` as developer `sander-h`.
- **[[modules/icenterlib-appsettings]]** — wraps the iCenter DB `T_ApplicationSettings` table via `SIP_GetAppSetting` SP. `IsInsideMaintenanceWindow` reads `My.Settings` of the *consuming* assembly (Q-112). `SmtDeburrSpeed = 0.225 / 60` m²/sec is a `Public Const` here (process-relevant constant in the wrong file — Q-122).
- **4 new business-rule notes**: [[business-rules/icenter-part-code-prefixes]], [[business-rules/icenter-status-code-default-range]], [[business-rules/icenterlib-maintenance-window]], [[business-rules/smt-deburr-speed]].
- **`Connections.UseIsahTestDb`** is a `Public Shared` field (process-mutable). Switches all iCenter→ISAH connections to test DB. Single global toggle.

## Recent Changes
- Created [[mocs/icenterlib]] top-MOC catalogueing all 35 folders + roadmap.
- Created 3 module notes: [[modules/icenterlib-common]], [[modules/icenterlib-connections]], [[modules/icenterlib-appsettings]].
- Created 4 business-rule notes (one of which — `smt-deburr-speed` — is the first business-rule note for a non-Elumatec subsystem).
- Opened Q-107..Q-131 (25 new questions, 11 `#safety-relevant`). Most consequential: Q-107 (hard-coded DB creds), Q-119 (sa login), Q-118 (dev IP in source), Q-109 (EXEC SQL pattern), Q-114 (SQL string concat).

## Active Threads
- Recommended next batches per the ICenterLib roadmap in [[mocs/icenterlib]]:
  1. **ISAH/** (65 files) — every Sales/WorkPreparation/Production note already references `oISAH`. Highest-priority subsystem. Will need a sub-MOC + ~10-15 module notes covering the major entities (TimeRegistration 72 KB, Part 49 KB, ProductionHeader 45 KB, Icenter2Isah 38 KB, PBOO 31 KB, DossierDetail 28 KB, BillOfMat 26 KB, DossierMain 19 KB, Employee 17 KB, PBOS/PBOM 11+10 KB, BillOfOper 9 KB, ShopDoc 8 KB).
  2. **iCenter/** (29 files) — `ProductionMachines` (41 KB), `IPPart` (19 KB), `IPBatch` (13 KB), `Client`, `IPOrder`, `Servicedesk`, etc.
  3. **SmtProduction/** (128 files) — TruTops Oseon plumbing. Needs a sub-MOC.
  4. **CAD/** (127 files) — Creo + Windchill PLM. Needs a sub-MOC.

## Notes from working tree
- Three Obsidian auto-stubs at wiki root (`jiba-portal.md`, `kardex.md`, `trutops-oseon.md`) and `.obsidian/` autoupdates remain unstaged.
- `architecture/external-surface.md` user-reformatted (table layout changed) — kept as-is.
