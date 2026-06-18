---
type: external-system
title: "Telesis laser engraver"
status: stub
tags: [external-system, needs-content, safety-relevant]
created: 2026-06-18
updated: 2026-06-18
---

# Telesis laser engraver

Laser part-marking device driven from iCenter via RS-232 serial port at **9600 baud, 8N1, no handshake, RTS forced high**.

> **Stub** — populate model number, firmware version, vendor URL.

## Quick links

- [[../modules/icenter-marktool|`iCENTER\MarkTool\`]] — the COM-port driver + UI.
- `T_ProdMachines.MarkToolComPort` — per-station COM-port assignment.
- Q-327 — confirm 9600 8N1 matches current firmware.
- Q-332 `#safety-relevant` — IPpart-prefix substitution silently engraves Modelname instead of scanned text.
