---
type: moc
title: "ICenterLib/MySystem — system utilities"
status: draft
module: "ICenterLib/MySystem"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\MySystem\\"
tags: [moc, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# ICenterLib/MySystem

## What this hub covers

`ICenterLib\MySystem\` — **system-level utilities** that wrap Windows / .NET APIs the application uses across modules.

20 source files (~66 KB) + 1 generated.

## Files

| File | Role |
|------|------|
| `Network.vb` | UNC/mapping conversion (`GetUNCfromMapping`), interface enumeration |
| `Network\TcpServer.vb` | minimal TCP server |
| `Printer.vb` | printer enumeration / queue mgmt |
| `Registry.vb` | Windows registry read/write |
| `FileSystem.vb` | file-system helpers |
| `Computer.vb` | computer info |
| `Environment.vb` | env-var helpers |
| `Math.vb` | numeric helpers |
| `MyProcess.vb` | Process.Start wrapper / process discovery |
| `Window.vb` | Window enumeration |
| `OpenWindowGetter.vb` | gets currently-open windows |
| `WindowsUser.vb` | current Windows user |
| `TerminalServerSessions.vb` | terminal-server session enumeration |
| `HelpHandler.vb` | F1 help handler |
| `HealthMonitorClient.vb` | health-monitor reporting client |
| `PowerShellWrapper.vb` | PowerShell execution |
| `ICenterObjectNotFoundException.vb` | typed exception |
| `ExceptionList.vb` | aggregate-exception bag |
| `Encryption\FrmEncrypt.vb` + `.Designer.vb` | encryption-test UI |
| `Encryption\SecurityController.vb` | encryption logic |

## Notable findings

1. **`Encryption\SecurityController.vb`** + **`FrmEncrypt.vb`** are separate from `ISAH.Helpers.EncryptionHelper` (the XOR cipher — Q-216). Phase-3 follow-up to determine which is actually used for what. `#safety-relevant`.
2. **`Network.GetUNCfromMapping`** — called by [[icenterlib-icenter-leaves|`XmlFile.AddDocument`]] when storing document paths. Resolves drive-letter-mapped paths to UNC for cross-host portability.
3. **`PowerShellWrapper`** — generic PowerShell invocation. Care needed — if any caller passes user input, command injection risk. Q-287.
4. **`ExceptionList`** — consumed by [[icenterlib-icenter-batch-hierarchy|`IPOrder.DeleteSmtProdOrd`]] to aggregate per-batch failures.
5. **`HealthMonitorClient`** — reports to a JAZO-internal health-monitor endpoint. Phase-3 follow-up.
6. **`TerminalServerSessions`** suggests iCenter supports running under terminal services / Remote Desktop — multi-session concerns relevant to file-locking, per-machine identification.

## Open questions

- **Q-287 (new):** `PowerShellWrapper` — find consumers; if any pass user-supplied strings, command-injection risk.
- **Q-288 (new):** `SecurityController` vs `EncryptionHelper` — two separate encryption families. Which is used where?

Logged in [[../needs-review/_index]].

## Coverage

All 20 source files marked `done` overview-level. Designer marked `generated`.
