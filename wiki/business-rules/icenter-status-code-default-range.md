---
type: business-rule
title: "Default work-view status range 40-49"
status: needs-review
module: "ICenterLib/(root)"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\Common.vb"
last-reviewed: ""
tags: [business-rule, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Default work-view status range — 40 to 49

## Rule

When iCenter opens a "work view" (the operator's pending-work list) without an explicit status filter, the default range is **status codes `"40"` through `"49"`** inclusive. Stringly-typed; the lower and upper bounds are `Public Const` strings on `Common`.

## Where it lives

- File: `ICenterLib\Common.vb` lines 524–525
- Symbols: `Public Const WorkViewDefaultFromStatusCode As String = "40"`, `WorkViewDefaultTillStatusCode As String = "49"`

## The code (minimal quote)

```vb
Public Const WorkViewDefaultFromStatusCode As String = "40"
Public Const WorkViewDefaultTillStatusCode As String = "49"
```

## Why this is a business rule

iCenter and ISAH use **two-digit string status codes** for production-state machines. Status codes 40-49 collectively represent "work in progress" on the shop floor — what an operator at a station should be seeing.

A separate `Common.GetStatusCodeColor(StatusCode)` provides the colour mapping for status codes 04-09 (a different range — likely a *different* status concept, possibly dossier status vs production status). The pattern of stringly-typed two-digit codes is consistent throughout.

Phase-4 follow-up: enumerate every meaningful status code (currently we know 04-09 are coloured; 40-49 are work-view defaults; SME-confirm what each means).

## Triggers / when it fires

- Anywhere code constructs a status filter without explicit user input (e.g. opening `FrmWorkView` for the first time).
- Phase-3 follow-up: callsites via Grep on `WorkViewDefaultFromStatusCode`.

## Effects

- Controls which production rows the operator sees by default in work-view-style screens.
- A row at status `"50"` (presumably "completed") is *not* shown in the default view.

## Edge cases / known exceptions

- **`Public Const String`** → inlined at compile time in any consuming assembly. Changing the value in ICenterLib without recompiling iCenter leaves iCenter with the old values.
- Range is `>= 40 AND <= 49` (inclusive). Status `"49"` IS shown; status `"50"` is not.
- Stringly-typed — sorting `"4"`, `"40"`, `"5"` lexically vs numerically gives different results. Phase-3 follow-up on how the filter comparison is actually built (`>=` and `<=` on strings work because both bounds are two-digit).

## Safety classification

- [ ] Touches physical process — no.
- [x] Touches operator UX → `#needs-review` to confirm with SME.
- [ ] Reversible if wrong? — Yes (config change + redeploy).
- [ ] Blocks production if it fails? — No (operator can override the filter).

## SME questions

- **Q-127 (new):** Enumerate every iCenter status code (40-49 range, plus 04-09 colour-mapped range from `GetStatusCodeColor`). Surface the SME-friendly name for each.
- **Q-128 (new):** Confirm that "default work view shows statuses 40-49" matches operator expectation.

Logged in [[../needs-review/_index]].

## Related

- [[../modules/icenterlib-common]] — defining module.
