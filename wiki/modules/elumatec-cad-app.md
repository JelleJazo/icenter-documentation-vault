---
type: module
title: "EluCadApp — Elumatec central controller"
status: draft
module: "iCENTER/Elumatec"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\iCENTER\\Elumatec\\EluCadApp.vb"
last-reviewed: 2026-06-18
tags: [module, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# `Elumatec\EluCadApp.vb` — central controller

> **Status: overview note only.** EluCadApp is 128 KB / ~3 200 lines. This page describes the *shape* of the class and the constants it owns; per-cluster behaviour notes will be written when their Phase-3 batches land.

## Purpose

Glue class that mediates between the Elumatec forms (`FrmProfileView`, `CtrlProfMillElu`, `CtrlProfMillCam`, `FrmEluMissingProfile`, …), the [[elumatec-machine-base|`Sbz14x` machine classes]], the **`.epd` profile database** parsed from the shared share, and the **DGX** table that represents the cuts to make.

## Class state

| Field | Type | Role |
|-------|------|------|
| `ProfileDB` | `DataTable` | parsed profile catalogue (one row per profile; columns: `MName`, `MComment`, `Vendor`, `Series`, `Color`, `Surface`, `Width`, `Height`, `Length`, `DgxOrient`, `MTime`, `IsCombined`, `Image`, `ImageDgx`). Primary key: `MName`. |
| `dtDgx` | `DataTable("DGX")` | cuts to make. Columns: `LineNr`, `BLength`, `CLength`, `Qty`, `QtySoll`, four corner angles (`CAngle*H/V`), `BIdentNo`, `Info`, `Box`, `Jobnr`, `Pos`, `Color`, `Barcode`, `BLineNr`. Primary key: `LineNr`. |
| `AppVersion` | `AppVersion` | resolved Elumatec EluCad install version; controls post-processor paths and settings layout. |

## Constants (the file is a constants museum)

| Const | Value | Meaning |
|-------|-------|---------|
| `MAXLENGTHINFO` | 14 | max chars for `Info` field on a job line |
| `MAXLENGTHJOB` | 10 | max chars for `Jobnr` field |
| `MAXLENGTHCOLOR` | 10 | max chars for `Color` field |
| `INF01_TEXT` | `"Ander profiel"` | sentinel "different profile" tag (max 14 chars) |
| `INF02_TEXT` | `"*Let op!*"` | sentinel "watch out!" tag (max 10 chars) |
| `UnknownBIdentNo` | `"100999"` | profile ID used when the actual one can't be resolved |
| `ProfileTimeStampFormat` | `"yyyy-MM-dd HH:mm:ss"` | timestamp format inside `.epd` |
| `DgxOnlineExportFilename` | `"dgxdata.dgx"` | filename written to drop the live DGX online |
| `DgxOnlineActiveFilename` | `"dgxdata.elu"` | filename when the DGX is "active" / consumed |
| `ManualProgFolder` | `\\jazo.local\dfs\pm\Elumatec_SBZ140\Aanpassen\` | shared "manual adjustments" drop folder |
| `ViewerProgFolder` | `c:\temp\` | local-machine viewer scratch folder |
| `EluSoftRegistryKey` | `HKEY_CURRENT_USER\Software\EluSoft GmbH` | EluSoft (Elumatec) registry root for per-user settings |
| `EluSoftSettingsRegistryKey` | `HKCU\Software\EluSoft GmbH\EluCad\Settings` | EluSoft settings subkey iCenter sometimes overwrites |
| `DefaultCellStyleFormat` | `"0.00"` | numeric display |
| `Precision` | `2` | rounding precision |
| `usfCultureInfo` | `en-US` with empty group separator | forced number culture used when writing NC files (decimal point, no thousands sep) |

## Public enums

```vb
Public Enum ProfileProperty
    BIdentNo, Description, BWidth, BHeight, BPartCode,
    BLength, BSeries, BColor, BSurface, MTime, DgxOrient
End Enum

Public Enum NcType
    Auf = 1     ' classic AUF (XML-ish) format — used by SBZ140 ALU/STL/RVS, SBZ141 ALU
    EluXml = 2  ' newer .eluxml format
End Enum
```

`NcType` is selected per machine variant (see [[elumatec-machine-base]]) and gates which `NcxExePath` arguments and which post-processor paths `Sbz14x.CreateNCX` uses. Adding a third variant means touching `Sbz14x.CreateNCX`, `Sbz14x.NCFileExtension`, and the per-machine override.

## How the profile DB is loaded

`PopulateProfileDB(Optional IncludeImages As Boolean = False)` (line 107):

1. Open `ProfileDbFile = My.Settings.Properties("profileDB").DefaultValue` for read with `FileShare.ReadWrite` (so it doesn't block other readers/writers of the live `.epd`).
2. Stream the file line-by-line. Records are bracketed by lines containing `:PROFILE`.
3. Within a `:PROFILE` block, every line has a `key=value` shape; pick the keys `MName`, `MComment`, `Vendor`, `Series`, `Color`, `Surface`, `Dimension` (parsed into Width/Height/Length), `DgxOrient`, etc.
4. Optionally load thumbnail bitmaps from `My.Settings.Properties("profileImageDir").DefaultValue + "64x64\"` (and the DGX-variant from `profileImageDirDgx`).

The default `profileDB` setting points at `\\jazo.local\dfs\PM\Elumatec_SBZ140\Directories\database\JZProfiles.epd`. Read failures result in an empty `ProfileDB` (the `Exit Sub` at line 111 returns silently).

## Behavior areas (Phase-3 sub-notes to come)

Roughly identifiable from the file's section structure — to be confirmed once each cluster is read:

- **Profile DB queries** — `GetProfileProperty`, `GetProfileImage`, lookups by `MName` / `BIdentNo`.
- **DGX management** — building, exporting, and importing the DGX table; the `dgxdata.dgx` / `dgxdata.elu` handshake with the live Elumatec system.
- **NCW (CAM file) building** — wraps `Sbz14x.CreateNCX` and the post-processor pipeline.
- **Registry sync** — overwrite Elumatec's per-user `HKCU\Software\EluSoft GmbH\EluCad\Settings` entries from `SourceRegFile` so all clients share the same EluCad configuration. Gated by `AppSettings.GetStringSetting("OverwriteClientEluCadRegistry") = "1"` (see `FrmMain.OverwriteEluCadRegistry`, line 185–189 of `FrmMain.vb`).
- **DGX online file roundtrip** — drop `dgxdata.dgx`, wait for it to become `dgxdata.elu` (presumably renamed by the saw's PC after consumption), then update iCenter.

## Surprises

- **DB schema is hardcoded inline** in the constructor. Adding a profile-DB column requires editing both `PopulateProfileDB` *and* the `dtDgx` / `ProfileDB` column declarations.
- **`en-US` culture is forced** with `NumberGroupSeparator = ""`. Files written from iCenter will use `.` decimals and no thousands separators — required by the Elumatec post-processor, but a foot-gun if the rest of iCenter writes the same DataTables under the Dutch culture.
- **Unknown profile sentinel `100999`** appears in the DGX rather than `NULL`. SME-confirm that `100999` is reserved and not a real `BIdentNo`. `#needs-review`
- The `ManualProgFolder` UNC and the `ViewerProgFolder` local path are `Public Const` — burning them into the binary. Moving the shared folder means re-deploy.

## Business rules surfaced here

Watchlist for Phase 4:

- The "Ander profiel" / "*Let op!*" `Info` sentinels (line 26–27) are user-visible warning markers. Any code that emits them is implementing a rule.
- `UnknownBIdentNo = "100999"` is a reserved profile ID. Any rule conditioned on it is a business rule.
- The `OverwriteClientEluCadRegistry` toggle silently replaces the user's EluCad settings on machine startup. Treat any change there as `#safety-relevant`.

## External systems touched

- [[../external-systems/dfs-share|`\\jazo.local\dfs\pm\Elumatec_SBZ140\...`]] — profile DB (`.epd`), tool DB, fixtures, offsets, manual-program drop folder.
- [[../external-systems/elumatec-sbz140|Elumatec SBZ140 / SBZ141 machines]] — indirectly via NC programs and DGX file handshake.
- Per-user Windows registry under `HKCU\Software\EluSoft GmbH\EluCad\Settings` — read/write.

## Open questions

- **Q-024 (new):** Confirm `UnknownBIdentNo = "100999"` is a reserved sentinel (no real profile with that ID). `#needs-review`
- **Q-025 (new):** The `dgxdata.dgx` → `dgxdata.elu` rename handshake is the only synchronisation between iCenter and the saw's PC for live work. What happens on rename failure? Is there a timeout? `#safety-relevant`
- **Q-026 (new):** The `OverwriteClientEluCadRegistry` switch silently overwrites the EluCad settings registry from `SourceRegFile`. When is this safe to run? What if EluCad is mid-edit?

## Coverage

`_coverage.md`:

- `Elumatec\EluCadApp.vb` → `needs-review` (overview-only; sub-notes per behaviour cluster pending Phase-3 batches)
