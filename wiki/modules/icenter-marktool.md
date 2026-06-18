---
type: module
title: "iCENTER/MarkTool — Telesis laser engraver COM-port driver"
status: done
module: "iCENTER/MarkTool"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\MarkTool\\ClsComPort.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\MarkTool\\ClsMarkToolDb.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\MarkTool\\frmMarkTool.vb"
last-reviewed: 2026-06-18
tags: [module, safety-relevant, needs-review, external-system]
created: 2026-06-18
updated: 2026-06-18
---

# `iCENTER\MarkTool\` — Telesis laser-engraver COM-port driver

> **Drives a Telesis laser engraver via a serial COM port.** Operator scans a Modelname (or IPpart-prefixed code which gets stripped to Modelname), the text is shipped to the engraver over RS-232, the engraver marks the part. All sessions are logged. `#safety-relevant` — driving a laser, even a marking-power one.

The author header (in `ClsComPort.vb`) gives provenance:
> `'Naam: René Rakitow / Datum: 08-07-2010 / Omschrijving: Geschreven voor testen van Telesis graveerapparaat`
>
> Originally written for **testing** a Telesis engraver on **8 July 2010**.

## Files

- **`ClsComPort.vb`** (~66 lines) — RS-232 serial-port wrapper.
- **`ClsMarkToolDb.vb`** (~146 lines) — message-log DB CRUD.
- **`frmMarkTool.vb`** (~205 lines) — operator UI + send loop.

## `ClsComPort.SetComPort()` — the serial config

Hardcoded:
```vb
.PortName  = ComPortName               ' from T_ProdMachines.MarkToolComPort
.BaudRate  = 9600
.Parity    = Parity.None
.DataBits  = 8
.StopBits  = StopBits.One
.Handshake = Handshake.None
.RtsEnable = True
```

**8N1 at 9600 baud, no handshake, RTS forced high.** Standard Telesis-engraver-friendly config. Q-327 — confirm this matches current Telesis firmware.

`ReadFromCom` has a **dead infinite loop**: `While abortThread = False` with no `abortThread = True` anywhere. Dead code — `ReadFromCom` is never called (search reveals no consumer). Q-328.

## `frmMarkTool.SendData(sString)` — the send loop

Core path:

1. Build session: `SessionId = oDb.GetSessionId + 1` (= MAX(SessionId)+1, **not** atomic — two concurrent sends can race the same SessionId). Q-329.
2. Look up `ComPortName` from `prodMachines.GetStringSettingValue(ProdMachineId, "MarkToolComPort")`.
3. If empty → MsgBox `"Configuratiefout: ... Er is geen COM-poort ingesteld voor dit station."`
4. Open port, **send `Chr(13)` first** (CR), then `sString.Replace("-", " ") & Chr(13)`. **All hyphens are converted to spaces** before sending. Q-330 — Telesis-encoding constraint, or text-formatting choice?
5. Each open/close + each send is logged via `ClsMarkToolDb.SetSessionLog(..., MessageTypeId, MessageText, EmpId)` with these MessageTypeIds:
   - **1** = data sent
   - **2** = port-busy wait (1 second)
   - **3** = exception/error
   - **4** = port state changes (opened/closed)
6. If port is already open ("busy"), **retry up to 100 times with 1-second sleeps** between checks. Total worst case: **100 seconds blocking** the UI thread before reporting failure. Q-331 `#safety-relevant`.

## `TXT_Code_KeyDown` — the operator-facing scan handler

```vb
If e.KeyCode = Keys.Enter Then
    Dim MarkText As String = TxtDesignCode.Text.Trim.ToUpper
    If MarkText.StartsWith(ICenterLib.Common.IPPARTPREFIX) Then
        MarkText = ICenterLib.ICenter.IPPart.GetIPpartInfo(
                       ClsICenter.StripIpIdFromIpNr(MarkText), "Modelname").ToString.ToUpper
    End If
    SetText(MarkText)
End If
```

**When the operator scans an IP-prefixed code**, the form looks up the corresponding `Modelname` from `T_IPpartLines` and engraves that **instead** of the raw scanned text. `#safety-relevant` — the engraved text differs from what the operator typed; the operator must trust the lookup. If the lookup returns wrong (or returns Nothing → engraves "Nothing"-string-ified), the part is mis-marked.

**No null guard** on the `GetIPpartInfo` result — `Nothing.ToString` throws NullReferenceException, but it's caught by `SendData`'s outer Try/Catch (silent). Better outcome would be to refuse-to-mark. Q-332.

## `ClsMarkToolDb` — the message log

- `T_MarkToolLogMessages(SessionId, ClientId, MessageTypeId, MessageText, EmpId, MessageDate)`.
- `T_LogMessageType(MessageTypeId, TypeActive)` — per-type kill-switch. If `TypeActive=0`, that MessageType is silently dropped — admins can throttle log noise.
- `MessageText` is **truncated to 125 chars** in `SetSessionLog` (Q-333 — silent truncation; if the engraved text is longer than 125 chars, only the first 125 are logged).
- `AutoCleanDb(DateToClean)` — deletes rows where `MessageDate < @DateToClean`. Called from `frmMarkTool.RefreshLoggingGrid` with `Today.AddDays(-MessageLogDaysToKeep)`. Default kept days driven by `AppSettings.GetIntegerSetting("MarkToolDaysToKeep")`.
- `GetMessages(ClientId)` filters by `MarkToolDaysToShow` from `T_ApplicationSettings`. Per-client log view.

## Surprises

1. **`SessionId = GetSessionId + 1`** race condition (Q-329).
2. **100-second blocking retry** on busy port (Q-331).
3. **Dead `ReadFromCom`** code (Q-328) — bidirectional comms never implemented.
4. **Hyphen→space replacement** silently rewrites the scanned text (Q-330).
5. **IPpart-prefix substitution** — operator scans `IP-12345`, engraver gets `MODEL-A` (whatever Modelname is associated). Trust-the-lookup hazard (Q-332).
6. **All form text + log entries are Dutch** (`"reeds geopend"`, `"beschikbaar opnieuw geopend"`, etc.).
7. **`frmMarkTool.SetParentForm(MyParentForm)`** — keeps a reference to FrmMain to null itself on close. Couples to the god-form.
8. **`SetSessionLog` silently swallows DB exceptions** — if logging fails, the engrave still proceeds. Operator has no log of a problem mark.
9. **`MessageDate` column has no explicit value** in the INSERT — relies on a DB default (presumably `GETDATE()`). Q-334.

## Business rules surfaced

- **9600 8N1 no-handshake** = Telesis engraver wire protocol.
- **Hyphens are spaces** in engraved text (Q-330).
- **IP-prefixed scans are auto-resolved to Modelname** (Q-332).
- **MessageType 1=data / 2=wait / 3=error / 4=state-change** convention.
- **MessageText silently truncated at 125 chars** (Q-333).

## Open questions

- **Q-327 (new):** Confirm Telesis engraver serial config (9600 8N1) still matches current firmware.
- **Q-328 (new):** `ClsComPort.ReadFromCom` is dead infinite-loop code. Delete or implement.
- **Q-329 (new):** `SessionId = MAX(SessionId) + 1` non-atomic — concurrent sessions can collide.
- **Q-330 (new):** Hyphen→space substitution in engraved text. Telesis encoding constraint or aesthetic?
- **Q-331 (new):** 100-second blocking retry on busy port locks UI thread. `#safety-relevant`
- **Q-332 (new):** IPpart-prefix substitution — no null-guard, no operator confirm-before-engrave. `#safety-relevant`
- **Q-333 (new):** `MessageText` silently truncated at 125 chars; engraved string longer than 125 → log is incomplete.
- **Q-334 (new):** `MessageDate` not in INSERT — relies on DB default.

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/icenter-remaining]] — parent MOC.
- [[icenterlib-icenter-production-machines|`ProductionMachines`]] — `T_ProdMachines.MarkToolComPort` column source.
- [[icenterlib-icenter-batch-hierarchy|`IPPart.GetIPpartInfo("Modelname")`]] — Modelname lookup for IP-prefix scans.
- [[../external-systems/telesis|Telesis engraver]] — destination device.

## Coverage

- `iCENTER\MarkTool\ClsComPort.vb` → `done`
- `iCENTER\MarkTool\ClsMarkToolDb.vb` → `done`
- `iCENTER\MarkTool\frmMarkTool.vb` → `done`
- `iCENTER\MarkTool\frmMarkTool.Designer.vb` → `generated`
