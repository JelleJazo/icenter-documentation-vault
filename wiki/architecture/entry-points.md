---
type: architecture
title: "Entry points — Sub Main and mode dispatch"
status: draft
module: "iCENTER/Modules"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Modules\\Main.vb"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\iCenter.vbproj"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\ApplicationEvents.vb"
last-reviewed: ""
tags: [architecture, entry-point]
created: 2026-06-18
updated: 2026-06-18
---

# Entry points

## What this view shows

How a launched `iCenter.exe` process picks one of three top-level run modes and which UI form it puts on the screen.

## The configured entry point

`iCenter.vbproj` declares:

- `<OutputType>WinExe</OutputType>` (no console window by default)
- `<MyType>WindowsFormsWithCustomSubMain</MyType>` — opts out of the auto-generated `My.Application` startup form and routes through a hand-written `Sub Main`.
- `<StartupObject>iCenter.Main</StartupObject>` — that hand-written `Sub Main` lives in the `iCenter` namespace, `Main` module.

The corresponding file is `Modules\Main.vb` → `Public Module Main` → `Sub Main()` at **line 198**.

`ApplicationEvents.vb` is the stock VB.NET `My.MyApplication` partial. The class body is empty (lines 8–9). All handlers (`Startup`, `Shutdown`, `UnhandledException`) are unwired, so there is **no** application-level exception handler — the only catch-all is the `Try…Catch` wrapped around the body of `Sub Main` itself.

## Dispatch tree

`Sub Main` reads command-line arguments via `Functions.GetApplicationArguments` (declared in `ICenterLib`), keys on `-m <mode>`, and branches:

| `-m` value | What launches | Form / call |
|------------|---------------|-------------|
| `cadbatchserver` | Headless-ish CAD batch worker | `New CadBatchserver.FrmCadBatchServer().ShowDialog()` |
| `updateicenter` | Self-updater | `Functions.UpdateIcenter()` |
| *(default)* | Interactive UI | `New FrmMain().ShowDialog()` |

After the interactive `FrmMain` closes, two **post-main** branches may fire — gated on globals that `FrmMain` itself flips before disposing:

- `BatchServerMode = True` → opens `Batchserver.FrmBatchServer` (note: distinct from `CadBatchserver`).
- `ComWatcherMode = True` → opens `Elumatec.FrmComWatcher`.

See [[runtime-modes]] for what each mode actually does.

## Startup side effects

Before the dispatch, `Sub Main` calls `Init()` (lines 149–169 of `Modules\Main.vb`), which:

1. Warns via `MsgBox` if `sXML_Root` ≠ the default in `ICenterLib.AppSettings.XML_ROOT_SERVER`.
2. Probes XML write access (`Functions.GetXMLWriteAccess` → sets `UserHasXMLWriteAccess`).
3. Loads two `DataTable`s from the ICenter DB via `oICENTER`:
   - `dtCadGenerics = oICENTER.GetGenericNames`
   - `DtGenericMachGrp = oICENTER.GetAllGenericMachGrpLines()`
4. `RefreshIsahSortOrder()` — pulls sort order from ISAH.
5. Picks the Creo CAD version (`ICenterLib.CAD.Creo.Environment.DefaultCadAppVersion`).
6. Constructs Oseon data services (`OseonAppContextDataService`, fetches the default `OseonAppContext`, wires the per-context Part/Doc/Status data services in `SetOseonDataServices`).
7. Constructs ISAH-customising / Elfsquad service + Product Configuration data service.
8. `Functions.InitProfMillMachGrps()` — populates the global `ProfMillMachGrps` list (Elumatec profile-mill machine groups).

If `UseCustomTempFolder` is `True` (it is — `Private Const = True`, line 134), the interactive path also creates a per-PID temp folder `%TEMP%\iCenter_PID<n>\` and rewrites the `TMP`/`TEMP` env vars for the process before showing `FrmMain`. The folder is deleted on clean exit (line 255).

## Key decisions / surprises

- **No `My.Application` lifecycle.** Because of `WindowsFormsWithCustomSubMain`, the standard VB.NET unhandled-exception hook is not wired. A crash outside `Sub Main`'s outer `Try` is silent at the application level — only the inner `Try` around `MyMainForm.ShowDialog()` catches and logs via `oApplicationLog.NewEntry(...)`.
- **Mode switching is positional in code, not data-driven.** Adding a new mode means editing `Sub Main`.
- **Per-PID temp folder** rewrites process env vars — any child process spawned from iCenter inherits this redirected `TEMP`. Worth knowing when debugging interop with Creo / SolidEdge / Ghostscript spawned tools.
- **Two distinct "batch server" forms** exist: `CadBatchserver.FrmCadBatchServer` (CAD jobs, command-line `-m cadbatchserver`) vs. `Batchserver.FrmBatchServer` (generic, post-FrmMain). Easy to confuse.

## Open questions (added to [[../needs-review/_index]])

- **Q-002:** Who launches `-m cadbatchserver`? A scheduled task? A service wrapper? The user `JZPUBLISH`? (See the commented-out `If GetUsername.ToUpper = "JZPUBLISH"` block at lines 226–233 of `Modules\Main.vb`.)
- **Q-003:** What conditions inside `FrmMain` flip `BatchServerMode` / `ComWatcherMode` to `True`? Search needed.
- **Q-004:** Is `Functions.UpdateIcenter()` (the `-m updateicenter` path) still in use given the ClickOnce signing was disabled in 2023-07?

## Related

- [[_index|Architecture MOC]]
- [[runtime-modes]]
- [[global-state]]
- [[../modules/_index|modules/]]
