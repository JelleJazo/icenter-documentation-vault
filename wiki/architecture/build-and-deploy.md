---
type: architecture
title: "Build and deploy"
status: draft
module: "iCENTER/(root)"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\iCenter.vbproj"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\packages.config"
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\app.config"
last-reviewed: ""
tags: [architecture, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# Build and deploy

## What this view shows

How `iCenter.vbproj` compiles, how artifacts are signed and published, and the deployment endpoint operators are expected to consume.

## Compile-time facts

| Setting | Value | Source |
|---------|-------|--------|
| Output type | `WinExe` | `iCenter.vbproj` line 10 |
| Target framework | `.NET Framework 4.8` | line 16 |
| Platform | `x86` (both Debug and Release) | lines 5, 47, 60 |
| Assembly name | `iCenter` | line 13 |
| Root namespace | `iCenter` | line 12 |
| Application icon | `Resources\iCenter.ico` | line 86 |
| Option Strict | **Off** | line 80 — late binding allowed; whole codebase is loose-typed |
| Option Explicit | On | line 74 |
| Option Compare | Binary | line 77 |
| Option Infer | On | line 83 |
| Warnings suppressed | `42016,41999,42017,42018,42019,42032,42036,42020,42021,42022` | line 55 — implicit-conversion family |

`Option Strict Off` is the single biggest readability hazard in this codebase: every `Object`-typed expression is implicitly cast at runtime. Phase 3 module docs should flag specific cases where this hides intent.

## Assembly version

`iCenter.vbproj`:

- `<ApplicationVersion>1.6.1.%2a</ApplicationVersion>` (`%2a` = `*`, auto-increment build/revision) — line 41
- `<ApplicationRevision>97</ApplicationRevision>` — line 40

Current published version is therefore `1.6.1.97`.

## Signing

- **Assembly** is signed with the strong-name key `JAZO Zevenaar bv.snk` (`<SignAssembly>true</SignAssembly>` line 105–108).
- **Manifest** signing via `<ManifestCertificateThumbprint>F62F2D18F9E94ED70D96DE453669920D3218DCAF</ManifestCertificateThumbprint>` (line 89) — but **`<SignManifests>false</SignManifests>`** (line 99) so the manifest is *not* in fact signed.
- An `AfterCompile` Authenticode-signing target using `signtool.exe` from `\\jazo.local\dfs\applications\Development\Tools\cmd\signtool.exe` is **commented out** with the note _"CB 2023-07-18: disabled signing because certificate is expired"_ (`iCenter.vbproj` lines 3382–3391).
- A follow-up `mage.exe`-based manifest update / re-sign block is similarly commented out.

→ **Operational consequence:** since 2023-07-18, builds ship unsigned at the manifest level. This is `#needs-review` for compliance/release-engineering.

## Publishing (ClickOnce)

| Setting | Value |
|---------|-------|
| `PublishUrl` | `Z:\iCenter\BetaRelease\` (line 26) |
| `PublisherName` | `JAZO Zevenaar bv` (line 37) |
| `ProductName` | `iCenter` (line 36) |
| `Install` | `false` (line 27) — install-from-network mode |
| `UpdateEnabled` | `false` (line 29) |
| `BootstrapperEnabled` | `false` (line 45) |
| `ApplicationManifest` | `My Project\app.manifest` (line 102) |
| `ManifestTimestampUrl` | `http://timestamp.sectigo.com` (line 111) |

`Z:\` is a mapped network drive. Together with `UpdateEnabled=false` this suggests **out-of-band update** — likely the `-m updateicenter` self-update path ([[runtime-modes]] / `Functions.UpdateIcenter()`) rather than ClickOnce update. **#needs-review** to confirm.

## NuGet packages (`packages.config`)

Modern packages restored alongside framework-2010s libraries:

- `DocumentFormat.OpenXml` 3.3.0 — Word/Excel output
- `HtmlAgilityPack` 1.12.0, `ExCSS` 4.2.3 — HTML/CSS parsing
- `Microsoft.Web.WebView2` 1.0.3124.44 — embedded Chromium control
- `Microsoft.Extensions.*` (DI, Http, Logging, Options) 9.0.4 — modern DI stack pulled in (likely used through `ICenterLib`)

## Non-NuGet DLL references

References by `<HintPath>` to `\\jazo.local\dfs\Applications\Development\Dot Net DLLs\DOT.NET 4\RELEASE\...` — see [[project-references]] for the full list. Build is **dependent on the DFS share being mounted**; offline builds fail.

## Project references

| Reference | Path |
|-----------|------|
| `ICenterLib` | `..\..\ICenterLib\ICenterLib\ICenterLib.vbproj` → `C:\DevOps\ICenterLib\ICenterLib\ICenterLib.vbproj` |
| `TruTopsLib` | `..\TruTopsLib\TruTopsLib.vbproj` → `C:\DevOps\iCenter\iCenter\TruTopsLib\TruTopsLib.vbproj` |

These projects are **outside this wiki's scope** per [[../../CLAUDE]] but compile into `iCenter.exe`. See [[project-references]] for the implications.

## Key decisions / surprises

- `x86` lock-in means iCenter cannot use 64-bit-only third-party libraries. Likely chosen because of Creo / Elumatec ActiveX bindings.
- `loadFromRemoteSources enabled="true"` in `app.config` line 42 — required because referenced DLLs are loaded from the DFS UNC path. Lowers `CAS` protection.
- The `app.config` includes a *commented-out* `assemblyBinding` pointing `nestBridge.dll` at `\\jazo.local\dfs\applications\iCenter\Resources\NestProfessor\nestBridge.dll` with the inline note _"dit werkt niet, folder moet subfolder zijn van de applicatie"_ (lines 35–41). NestProfessor integration is therefore **partial** — `#needs-review` whether the feature is dead or worked around elsewhere.

## Open questions

- **Q-010:** Has Authenticode signing been re-enabled since 2023-07-18 in any branch / scripted pipeline outside `iCenter.vbproj`? `#needs-review`
- **Q-011:** Is `Z:\iCenter\BetaRelease\` the production publish target or only "Beta"? If only Beta, where does the production release go?
- **Q-012:** Confirm `Functions.UpdateIcenter()` is the live update mechanism and document its source/destination paths and signing model.

## Related

- [[_index]]
- [[project-references]]
- [[entry-points]]
