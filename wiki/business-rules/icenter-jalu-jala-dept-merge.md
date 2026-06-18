---
type: business-rule
title: "JALU+JALA dept merge in identification UI"
status: needs-review
module: "ICenterLib/iCenter"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\FrmIdentification.vb"
last-reviewed: ""
tags: [business-rule, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# JALU + JALA dept merge in identification UI

## Rule

When the **shop-floor identification form** builds its department-button row, it **merges department code `JALA` into `JALU`** — the JALU button shows employees from both departments, and a standalone `JALA` button is suppressed.

## Where it lives

- File: `ICenterLib\iCenter\FrmIdentification.vb` lines 34-45
- Symbol: `FrmIdentification_Load` (department-button construction loop)

## The code (verbatim)

```vb
For Each drDepartment As DataRow In dtDepartments.Rows
    Dim DeptCodes As String = drDepartment.Item("DeptCode").ToString.Trim
    If DeptCodes.ToUpper = "JALU" Then
        DeptCodes &= ", JALA"
    ElseIf DeptCodes.ToUpper = "JALA" Then
        Continue For
    End If
    Dim RadDep As New CtrlRadButtonIdent(drDepartment.Item("DeptCode").ToString.Trim, DeptCodes)
    AddHandler RadDep.CheckedChanged, AddressOf Me.RadioButton_CheckedChanged

    Me.FLPDepartMents.Controls.Add(RadDep)
Next
```

So the JALU radio button is constructed with `Name = "JALU"` and `DeptCodes = "JALU, JALA"`. When clicked, its `CheckedChanged` handler queries `ISAH.Employee.GetEmpIdsbyDeptCodes("JALU, JALA")` which returns employees from both departments.

JALA's own row is skipped entirely via `Continue For`.

## Why this is a business rule

`JALU` and `JALA` are JAZO's distinct dept codes for **aluminium production (JALU)** and **aluminium assembly (JALA)** — at least, that's the strong inference from the codes. The two departments share **physical workstations and operators**: it would be confusing to make operators tap two separate buttons for what is effectively one place. So the UI collapses them.

The merge is **UI-presentational only**:
- `T_Employee.DeptCode` rows still carry distinct `JALA` / `JALU` values.
- `T_ProdMachinesMP`, `T_Operations`, etc. still distinguish.
- Only the identification form's button-row collapses the two.

## Triggers / when it fires

- Every time `FrmIdentification` is loaded (any login or MP-machine flow).

## Effects

- One button labelled `JALU` appears instead of two (JALU + JALA).
- Tapping it lists employees from both departments.
- Selecting an employee from JALA via this button returns their actual `JALA` `EmpId` and original `DeptCode = JALA` — the merge doesn't rewrite the data.

## Edge cases / known exceptions

- **JALA-only with no JALU**: if `dtDepartments` contains a `JALA` row but no `JALU` row, the JALA row gets `Continue For`-skipped and **no JALU button is created** — JALA employees become unreachable through this UI. Q-237 / Q-257.
- **Case**: comparison is `.ToUpper = "JALU"` — case-insensitive match. Lowercase `jalu` rows get merged too.
- **Order-sensitive in display only**: the order in `dtDepartments` determines whether JALU appears before/after where JALA would have been.
- **`drDepartment.Item("DeptCode").ToString.Trim`** is read twice (line 35 and line 41) — the second call passes raw `"JALU"` (without the appended ", JALA") as `Name` for the radio button, while `DeptCodes` carries the merged `"JALU, JALA"`. Intentional but easy to miss.

## Safety classification

- [ ] Touches physical process — no.
- [ ] Drives cost / pricing — no.
- [x] Reversible if wrong? — yes (UI-only).
- [ ] Blocks production if it fails? — only if JALA-only ever happens (then JALA employees can't identify themselves).

## SME questions

- **Q-257 (new):** What happens if there are JALA employees but no JALU active employees? They become unreachable on this form.
- **Q-258 (new):** Are JALU and JALA truly co-located? Should the data model merge them (single dept) instead of the UI?

Logged in [[../needs-review/_index]].

## Related

- [[../modules/icenterlib-icenter-identification|`FrmIdentification`]] — defining module.
- [[../mocs/icenterlib-icenter]] — parent MOC.
