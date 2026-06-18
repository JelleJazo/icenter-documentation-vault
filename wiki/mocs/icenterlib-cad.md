---
type: moc
title: "ICenterLib/CAD — Creo / Windchill / Geometry / OpenGL"
status: draft
module: "ICenterLib/CAD"
source-paths:
  - "C:\\Users\\jelle-r\\source\\repos\\JIBA\\iCenter And Tools\\ICenterLib\\ICenterLib\\CAD\\"
tags: [moc, safety-relevant, needs-review]
created: 2026-06-18
updated: 2026-06-18
---

# ICenterLib/CAD

## What this hub covers

`ICenterLib\CAD\` — the **CAD-pipeline shared library**. 116 source files (~577 KB) across half a dozen sub-folders. Touches Creo (PTC's parametric-CAD app), Windchill (PTC's PLM), OpenGL preview, geometry math, Modelgenerator (parametric model creation), DXF/STEP/STL conversion, publisher utilities.

> **The most CAD-heavy folder in ICenterLib.** Anything that automates Creo, queries Windchill, generates a CAD-input file, or transforms geometry probably lives here.

## Sub-folder breakdown

```
CAD\
├── Creo\                    ' Creo automation (Trailfile, ProProgram, Model info, Features, Params)
├── Geometry\                ' geometry math (Earcut triangulation, DXF profile mill, etc.)
├── Modelgenerator\          ' parametric-model generation (separate from CadBatchServer's Modelgenerator)
├── OpenGL\                  ' OpenGL viewer (CtrlOpenGLViewer)
├── PLM\                     ' Windchill PLM integration (EPMDocument, etc.)
├── Publisher\               ' publish-job utilities
├── ProProgram\              ' Pro/Program (Creo's parametric-config language)
└── … (root CAD files)
```

## Top files by size

| File | KB | Role |
|------|---:|------|
| `Geometry\Earcut_CSharp.vb` | 36 | Earcut (Mapbox) polygon triangulator port |
| `Creo\ProProgram\VBCodeConverter.vb` | 34 | converts Pro/Program-language constructs to VB equivalents |
| `Geometry\Earcut.vb` | 30 | second Earcut implementation |
| `OpenGL\CtrlOpenGLViewer.vb` | 24 | OpenGL preview UserControl |
| `Creo\Trailfile.vb` | 20 | Creo trail-file (.txt scripting) generator |
| `Modelgenerator\Configuration.vb` | 20 | model-generator configuration |
| `PLM\EPMDocument.vb` | 18 | EPM document (Windchill) entity |
| `Creo\ModelInformation.vb` | 18 | Creo model-info reader |
| `Geometry\Dxf3DProfileMill.vb` | 15 | 3D-DXF generator for profile-mill geometry — **`#safety-relevant`** |
| `Creo\ProProgram\Functions.vb` | 13 | Pro/Program function library |
| `Creo\ProProgram\Design.vb` | 13 | Pro/Program design rules |
| `Creo\Feature.vb` | 12 | Creo feature entity |
| `Publisher\Common.vb` | 12 | publisher helpers |
| `Creo\ParameterCollection.vb` | 12 | Creo parameter-collection |
| `Creo\AppVersion.vb` | 9 | Creo app-version detection |

## Notable findings (at a glance)

1. **`Geometry\Earcut.vb` and `Earcut_CSharp.vb`** — two Earcut implementations, totalling ~66 KB. Likely a hand-port and a C#-port living side by side (Q-295 — deduplicate?).
2. **`Geometry\Dxf3DProfileMill.vb`** — geometry generator for Elumatec profile-milling. Output feeds `iCENTER\Elumatec\` ([[elumatec-ncpipeline]]). `#safety-relevant`.
3. **`Creo\ProProgram\VBCodeConverter.vb`** — converts Pro/Program (Creo's config language) to VB. Single biggest file outside Earcut. Q-296 — what's the conversion purpose?
4. **`Creo\Trailfile.vb`** — generates Creo trail files (scripted automation). Direct Creo-driving code.
5. **`OpenGL\CtrlOpenGLViewer.vb`** — embedded OpenGL viewer (24 KB). Phase-3 follow-up for embedded native-OpenGL usage.
6. **`Modelgenerator\`** subfolder exists in BOTH `ICenterLib\CAD\` AND `ICenterLib\CadBatchServer\Modelgenerator\`. Two distinct subsystems with the same name. Q-297.
7. **PLM\EPMDocument.vb** — Windchill EPMDocument wrapper. Phase-3 follow-up.

## Open questions

- **Q-295 (new):** `Geometry\Earcut.vb` + `Earcut_CSharp.vb` — deduplicate?
- **Q-296 (new):** `Creo\ProProgram\VBCodeConverter.vb` — what does it convert? Pro/Program → VB or reverse?
- **Q-297 (new):** Two Modelgenerator subsystems exist (one in CAD\, one in CadBatchServer\). Document both purposes.
- **Q-298 (new):** Confirm `Dxf3DProfileMill.vb` is the source of the DXF files consumed by Elumatec's profile-mill pipeline. `#safety-relevant`

Logged in [[../needs-review/_index]].

## Coverage decision

**Folder is large + business-critical. Phase 3e marks all 116 source files as `done` overview-level via this MOC**, but flags `Dxf3DProfileMill`, `Trailfile`, `EPMDocument`, `ProProgramFunctions`, `Modelgenerator\Configuration`, and `Publisher\Common` for **Phase 4 deep-read** (business-logic sweep).
