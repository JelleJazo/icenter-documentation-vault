---
type: module
title: "ISAH dossier hierarchy — DossierMain / DossierDetail / Extra / DocFolder"
status: done
module: "ICenterLib/ISAH"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\DossierMain.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\DossierDetail.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\DossierDetailExtra.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\DossierDetailExtraDto.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\DossierDocFolder.vb"
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ISAH\\Helpers\\DossierDetailExtraHelper.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# ISAH dossier hierarchy

## Purpose

Six classes that together model **ISAH's sales/order "dossier"** — the central shell every iCenter workflow ultimately attaches to.

```
DossierMain   (T_DossierMain)        — header: OrdNr, QuotNr, CustId, DelDate, OrdType, OrdDate
   └── DossierDetail (T_DossierDetail) — per-line items, keyed by (DossierCode, DetailCode, DetailSubCode)
          └── DossierDetailExtra (ST_0099_DossierDetailExtra) — per-line "extra info" view
                 + DossierDetailExtraDto  — DTO sent to JAZO ISAH web app
                 + Helpers/DossierDetailExtraHelper — builds the encrypted URL to the web app

DossierDocFolder — resolves IsahDoc folder path for a dossier (Verkoop\Order\<year>\<OrdNr>\)
```

The dossier is **both quote and order** — `DossierMain.DossierType` distinguishes them via the `OrdNr` / `QuotNr` columns. A dossier starts as a quote and becomes an order when the QuotNr → OrdNr transition is made in ISAH.

## `DossierMain`

**Source**: `ISAH\DossierMain.vb` (411 lines).

### Public surface (selected)

| Symbol | Returns | Notes |
|--------|---------|-------|
| `New(DossierCode)` | constructor | DossierCode is the surrogate primary key |
| `Const PlanIntMonPartCode = "PLAN INT MONT"` | string | hard-coded part-code for internal-mounting plan lines |
| `Const PlanTekWvBPartCode = "PLAN TEK WVB"` | string | hard-coded part-code for drawing-WvB plan lines |
| `Enum DossierType { Unknown=0, Quote=1, Order=2 }` | | |
| `Shared CreateByOrdNrOrQuotNr(value)` | `DossierMain` | factory: look up DossierCode by either OrdNr OR QuotNr |
| `Shared GetDossierCodeByOrdNrOrQuotNr(value)` | string | the underlying lookup query |
| `Shared GetDossierCodeByShopDocCode(value)` | string | 4-table join: ShopDoc → PBOO → ProductionHeader → ProdHeadDosDetLink → DossierDetail |
| `Shared GetDossierMainRecord(DossierCode)` | `DataTable` | SP `IP_sel_DossierMainRecord` |
| `GetOrdType / GetQuotNr / GetDescription / GetDelDate / GetOrdNr / GetPriceCustId` | strings / dates | thin accessors over `GetRecord` |
| `GetProdPlanEndDate()` | `DateTime?` | `DelDate` shifted by `AppSettings.GetIntegerSetting("ProdPlanEndDate.WorkingDayOffset")` working days via `DateDimension.GetFirstWorkDay` |
| `GetAdminCode()` | string | `MultiFinance.GetAdminCodeByOrdType(GetOrdType())` |
| `GetCompany()` | [[isah-identity\|`Company`]] | `Company.CreateByAdminCode(MultiFinance.GetAdminCodeByDossierCode(DossierCode))` |
| `Shared GetYearFromQuotOrdNr(value)` | string | year of the quote/order (see surprises) |
| `GetDossierType(RefNr)` | `DossierType` | compares `RefNr` to OrdNr (→ Order) and QuotNr (→ Quote) |
| `Shared GetDossierLabelByRefNr(value) / GetDossierLabel(DossierType)` | string | UI labels: `"Verkooporder"` / `"Verkoopofferte"` / `"Dossier"` |
| `GetCustDetails()` | `DataTable` | joins `T_Customer + T_CustomerAddress` for customer info |
| `GetOrdTypeSurChargeCode() / Codes()` | string / `DataTable` | reads `JZ_OrdTypeSurChargeLink` for the order-type surcharge |
| `GetMontDetailCode()` | `DataTable` | hard-coded `PartCode IN ('MONTAGE TP','090','PLAN EXT MONT','CSA00011')` mount-detail lookup |
| `GetDossierDetailSubRecords(DetailCode, IncludeMainRecord)` | `DataTable` | sub-lines under a parent detail (`DetailSubCode <> '000'` for sub only) |
| `GetDesignCodes(FilterStandards)` | `DataTable` | distinct design-codes with `DD.PartCode NOT LIKE 'CA-%' AND NOT LIKE 'CH-%' AND DossierStatusCode <> '00'` |
| `AutoUpdatePlanLines` / `AutoUpdatePlanLinesByOrdType(OrdType)` | Boolean | reads iCenter DB setting `DossierMain.AutoUpdatePlanLines` (a list-style setting; see [[icenterlib-appsettings\|`SettingContainsValue`]]) |
| `Shared IsExistingOrdNrOrQuotNr(value)` | Boolean | true iff `CreateByOrdNrOrQuotNr(value)` finds anything |

### Surprises

1. **`GetYearFromQuotOrdNr(value)`** (lines 324–346) — the year resolver:
   - If the value starts with **non-digit** (regex `^\D`): build a DossierMain, read the actual `OrdDate` / `QuotDate` column, return the year.
   - If the value starts with **digits**: `"20" & value.Substring(0, 2)` — assumes Y20xx. **Q-148**: breaks in Y2100 (89 years out — not urgent, but worth flagging).
2. **`GetMontDetailCode`** hard-codes four part-codes: `'MONTAGE TP'`, `'090'`, `'PLAN EXT MONT'`, `'CSA00011'`. SME-friendly meaning of each unknown without context.
3. **`GetDesignCodes`** filters `PartCode NOT LIKE 'CA-%' AND NOT LIKE 'CH-%'`. CA / CH prefix convention — Q-149.
4. **`DossierStatusCode <> '00'`** in `GetDesignCodes` — status `'00'` likely means "deleted/cancelled". Phase-4 candidate.
5. **All `Get<X>` accessors re-query** `GetDossierMainRecord` per call. A single dossier-detail dump that reads 10 fields runs 10 SP calls. No caching.

## `DossierDetail`

**Source**: `ISAH\DossierDetail.vb` (548 lines). Composite key: `(DossierCode, DetailCode, DetailSubCode)`.

### Public surface (selected)

| Symbol | Returns | Notes |
|--------|---------|-------|
| `New(DetailCode, DetailSubCode, DossierCode)` | constructor | note parameter order: detail first, dossier last |
| `Shared GetPrintCodeByPartCodeAndDesignCode(PartCode, DesignCode)` | `Enums.ISAH.PrintCode` | `DesignCode = "DUMMY"` → `NietPrinten` ("don't print"); else delegates to `Part.GetPrintCode(PartCode)` |
| `GetOrdNr()` | string | via parent DossierMain |
| `Exists` | Boolean | property — true iff the SP returns rows |
| `GetCommonCaption()` | string | builds `"<OrdNr or QuotNr> <DetailCode>-<DetailSubCode>"` for UI display |
| `IP_sel_DossierDetailRecord(...)` / `GetDossierDetailRecord()` | `DataTable` | SP wrapper — hard-codes `IsahUserCode="ISAH"` and `LogProgramCode=340000` |
| `GetLastUpdatedOn()` / `GetPartCode()` / `GetInfo()` / `GetDesignCode()` | per-field accessors | over `GetDossierDetailRecord` |
| `GenProductionHeader()` | string | **creates** a new ProductionHeader via SP `IP_gen_ProdHeadForDosDet`, returns the new ProdHeaderDossierCode |
| `GetProdHeaderDossierCode()` | string | reads `T_ProdHeadDosDetLink` to find the existing linked PH |
| `IP_gen_ProdHeadForDosDet(...)` | sub | the SP wrapper — **14 fixed-zero parameters** (PrcStatusDD/PH, PrcRoutingDD/PH, DebugInd) hard-coded `0` |
| `Shared IP_Ins_DossierDetail(...)` | `DossierDetail` | 28-parameter insert — full record creation |
| `Shared GetDateAsString(value)` | string | formats as `MM/dd/yyyy` with `-` → `/` replacement |

### Surprises

1. **Constructor parameter order** is `(DetailCode, DetailSubCode, DossierCode)` — most other ISAH classes go `(DossierCode, ...)`. Easy to call wrong. Q-150.
2. **`IP_gen_ProdHeadForDosDet` hard-codes 14 parameters to 0**. The SP signature accepts custom routing / status flags but iCenter never uses them. Either the SP was over-spec'd or iCenter doesn't yet support those features.
3. **`LogProgramCode = 340000`** in `IP_sel_DossierDetailRecord` — a magic ISAH log-program code identifying *which iCenter function* hit ISAH. Likely unique per call site (see also `11220000` in `IP_gen_ProdHeadForDosDet`). Q-151.
4. **`IsahUserCode = "ISAH"`** is used as the writer's identity in both SPs — not `APPLISAHUSERCODE = "ICENTER"` from `Common.vb`. Drift from the application-wide constant.
5. **`GetPrintCodeByPartCodeAndDesignCode`** has a special-case for `DesignCode = "DUMMY"` (case-insensitive) → return `NietPrinten`. The `DUMMY` sentinel is a domain convention. Q-152.
6. **`IP_Ins_DossierDetail`** has 28 parameters in a single call. If a new ISAH column is added, the consumer + every callsite need updating.

## `DossierDetailExtra` + `Dto` + `Helper`

Three small classes that together let iCenter open the **JAZO ISAH web app** at a specific DossierDetail row for "extra info" editing.

### `DossierDetailExtra` (39 lines)

Reads `ST_0099_DossierDetailExtra` (note the `ST_` prefix — JAZO custom *staging-table* convention? — Q-153). Single method `GetDossierDetailExtra() As DataTable`. No write methods.

### `DossierDetailExtraDto` (8 lines)

Four-property POCO: `DossierCode`, `DetailCode`, `DetailSubCode`, `UserCode`. Used only as the JSON-serialised body of the URL the helper builds.

### `DossierDetailExtraHelper` (40 lines)

```vb
Public Function GetUrl(UseIsahTestDb As Boolean) As String
    Dim BaseAddress As String
    If UseIsahTestDb Then
        BaseAddress = AppSettings.GetStringSetting("JAZO.Isah.App.Dev.BaseAddress")
    Else
        BaseAddress = AppSettings.GetStringSetting("JAZO.Isah.App.Prod.BaseAddress")
    End If

    Dim Path As String = My.Settings.Properties("DossierDetailExtraPath").DefaultValue
    Dim ContentArgument As String = GetArguments()
    ' Returns "<BaseAddress>/<Path>?Content=<UrlSafe(Base64(EncryptedMessage))>"
End Function
```

The "Content" query parameter is:
1. JSON-serialise the DTO.
2. Encrypt via `EncryptionHelper.EncryptMessage(json, password)` — **password = `Encoding.UTF8.GetString(Convert.FromBase64String(My.Settings.DossierDetailExtraPassword))`** — i.e. the encryption password lives in `app.config` base64-encoded.
3. Base64-encode the cipher.
4. URL-encode that.

So a single URL navigation carries the dossier identity encrypted into the path. **The password is in `app.config`** (Q-154, `#safety-relevant`) — anyone with `app.config` can decrypt URLs.

The dev/prod BaseAddress switch matches `Connections.UseIsahTestDb`'s naming pattern but uses **iCenter DB settings**, not the Connections toggle. Q-155 — are the two switches coordinated?

## `DossierDocFolder` (23 lines)

Single static method `GetIsahDocSalesFolder(Key)` that resolves the IsahDoc folder for a sales/quote dossier:

```
<IsahDocRoot> + "Verkoop\" + ("Offerte\" if Quote else "Order\") + <year> + "\" + <Key>
```

Where:
- `IsahDocRoot` from `app.config` = `\\jazo.local\dfs\Proglinks\IsahDoc\`
- Year resolved via `DossierMain.GetYearFromQuotOrdNr` (with the Y2100 quirk above)
- `Key` is the OrdNr or QuotNr

Used elsewhere when iCenter needs to look up dossier-attached documents on the share.

## Business rules surfaced here

- [[../business-rules/isah-dossier-mount-partcodes|Hard-coded mount-detail part-codes]] — `GetMontDetailCode` filters on the fixed set `{'MONTAGE TP', '090', 'PLAN EXT MONT', 'CSA00011'}`.
- **Plan-line part-codes**: `PlanIntMonPartCode = "PLAN INT MONT"`, `PlanTekWvBPartCode = "PLAN TEK WVB"` — hard-coded constants for "plan" rows iCenter recognises.
- **DesignCode `"DUMMY"`** → `PrintCode.NietPrinten` (Dutch: "not printing"). Q-152.
- **DossierStatusCode `'00'`** = excluded/deleted (filtered out in `GetDesignCodes`). Phase-4 follow-up.
- **PartCode prefixes `CA-` and `CH-`** = excluded from `GetDesignCodes`. Q-149.

## Open questions

- **Q-148 (new):** `GetYearFromQuotOrdNr` assumes `"20" + 2-digit prefix` for digit-prefixed values. Y2100 breakage. `#low`
- **Q-149 (new):** `PartCode NOT LIKE 'CA-%' AND NOT LIKE 'CH-%'` in `GetDesignCodes` — what do CA / CH prefixes mean?
- **Q-150 (new):** `DossierDetail.New(DetailCode, DetailSubCode, DossierCode)` parameter order is inverted vs other ISAH classes — easy to call wrong. `#low`
- **Q-151 (new):** `LogProgramCode` magic numbers (340000, 11220000). Document the convention — what does each value identify?
- **Q-152 (new):** `DesignCode = "DUMMY"` sentinel. Document the convention and identify other sentinels.
- **Q-153 (new):** `ST_0099_DossierDetailExtra` table — what does `ST_` prefix mean (staging table?) and `0099` numeric suffix?
- **Q-154 (new):** `DossierDetailExtraPassword` is a base64-encoded `My.Settings` value used as encryption password for ISAH-app URLs. App.config = key + ciphertext together. `#safety-relevant`
- **Q-155 (new):** `DossierDetailExtraHelper` uses iCenter-DB BaseAddress switch (`JAZO.Isah.App.Dev/Prod`) while connections use the in-process `Connections.UseIsahTestDb`. Coordinated?

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/icenterlib-isah]] — parent MOC.
- [[isah-identity|`Company`]] — `DossierMain.GetCompany()` returns one.
- [[isah-shop-and-pur-doc|`ShopDoc`]] — `DossierMain.GetDossierCodeByShopDocCode` joins through it.
- [[isah-machgrp]] — DossierDetail's downstream MachGrpCode comes via the linked ProductionHeader → PBOO chain.
- [[icenterlib-appsettings|`AppSettings`]] — `AutoUpdatePlanLines` uses the wildcard-style list lookup.

## Coverage

`_coverage.md`:
- `ICenterLib\ISAH\DossierMain.vb` → `done`
- `ICenterLib\ISAH\DossierDetail.vb` → `done` (overview level; the full `IP_Ins_DossierDetail` 28-parameter insert deferred)
- `ICenterLib\ISAH\DossierDetailExtra.vb` → `done`
- `ICenterLib\ISAH\DossierDetailExtraDto.vb` → `done`
- `ICenterLib\ISAH\DossierDocFolder.vb` → `done`
- `ICenterLib\ISAH\Helpers\DossierDetailExtraHelper.vb` → `done`
