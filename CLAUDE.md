# Codebase Wiki Generation — Standing Instructions

## Mission

Build a complete Obsidian wiki for this legacy codebase, which controls business processes in a production factory (sales, engineering, work preparation, production, transport). Two non-negotiable goals:

1. FULL COVERAGE — every source file is accounted for: documented, or explicitly marked (config/generated/dead).
2. BUSINESS LOGIC — every rule affecting a factory or business process is surfaced,
   located in code (path + symbol), explained in plain and easy to understand language, and flagged for SME review.

## Critical rules

- NEVER invent behavior. If intent is unclear, describe what the code does literally,
  set `status: needs-review`, add #needs-review, and write the specific open question.
- This software touches upon physical processes in a factory. Your inferences are NOT authoritative.
  Anything process-, setpoint-, interlock-, or safety-relevant gets #needs-review and #safety-relevant.
- Link to source (path + function/class) instead of pasting large code blocks.
  Quote at most the few lines that embody a rule.
- Work in small batches. After each batch: update \_coverage.md and `git commit`.
- Build the file inventory from `git ls-files` / ripgrep — never from memory.

## Workflow (phases, in order)

1. Inventory — list every source file in \_coverage.md with status=todo. Shallow only.
2. Architecture — entry points, data flow, external systems, deployment.
3. Deep docs — one note per module; mark each file done in \_coverage.md as you go.
4. Business-logic pass — dedicated cross-cutting sweep (see search patterns below).
5. Linking & review — MOCs, graph cleanup, list all #needs-review items.

## Coverage definition

A module is "done" only when every file under it has a non-todo status in \_coverage.md.

## Obsidian conventions

- Every note starts with YAML frontmatter: type, status, module, source-paths, tags, last-reviewed.
- Connect notes with [[wikilinks]]. Each subsystem gets a MOC (Map of Content) hub note.
- Tags: #business-rule #domain-concept #external-system #entry-point #dead-code #needs-review #safety-relevant
- Note types: architecture | module | business-rule | domain-concept | external-system | moc
