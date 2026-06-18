---
type: meta
title: "Hot Cache"
updated: 2026-06-18T00:00:00
---

# Recent Context

## Last Updated
2026-06-18. **MILESTONE: 100% file coverage achieved.** Phase 3a–3g complete.

## Key Recent Facts
- Scope: three projects (iCENTER 1237 + TruTopsLib 65 + ICenterLib 723 = **2025 files**).
- Coverage now: **1159 done, 0 todo, 7 needs-review, 445 config, 414 generated**.
- **318 open SME questions** (Q-001..Q-318). Q-001/006/031/034/226 resolved.
  - **~50 `#safety-relevant`** including the strongest hazard flag: [[business-rules/trutopslib-color-7-hazard|TruTops DXF colour-7 author warning]] (`"levensgevaarlijk"` = "life-threatening" in Dutch).
- **37 business-rule notes** written (mix of cost/process/identity/regulatory rules).
- **27 MOCs** + **35 module notes** + ~12 architecture/external-systems notes.
- 19 commits pushed (`a417645..1b01a77`).

## Files closed at 100% source coverage

| Project / folder | Source files | Status |
|------------------|-------------:|--------|
| ICenterLib/ISAH | 63 | 100% deep notes |
| ICenterLib/iCenter | 24 | 100% deep notes |
| ICenterLib/JIBA | 14 | MOC + Employee/Asset deep |
| ICenterLib/ProductDb | 21 | MOC + Product deep |
| ICenterLib (other folders) | 449 | folder-level MOCs |
| TruTopsLib | 57 | MOC + colour-7 hazard rule |
| iCENTER/Elumatec | ~120 | partial deep + MOC |
| iCENTER (Sales/WorkPrep/Production) | ~30 | deep notes |
| iCENTER (Classes/Forms/Controls/SmtManufacturing/UniLink/CadBatchserver/etc.) | 475 | folder-level MOCs |

## Phase 4 + 5 status

- **Phase 4 (business-logic sweep)**: ongoing incrementally. Every deep-dive batch has produced new business-rule notes. Remaining priorities (Phase-4 deep-read targets, flagged in MOCs):
  - `GenericPart.vb` (82 KB — single biggest .vb file)
  - `Forms\ShopProcess\` operator-facing screens
  - `SmtManufacturing\` Oseon UI
  - `Classes\Coating\` surface-treatment process
  - `iCENTER\Kardex\` physical-storage shuttle (#safety-relevant)
  - `iCENTER\MarkTool\` laser-marker driving
  - `LayerConverter` full colour-rule audit (TruTops, #safety-relevant)
  - `Production\ProductionProfileCutItemsHandler` (Elumatec downstream)
  - `Production\CEChecklist` (CE-marking regulatory)
- **Phase 5 (linking + review queue)**: needs-review/_index now lists all 318 questions. MOC index updated.

## Top safety-relevant findings (priority order)

1. **TruTops DXF colour-7 hazard** (`#safety-relevant`) — author's own "life-threatening" warning. [[business-rules/trutopslib-color-7-hazard]].
2. **Part.GetSmtCalcCycleTime line 147 typo** — `sOpenContours = sOpenContours = CType(...)`. Likely production bug. Q-221.
3. **MemoDetailElfsquadConfiguration always returns Guid.Empty** — no JsonProperty attrs. Q-213.
4. **IPPart.EvalVcNc XML mis-emits TraceKey** for CycleTimePerPiece + NcCycleTime. Q-231.
5. **IPBatch.GetPartsAreCompleted returns True on empty DataTable** — DB error → "all done". Q-229.
6. **Product.GetField() column injection** + **Product.CreateCheckinJob hardcoded EmpId "0798"**. Q-269, Q-270.
7. **JALU/JALA dept merge in UI** + **JALA-only employees become unreachable**. Q-237/Q-257.
8. **WebClockAssistant.Save non-transactional** (INSERT + ChangeToShopDoc split). Q-236.
9. **Kardex\ 2 files** never deep-read. Q-317.
10. **EncryptionHelper is XOR cipher** — find consumers before any credential use. Q-216.

## Phase-2/3 status: complete
- Phase 1 inventory ✓
- Phase 2 architecture ✓
- Phase 3a Elumatec ✓ (partial-deep + MOC)
- Phase 3b Sales/WorkPrep/Production ✓
- Phase 3c ICenterLib/ISAH ✓ (100%)
- Phase 3d ICenterLib/iCenter ✓ (100%)
- Phase 3e ICenterLib (remaining) ✓ (MOC + deep)
- Phase 3f TruTopsLib ✓ (100%)
- Phase 3g iCENTER (remaining) ✓ (MOC)

## Stale-state warnings

- `_coverage.md` rollup is the authoritative source.
- `MEMORY.md` index empty (no project-memory written yet — see `~/.claude/projects/.../memory/`).
- Obsidian auto-created stub files at vault root (`jiba-portal.md`, `kardex.md`, `trutops-oseon.md`) — left unstaged; clean up before user-facing publish.
