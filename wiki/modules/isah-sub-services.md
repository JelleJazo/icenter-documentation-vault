---
type: module
title: "ISAH sub-services — DataServices, Handlers, ViewModels, Helpers"
status: done
module: "ICenterLib/ISAH"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\DataServices\\EmployeeDataService.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\DataServices\\IsahCustomisingElfsquadDataService.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\DataServices\\MemoDetailDataService.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\DataServices\\ToolboxDataService.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\DataServices\\UpdateProdLeadTimeDataService.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\Handlers\\UpdateProdLeadTimeHandler.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\ViewModels\\CustomerAddressViewModel.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\ViewModels\\PartBasicViewModel.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\Helpers\\EncryptionHelper.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\Helpers\\TextStyling\\IPlainTextHelper.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\Helpers\\TextStyling\\PlainTextHelper.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\Helpers\\TextStyling\\HtmlPlainTextHelper.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\Helpers\\TextStyling\\RtfPlainTextHelper.vb"
last-reviewed: 2026-06-18
tags: [module, needs-review, safety-relevant]
created: 2026-06-18
updated: 2026-06-18
---

# ISAH sub-services — `DataServices`, `Handlers`, `ViewModels`, `Helpers`

> 13 small files grouped together. These are the modern-style additions to the ISAH layer: namespaced `ISAH.DataServices`, `ISAH.Handlers`, `ISAH.ViewModels`, `ISAH.Helpers` — a partial migration from the older root-level `ISAH` classes (which use shared static methods and string-key DataTables).

## `ISAH.DataServices`

### `EmployeeDataService` (16 lines)

Single function `GetFullNameByUsername(username) → String`. Returns the username itself if `Employee.CreateByUsername` returns Nothing/empty `EmpId`; otherwise returns `Employee.GetEmpFullName`. **Safe fallback**: never returns blank. Used as a display helper.

### `IsahCustomisingElfsquadDataService` (80 lines) `#safety-relevant`

HTTP client for the **IsahCustomising REST API** (the bridge service to Elfsquad's external configurator). All three methods are `Async Function`:

- `GetConfigurationIdByDesignCode(designCode) → Task(Of Guid)` — GET `Elfsquad/JIP_Get_ConfigurationIdByDesignCode?DesignCode=...`. Returns `Guid.Empty` on null/empty designCode or zero rows.
- `PostConfigurationIdDesignCodeMapping(designCode, configurationId) → Task(Of ConfigurationDesignCodeMapping)` — POST `Elfsquad/JIP_Ins_ConfigurationIdDesignCodeMapping`, JSON body.
- `DeleteConfigurationIdDesignCodeMapping(designCode, configurationId) → Task(Of ConfigurationDesignCodeMapping)` — DELETE with JSON body (unusual — DELETE with body works but isn't universally supported).

HttpClient pulled from `Connections.GetIsahCustomisingHttpClient()` — defined in [[icenterlib-connections]]. Non-success throws `HttpRequestException`. Q-210 — what's the auth/transport for the IsahCustomising service?

The `designCode` is interpolated raw into the URL (`?DesignCode={designCode}`) — **no URL-encoding**. Q-211 — designCodes with `&`, `?`, `=`, `#` characters break the request. `#safety-relevant`.

### `MemoDetailDataService` (78 lines) `#safety-relevant`

Two `Public Function` plus one helper, all using `DataHandler.GenericQuery` (so they go through the safe parameterised path):

- `GetElfsquadConfigurationId(dossierDetail) → Guid` — joins `T_MemoDetail + T_DossierDetail` on `MemoGrpId`, filters by `MemoTypeCode = @MemoTypeCode` where the type code is **read from `My.Settings.Properties("MemoTypeCodeEC").DefaultValue`**. Deserialises the `Info` column as JSON `ElfsquadConfiguration { Configuration : Guid }`. **`DefaultValue`** is taken, not the user-overridden Value — so app.config settings panel changes never affect this. Q-212 (matches earlier Q-002 pattern).
- `GetElfsquadConfigurationModelId(part) → Guid` — joins `T_MemoDetail + T_Part` on `MemoGrpId`, filters by `MemoTypeCode = @MemoTypeCode` (`MemoTypeCodeECM` setting). Returns the ConfigurationModelId.
- `ParseElfsquadConfigurationModelId(responseBody) → Guid` — JSON helper; deserialises as the empty-shell `MemoDetailElfsquadConfiguration` class (which has zero properties). **The class has no `<JsonProperty>` attributes**, so this always returns `Guid.Empty` — Q-213. `#safety-relevant` because the **bend-station Elfsquad-model lookup will always return Guid.Empty**, possibly disabling the integration silently. Compare to `ElfsquadConfiguration` in the same file which DOES have `<JsonProperty("Configuration")>`.

### `ToolboxDataService` (29 lines)

One `Public Sub`: `JIPX_prc_CalcDeltaWorkingDay(StartDate, EndDate, ByRef Delta)`. SP `JIPX_prc_CalcDeltaWorkingDay`. Same delta-working-day SP family as [[isah-leaves|`DateDimension`]] but uses the `JIPX_` (extension?) prefix. Q-214 — what distinguishes `JIPX_` from `JIP_` from `IPX_` prefixes? `IP_` = stock ISAH, `JIP_` = JAZO custom, but the X variants?

### `UpdateProdLeadTimeDataService` (103 lines) `#safety-relevant`

Two functions wrapping the `JZ_ProdLeadTime` table — **the empirical actuals tracker** that captures real start/end times per production-BOO line:

- `JIP_get_LeadTimeBetweenDates(FromDate, TillDate, MachGrpCode)` → SP `JIP_get_LeadTimeBetweenDates`. **Inner exception silently swallowed** (`Catch ex As Exception ' Empty`) — returns empty DataTable. Outer exception returns `Nothing`. So callers can't distinguish "no data" from "DB error".
- `UpdateRecord(...)` — inline T-SQL **IF NOT EXISTS / BEGIN INSERT / ELSE / BEGIN UPDATE** upsert into `JZ_ProdLeadTime`. Inserts `PrevOperEndTime` and `RealTotalLeadTime` as `DBNull` when `PrevOperEndTime` is `Nothing` (no previous operation found). Uses `DataHandler.GenericQuery` (parameterised — safe from injection).

The upsert key is `(ProdHeaderDossierCode, ProdBOOLineNr)`. This is the **production-lead-time learning table** — possibly fed back into the planning system.

## `ISAH.Handlers`

### `UpdateProdLeadTimeHandler` (68 lines) `#safety-relevant`

The orchestrator that runs the lead-time-actuals job. Constructor takes `(startDate, endDate, machGrpCode)`. `Process()`:

1. `GetWorkData()` → calls `JIP_get_LeadTimeBetweenDates` to find production-BOO lines completed in the window for the machine group.
2. For each row:
   - `GetEndTimePreviousOperation(ProdHeaderDossierCode)` — **switch on MachGrpCode**:
     - `"M38"` → `CoatingLayerThicknessDataService.GetLatestEntryDate(ProdHeaderDossierCode)`.
     - Any other → `throw NotImplementedException`.
   - If a previous-operation end-time was found, `ToolBoxDataService.JIPX_prc_CalcDeltaWorkingDay(prev, RealEndTime, ByRef RealTotalLeadTime)`.
   - **If `RealTotalLeadTime < 0`** ("Unrealyable EndTimePreviousOperation, ignore value" — sic), discard the previous time and set total lead time to 0. Indicates clock drift or data-entry mistake; recovery is silent.
   - `UpdateProdLeadTimeDataService.UpdateRecord(...)` upserts the row.

**Hardcoded machine group `M38`** — same machine-group code that appears in [[isah-part-and-dispatch|the Part dispatch flow]]. Q-215 — if iCenter ever adds another `MachGrpCode` consumer, this handler throws `NotImplementedException`. Should this fall back to "no previous operation"?

`CoatingLayerThicknessDataService` is in `ICenterLib.ICenter.DataServices` (not ISAH) — to be documented in Phase 3d.

## `ISAH.ViewModels`

### `CustomerAddressViewModel` (30 lines)

Plain DTO — 25 auto-properties matching `T_CustomerAddress` columns (CustId, CustAddrCode, CountryCode, Name, Addr/Addr2/Addr3, HouseNumber, PostCode, City, State, Phone, Fax, Email, WebSiteAddr, LastUpdatedOn, Telex, VATNr, VATVerifyDate, CountryDescription, RelRelCode, RelRelName, LastShippingMainDate, CustAddressObsInd). No behavior. Likely UI binding target.

### `PartBasicViewModel` (17 lines)

Plain DTO — `PartCode`, `Description`, `DispatchWarehouseCode`, `DispatchLocationCode`, `DesignCode` + a computed `FullDispatchLocationCode → DispatchWarehouseCode & "." & DispatchLocationCode`. Same dot-notation as [[isah-part-and-dispatch|`Part.FullDispatchLocation`]] (Q-180). Two encodings of the same rule.

## `ISAH.Helpers`

### `EncryptionHelper` (26 lines) `#safety-relevant`

```vb
Public Function EncryptMessage(Message, Password)
    ' XOR each character of the message with the password, password repeats
End Function
```

**XOR cipher** — symmetric, key-repeating, ASCII-only. Not cryptography by any modern definition. If used for any **secret** (password, API key, license token), the secret can be recovered trivially from any ciphertext-plaintext pair. Q-216 — find consumers; if any of them are storing credentials, **escalate**. `#safety-relevant`.

Also: parameters are untyped (`Message, Password` — implicit `Object`). On non-ASCII input the `Chr(Asc(...))` round-trip mangles bytes ≥ 128. No `Public Class EncryptionHelper` namespace pollution issue but **`EncryptMessage = Result`** is VB6-style return assignment — old-VB6 import.

### `TextStyling` interface family

Four files (one interface + three implementations + one dispatcher):

- `IPlainTextHelper` — interface: `Function ToPlainText(value As String) As String`. **Lives in the root namespace, not `ISAH.Helpers.TextStyling`** — Q-217 — namespace mismatch; the implementations also live in root namespace (not `ISAH.Helpers.TextStyling`), only the dispatcher `PlainTextHelper` is in `ISAH.Helpers.TextStyling`.
- `RtfPlainTextHelper` — loads value into a `System.Windows.Forms.RichTextBox`, returns the `.Lines` joined with `vbCrLf`. **Instantiates a WinForms control in a helper** — Q-218 — breaks in headless contexts (services, web).
- `HtmlPlainTextHelper` — XmlDocument.LoadXml(value), iterates `<p>` tags, joins InnerText with vbCrLf. Treats HTML as XML — will fail on real-world HTML with unclosed tags or attributes without quotes.
- `PlainTextHelper` (dispatcher): if value starts (case-insensitive) with `{\rtf` → RTF; with `<body>` → HTML; else returns the value unchanged. The format-sniffing is **string-prefix-only** — well-formed HTML starting with `<html>` (not `<body>`) won't trigger HTML path.

All three catch any exception and return the original value — silent best-effort.

## Surprises

1. **`MemoDetailElfsquadConfiguration` ParseElfsquadConfigurationModelId** always returns `Guid.Empty` because the target class has no `<JsonProperty>` attributes. Possible production bug. Q-213. `#safety-relevant`.
2. **DELETE with JSON body** in `IsahCustomisingElfsquadDataService.DeleteConfigurationIdDesignCodeMapping` — unusual REST style.
3. **`My.Settings.Properties("X").DefaultValue`** pattern (vs `My.Settings.X`) reads the app.config default, **not** the user-scoped overridden value. Same anti-pattern as Q-002.
4. **Designcode URL injection vector** in `GetConfigurationIdByDesignCode` — raw concatenation into URL.
5. **`UpdateProdLeadTimeHandler` hardcoded for M38** — single coating-layer code path. Adding another MachGrpCode throws `NotImplementedException`. Q-215.
6. **XOR cipher in `EncryptionHelper`** — `#safety-relevant` if used for any actual secret.
7. **Helpers/TextStyling namespace inconsistency** — interface + 2 implementations live in root namespace, only the dispatcher is in `ISAH.Helpers.TextStyling`. Q-217.
8. **`RtfPlainTextHelper` instantiates `System.Windows.Forms.RichTextBox`** — kills the helper in headless contexts (CadBatchserver). Q-218.
9. **`ISAH.DataServices` + `ISAH.Handlers` + `ISAH.ViewModels` + `ISAH.Helpers` are the modern style** — using `DataHandler.GenericQuery`, dependency injection in handlers, async HTTP — vs the older root-level `ISAH.*` classes using static methods on raw `SqlCommand` (e.g., `WorkView`, `Customer`). **Partial-migration debt.**

## Business rules surfaced here

- **`M38` → CoatingLayerThickness** is the previous-operation lookup for the lead-time-actuals job. Same `M38` as the Part-dispatch hardcode (Q-176) — coating is JAZO's surface-treatment dept.
- **`MemoTypeCodeEC` setting** = the memo-type code that stores the per-dossier-detail Elfsquad ConfigurationId.
- **`MemoTypeCodeECM` setting** = the memo-type code that stores the per-part Elfsquad ConfigurationModelId.

## Open questions

- **Q-210 (new):** Document the `IsahCustomising` service contract — auth, transport, hosting. (`Connections.GetIsahCustomisingHttpClient`.)
- **Q-211 (new):** `GetConfigurationIdByDesignCode` raw-concatenates `designCode` into URL. URL-encode? `#safety-relevant`
- **Q-212 (new):** `MemoDetailDataService` uses `My.Settings.Properties("X").DefaultValue` instead of `My.Settings.X`. Same anti-pattern as Q-002 — review.
- **Q-213 (new):** `MemoDetailElfsquadConfiguration` has no `<JsonProperty>` attrs, so `ParseElfsquadConfigurationModelId` always returns `Guid.Empty`. **Possible production bug**. `#safety-relevant`
- **Q-214 (new):** Document the SP-prefix system: `IP_` (ISAH), `JIP_` (JAZO ISAH), `JIPX_` (JAZO ISAH extension?), `IPX_` (ISAH extension?), `SIP_` (??).
- **Q-215 (new):** `UpdateProdLeadTimeHandler` hardcoded for `MachGrpCode = "M38"`. Other machine groups throw `NotImplementedException`. Should this fall back to "no previous operation"?
- **Q-216 (new):** `EncryptionHelper` is a XOR cipher. Find consumers — if any store credentials, escalate. `#safety-relevant`
- **Q-217 (new):** `IPlainTextHelper` + `HtmlPlainTextHelper` + `RtfPlainTextHelper` live in root namespace; `PlainTextHelper` in `ISAH.Helpers.TextStyling`. Fix.
- **Q-218 (new):** `RtfPlainTextHelper` instantiates `System.Windows.Forms.RichTextBox` — kills headless usage. Use a non-WinForms RTF parser (e.g., regex strip).

Logged in [[../needs-review/_index]].

## Coverage

`_coverage.md` — 13 files in this batch → `done`:
- `ISAH\DataServices\EmployeeDataService.vb`
- `ISAH\DataServices\IsahCustomisingElfsquadDataService.vb`
- `ISAH\DataServices\MemoDetailDataService.vb`
- `ISAH\DataServices\ToolboxDataService.vb`
- `ISAH\DataServices\UpdateProdLeadTimeDataService.vb`
- `ISAH\Handlers\UpdateProdLeadTimeHandler.vb`
- `ISAH\ViewModels\CustomerAddressViewModel.vb`
- `ISAH\ViewModels\PartBasicViewModel.vb`
- `ISAH\Helpers\EncryptionHelper.vb`
- `ISAH\Helpers\TextStyling\IPlainTextHelper.vb`
- `ISAH\Helpers\TextStyling\PlainTextHelper.vb`
- `ISAH\Helpers\TextStyling\HtmlPlainTextHelper.vb`
- `ISAH\Helpers\TextStyling\RtfPlainTextHelper.vb`

ISAH coverage: **63/65 → 97%**. Remaining: `BillOfMaterialRecord.vb` (32 lines) + `BillOfOperationRecord.vb` (32 lines) — both in BOM/BOO families already covered in [[isah-production-hierarchy]]; will be marked done by reference.
