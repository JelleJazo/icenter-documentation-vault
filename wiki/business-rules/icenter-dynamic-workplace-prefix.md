---
type: business-rule
title: "Dynamic-workplace naming prefix WP"
status: needs-review
module: "ICenterLib/iCenter"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\Client.vb"
last-reviewed: ""
tags: [business-rule, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Dynamic-workplace naming prefix — `WP`

## Rule

A Windows computer named **`WP<integer>`** (e.g., `WP123`) is treated by iCenter as a **dynamic / virtual workplace** whose MachineId is the integer after the `WP` prefix. iCenter validates the MachineId exists in `T_ProdMachines` and treats the computer as that production machine.

## Where it lives

- File: `ICenterLib\iCenter\Client.vb`
- Constant: line 7 — `Public Const DynamicWorkplacePrefix As String = "WP"`
- Parser: `Client.GetMachineIdFromName(Value)` lines 30-48.

## The code

```vb
Public Const DynamicWorkplacePrefix As String = "WP"

Public Shared Function GetMachineIdFromName(ByVal Value As String) As Integer
    Dim DefaultValue As Integer = -1
    If String.IsNullOrEmpty(Value) Then Return DefaultValue
    ElseIf Value.Length < 4 Then Return DefaultValue
    ElseIf Not Value.StartsWith(DynamicWorkplacePrefix) Then Return DefaultValue
    ElseIf Not IsNumeric(Value.Substring(2)) Then Return DefaultValue
    Else
        Dim Id As Integer = Integer.Parse(Value.Substring(DynamicWorkplacePrefix.Length))
        If GetMachineIdExist(Id) Then Return Id Else Return DefaultValue
End Function
```

## Why this is a business rule

JAZO's network naming convention reserves the `WP` prefix for **transient operator-laptop / virtual-machine-name workplaces**. A machine literally named `WP123` is the convention by which iCenter knows to look up MachineId 123 rather than treating it as a fixed-name production machine. Permanent stations get descriptive computer names (e.g., `JAZO-SBZ140-01`).

If JAZO ever needs to deploy a different prefix or use the existing `WP` prefix for other purposes (e.g., a Windows Pro tag), this parser conflicts.

## Triggers / when it fires

- Anywhere `Client.GetMachineIdFromName(computername)` is called.
- Phase-3 follow-up to enumerate callers.

## Effects

- Computer named `WP123` → resolves to MachineId 123 (if exists in `T_ProdMachines`).
- Computer named `JAZO-XXX-01` → returns -1, falls back to other lookup paths (typically `PrefClient` matching).
- Computer name shorter than 4 chars OR not starting with `WP` OR with non-numeric suffix → -1.

## Edge cases / known exceptions

- `Value.Length < 4` rejects `WP1` and `WP12` (but accepts `WP123`).
- `Value.Substring(2)` — hardcoded `2` (length of "WP"). If `DynamicWorkplacePrefix` changes to e.g. "WKS", this breaks. The `IsNumeric` check uses the hardcoded substring, while the actual `Integer.Parse` uses `DynamicWorkplacePrefix.Length` — **inconsistent** between checker and parser. Q-247.
- Numeric check via `IsNumeric` — accepts negatives, decimals, scientific notation in theory; `Integer.Parse` would throw on those. Latent crash path.

## Safety classification

- [ ] Touches physical process — indirectly (determines which station the laptop becomes).
- [ ] Drives cost / pricing — no.
- [x] Reversible if wrong? — yes (rename Windows host).
- [ ] Blocks production if it fails? — yes if a `WP*` laptop is provisioned with a wrong MachineId in the suffix; the operator can't clock in to the intended station.

## SME questions

- **Q-247 (new):** `GetMachineIdFromName` uses hardcoded `Value.Substring(2)` (length of "WP") in the IsNumeric check but `DynamicWorkplacePrefix.Length` in the parse. Mismatched if prefix is ever changed.
- **Q-248 (new):** Is `WP` the only dynamic prefix? Are there `WS`, `MOB`, etc. variants?

Logged in [[../needs-review/_index]].

## Related

- [[../modules/icenterlib-icenter-production-machines|`Client`]] — defining module.
- [[../mocs/icenterlib-icenter]] — parent MOC.
