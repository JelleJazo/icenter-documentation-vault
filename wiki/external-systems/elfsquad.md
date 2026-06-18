---
type: external-system
title: "Elfsquad"
status: stub
tags: [external-system, needs-content]
created: 2026-06-18
updated: 2026-06-18
---

# Elfsquad

Visual product-configurator SaaS. Integrated via the **IsahCustomising** intermediary service.

> **Stub** — populate tenant, endpoints, contact, customer-facing URL.

## Quick links

- [[../modules/isah-sub-services|`IsahCustomisingElfsquadDataService`]] — HTTP client.
- ConfigurationId stored in `T_MemoDetail` rows keyed by `MemoTypeCodeEC` setting; ConfigurationModelId by `MemoTypeCodeECM`.
- Q-213 `#safety-relevant` — `MemoDetailElfsquadConfiguration` has no JsonProperty attrs; `ParseElfsquadConfigurationModelId` always returns `Guid.Empty`.
- Q-211 `#safety-relevant` — URL-encoding gap in `GetConfigurationIdByDesignCode`.
