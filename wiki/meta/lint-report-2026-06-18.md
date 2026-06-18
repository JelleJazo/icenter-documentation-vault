---
type: meta
title: "Lint Report 2026-06-18"
created: 2026-06-18
updated: 2026-06-18
tags: [meta, lint]
status: post-auto-fix
---

# Lint Report — 2026-06-18

Wiki health check after Phases 1–4 (full file coverage + business-logic deep-reads).

> **Update 2026-06-18 (post-auto-fix)**: L-1 frontmatter fix applied, H-3 reclassified as false-positive (Markdown-table pipe escapes), M-1 stubs created. Dead-link count **57 → 30** (-27, 47% reduction). See "Post-Fix Status" at the bottom.

## Transport

- Filesystem (no `transport.json`; no MCP; no Obsidian CLI configured).
- DragonScale addresses: **not in use** — Address Validation skipped.
- Semantic tiling: **not in use** (no ollama / no `scripts/tiling-check.py`) — Semantic Tiling skipped.

## Summary

| Metric | Value |
|--------|------:|
| Pages scanned (`wiki/**/*.md` excl. `_templates/`) | **134** |
| Distinct page filenames | 128 (6 intentional `_index.md` collisions across folders) |
| Wikilinks parsed | 2 696 |
| Distinct link targets | 185 |
| **Orphan pages** | **0** ✅ |
| **Dead links** | **57** (across the 185 targets) |
| **Empty files** | **8** ⚠ (auto-created Obsidian stubs at vault root) |
| **Empty sections** | 0 ✅ (false-positive after sub-heading-aware check) |
| **Frontmatter gaps** | 1 (`hot.md` missing status/tags/created) |
| Index integrity (`wiki/index.md`) | 1 dead link (`CLAUDE`) |
| `needs-review/_index.md` integrity | 0 dead links across 108 internal refs ✅ |

**Severity rollup**: 0 BLOCKERs · 4 HIGH · 4 MEDIUM · 3 LOW

---

## HIGH — fix before next bulk ingest

### H-1. Empty Obsidian-stub files at vault root (8)

Earlier sessions flagged these in [[../hot]]: Obsidian auto-created zero-byte files when wikilinks were clicked in the editor.

| File | Referenced by |
|------|---------------|
| `wiki/cad-batchserver.md` | (none — vestigial) |
| `wiki/design-comments.md` | (none — vestigial) |
| `wiki/elumatec-sbz140.md` | (none — vestigial) |
| `wiki/jiba-portal.md` | (none — vestigial) |
| `wiki/kardex.md` | (none — vestigial) |
| `wiki/smt-manufacturing.md` | (none — vestigial) |
| `wiki/trutops-oseon.md` | (none — vestigial) |
| `wiki/windchill.md` | (none — vestigial) |

> They satisfy 0 inbound links (so they don't show as orphans because no page references the **bare-name** version — references use `[[../external-systems/oseon]]` etc.).

**Recommendation**: **delete all 8**. They have no content, no inbound links, and represent the canonical external-systems pages that DO need to be created (see M-1 below) — but at the proper paths (`wiki/external-systems/oseon.md`, etc.), not at the vault root.

### H-2. Dead link `CLAUDE` (7 refs)

Wikilinks like `[[../CLAUDE]]` and `[[../../CLAUDE]]` appear in 7 pages pointing at the **project-instructions file** (`C:\DevOps\icenter-vault\CLAUDE.md`) rather than a wiki page.

Sources: `index`, `log`, `overview`, `_coverage`, `build-and-deploy`, `project-references`, `_index` (architecture).

**Recommendation**: replace the wikilinks with Markdown links to the actual file path: `[CLAUDE.md](../../CLAUDE.md)` (paths relative to each page). Obsidian will still render them as clickable; the wiki-graph won't report them as dead.

### H-3. Backslash-typo wikilinks (4)

| Bad target | Should be | Source(s) |
|------------|-----------|-----------|
| `icenter-status-code-default-range\` | `icenter-status-code-default-range` | (1 page) |
| `icenterlib-appsettings\` | `icenterlib-appsettings` | (1 page) |
| `_index\` | `_index` | (1 page) |
| `isah-identity\` | `isah-identity` | (1 page) |

**Recommendation**: search-and-replace `\\]]` → `]]` across the wiki. Safe to auto-fix.

### H-4. Six `_index.md` filename collisions (architectural — intentional)

Each folder has its own `_index.md`:

- `wiki/architecture/_index.md`
- `wiki/business-rules/_index.md`
- `wiki/domain-concepts/_index.md`
- `wiki/external-systems/_index.md`
- `wiki/mocs/_index.md`
- `wiki/modules/_index.md`
- `wiki/needs-review/_index.md`

Obsidian resolves these by path. **Not a bug**, but bare `[[_index]]` wikilinks could in theory be ambiguous. Searched the wiki: every `[[_index]]` reference uses a folder-prefixed form (`[[../mocs/_index]]`, etc.). **No action needed**, just noted.

---

## MEDIUM — schedule for next housekeeping pass

### M-1. External-system stubs missing (24 distinct targets, ~50 link references)

The wiki references 24 external systems that have no `wiki/external-systems/<name>.md` stub. Most are referenced by `external-systems/_index.md`, `architecture/external-surface.md`, and the main MOCs.

| External system | Refs | Suggested page |
|-----------------|-----:|----------------|
| `isah` | 11 | `wiki/external-systems/isah.md` |
| `icenter-db` | 9 | `wiki/external-systems/icenter-db.md` |
| `dfs-share` | 4 | `wiki/external-systems/dfs-share.md` |
| `oseon` | 3 | `wiki/external-systems/oseon.md` |
| `creo` | 3 | `wiki/external-systems/creo.md` |
| `dymo-label` | 3 | `wiki/external-systems/dymo-label.md` |
| `elfsquad` | 3 | `wiki/external-systems/elfsquad.md` |
| `trutops` | 2 | `wiki/external-systems/trutops.md` |
| `boost-pps`, `solid-edge`, `webclock`, `zammad`, `xwiki`, `pdf-xchange`, `jmail-launcher`, `ghostscript`, `active-directory`, `jconfigurator`, `elumatec-dg`, `icenter2-db`, `sola-data-connector`, `outlook` | 2 each | one stub each in `wiki/external-systems/` |
| `telesis`, `web-clock` (likely dup of webclock) | 1 each | one stub each |

**Recommendation**: stub each in `wiki/external-systems/` with frontmatter (`type: external-system`, `status: stub`, `tags: [external-system, needs-content]`) and a one-line description. SME can fill in deployment details, contact, URL later. **Safe to auto-create stubs**.

### M-2. Folder-MOC stubs missing (18 distinct targets)

`overview.md` and `_index.md` (mocs) reference folder-level MOCs that don't exist as pages:

- `work-preparation`, `production`, `engineering`, `sales` (referenced in `overview.md` / `_index.md`)
- `cad`, `cam`, `forms`, `controls`, `classes`, `data-migration`, `comparers`, `batchserver`, `mark-tool`, `pcf-net-studio`, `uni-link`, `vent-duct-configurator`, `ic-importer`, `modules-folder`

Most are already covered under one of:
- `mocs/office-to-shopfloor.md` (sales/engineering/workprep/production)
- `mocs/icenter-remaining.md` (forms/controls/classes/data-migration/etc.)
- `mocs/icenterlib-cad.md`, `mocs/icenterlib-pcfnet.md`, etc.

**Recommendation**: don't create new pages — instead, **update the linking pages** to point at the existing canonical MOCs (e.g., replace `[[work-preparation]]` with `[[office-to-shopfloor]]` and `[[classes]]` with `[[icenter-remaining]]`). Mostly in `wiki/overview.md` and `wiki/mocs/_index.md`. Needs review (the mapping is judgment).

### M-3. Template-type-name wikilinks (6 leakage cases)

`[[business-rule]]`, `[[module]]`, `[[domain-concept]]`, `[[Note Name]]`, `[[name]]`, `[[wikilinks]]` — these are template placeholder strings that leaked into actual pages.

Sources: `log`, `_index` (business-rules), plus a few others.

**Recommendation**: grep the wiki for these patterns and either remove the wikilink (turn into plain text) or fix the target. Needs review.

### M-4. Future-page references (4)

| Target | Source |
|--------|--------|
| `icenter-flowgrill-quality` | `icenter-smt-deburr-cycle-time` |
| `icenterlib-sub-services` | `icenter-coating-dept-codes` |
| `isah-coating-selection-codes` | (1 ref) |
| `hour-codes` | `icenterlib-icenter-identification` (mentioned as TODO domain-concept) |

**Recommendation**: leave for now (intentional forward-references); convert to TODO list. Phase-5 follow-up.

---

## LOW — defer / housekeeping

### L-1. Frontmatter gap in `hot.md`

```
wiki/hot.md  missing: status, tags, created
```

`hot.md` is a meta page with `updated:` set but lacks `status`/`tags`/`created`. **Safe to auto-fix** by adding `type: meta`, `status: active`, `tags: [meta]`, `created: 2026-06-18`.

### L-2. Naming convention divergence (informational only)

The wiki uses **kebab-case** filenames (e.g., `icenter-coating.md`, `pcfnet-generic-part.md`) throughout, NOT the Obsidian-style **Title Case with spaces**. This is an **intentional project convention** matching the source-folder naming (cf. CLAUDE.md `## Obsidian conventions`). All wikilinks accordingly use the kebab-case names. **No fix needed**; documented here so future contributors don't "correct" file names back to Title Case.

### L-3. needs-review index target validity (informational)

`wiki/needs-review/_index.md` has **108 internal `[[...]]` references**, all of which resolve. Most reference the canonical module + business-rule notes. The index is the SME work-queue and is the largest single file in the wiki — its link health is critical and is currently **perfect**.

---

## Naming Conventions audit

| Element | This wiki | Standard convention |
|---------|-----------|---------------------|
| Filenames | kebab-case (`icenter-coating.md`) | Title Case with spaces |
| Folders | lowercase plural (`modules/`, `business-rules/`) | lowercase with dashes ✅ |
| Tags | lowercase, hierarchy-free (`#safety-relevant`, `#needs-review`) | hierarchical (`#domain/architecture`) |
| Wikilinks | match filename | match filename ✅ |

Two divergences from the lint-skill defaults are **deliberate project choices**:
1. **kebab-case filenames** (matches code-folder naming).
2. **Flat tags** instead of hierarchical (per CLAUDE.md fixed-set: `#business-rule #domain-concept #external-system #entry-point #dead-code #needs-review #safety-relevant`).

No action needed — flagged so the lint baseline reflects the actual convention.

---

## Writing style spot-check (sample)

Scanned 5 pages at random for style-guide compliance:

| Page | Declarative? | Source-cited? | Uncertainty flagged? |
|------|-------------|---------------|----------------------|
| `modules/pcfnet-generic-part.md` | ✅ | ✅ (line numbers throughout) | ✅ (Q-### markers) |
| `business-rules/trutopslib-color-7-hazard.md` | ✅ | ✅ (verbatim author comment) | ✅ |
| `mocs/smtmanufacturing.md` | ✅ | ✅ | ✅ |
| `business-rules/icenter-kardex-warehouse-codes.md` | ✅ | ✅ | ✅ |
| `modules/icenter-marktool.md` | ✅ | ✅ | ✅ |

Style is consistent. The `Q-###` pattern serves the role of `> [!gap]` flags throughout.

---

## Suggested auto-fix order (ask before each)

1. **Safe to auto-fix without review** (run as a batch):
   - **L-1**: add missing frontmatter fields to `wiki/hot.md`.
   - **H-3**: search-and-replace 4 backslash-typo wikilinks.
   - **M-1 (stubs only)**: auto-create 24 minimal external-systems stubs (`type: external-system`, `status: stub`, `tags: [external-system, needs-content]`, 1-line description).
2. **Needs human review**:
   - **H-1**: delete the 8 Obsidian-auto-stub files at vault root.
   - **H-2**: convert `[[CLAUDE]]` wikilinks to Markdown file links.
   - **M-2**: re-target 18 folder-MOC wikilinks to the canonical existing MOC pages.
   - **M-3**: clean up 6 template-leakage wikilinks.
3. **Defer to Phase 5**:
   - **M-4**: 4 future-page references.

---

## Recurring patterns worth process changes

1. **External-system stubs were promised in Phase 2 (`external-systems/_index.md` has a "Pages (stubs to populate during Phase 3)" empty section) but never created.** Process: when a wikilink references an external system, the linker should immediately create a stub.
2. **Obsidian auto-stub-creation at vault root** when users click un-resolved wikilinks. Process: educate that auto-created files at the root must be moved to the correct folder OR the wikilink fixed to its proper folder-prefixed form.
3. **6 backslash-typo wikilinks** suggest a copy-paste-from-coverage-table pattern that included the trailing markdown `\` line-continuation. Process: when copying paths into wikilinks, strip trailing whitespace and slashes.

---

## Post-Fix Status (2026-06-18)

### Applied

- ✅ **L-1**: `wiki/hot.md` frontmatter completed (`status`, `tags`, `created` added).
- ⏭️ **H-3**: Re-scanned — these are NOT typos. The `\|` inside `[[isah-identity\|`Company`]]` is the **Markdown-table pipe escape**, required so the pipe doesn't break the table column. Obsidian honours the escape and resolves the link target correctly. **Lint detector was wrong** — the regex `[^\]\|#]+?` ate the backslash as if it were part of the target name. Updated detector to strip trailing `\` before comparison; with that fix, 0 backslash-typo dead links remain.
- ✅ **M-1**: Created **29 external-system stubs** in `wiki/external-systems/` (24 unique + 2 redirect stubs + 3 satisfying existing wikilinks like `web-clock` alias for `webclock`). All have `type: external-system`, `status: stub`, `tags: [external-system, needs-content, ...]` frontmatter. Each carries 1-3 quick links into the relevant MOC + business-rule notes.

### Held (needs human review)

- ⚠️ **H-1**: 8 zero-byte root stubs (`wiki/kardex.md`, `jiba-portal.md`, `trutops-oseon.md`, `elumatec-sbz140.md`, `windchill.md`, `smt-manufacturing.md`, `cad-batchserver.md`, `design-comments.md`) remain. **Now in collision** with 5 new external-systems stubs (kardex / jiba-portal / trutops-oseon / elumatec-sbz140 / windchill).
  - Obsidian's wikilink resolution with colliding filenames is **non-deterministic** — `[[kardex]]` may resolve to either the empty root file OR the new external-systems file. Risk: operator clicks link and sees empty page.
  - **Recommendation**: delete the 5 colliding root stubs immediately (truly safe — zero bytes, content elsewhere). The other 3 root stubs (`smt-manufacturing`, `cad-batchserver`, `design-comments`) need MOC counterparts created OR the bare-name wikilinks in `modules/_index.md` retargeted before deletion.
- ⚠️ **H-2**: 8 `[[CLAUDE]]` wikilinks point at the project-instructions file at the repo root, not a wiki page. Convert to Markdown links: `[CLAUDE.md](../../CLAUDE.md)`.
- ⚠️ **M-2**: 17 folder-MOC bare-name wikilinks (`work-preparation`, `classes`, `forms`, `controls`, `cam`, etc.) need retargeting at the existing canonical MOCs (`office-to-shopfloor`, `icenter-remaining`, `icenterlib-cad`, etc.). Mostly in `wiki/overview.md` and `wiki/mocs/_index.md` and `wiki/modules/_index.md`.
- ⚠️ **M-3**: 13 template-leakage wikilinks (`[[business-rule]]`, `[[module]]`, `[[domain-concept]]`, `[[Note Name]]`, `[[name]]`, `[[wikilinks]]`, `[[...]]`) appear in body text where they should be plain text. Mostly in `_index` pages where they were copy-pasted from template instructions.
- ⚠️ **M-4**: 4 future-page references intact (icenter-flowgrill-quality, isah-coating-selection-codes, icenterlib-sub-services, hour-codes). Leave as Phase-5 TODOs.

### New collision warnings (introduced by M-1)

Creating external-systems stubs at filenames that match the empty root stubs introduces 5 filename collisions:

| Bare name | Empty root stub | New external-systems stub |
|-----------|-----------------|---------------------------|
| `kardex` | `wiki/kardex.md` (0 bytes) | `wiki/external-systems/kardex.md` (1.2 KB) |
| `jiba-portal` | `wiki/jiba-portal.md` (0 bytes) | `wiki/external-systems/jiba-portal.md` (1.1 KB) |
| `trutops-oseon` | `wiki/trutops-oseon.md` (0 bytes) | `wiki/external-systems/trutops-oseon.md` (0.5 KB) |
| `elumatec-sbz140` | `wiki/elumatec-sbz140.md` (0 bytes) | `wiki/external-systems/elumatec-sbz140.md` (1.3 KB) |
| `windchill` | `wiki/windchill.md` (0 bytes) | `wiki/external-systems/windchill.md` (0.6 KB) |

Resolving the collisions = deleting the 5 empty root files. Already a planned H-1 action; tracking here for clarity.

### Numerical comparison

| Metric | Before | After | Δ |
|--------|------:|------:|---:|
| Pages | 134 | 163 | +29 |
| Dead links | 57 | 30 | **-27** |
| Empty files | 8 | 8 | 0 (H-1 not actioned) |
| Frontmatter gaps | 1 | 0 | **-1** |
| Filename collisions | 6 | 7 (1 intentional `_index`, 5 new H-1-resolvable, 1 stale) | (expected; H-1 will resolve) |

The 30 remaining dead links all sit in the "Needs human review" tier of the original auto-fix order.

