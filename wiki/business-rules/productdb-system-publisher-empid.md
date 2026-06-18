---
type: business-rule
title: "System-publisher EmpId 0798 for ProductDb checkin jobs"
status: needs-review
module: "ICenterLib/ProductDb"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ProductDb\\Product.vb"
last-reviewed: ""
tags: [business-rule, needs-review, safety-relevant]
created: 2026-06-18
updated: 2026-06-18
---

# System-publisher EmpId — `0798`

## Rule

When iCenter submits a **product-checkin automation job** to JIBA (`SIP_Ins_AutomJob_v2`), the SP-level `@EmpId` parameter is **hardcoded to `"0798"`** — JAZO's system-publisher service account. The iCenter user's actual EmpId is carried separately in the `<JIBATask><EmpId>` XML payload.

## Where it lives

- File: `ICenterLib\ProductDb\Product.vb` line 81
- Symbol: `Product.SIP_Ins_AutomJob_v2(XmlParams, Description)` — `@EmpId` parameter hardcoded.

## The code

```vb
.AddWithValue("@EmpId", "0798")
.AddWithValue("@ProductionJob", 1)
```

(For context, the iCenter user's EmpId is in the XML payload:)

```xml
<JIBATask>
  <EmpId>{passed-in EmpId}</EmpId>
  ...
</JIBATask>
```

## Why this is a business rule

JAZO's JIBA automation framework attributes scheduled / triggered jobs to **an EmpId for audit and ownership**. Submitting every iCenter→ProductDb checkin under the real user's EmpId would clutter that audit trail with hundreds of one-shot jobs and require every user to have JIBA-job-submitter permissions. Instead, all such submissions are attributed to a single service-account EmpId, **`0798`**, and the real user is recorded inside the XML payload.

Whoever owns EmpId `0798` in JIBA's `T_Employee` is effectively the **iCenter-product-checkin service account**. Q-279.

## Triggers / when it fires

- Anywhere `Product.CreateCheckinJob(Environment, EmpId, UploadToWeb)` is called.
- Phase-3 follow-up: enumerate UI callers (likely `FrmProductDbPriceList.btnPublish_Click` or similar).

## Effects

- Row inserted into JIBA's `T_AutomJob_v2` table with `EmpId = "0798"`.
- Job audit-trail attributes every product-checkin to the service account.
- Real user only recoverable by parsing `XmlParams`.

## Edge cases / known exceptions

- If EmpId `0798` is ever removed from `T_Employee` (e.g., termination process), `SIP_Ins_AutomJob_v2` may fail FK validation (depending on schema).
- If the service account's permissions get pruned, every iCenter user loses the ability to publish products.
- **Hardcoded** — no fallback if "0798" is the wrong EmpId for a future JIBA deployment / environment.

## Safety classification

- [ ] Touches physical process — no.
- [x] Drives cost / pricing — indirectly (gates whether products reach the public catalog).
- [ ] Reversible if wrong? — yes (delete the job).
- [x] Blocks production if it fails? — yes (no user can publish a product if `0798` becomes invalid). `#safety-relevant`

## SME questions

- **Q-279 (new):** Document the JIBA service account EmpId `0798` — what permissions does it carry? Who manages it?
- **Q-280 (new):** Move to AppSettings to avoid source change if the account ever moves.
- **Q-281 (new):** Confirm JIBA `T_AutomJob_v2` schema's FK on EmpId.

Logged in [[../needs-review/_index]].

## Related

- [[../modules/icenterlib-productdb-product|`Product`]] — defining module.
- [[productdb-upload-status-gate|Upload-to-environment gate]] — the gate that decides if `CreateCheckinJob` is even called.
- [[../modules/icenterlib-jiba-employee-asset|`JIBA.Employee`]] — backing identity store.
