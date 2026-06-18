---
type: architecture
title: "Project & DLL references"
status: draft
module: "iCENTER/(root)"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\iCenter.vbproj"
last-reviewed: ""
tags: [architecture, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Project & DLL references

## What this view shows

Everything `iCenter.exe` compiles against — the in-tree companion projects, the DFS-hosted house DLLs, the NuGet packages, and the GAC / .NET framework refs. Anything that *isn't* in the in-scope `iCENTER\` folder but still ends up in the EXE is here.

## Companion projects (`<ProjectReference>`)

| Project | Project-file path (vbproj) | GUID | Scope |
|---------|----------------------------|------|-------|
| **`ICenterLib`** | `C:\Users\jelle-r\source\repos\JIBA\iCenter And Tools\ICenterLib\ICenterLib\ICenterLib.vbproj` ⚠️ | `9f3078de-2386-4ff8-b66d-a6df9e100953` | **in scope** (added 2026-06-18) |
| **`TruTopsLib`** | `C:\DevOps\iCenter\iCenter\TruTopsLib\TruTopsLib.vbproj` | `afd6ec7d-cfd8-4184-a029-13e2503bb0e4` | **in scope** (added 2026-06-18) |

> ⚠️ The reference inside `iCenter.vbproj` line 3335 (`..\..\ICenterLib\ICenterLib\ICenterLib.vbproj`) and `iCENTER.sln` line 11 (`..\ICenterLib\ICenterLib\ICenterLib.vbproj`) both resolve to `C:\DevOps\iCenter\ICenterLib\ICenterLib\` — a path that does **not** exist on this machine. The actual source lives under the user's personal `source\repos\` tree. The build relies on this directory being present on each developer's box; CI behavior unclear. `#needs-review` (tracked as **Q-018**).

Both projects are now part of this wiki's scope per the 2026-06-18 [[../../CLAUDE|standing-instructions]] update. Their types are everywhere in the iCENTER source. The `Imports` block in `iCenter.vbproj` (lines 277–289) makes them ambiently available:

```xml
<Import Include="ICenterLib" />
<Import Include="ICenterLib.UserControls" />
```

A randomly chosen file (`FrmMain.vb`, lines 18–24) imports:

```
ICenterLib.DataHandler
ICenterLib.ICenter
ICenterLib.ISAH
ICenterLib.JMail
ICenterLib.SmtProduction
ICenterLib.SmtProduction.Entities
ICenterLib.SmtProduction.UI
```

So `ClsICenter`, `ClsISAH`, `JMail`, the Oseon SMT integration, the central `Common` helpers, the PLM bridge, the application-culture singleton, and most of the data-access layer all live in `ICenterLib`. Module notes targeting iCENTER types should link to corresponding `modules/icenterlib-*` notes once those are written; Phase-3 prioritisation will likely traverse iCENTER and ICenterLib together when they're closely coupled (e.g. anything in `iCENTER\Elumatec\` that calls into `ICenterLib.CAD.*`).

> **Q-001 — resolved 2026-06-18.** Decision: both companion projects are in scope. See [[../needs-review/_index]] Q-001 row.

## House DLLs from the DFS share

References by `<HintPath>` to `\\jazo.local\dfs\Applications\Development\Dot Net DLLs\DOT.NET 4\RELEASE\<Name>\<Name> <Version>\<Name>.dll`. Reproduced from `iCenter.vbproj` lines 129–275:

| DLL | Version | Purpose (inferred — confirm) |
|-----|---------|------------------------------|
| `AxInterop.PDFXCviewAxLib` | 1.0.0.0 | ActiveX wrapper for PDF-XChange Viewer (UI) |
| `Interop.PDFXCviewAxLib` | 1.0.0.0 | COM interop for PDF-XChange |
| `DataGridViewAutoFilter` | 1.0.0.0 | MS sample auto-filter for DataGridView |
| `ICSharpCode.SharpZipLib` | 0.86.0.518 | Zip / tar archive support |
| `ILCalc` | 0.9.7.5 | Runtime arithmetic expression evaluator |
| `Interop.IWshRuntimeLibrary` | 1.0.0.0 | Windows Script Host — shortcut (`.lnk`) creation |
| `Interop.SHDocVw` | 1.1.0.0 | Internet Explorer ShellDocObject (web hosting / shell folder browsing) |
| `itextsharp` | 4.1.6.0 | PDF generation / manipulation |
| `WW`, `WW.Cad`, `WW.GL`, `WW.License` | 3.5.35.96 / 3.5.0.0 | CadLib (Würste Werner CadLib) — DWG/DXF read/write, OpenGL render, licensing |

→ Build requires the DFS share to be reachable. → If any of these DLLs is updated on the share, **every iCenter build picks up the change without a NuGet manifest bump.** `#needs-review` whether that's the intended deployment model.

## NuGet packages

See `packages.config`. The notable additions:

- **`Microsoft.Extensions.*` 9.0.4** — modern .NET Core-style DI/Logging/Http. Likely consumed by `ICenterLib`'s newer code paths (the iCenter project itself appears to still use globals — see [[global-state]]).
- **`Microsoft.Web.WebView2` 1.0.3124.44** — Chromium WebView2 control. Indicates embedded browser views inside `FrmMain` / controls — used for ASPX-based dashboards like the [[../external-systems/kardex|Kardex interface]] and [[../external-systems/jiba-portal|JIBA portal]].
- **`DocumentFormat.OpenXml` 3.3.0** — XLSX/DOCX generation (likely sales-document workflows).
- **`HtmlAgilityPack` 1.12.0 + ExCSS 4.2.3** — HTML/CSS parsing (email templates? web-scraping?). `#needs-review`.

## Framework GAC refs (selected — full list in `iCenter.vbproj`)

`System.Windows.Forms`, `System.Windows.Forms.DataVisualization` (Charts), `PresentationCore`/`WindowsBase`/`System.Printing` (WPF print pipeline used from Forms), `System.DirectoryServices` + `.AccountManagement` (AD lookups), `System.EnterpriseServices`, `System.ServiceModel` (WCF), `System.Web.Services` (ASMX clients — `jconfigurator.asmx`), `System.Management` (WMI).

`System.Xml` and `System.Xml.Linq` use **explicit `HintPath`s** to `C:\Program Files (x86)\Reference Assemblies\Microsoft\Framework\.NETFramework\v4.6.2\` (lines 254–259) — this is unusual; default would be GAC. `#needs-review` whether this is leftover from an earlier target framework or intentional.

## Open questions

- **Q-018:** the `ICenterLib` project-reference path (`..\..\ICenterLib\ICenterLib\...`) resolves to a non-existent directory; the real source is under the user's personal repo tree. Where is the canonical / build-server copy? `#needs-review`
- **Q-013:** house DLLs are loaded from a live DFS share with no version-pinning beyond the path. Is the DFS folder treated as immutable per-version (the path includes the version), or do operators update in place? `#needs-review`
- **Q-014:** `System.Xml.dll` referenced from the v4.6.2 reference-assemblies folder while `TargetFrameworkVersion=v4.8`. Drift? Confirm.

## Related

- [[_index]]
- [[build-and-deploy]]
- [[../external-systems/_index|external-systems/]]
