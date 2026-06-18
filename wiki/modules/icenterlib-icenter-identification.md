---
type: module
title: "Identification — FrmIdentification + photo-button controls + WebClockAssistant"
status: done
module: "ICenterLib/iCenter"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\FrmIdentification.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\CtrlImageIdentification.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\CtrlRadButtonIdent.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\WebClockAssistant\\WebClockAssistantRepository.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\WebClockAssistant\\WebClockAssistantGenericHandler.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\WebClockAssistant\\WebClockAssistantUserSelectionHandler.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\iCenter\\WebClockAssistant\\WebClockAssistantAnonymousHandler.vb"
last-reviewed: 2026-06-18
tags: [module, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Identification — `FrmIdentification` + photo-button controls + `WebClockAssistant`

> **The shop-floor login UX.** When an operator approaches a station they tap a department (radio button), then a photo button (their face), and they're identified. The form supports two flavours: real-employee login and MP-machine login (where a "machine emp id" stands in for the unattended workplace).

## `FrmIdentification` — the picker form

187 lines. Constructor: `New(LoginType, DefaultEmpId)`. Two LoginTypes:

- `UserLogin` — real-employee identification.
- `ProdMachinesMP` — multi-purpose-workplace identification.

### Departments via `FLPDepartMents`
On Load:

- If `UserLogin`: `ISAH.Employee.GetDeptCodesByActiveEmpIds()` — distinct DeptCodes that have at least one active employee.
- If `ProdMachinesMP`: `ProductionMachineMultiPurpose.GetProdMachineMP()` distincted on DeptCode.

For each row, **special-case**: if DeptCode = "JALU" → display as "JALU, JALA" (i.e., merge JALA into the JALU button). If DeptCode = "JALA" → skip (the JALA button is already covered by JALU). Documented as [[../business-rules/icenter-jalu-jala-dept-merge]].

Builds one `CtrlRadButtonIdent` per dept; wires `CheckedChanged` → `RadioButton_CheckedChanged`.

### Employees via `flpEmpIds`
`RadioButton_CheckedChanged(sender)`:

- Clear the FlowLayoutPanel.
- `ISAH.Employee.GetEmpIdsbyDeptCodes(DeptCodes)` → base DataTable.
- If MP mode: clear the rows, re-fetch `ProductionMachineMultiPurpose.GetProdMachineMP()` filtered by DeptCode IN `(...)` via `Common.GetSqlInString`, and for each MP-machine populate the employee row from `Employee.IP_sel_EmployeeRecord()`. If the employee row is empty, **synthesise a "Geen / LEAN modus" placeholder row** with GenderCode=1, HourCode="" — represents an unmanned machine ready for lean-mode use.
- For each employee row, instantiate a `CtrlImageIdentification` (the photo button). If the employee is in `DisabledMembers`, set `.Enabled = False`.

### Default-department selection on `Shown`
`FrmIdentification_Shown`:
1. Try `JIBA.Asset.GetAssetInfoByComputername(ComputerName, AssetField.DeptCode)` — the **per-asset-tag default department**.
2. If that DeptCode isn't in the departments list, fall back to `Employee(DefaultEmpId).GetDeptCode()` — the **operator's home department**.
3. If still empty and MP mode, fall back to the **first dept button** in the FlowPanel.
4. Find the matching `CtrlRadButtonIdent` (via `ContainsDeptCode`) and click it.

### `SetEmpSelected(o As CtrlImageIdentification)`

When an operator taps a photo button, the form is set to `DialogResult.OK` after caching `EmpId/Id/FirstName/LastName`. Two safeguards:

- **MP-mode + EmpPresent**: rejects with `"... is al in gebruik bij werkplek '{PrefClients}'"` MsgBox. **An already-clocked-in employee can't double-clock at an MP station** — looks up via `ProductionMachines.GetPrefClientsByMachineEmpId`. Q-234 `#safety-relevant` — what if the employee crashed out and the row wasn't cleared? Lockout?

## `CtrlImageIdentification` — the photo button

A `UserControl` showing employee photo + name. Visual semantics:

- **Border colour** = green (Lime) if present (`HourCode == "AW"`), black if absent.
- **MouseEnter when absent**: re-checks `Employee.GetIsPresent()` to refresh state (in case they clocked in on another station meanwhile).
- **Click handler**: if not present and `AllowAbsent=False` → `MsgBox("Je dient je eerst aan te melden.", ...)` and abort; else delegate to `FrmIdentification.SetEmpSelected(Me)`.

`HourCode "AW"` = Dutch "Aanwezig" (present). Documented as a domain concept in [[../domain-concepts/hour-codes]] (to be written).

## `CtrlRadButtonIdent` — the department button

A `RadioButton` rendered as a Button. Stores `DeptCodes` (the comma-separated list — for the JALU-JALA merger case it's literally "JALU, JALA"). `ContainsDeptCode(value)` splits on `,` and matches case-insensitively.

Visual: Navy background, white text, red text when Checked. Bold, 10pt.

## `WebClockAssistant` family — virtual-assistant clocking

> When operator A is already clocked in on station X and needs help from operator B on the same task, B can be added as **assistant**. The assistant inherits the same ShopDoc / HourCode time-reg line as the primary.

### `WebClockAssistantRepository` (113 lines)

Direct CRUD on `T_WebClockAssistant(EmpIdPrim, EmpIdSec)`:
- `Shared GetAllWebClockAssistants()` — all rows.
- `Shared GetWebClockAssistants(EmpId)` — by EmpIdPrim.
- `InsertWebClockAssistant(empIdPrim, empIdSec)` — `IF NOT EXISTS` upsert (idempotent).
- `DeleteWebClockAssistant(empIdSec)` — **DELETE by EmpIdSec ONLY** (not by combined key). If a secondary is assistant to two primaries, deleting them as secondary removes BOTH pairings. Q-235 — intentional or bug?

### `WebClockAssistantGenericHandler` (60 lines, `MustInherit`)

Abstract base. Properties: `PrimaryEmpId`, `SecondaryEmpId`. Two abstract methods: `FindAvailableAssistant()`, `GetAssistantToRemove()`.

`Save()`:
1. Repository: `InsertWebClockAssistant(PrimaryEmpId, SecondaryEmpId)`.
2. Read primary's `Employee.GetStatus()`.
3. If primary has an active ShopDoc, call `ISAH.TimeRegistration.ChangeToShopDoc(SecondaryEmpId, ShopDocCode, HourCode, True, ..., False, ...)` to switch the assistant to the primary's task.

**The Save() is non-transactional** — INSERT + ChangeToShopDoc are independent. If the second call fails, the assistant is registered but not actually clocked in. Q-236 `#safety-relevant`.

`RemoveAssistant(empId)`:
1. `WebClockAssistantRepository.DeleteWebClockAssistant(empId)`.
2. `TimeReg.CloseTimeRegLine(empId, today, False)` — close the assistant's open time-reg line.

### `WebClockAssistantAnonymousHandler` (54 lines)

Auto-picks the next available "virtual assistant" from `ISAH.Employee.GetWebClockAssistants(PrimaryEmpId)` (the **per-primary pool of allowed assistants** — distinct from `T_WebClockAssistant` which is the current-assignment table). Picks the first one not already in active assignment.

If none available, throws Dutch exception: `"Er zijn geen virtuele assistenten meer beschikbaar.\n\nRoep je teamleider er bij voor verdere afhandeling."`

### `WebClockAssistantUserSelectionHandler` (62 lines)

Pops `FrmIdentification(UserLogin, PrimaryEmpId)` with `AllowAbsent=False` and `DisabledMembers = {PrimaryEmpId} + current assistants`. Operator picks; **rejects** if the picked employee is already clocked in at any ProdMachine (throws `GetCurrentTasksMessage`). Otherwise sets `SecondaryEmpId`.

`GetAssistantToRemove()` pops a `FrmUserSelection` from `UserControls.FrmUserSelection`.

## Surprises

1. **JALU/JALA merging in UI but not elsewhere**. The merge is presentational only; queries elsewhere still treat them as distinct codes. Q-237 — what happens if an employee belongs to JALA only? They'd appear under the merged button but DeptCode field would still be JALA. Confusion risk.
2. **`AllowAbsent` default = False** on both `FrmIdentification` and `CtrlImageIdentification` — operators must be present to identify themselves. Caller must explicitly opt-in for absent-allowed flows (e.g., admin override).
3. **MP-mode synthesises a "Geen / LEAN modus" placeholder** when the MachineEmpId has no matching `T_Employee` row. The placeholder has GenderCode=1 (male title), Initials/JobDescription="". Effectively "this machine has no operator — proceed in lean mode" path.
4. **`FrmIdentification.SelectDefaultDeptCode(DeptCode)`** is a 3-line stub returning `False`. Dead code? Q-238.
5. **`WebClockAssistantAnonymousHandler.FindAvailableAssistant`** does `dtAvailableAssistants.Rows.Count` after checking only `dtWebClockAssistant IsNot Nothing` — if `dtAvailableAssistants` is Nothing, NullReferenceException. Q-239.
6. **`SetEmpSelected` MsgBox + Exit** = blocks UI thread. Standard WinForms; fine here.
7. **`CtrlImageIdentification.PBImage_MouseEnter`** triggers a DB call (`Employee.GetIsPresent`) on hover when not-present — **N×mouse-hover DB lookups** if the operator drags their cursor across 30 photos. Q-240 — performance.

## Open questions

- **Q-234 (new):** MP-mode lockout: `SetEmpSelected` rejects if employee.HourCode='AW'. If a prior session crashed without clearing, employee stays locked. Recovery path? `#safety-relevant`
- **Q-235 (new):** `DeleteWebClockAssistant(empIdSec)` deletes by secondary-only. Removes all pairings if same secondary helps multiple primaries.
- **Q-236 (new):** `WebClockAssistantGenericHandler.Save` is non-transactional. INSERT succeeds but ChangeToShopDoc fails → orphaned assistant assignment. `#safety-relevant`
- **Q-237 (new):** JALU/JALA merge in UI only. Confusing semantics for JALA-only employees.
- **Q-238 (new):** `FrmIdentification.SelectDefaultDeptCode` is a dead stub. Delete?
- **Q-239 (new):** `WebClockAssistantAnonymousHandler.FindAvailableAssistant` NRE risk if `dtAvailableAssistants Is Nothing`.
- **Q-240 (new):** `CtrlImageIdentification.PBImage_MouseEnter` triggers a per-hover DB call when not-present. Performance review with 30+ employees.

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/icenterlib-icenter]] — parent.
- [[isah-identity|`ISAH.Employee`]] — GetDeptCodesByActiveEmpIds, GetEmpIdsbyDeptCodes, GetWebClockAssistants, GetIsPresent.
- [[isah-time-registration|`ISAH.TimeRegistration`]] — `ChangeToShopDoc` / `CloseTimeRegLine` consumers.
- [[icenterlib-icenter-production-machines|`ProductionMachines`]] — `GetPrefClientsByMachineEmpId` + `GetCurrentTasksMessage`.

## Coverage

- `iCenter\FrmIdentification.vb` → `done`
- `iCenter\CtrlImageIdentification.vb` → `done`
- `iCenter\CtrlRadButtonIdent.vb` → `done`
- `iCenter\WebClockAssistant\WebClockAssistantRepository.vb` → `done`
- `iCenter\WebClockAssistant\WebClockAssistantGenericHandler.vb` → `done`
- `iCenter\WebClockAssistant\WebClockAssistantUserSelectionHandler.vb` → `done`
- `iCenter\WebClockAssistant\WebClockAssistantAnonymousHandler.vb` → `done`
- `iCenter\FrmIdentification.designer.vb`, `CtrlImageIdentification.designer.vb`, `CtrlRadButtonIdent.designer.vb` → `generated`
