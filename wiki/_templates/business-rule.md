---
type: business-rule
title: ""
status: needs-review
module: "iCENTER/<SubFolder>"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\<SubFolder>\\<File>.vb"
last-reviewed: ""
tags: [business-rule, needs-review]
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# <Rule name in plain language>

## Rule

<One sentence. What the rule *says*, not how it's coded.>

## Where it lives

- File: `<path>` (in `source-paths`)
- Symbol: `<Class>.<Method>` line ~<N>

## The code (minimal quote)

```vb
' Only the few lines that EMBODY the rule. Not the whole method.
```

## Why this is a business rule

<What factory or business process does this affect? Who cares if it changes?>

## Triggers / when it fires

<Inputs, schedule, user action, machine event.>

## Effects

<What changes in the world: DB row, machine command, file written, email sent.>

## Edge cases / known exceptions

- <…>

## Safety classification

- [ ] Touches physical process / setpoint / interlock / safety → add `#safety-relevant`
- [ ] Reversible if wrong?
- [ ] Blocks production if it fails?

## SME questions

- <Open questions, then add to [[needs-review/_index]].>
