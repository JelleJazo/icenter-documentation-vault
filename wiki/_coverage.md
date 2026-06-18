---
type: meta
title: "File Coverage Tracker"
status: active
created: 2026-06-18
updated: 2026-06-18
tags: [meta, coverage]
---

# File Coverage Tracker

**Source of truth for "done".** A module is *done* only when every file under it has a non-`todo` status here.

## Status values

| Status | Meaning |
|--------|---------|
| `todo` | Not yet documented |
| `done` | Documented in a [[modules/_index\|module]] note with frontmatter + source-paths |
| `config` | Configuration file: no behavior; pointer in module note is enough |
| `generated` | Generated code: do not document; record generator |
| `dead-code` | Verified unused: flagged with #dead-code, SME confirmation pending |
| `needs-review` | Documented but intent unclear: open question in [[needs-review/_index]] |

## How to populate

Phase 1 of the workflow:

```powershell
# from the codebase root
cd C:\DevOps\iCenter2
git ls-files | Out-File -Encoding utf8 C:\DevOps\icenter-vault-preparator\wiki\.coverage-raw.txt
```

Then convert each line into a row below. **Do not** populate from memory.

---

## Inventory

_(Empty — Phase 1 not yet run. After running `git ls-files`, append one table per top-level project.)_

### Template per project

```
## JAZO.iCenter.<ProjectName>

| File | Status | Note | Last reviewed |
|------|--------|------|---------------|
| src/Foo.cs | todo | — | — |
| src/Bar.cs | done | [[modules/bar-cs]] | 2026-06-18 |
```

---

## Roll-up

_(Updated by the lint pass — `claude-obsidian:wiki-lint`.)_

| Project | Total | Done | Todo | Config | Generated | Dead | Needs-review |
|---------|------:|-----:|-----:|-------:|----------:|-----:|-------------:|
| _(pending Phase 1)_ | — | — | — | — | — | — | — |
