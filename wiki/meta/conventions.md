---
type: meta
title: "Wiki Conventions"
status: active
tags: [meta]
created: 2026-06-18
updated: 2026-06-18
---

# Wiki Conventions

The rules every note in this wiki follows. Mirrors the standing instructions in `../../CLAUDE.md`.

## Note types

| `type` | What it documents | Folder |
|--------|-------------------|--------|
| `architecture` | Cross-cutting structural view | `architecture/` |
| `module` | One source file or tight group of files | `modules/` |
| `business-rule` | One rule that affects factory or business process | `business-rules/` |
| `domain-concept` | One vocabulary term | `domain-concepts/` |
| `external-system` | One external integration | `external-systems/` |
| `moc` | Subsystem hub / Map of Content | `mocs/` (and `_index.md` files) |
| `meta` | Wiki-about-the-wiki | `meta/` and top-level meta files |

## Required frontmatter

Every note **must** start with at minimum:

```yaml
---
type: <one of the types above>
status: <todo | draft | done | needs-review>
module: "<project path, e.g. JAZO.iCenter.Domain>"   # for module/rule/concept notes
source-paths:                                         # for any note that points at code; use the absolute path inside one of the in-scope roots
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\path\\to\\File.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\TruTopsLib\\path\\to\\File.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\path\\to\\File.vb"
tags: [<at least one>]
last-reviewed: YYYY-MM-DD                             # set when status flips to done
---
```

## Tag glossary

| Tag | Meaning |
|-----|---------|
| `#business-rule` | A rule affecting factory/business behavior |
| `#domain-concept` | Vocabulary term |
| `#external-system` | An external integration |
| `#entry-point` | A `Main`, service host, scheduled job, or UI launcher |
| `#dead-code` | Verified unused — keep until SME confirms |
| `#needs-review` | Intent unclear, requires SME |
| `#safety-relevant` | Touches process, setpoints, interlocks, or safety. **Never close without SME sign-off.** |

## Status values

| Status | Meaning |
|--------|---------|
| `todo` | Not yet written |
| `draft` | Written but incomplete |
| `done` | Reviewed and complete; `last-reviewed` set |
| `needs-review` | Awaiting SME |

## Citation rule

**Never paste long code blocks.** Link to source via `source-paths:` and reference the function/class name in prose. Quote at most the few lines that *embody* a rule.

## Wikilink style

- Use `[[Note Name]]` for same-folder links where filenames are unique.
- Use `[[folder/name|Display Text]]` for cross-folder links.
- Index pages are conventionally `_index.md` so they sort to the top of their folder.

## Commit cadence

Per CLAUDE.md: work in small batches. After each batch — update `_coverage.md` and `git commit`.
