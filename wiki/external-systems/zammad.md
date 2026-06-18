---
type: external-system
title: "Zammad servicedesk"
status: stub
tags: [external-system, needs-content]
created: 2026-06-18
updated: 2026-06-18
---

# Zammad

Open-source ticketing system. JAZO's servicedesk. Accessed from iCenter via the **Servicedesk widget**.

> **Stub** — populate URL, version, hosting.

## Quick links

- [[../modules/icenterlib-icenter-leaves|`Servicedesk.vb`]] — URL builder + `FrmWebView` host.
- [[../business-rules/icenter-servicedesk-fallback-email|`pvs@jazo.com` fallback email]] for guest / machine-emp tickets.
- `AppSettings.GetStringSetting("ServicedeskFeedbackUrl")` — the new-ticket URL template.
- `AppSettings.GetStringSetting("ServicedeskBaseUrl")` — overview base URL.
- Q-259 / Q-260 — document `pvs@jazo.com` distribution + move literal to AppSettings.
