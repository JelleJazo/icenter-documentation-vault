---
type: module
title: "ISAH lookups — Selection, Setting"
status: done
module: "ICenterLib/ISAH"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\Selection.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\Setting.vb"
last-reviewed: 2026-06-18
tags: [module, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# ISAH lookups — `Selection` and `Setting`

## Purpose

Two small accessor classes that wrap ISAH's universal code-tables:

- **`Selection`** — wraps `T_Selection`, ISAH's universal *code-with-description* table. Used everywhere a string-identifier needs a human-readable label (sales-team codes 031/032/033, company codes 041/042, dept codes, status codes, etc.). Most accessor patterns elsewhere in ICenterLib that look up "what's the description of code X?" go through this class.
- **`Setting`** — wraps ISAH's `Setting` SP family. Currently exposes exactly one accessor (`GetStandCapacityType`). The class is a stub for the broader ISAH settings surface.

## `Selection`

```vb
Public Class Selection
    Public ReadOnly Property SelectionCode As String

    Public Sub New(SelectionCode As String)

    Public Shared Function GetSelectionField(SelectionCode As String, ReturnValue As String) As Object
    Public Function GetDescription() As String
    Public Function IP_sel_SelectionRecord() As DataTable
    Public ReadOnly Property Exists As Boolean
End Class
```

Backed by stored procedure **`IP_sel_SelectionRecord`** (`@SelectionCode` parameter). Returns one row keyed on `SelectionCode`, with columns including at least `Description`.

`GetSelectionField(code, columnName)` is the generic-column accessor. `GetDescription()` is the common convenience wrapper for `GetSelectionField(code, "Description")`. `Exists` returns `True` if the SP yields any rows.

Reference example: [[sales-customer-team|`Sales\FrmCustomerTeam`]] constructs `New ISAH.Selection("031")`, `("032")`, `("033")` and reads each's `.GetDescription`. If the SP returns no description, the form falls back to the bare code as the display label.

## `Setting`

```vb
Public Class Setting
    Public Shared Function GetStandCapacityType() As Integer
End Class
```

The class has **exactly one method**. It calls SP `IP_get_Setting` with `@SettingCode = "1"` and reads back the `@StandCapacityType` **output parameter** (the only place in ICenterLib that uses SP output parameters — every other accessor uses `ExecuteReader` or `ExecuteScalar`).

Inline comment _"Query retrieved from IP_sel_MachGrpExtRS"_ explains the SP's behaviour: it's the same query that lives inside the larger `IP_sel_MachGrpExtRS` SP, extracted as a standalone for ISAH-side reuse.

## Surprises

1. **`Selection.GetSelectionField`** has a final `Return Result` (line 23) after the `If…Else` branches — both branches `Return` already, so this is unreachable. Cosmetic dead code.
2. **`Selection.IP_sel_SelectionRecord`** wraps its SP call in two try/catches (outer + inner) — the inner catch logs `Critical`, the outer catch swallows silently. Asymmetric error handling.
3. **`Setting.GetStandCapacityType`** has its `oApplicationLog.NewEntry` line commented out (line 32). Currently no logging on SP failure.
4. **`Setting.GetStandCapacityType` uses `@SettingCode = "1"`** as a magic value. There's no enum or named constant for which setting is "1". Q-138.
5. **The `Setting` class has no instance state** — `GetStandCapacityType` is a `Public Shared` static. Adding more settings means adding more shared methods. No `Setting(SettingCode)` constructor pattern.

## Business rules surfaced here

- `T_Selection` is iCenter's universal code-lookup table. Anything that resolves a string code (`"031"`, `"041"`, etc.) to a description ultimately reads it.
- The `IP_sel_SelectionRecord` SP is the authoritative way to read one row. **Don't** hand-write `SELECT FROM T_Selection` — go through `Selection`.

## Open questions

- **Q-138 (new):** `Setting.GetStandCapacityType` uses magic `@SettingCode = "1"`. Document what setting code 1 is and add a named constant.
- **Q-139 (new):** `Setting` is a single-method class. Are there other ISAH settings (codes 2, 3, …) that should be added here?

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/icenterlib-isah]] — parent MOC.
- [[../modules/sales-customer-team]] — example caller of `Selection`.
- [[isah-identity|`Company`]] — uses Selection-codes `"041"` (JAZO) and `"042"` (FlowGrill).

## Coverage

`_coverage.md`:
- `ICenterLib\ISAH\Selection.vb` → `done`
- `ICenterLib\ISAH\Setting.vb` → `done`
