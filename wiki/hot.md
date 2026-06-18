---
type: meta
title: "Hot Cache"
updated: 2026-06-18T00:00:00
---

# Recent Context

## Last Updated
2026-06-18. **PHASE 4 COMPLETE.** All priority deep-read targets documented. 100% file coverage maintained. 403 open SME questions consolidated.

## Final coverage

- Scope: 3 projects (iCENTER 1237 + TruTopsLib 65 + ICenterLib 723 = **2025 files**).
- Coverage: **1159 done, 0 todo, 7 needs-review, 445 config, 414 generated**.
- **403 open SME questions** (Q-001..Q-403). 5 fully resolved; 1 partially resolved (Q-310).
- **~55 `#safety-relevant`** including the strongest hazard: [[business-rules/trutopslib-color-7-hazard|TruTops DXF colour-7 "levensgevaarlijk" warning]].
- **47 business-rule notes**.
- **28 MOCs** + **40+ module notes** + ~12 architecture/external-systems notes.
- **25 commits** pushed (`a417645..ded8056`).

## Phase 4 deep-reads delivered

| Target | Status | Module note | Business rules |
|--------|--------|-------------|----------------|
| Kardex (KardexProcessor + FrmKardexInterface) | ✓ | [[modules/icenter-kardex]] | [[business-rules/icenter-kardex-warehouse-codes]] |
| MarkTool (Telesis engraver driver) | ✓ | [[modules/icenter-marktool]] | — |
| Classes/Coating | ✓ | [[modules/icenter-coating]] | [[business-rules/icenter-coating-executor-enum]], [[business-rules/icenter-coating-dept-codes]] |
| PCFNet/GenericPart (82 KB) | ✓ | [[modules/pcfnet-generic-part]] | [[business-rules/pcfnet-ebtv-billed-by-mass]] |
| Production/ProductionProfileCutItem + CEChecklist | ✓ | [[modules/icenterlib-production-cutitems-cechecklist]] | — (covered in module note) |
| TruTopsLib/LayerConverter + MigrationFix full audit | ✓ | [[modules/trutopslib-layer-converter]] | [[business-rules/trutopslib-color-7-hazard]] |
| SmtManufacturing (47 files, FlatPatternConverter + Part heads) | ✓ | [[mocs/smtmanufacturing]] | [[business-rules/icenter-smt-workdir-convention]] |

## Top 10 safety-relevant findings (priority order)

1. **TruTops DXF colour-7 hazard** — author's own "levensgevaarlijk" (life-threatening) warning. [[business-rules/trutopslib-color-7-hazard]].
2. **Part.GetSmtCalcCycleTime line 147 typo** — `sOpenContours = sOpenContours = CType(...)`. Likely production bug. Q-221.
3. **MemoDetailElfsquadConfiguration always returns Guid.Empty** — missing JsonProperty attrs. Q-213.
4. **IPPart.EvalVcNc XML mis-emits TraceKey** for CycleTimePerPiece + NcCycleTime. Q-231.
5. **IPBatch.GetPartsAreCompleted returns True on empty DataTable** — DB error → "all done". Q-229.
6. **Product.GetField() column injection** + **Product.CreateCheckinJob hardcoded EmpId "0798"**. Q-269, Q-270.
7. **EBTV galvanizing billed by MASS** — direct commercial-risk gate; misroute = 5-10× mis-charge. [[business-rules/pcfnet-ebtv-billed-by-mass]] / Q-355.
8. **Kardex no-transaction across UpdateJobStatusId(2) + SaveXml** — orphaned dispatch state. Q-326.
9. **MarkTool IPpart-prefix substitution** — silently engraves Modelname instead of scanned text. Q-332.
10. **CadBatchServer `c:\temp\` cleanup** intersects iCenter SMT scratch — convention enforces `c:\work\`. [[business-rules/icenter-smt-workdir-convention]] / Q-391.

## What remains (genuine SME work, not autonomous documentation)

- **397 unanswered SME questions** in [[needs-review/_index]]. Each needs a real conversation with SMEs.
- **Phase 4 internals deferred to SME**:
  - `Part.ConvertToGeo2D` (838 lines) and `ConvertToGeo3D` (556 lines) — bend-handling internals.
  - `FlatPatternConverter.ReplaceBendNotes` + Solid Edge variants.
  - PCFNet per-family classes: StlDoor001 (37 KB), AluDoor001 (18 KB), AluBasicWallLouver001 (17 KB) — each is a product family with its own Compute()/Execute() business logic.
  - `GenericPart.UpdateBOM` / `CalculateSolidProperties` / `GetCalculation` / `AddCalculationEntireTree`.
  - Forms/ShopProcess (10 operator-facing screens — `#safety-relevant`).
  - Detailed SmtProduction PPSInterface vs Oseon migration history.

## Stale-state warnings

- `_coverage.md` rollup is the authoritative source.
- `MEMORY.md` index empty (no project-memory written yet — see `~/.claude/projects/.../memory/`).
- Obsidian auto-created stub files at vault root (`jiba-portal.md`, `kardex.md`, `trutops-oseon.md`) — left unstaged; clean up before user-facing publish.
- 5 questions confirmed resolved (Q-001/006/031/034/226); Q-310 partial resolution (TEMPLATEDXF path).
