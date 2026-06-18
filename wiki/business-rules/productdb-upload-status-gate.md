---
type: business-rule
title: "Product upload-to-environment status gate"
status: needs-review
module: "ICenterLib/ProductDb"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\ProductDb\\Product.vb"
last-reviewed: ""
tags: [business-rule, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Product upload-to-environment status gate

## Rule

The rule deciding **which products can be uploaded to which environment**:

| `Product.StatusId` | Upload PROD? | Upload TEST? |
|-------------------|:---:|:---:|
| `Progress (1)` | ❌ | ❌ |
| `Ready (2)` | ❌ | ✅ |
| `Released (99)` | ✅ | ✅ |
| `Obsolete (999)` | ❌ | ❌ |

Released → both envs; Ready → TEST only; Progress / Obsolete → no upload at all.

## Where it lives

- File: `ICenterLib\ProductDb\Product.vb` lines 261-281
- Symbol: `Product.GetIsUploadAllowed(Environment As TypeOfEnvironment[, StatusId As StatusId]) As Boolean`

## The code

```vb
Public Function GetIsUploadAllowed(Environment As TypeOfEnvironment, StatusId As StatusId) As Boolean
    Dim Result As Boolean = False

    Select Case StatusId
        Case StatusId.Released
            Result = True

        Case StatusId.Ready
            If Environment = TypeOfEnvironment.TEST Then
                Result = True
            End If

    End Select

    Return Result
End Function
```

`StatusId.Progress` and `StatusId.Obsolete` fall through with `Result = False`.

## Why this is a business rule

This gate **enforces author-discipline**:
- A product still in Progress can't accidentally reach a customer because PROD upload is locked.
- A product Ready for testing can be deployed to TEST without committing to PROD.
- Only after the explicit Released transition can the product face PROD.
- Obsolete products are frozen — no overrides.

The rule sits in the data layer (`Product` entity), not in a separate authorization service. This means any caller calling `CreateCheckinJob` directly **bypasses the gate** unless they also call `GetIsUploadAllowed` first. Q-277.

## Triggers / when it fires

- Anywhere `Product.GetIsUploadAllowed(env)` is consulted — primarily on the upload-button enable/disable logic in the product UI (likely `FrmProductDbPriceList` or sibling forms).

## Effects

- Returns `True` → upload allowed; UI button enabled; `CreateCheckinJob` may be called.
- Returns `False` → button disabled; user sees no path to upload.

## Edge cases

- **No `Default` case** — any unrecognised `StatusId` (e.g., `-1` from `GetStatusId` when the row doesn't exist) returns `False`. Safe default.
- **PROD also requires the upload context** to honour the Boolean — if the caller ignores `GetIsUploadAllowed`, nothing prevents the upload. Trust-the-caller design.
- **No `Released → TEST only` option** — once Released, both envs are open. If JAZO ever needs to release-to-PROD without TEST-availability, the rule needs reversal.

## Safety classification

- [ ] Touches physical process — no.
- [x] Drives cost / pricing — yes (controls customer-facing product catalog).
- [x] Reversible if wrong? — yes (rerun status update).
- [x] Blocks production if it fails? — no, but a wrongly-allowed upload puts a Progress product on the public site.
- `#safety-relevant` because it's the **only gate between authors and the public catalog**.

## SME questions

- **Q-277 (new):** Is the gate enforced anywhere except in `Product.GetIsUploadAllowed`? Should there be a server-side check?
- **Q-278 (new):** Should Released-to-TEST-only be a possible state?

Logged in [[../needs-review/_index]].

## Related

- [[../modules/icenterlib-productdb-product|`Product`]] — defining module.
- [[productdb-status-id|Product.StatusId lifecycle]] — referenced enum.
- [[productdb-system-publisher-empid|System EmpId 0798]] — what actually runs the upload.
