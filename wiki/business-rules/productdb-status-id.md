---
type: business-rule
title: "Product.StatusId lifecycle (Progress=1 / Ready=2 / Released=99 / Obsolete=999)"
status: needs-review
module: "ICenterLib/ProductDb"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ProductDb\\Product.vb"
last-reviewed: ""
tags: [business-rule, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `Product.StatusId` lifecycle

## Rule

JAZO products have **four lifecycle states**, encoded as integer sentinels with intentional gaps:

| State | Value | Meaning |
|-------|------:|---------|
| Progress | 1 | being authored, not yet quote-able |
| Ready | 2 | author finished, can be uploaded to TEST environment only |
| Released | 99 | approved, can be uploaded to PROD environment |
| Obsolete | 999 | discontinued, no further uploads |

The 2→99 gap leaves room for intermediate statuses; the 99→999 gap likewise.

## Where it lives

- File: `ICenterLib\ProductDb\Product.vb` lines 15-20
- Symbol: `Public Enum StatusId`

## The code

```vb
Enum StatusId
    Progress = 1
    Ready = 2
    Released = 99
    Obsolete = 999
End Enum
```

## Why this is a business rule

The status drives:

- **Upload-to-environment** ([[productdb-upload-status-gate]]) — Released → both envs, Ready → TEST only.
- **Display filters** (`GetTreeNodes(BasicReadOnly, ...)` shows only Released; `AdminMode` shows both Progress and Released — but **not Obsolete**, since the upper bound is Released=99 not 999).
- **DB-side queries** must use the same values; `SIP_Get_Products(@SelectionType, @MaxStatus)` ranges by these.
- **Quote validity** — only Released products can be customer-facing.

Magic numbers everywhere; any code that hardcodes a comparison `If StatusId = 99` rather than `If StatusId = Product.StatusId.Released` is invisible to refactors.

## Triggers / when it fires

- Every `Product.GetStatusId()` consumer.
- Every WHERE clause filtering by status in ProductDb.

## Effects

- **Progress (1)**: invisible to non-admin users; cannot upload.
- **Ready (2)**: visible to admins; can upload to TEST only (per Q-267 / business-rule).
- **Released (99)**: visible to all; can upload to PROD or TEST.
- **Obsolete (999)**: hidden from default tree (`GetTreeNodes` filters `MaxStatus = Released = 99`).

## Edge cases

- `GetStatusId` returns `-1` if the DB query returns no value (`If o Is Nothing Then Return -1`). Callers must guard against `-1` matching no enum value.
- The `DisplayMode.AdminMode = 99` coincides numerically with `Released = 99`. Different domains but **easy confusion**.
- `GetTreeNodes` upper bound is `StatusId.Released` not `StatusId.Obsolete`, so **Obsolete products never appear** in the tree-view even in AdminMode. Probably intentional.

## Safety classification

- [ ] Touches physical process — no.
- [x] Drives cost / pricing — yes (only Released products are quote-able).
- [x] Reversible if wrong? — yes (rerun status update).
- [ ] Blocks production if it fails? — no.

## SME questions

- **Q-275 (new):** Are statuses 3-98 reserved? If so for what? Document the gap semantics.
- **Q-276 (new):** Why does `GetTreeNodes` cap at Released rather than Obsolete? Should there be an "Include Obsolete" toggle?

Logged in [[../needs-review/_index]].

## Related

- [[../modules/icenterlib-productdb-product|`Product`]] — defining module.
- [[productdb-upload-status-gate|Upload-to-environment gate]] — derived rule.
- [[productdb-system-publisher-empid|System EmpId 0798]] — submits checkin jobs.
