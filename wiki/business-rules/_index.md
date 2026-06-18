---
type: moc
title: "Business Rules — Index"
status: stub
tags: [moc, business-rule]
created: 2026-06-18
updated: 2026-06-18
---

# Business Rules

Every rule affecting a factory or business process. Surfaced in plain language; located in code by path + symbol; flagged for SME review.

> **Critical:** rules touching physical processes, setpoints, interlocks, or safety MUST carry `#safety-relevant` and `#needs-review` until SME confirms.

## How to find them (Phase 4 search patterns)

Use these in iCenter2 during the business-logic pass:

- Magic numbers / thresholds: `(?<![\w.])\d{2,}(?![\w.])` in non-test C#
- Conditional dispatch on enums / status: `switch.*[Ss]tatus`, `case .*:` near domain types
- Time windows: `TimeSpan\.From`, `DateTime\.Now`, cron expressions, `IDbCommand.*Timeout`
- Quantity limits: `>= ?\d+`, `<= ?\d+` in domain/application layers
- Permission/role gates: `[Authorize`, `Role\s*==`, claim checks
- External-system commands: writes to PLC/machine, file drops, print jobs
- Configurable behavior: `appsettings.*\.json` reads, feature flags

## Pages

_(Populated during Phase 4. Use the [[../_templates/business-rule|business-rule template]].)_

### By domain (placeholder)
- Order / job flow — _pending_
- Sheet-metal production — _pending_
- Nesting / Trumpf / TruTops handoff — _pending_
- Authentication / authorization — _pending_
- Notifications / email (jMail) — _pending_
- Data migration / lifecycle — _pending_

### By severity
- `#safety-relevant` — _none yet_
- `#needs-review` — _none yet_
