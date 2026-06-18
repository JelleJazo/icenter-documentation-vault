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

## In-tree companion projects (`<ProjectReference>`)

| Project | Project-file path | GUID |
|---------|-------------------|------|
| **`ICenterLib`** | `C:\DevOps\ICenterLib\ICenterLib\ICenterLib.vbproj` | `9f3078de-2386-4ff8-b66d-a6df9e100953` |
| **`TruTopsLib`** | `C:\DevOps\iCenter\iCenter\TruTopsLib\TruTopsLib.vbproj` | `afd6ec7d-cfd8-4184-a029-13e2503bb0e4` |

**These projects are out of scope** per [[../../CLAUDE]] but their types are everywhere in the iCENTER source. The `Imports` block in `iCenter.vbproj` (lines 277–289) makes them ambiently available:

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

So `ClsICenter`, `ClsISAH`, `JMail`, the Oseon SMT integration, the central `Common` helpers, the PLM bridge, the application-culture singleton, and most of the data-access layer all live in `ICenterLib`, not here. Phase-3 module notes will need to **say "see ICenterLib" rather than try to document the symbol** — but each such pointer should be **explicit**, so the wiki stays honest about how much logic actually lives outside its boundary.

> **Q-001 (Tracked in [[../needs-review/_index]]):** confirm scope decision with SME. Options: (a) document `ICenterLib` and `TruTopsLib` here too; (b) keep them out of scope but maintain an inventory page that lists every `ICenterLib.X` symbol referenced from iCENTER and the file/line; (c) defer to a sibling wiki.

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

- **Q-001 (open):** in-or-out for `ICenterLib` and `TruTopsLib`.
- **Q-013:** house DLLs are loaded from a live DFS share with no version-pinning beyond the path. Is the DFS folder treated as immutable per-version (the path includes the version), or do operators update in place? `#needs-review`
- **Q-014:** `System.Xml.dll` referenced from the v4.6.2 reference-assemblies folder while `TargetFrameworkVersion=v4.8`. Drift? Confirm.

## Related

- [[_index]]
- [[build-and-deploy]]
- [[../external-systems/_index|external-systems/]]
