---
type: moc
title: "ICenterLib/ProductDb — Product database wrappers"
status: draft
module: "ICenterLib/ProductDb"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ProductDb\\"
tags: [moc, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# ICenterLib/ProductDb

## What this hub covers

`ICenterLib\ProductDb\` — wrappers over the **ProductDb** SQL database (`Connections.ConnectProductDb`), the JAZO **product-master**. Products carry: status (Progress→Ready→Released→Obsolete), control-definition XML, Excel templates for quoting, language translations, price-lists, configurator mappings, product-group tree, surface-treatment defaults.

21 source files (~150 KB) + 5 generated (.Designer). Two large forms: `FrmProductDbPriceList`, `FrmProductPricePart`, `UCPropertyDefinitionEditor`, `UCPropertyDefinitionManager`.

## Files

| File | Role |
|------|------|
| `Product.vb` (35 KB) | core entity — see [[../modules/icenterlib-productdb-product]] |
| `CloneProductHandler.vb` | clone-job orchestrator |
| `CommonDb.vb` | static helpers for ProductDb |
| `DeclarationOfPerformance.vb` | CE doc generation (`PdfSuffix` const) |
| `ExcelTemplate.vb` | Excel-template entity |
| `ExcelWorkBookHelper.vb` | Excel I/O helper |
| `FrmCloneProduct.vb` | clone-product UI |
| `FrmProductDbPriceList.vb` | price-list editor |
| `FrmProductPricePart.vb` | price-part editor |
| `Language.vb` | language enum/lookup |
| `PriceListHelper.vb` | price-list helper |
| `PricePart.vb` | price-part entity |
| `ProductConfiguration.vb` | configuration entity |
| `ProductConfigurationDataService.vb` | config DataService |
| `ProductConfiguratorMapping.vb` | configurator → product mapping |
| `ProductFilter.vb`, `ProductFilterGroup.vb` | filter tree |
| `ProductGroup.vb` | top-level + sub-group classification |
| `ProductPricePart.vb` | linkage of product + price-part |
| `UCPropertyDefinitionEditor.vb` | property-def editor UI |
| `UCPropertyDefinitionManager.vb` | property-def manager UI |

## Business rules surfaced

- [[../business-rules/productdb-status-id|Product.StatusId enum]] — `Progress=1, Ready=2, Released=99, Obsolete=999`. Magic-number lifecycle.
- [[../business-rules/productdb-upload-status-gate|Upload-to-environment status gate]] — `Released` → both PROD and TEST allowed; `Ready` → TEST only; else blocked.
- [[../business-rules/productdb-system-publisher-empid|System-publisher EmpId "0798" for checkin jobs]] — hardcoded JAZO automation account in `Product.CreateCheckinJob`. `#safety-relevant`
- **Magic ControlIds** in surface-treatment-default extraction:
  - `ControlId='81'` → color code
  - `ControlId='168'` → coating-warranty id
  - `ControlId='82'` → product-type id
  - Fallback `AttrName='DefaultTestValue'` when `DefaultValue` absent. Q-267.
- **`ControlDefinitionBuilder.Version = 3` triggers verbose XML dump** to `c:\temp\dump_{ControlDefId}_{LangCode}.xml`. **Hardcoded local path** — only useful on dev machines. Q-268.

## Notable findings

1. **`Product.GetField(Field) → Object`** uses `"SELECT " & Field & " FROM T_Product..."` — **column-name SQL injection** if Field comes from anywhere external. Probably internal-only. Q-269 `#safety-relevant`.
2. **`SIP_Ins_AutomJob_v2` hardcoded params**: `@AutomationId=44`, `@JobTypeId=2`, `@EmpId="0798"` — embedding both the automation job-type and the system-publisher EmpId. Q-270.
3. **`CopyToNew`**: iterates DataTable rows and File.Copy each Source→Destination per row. **No transaction**: partial copy on failure. Also creates the destination directory if missing.
4. **`UpdateExcelTemplateId` / `UpdateControlDefId` / `UpdateControlMapId`** — all use the `IF EXISTS ... UPDATE ... ELSE ... INSERT` inline-SQL upsert pattern (same as `UpdateProdLeadTimeDataService`).
5. **`Product` mixes ProductDb-side and ISAH-side**: `GetDesignCode()` resolves PartCode via ProductDb then queries `ISAH.Part(PartCode).GetDesignCode`. The class spans two databases.
6. **`OpenFilterUrl` / `OpenSettingsUrl`** read `JIBA.AppParameter("JibaUrl").GetValue` for the portal base URL, then construct `{BaseUrl}js/ProductDb/Filters.html?ProductId={ProductId}` and call `Common.OpenInChrome`. **Chrome explicitly named** — Edge/Firefox don't get the alternative path. Q-271.
7. **`SIP_Get_Products` SelectionType=5**, `SIP_Get_ProductDefinition`, `SIP_Get_ProductForXmlXchange_v1` — three of many ProductDb SPs.
8. **Surface-treatment extraction in `GetDefaultSurfaceTreatmentDefinition` accesses `DS.Tables(24)`** — magic table-index. Fragile to SP-result-shape changes. Q-272 `#safety-relevant`.

## Open questions

- **Q-267 (new):** Document the meaning of ControlIds `81` (color), `82` (product type), `168` (coating warranty). Where defined?
- **Q-268 (new):** `Product.SIP_Get_ProductForXmlXchange` dumps to `c:\temp\dump_*.xml` when `ControlDefinitionBuilder.Version = 3 AndAlso VerboseLogging` — review before prod.
- **Q-269 (new):** `Product.GetField(Field)` interpolates column name into SQL. Column injection risk. `#safety-relevant`
- **Q-270 (new):** Hardcoded `@EmpId="0798"` for automation-job submission — see [[../business-rules/productdb-system-publisher-empid]].
- **Q-271 (new):** `OpenInChrome` hardcoded — what if Chrome isn't installed?
- **Q-272 (new):** `DS.Tables(24)` magic index in `GetDefaultSurfaceTreatmentDefinition`. `#safety-relevant`

Logged in [[../needs-review/_index]].

## Coverage

All 21 source files marked `done` via this MOC + the dedicated Product note. Designer files marked `generated`.
