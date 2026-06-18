---
type: module
title: "ISAH leaves — 20+ small entities and helpers"
status: done
module: "ICenterLib/ISAH"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\Customer.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\CustomerSelection.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\CustomerRelation.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\Vendor.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\Contact.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\Language.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\Database.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\DateDimension.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\MultiFinance.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\WeighingFactor.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\MemoDetailElfsquadConfiguration.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\Design.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\CallRegistration.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\WorkView.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\IsahFieldML.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\ShopDocCollection.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\JConfigParam.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\FrmJConfigParamDesignCode.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\DeliveryLine.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\PurchaseDocumentPartLine.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\DossierDetailExtra.vb"
last-reviewed: 2026-06-18
tags: [module, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# ISAH leaves — small entities and helpers

> **Sweep-style note.** Each section is short — under-200-line files with one or two methods. Anything that warranted a deep dive lives in its own module note already.

## Customer family

### `Customer` (130 lines)

```vb
Public Class Customer
    Public ReadOnly Property CustId As String
    Public ReadOnly Property CustomerSelection As CustomerSelection   ' eagerly built

    Public Shared Function IP_sel_CustomerBrowse(CustObsInd As Boolean) As DataTable   ' SP-driven list
    Public Sub      InsertCustomerSelection(NewSelectionCode)        ' delegates to CustomerSelection
    Public Sub      DeleteCustomerSelection(SelectionCode)
    Public Shared Function GetByOrdConfDate(SelectionCode, TillDate, FromDate) As DataTable   ' big inline query
End Class
```

`GetByOrdConfDate` is a **75-line inline T-SQL query** that joins `T_DossierMain + T_CustomerAddress + T_CustDefAddr + T_Customer + ST_MFORDTYPE + T_CustomerSelection` and filters by:

- `ISNUMERIC(C.CustId) = 1` — numeric CustIds only.
- `C.CustObsInd = 0` — non-obsolete.
- `CDA.CustAddrType = 0` — default address.
- `OT.AdminCode IS NULL OR OT.AdminCode = 'JAZO'` — JAZO entity only (not FlowGrill — Q-201 — is this intentional?).
- `LastConfirmDate > @FromDate AND <= @TillDate`.

When `SelectionCode IS NULL` (the "no team filter" path), the query **excludes** any customer that has a row in `T_CustomerSelection` for any of the three team codes `{'031', '032', '033'}` — encoded as a hardcoded `DECLARE @TeamSelection TABLE ... VALUES('031'),('032'),('033')`. **Sales-team-code duplication** with [[sales-team-codes|`FrmCustomerTeam`]] (Q-202).

Called from [[../modules/sales-customer-team|`Sales\FrmCustomerTeam`]].

### `CustomerSelection` (191 lines)

```vb
Public Class CustomerSelection
    Public ReadOnly Property CustId As String

    Public Sub InsertCustomerSelection(SelectionCode)           ' SP IP_ins_CustomerSelect
    Public Sub DeleteCustomerSelection(SelectionCode)           ' SP IP_del_CustomerSelect, needs optimistic LastUpdatedOn
End Class
```

Many-to-many wrapper for `T_CustomerSelection` (the join between customers and selection codes). Same pattern as [[isah-part-and-dispatch|`PartSelection`]]. Hardcodes `IsahUserCode = "ISAH"` (Q-185 family).

### `CustomerRelation` (47 lines)

Two static functions:

- `IP_sel_CustomerRelationRecord(CustId, CustRelCode)` — SP wrapper.
- `GetContacts(CustId)` — inline SQL: `SELECT Initials, Name, FirstName, GenderCode, JobDescription, Email, CustId, CustRelCode FROM T_CustomerRelation WHERE CustId=@CustId AND CustomerRelationObsInd=0 AND CustRelCode<>''`.

Reads contact-person rows for a customer.

### `Vendor` (31 lines)

```vb
Public Class Vendor
    Public ReadOnly Property VendId As String

    Public Function GetName() As String
    Public Function GetRecord() As DataTable           ' SP IP_sel_VendorRecord via DataHandler.GenericQuery
    Public Function GetVendorInfo(ReturnValue) As Object  ' dynamic-column accessor
End Class
```

Same dynamic-accessor pattern as `Part.GetPartField` (Q-170) but without the SQL-string-concat risk — uses `GenericQuery` and DataTable column lookup. Modern style.

### `Contact` (24 lines)

Just two static functions: `GetTitle(GenderCode, LangCode)` and `GetShortTitle(GenderCode, LangCode)`. Returns Dutch titles:

| GenderCode | `GetTitle` | `GetShortTitle` |
|-----------|------------|-----------------|
| 1 (male) | `"heer"` | `"Dhr."` |
| 2 (female) | `"mevrouw"` | `"Mevr."` |
| Else | `"heer/mevrouw"` | `"Dhr./Mevr. "` |

**`LangCode` parameter is accepted but ignored.** Always returns Dutch titles. Q-203 — if iCenter ever ships non-Dutch UI, this breaks. (Compare with [[isah-identity|`Employee.GenderCode`]] which has 4 states including `Neutral`; Contact only handles 2.)

## Reference data

### `Database` (73 lines)

ISAH-side static helpers:

- `RecordIsModified(DT, CurrentLastUpdatedOn)` — optimistic-concurrency check. If the DataTable's `LastUpdatedOn` differs from the supplied value, **shows a MsgBox** (`"Dit record is al gewijzigd door: ..."`) and returns `True`. Q-204 — **MsgBox in a data-layer class** is a layering violation.
- `GetLastUpdatedOnValueAsString(date)` → `yyyy-MM-dd HH:mm:ss.fff` formatter (matches `Common.DATETIMEFORMAT`).
- `GetAsDateTime(value)` → `CType(value, DateTime)`.
- `GetDateTimeAsString(date?)` → same as above but nullable-friendly.
- `GetLastLineNr(DT)` → `DT.Compute("MAX(LineNr)", Nothing)`.
- `GetTruncatedValue(value, fieldSize)` → `Substring(0, Min(Length, fieldSize))`.
- `IsOnline()` — runs `SELECT TOP 1 DossierCode FROM T_DossierMain`; returns True on success, False on any exception. The **ISAH health probe**.
- `FieldSize` enum with one value `T_Description_30 = 30`.

### `DateDimension` (72 lines)

Three SP wrappers around JAZO's working-day calendar:

- `CalcDeltaWorkingDay(OldDate, NewDate)` → SP `IPX_prc_CalcDeltaWorkingDay` — output param `@Delta`.
- `GetNextWorkDay_v2(FromDate, AddDays)` → SP `JIP_Get_NextWorkDay_v2`.
- `GetFirstWorkDay(FromDate, AddDays)` → SP `JIP_cmp_WorkingDay`. Used by [[isah-dossier|`DossierMain.GetProdPlanEndDate`]].

ISAH-side calendar with JAZO-custom working-day overrides (holidays, factory shutdowns).

### `MultiFinance` (45 lines)

Three static functions for ISAH's multi-finance (multi-entity) layer:

- `GetAdminCodeByOrdType(OrdType)` → `SELECT AdminCode FROM ST_MFORDTYPE WHERE OrdType = @OrdType`.
- `GetAdminCodeByDossierCode(DossierCode)` → delegates: get OrdType from `DossierMain`, then GetAdminCodeByOrdType.
- `GetCompanyByAdminCode(AdminCode)` → `Company.CreateByAdminCode(AdminCode)`.

The bridge between an order type and the legal entity (JAZO or FlowGrill — see [[isah-company-codes]]). The `ST_MFORDTYPE` table is the master mapping.

### `Language` (19 lines)

One static function `GetLangId(LangCode) → Integer`:

| `LangCode` | `LangId` |
|-----------|---------|
| `nl-NL` / `nl` | 1 |
| `de-DE` / `du` | 2 |
| `en-EN` / `en-US` / `en` | 5 |
| `fr-FR` / `fr` | 11 |
| else | 1 (default Dutch) |

Hardcoded language-code → ISAH-internal-id mapping. Q-205 — adding a fifth language needs a source change.

### `WeighingFactor` (16 lines)

Single method `GetRecords()` — SP `JIP_Get_Sal008`. Returns a DataTable of weighing factors. Phase-3 follow-up on what these are used for in salary calculations.

### `MemoDetailElfsquadConfiguration` (7 lines)

Empty class shell — `Public Class MemoDetailElfsquadConfiguration` with no members. Placeholder for future Elfsquad memo-detail integration. **`#dead-code`** in the strict sense.

## Settings + design

### `Design` (155 lines)

Wraps ISAH's `T_Design` (drawing) table. Constants:

- `DesignCodeMaxLength = 25`
- `DesignDescriptionMaxLength = 30`

Methods:

- `GetDescription()`, `GetExists()` — accessors.
- `IP_sel_DesignRecord()` → SP wrapper.
- `Shared GetJConfigParamDesignCode(PartCode)` → SP `JIP_Get_jConfigParamDesignCode_v2`.
- `Shared CmpTruncatedDesignCode(value)` / `CmpTruncatedDescription(value)` → enforce the length caps via `Substring`.
- `IP_Ins_Design(EmpId, Description, UserCode)` → SP wrapper with 15 parameters, most empty strings.
- `CheckExistsGUI(AllowCreation, Employee)` / `CheckExists(AllowCreation, Employee)` — if design doesn't exist, optionally prompt the user (`"Tekeningnummer ... bestaat niet. Wil je deze aanmaken?"`) and auto-create.

**`CheckExistsGUI` shows MsgBox from data layer** — same layering violation as `Database.RecordIsModified` (Q-204).

### `JConfigParam` (168 lines)

Wraps `JZ_jConfigParam` table — XML-blob configurator parameters stored per `DesignCode`. Loads the XML into a DataSet, exposes the `Parameter` DataTable via `GetConfigParameters()`.

- `Update(XmlParam, Employee)` runs SP `SIP_Prc_jConfigParam` to upsert. Uses `Database.RecordIsModified` for optimistic-concurrency — silently exits if `ModifiedInDatabase()`.
- `Delete()` — direct DELETE SQL guarded by `ModifiedInDatabase()`.
- `TrySetValue(Name, ByRef Parameter)` — typed parameter setter with reflection-by-instance-type (Integer/Double/Decimal/Boolean). Uses `Common.ApplicationCulture` for parsing.
- `GetParamValue(Name)` — DataTable.Select by name; returns the `value` column.
- `GetProductId()` — TrySetValue("PartCode", ...) then `ProductDb.Product.CreateByPartCode(PartCode).ProductId`. Bridges to ProductDb.
- `CheckDesignCode[GUI](AllowCreation, Employee)` — delegates to `Design.CheckExists[GUI]`.

The JConfigParam is the **configurator parameter blob** for a design — likely tied to Elfsquad / JIBA-ConfigPart flow visible in [[isah-icenter-to-isah|`Icenter2Isah.GenerateXML`]].

### `FrmJConfigParamDesignCode` (160 lines)

UI form (not deep-read). Likely the editor for `JConfigParam` blobs keyed by DesignCode. Phase-3 follow-up.

## Workflow + collections

### `CallRegistration` (103 lines)

Three static functions wrapping ISAH's customer-call log:

- `IP_Ins_CallRegistration(CustId, CustRelCode, VariantType, CallTypeCode, CallStatusCode, UserCode, Description, RegistrationDate, RefTab, RefTab2, EmpId, CallRegText, DossierCode, OrdNr)` — SP `JIP_Ins_CallRegistration` with 3 output params (`@New_CallNr`, `@New_RefTab`, `@New_RefTab2`). Sets `@RegistrationTime = DateTime.Now.TimeOfDay.TotalSeconds` (**sub-minute precision here, unlike `TimeRegistration.GetCurrentTimeInSeconds`** which rounds to minutes — Q-187 / Q-206 inconsistency).
- `IP_Get_AuxMemoId()` — fetches an aux-memo id via inline T-SQL DECLARE+EXECUTE pattern (same style as [[isah-production-hierarchy|`PBOS.IP_Ins_ProdBOS`]] — Q-158).
- `UV_AuxMemo(Info, AuxMemoId)` — updates the `UV_AuxMemo` view with the call-reg text.

Used for service-desk / customer-call tracking. Uses `Common.APPLISAHUSERCODE` (correct convention — matches TimeRegistration).

### `WorkView` (45 lines)

Two methods that build workflow lists:

- `GetSmtBendQueue()` — SP `SIP_GetSmtBendQueue` on the **iCenter DB** (not ISAH). **Hardcoded filters**: `FromProdStatusCode = "40"`, `TillProdStatusCode = "40"`, `FromDeptCode = "JPLT"`, `TillDeptCode = "JPLT"`, `MachGrpCodes = "P03"`. The sheet-metal bend queue is a fixed-shape view. Q-207.
- `GetWvBWorkView()` — inline SQL: `SELECT DD2.* FROM T_DossierDetail DD1 INNER JOIN T_DossierDetail DD2 ON DD2.DossierCode=DD1.DossierCode WHERE DD1.DossierStatusCode='09' AND DD2.DesignCode NOT IN ('', 'DUMMY') AND DD2.DossierStatusCode < 51`. The WvB ("werkvoorbereiding") work-view. Status `'09'` and `< 51` are magic ISAH status codes. Q-208.

Both methods are **iCenter-DB-and-ISAH-DB mixed** — `GetSmtBendQueue` uses iCenter DB, `GetWvBWorkView` uses ISAH. Asymmetric.

### `IsahFieldML` (58 lines)

Multi-language label lookup for ISAH fields. Three static functions:

- `GetDescription(IsahTableName, IsahFieldName, LangId)`.
- `GetBrowseLabel(IsahTableName, IsahFieldName, LangId)`.
- `GetRecordByNames(IsahTableName, IsahFieldName, LangId)` → joins `T_IsahFieldML + T_IsahField + T_IsahTable`.

Used to render Dutch (or other-language) labels for ISAH table columns in iCenter's UI.

### `ShopDocCollection` (64 lines)

`CollectionBase`-derived strongly-typed collection of `ShopDoc` instances. Standard `Add/Find/Item/Remove/Contains` pattern. `Add` dedups by `ShopDocCode` (case-insensitive). Used by [[isah-time-registration|`TimeRegCollector.SetStartedList` / `SetFinishedList`]].

## Document leaves

### `DeliveryLine` (36 lines)

Two static functions wrapping `T_DeliveryLine`:

- `GetIncompleteCount(OrdNr)` → returns `-1` on no-rows, else `Incomplete` column.
- `GetDeliveryLines(OrdNr)` → counts incomplete (`DelCompletedInd = 0`) lines for an order: `SELECT ISNULL(sum(1),0) AS Incomplete FROM T_DeliveryLine WHERE DossierCode = (SELECT DossierCode FROM T_DossierMain WHERE OrdNr = @OrdNr) AND DelCompletedInd = 0`.

### `PurchaseDocumentPartLine` (33 lines)

Simple two-key wrapper around `T_PurDocPartLine`. Single method `GetRecord()` runs SP `IP_sel_PurDocPartLineRecord(@PurDocCode, @PDPartLineNr)`. Companion to [[isah-shop-and-pur-doc|`PurDoc`]].

### `DossierDetailExtra`

Already covered in [[isah-dossier|the dossier hierarchy note]]. Reads `ST_0099_DossierDetailExtra` (`ST_` prefix per Q-153).

## Surprises across the sweep

1. **MsgBox in data-layer classes** (Q-204) — `Database.RecordIsModified` and `Design.CheckExistsGUI` both show MsgBox to the user. Mixed concerns; a CadBatchserver job running these classes on a headless server would block waiting for confirmation.
2. **Sales-team codes duplicated** (Q-202) — `Customer.GetByOrdConfDate` hardcodes `('031'), ('032'), ('033')` in a DECLARE TABLE; [[sales-team-codes|`Sales\FrmCustomerTeam`]] hardcodes the same in VB. Two sources of truth.
3. **JAZO-only filter** in `Customer.GetByOrdConfDate` (Q-201) — `OT.AdminCode IS NULL OR OT.AdminCode = 'JAZO'` excludes FlowGrill orders from customer-team management.
4. **Sub-minute vs minute precision drift** (Q-206) — `CallRegistration.IP_Ins_CallRegistration` uses `TimeOfDay.TotalSeconds` (sub-minute) while `TimeRegistration.GetCurrentTimeInSeconds` rounds to minutes (Q-187). Inconsistent.
5. **`Contact` ignores LangCode** — returns Dutch titles regardless.
6. **`Language` hardcoded** — adding a 5th language requires source edit (Q-205).
7. **`WorkView` mixes iCenter-DB and ISAH-DB calls** without distinguishing in the class name or namespace.
8. **`MemoDetailElfsquadConfiguration` is an empty shell** — `#dead-code` placeholder.

## Open questions

- **Q-201 (new):** `Customer.GetByOrdConfDate` filters to JAZO-only (`AdminCode IS NULL OR AdminCode = 'JAZO'`). Why exclude FlowGrill? `#needs-review`
- **Q-202 (new):** Sales-team codes `('031'), ('032'), ('033')` hardcoded in BOTH `Customer.GetByOrdConfDate` and `Sales\FrmCustomerTeam`. Single source of truth?
- **Q-203 (new):** `Contact.GetTitle/GetShortTitle` accepts `LangCode` but ignores it. Always Dutch.
- **Q-204 (new):** `Database.RecordIsModified` and `Design.CheckExistsGUI` show `MsgBox` from data-layer classes. Layering violation. Refactor?
- **Q-205 (new):** `Language.GetLangId` hardcodes 4 languages. Adding more requires source edit. Move to `T_Language` lookup?
- **Q-206 (new):** Precision drift — `CallRegistration` uses sub-minute `TotalSeconds`, `TimeRegistration` rounds to minutes (Q-187). Reconcile?
- **Q-207 (new):** `WorkView.GetSmtBendQueue` hardcodes `MachGrpCodes='P03'`, `DeptCode='JPLT'`, status range `'40'..'40'`. Single bend-queue target — refactor path if JAZO needs more?
- **Q-208 (new):** `WorkView.GetWvBWorkView` magic status codes `'09'` and `< 51`. Document.

Logged in [[../needs-review/_index]].

## Coverage

`_coverage.md` — all 18 files in this batch → `done`:
- `ICenterLib\ISAH\Customer.vb`
- `ICenterLib\ISAH\CustomerSelection.vb`
- `ICenterLib\ISAH\CustomerRelation.vb`
- `ICenterLib\ISAH\Vendor.vb`
- `ICenterLib\ISAH\Contact.vb`
- `ICenterLib\ISAH\Language.vb`
- `ICenterLib\ISAH\Database.vb`
- `ICenterLib\ISAH\DateDimension.vb`
- `ICenterLib\ISAH\MultiFinance.vb`
- `ICenterLib\ISAH\WeighingFactor.vb`
- `ICenterLib\ISAH\MemoDetailElfsquadConfiguration.vb` (`#dead-code` empty shell)
- `ICenterLib\ISAH\Design.vb`
- `ICenterLib\ISAH\CallRegistration.vb`
- `ICenterLib\ISAH\WorkView.vb`
- `ICenterLib\ISAH\IsahFieldML.vb`
- `ICenterLib\ISAH\ShopDocCollection.vb`
- `ICenterLib\ISAH\JConfigParam.vb`
- `ICenterLib\ISAH\FrmJConfigParamDesignCode.vb` (overview-only — UI form deferred)
- `ICenterLib\ISAH\DeliveryLine.vb`
- `ICenterLib\ISAH\PurchaseDocumentPartLine.vb`

(Customer + Vendor previously marked `needs-review` in earlier batch; this note completes them.)
