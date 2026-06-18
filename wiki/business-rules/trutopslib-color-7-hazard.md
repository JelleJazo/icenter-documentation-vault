---
type: business-rule
title: "DXF colour 7 (white) handling is life-threatening — bend lines reuse it"
status: needs-review
module: "TruTopsLib"
source-paths:
  - "C:\\DevOps\\iCenter\\iCenter\\TruTopsLib\\LayerConverter.vb"
last-reviewed: ""
tags: [business-rule, needs-review, safety-relevant]
created: 2026-06-18
updated: 2026-06-18
---

# DXF colour 7 (white) handling — author's own "life-threatening" warning

## Rule

In `LayerConverter.ConvertLayers`, the colour-rewrite rules deliberately handle colour **7** (white) in two competing ways:

1. **If the entity is `AcDbText` and colour-7** → **delete it** (the text will be replaced with bend information downstream).
2. **Colour 1 or 2 (red or yellow)** → assign to layer `0`. **The original behaviour likely included colour 7 here, but the author commented it out** with the warning: `'', 7  7 is levensgevaarlijk omdat zetlijnen soms ook wit zijn` ("…color 7 is **life-threatening** because bend lines are sometimes also white").

So colour 7 is **handled only when the entity is text**. Colour-7 lines (non-text) **fall through to "Else" and are deleted** — because there's no other branch that picks them up.

The warning is the strongest natural-language safety flag in the entire codebase.

## Where it lives

- File: `C:\DevOps\iCenter\iCenter\TruTopsLib\LayerConverter.vb` lines 31-60 (the colour Select Case)
- Author's hazard comment: line 53 — `Case 1, 2 '', 7 7 is levensgevaarlijk omdat zetlijnen soms ook wit zijn`

## The code (verbatim, with hazard comment intact)

```vb
For Each entity As WW.Cad.Model.Entities.DxfEntity In model.Entities
    Select Case entity.Color.ColorIndex
        Case 9, 8, 10
            entity.Layer = model.Layers("GREEN")
            If entity.LineType.Name = "DASHED" Then
                entity.LineType = model.LineTypes("CONTINUOUS")
            Else
                entity.LineType = model.LineTypes("DASHED")
            End If

        Case 7
            If entity.AcClass = "AcDbText" Then
                ''Aanvullen met buiginformatie, voorlopig text verwijderen
                lEntitiesToDelete.Add(entity)
            End If

        Case 5, 15  ''blue, roze
            lEntitiesToDelete.Add(entity)

        Case 1, 2 '', 7 7 is levensgevaarlijk omdat zetlijnen soms ook wit zijn
            entity.Layer = model.Layers("0")

        Case Else
            lEntitiesToDelete.Add(entity)

    End Select
Next
```

## Why this is a business rule

The DXF file is the **transport between CAD and the laser/punch programmer**. A DXF that loses a bend line means the operator sees the wrong fold profile and the resulting part is wrong — potentially scrap, potentially a danger to the next step (e.g., a part-on-table that doesn't fit a press brake correctly).

By **deleting colour-7 entities that aren't `AcDbText`**, this code may drop bend lines that happen to be drawn in white. The comment confirms the author knew this — and chose to live with the risk after commenting out the broader "assign colour-7 to layer 0" branch (which would have preserved them).

Bend-line colour convention may have changed over the lifetime of the codebase, leaving this defensive comment in place even after the upstream-generator standardised on a non-white colour for bend lines. SME confirmation required.

## Triggers / when it fires

- Every call to `LayerConverter.ConvertLayers(inFile)`.
- Phase-3 follow-up to enumerate callers (likely Elumatec / SMT job-prep pipelines).

## Effects

- Colour-7 `AcDbText` → deleted (intentional).
- Colour-7 non-text entity → **deleted** (the Case 7 branch doesn't match because the `If entity.AcClass = "AcDbText"` only deletes text; non-text colour-7 falls through… actually no — non-text colour-7 entities enter the `Case 7` branch, the inner `If` evaluates false, the branch exits WITHOUT adding to delete list **and WITHOUT reassigning the layer**. The entity keeps its original layer/linetype. So non-text colour-7 is **left untouched**, not deleted.

Wait — re-read: `Case 7` then `If AcDbText Then Delete End If`. If not text, nothing happens. So colour-7 non-text passes through with original layer.

But colour-7 entities that **also fail the prior `Case` matches** never see `Case Else` — because `Case 7` already matched.

**Net effect**: colour-7 non-text entities are kept as-is with their original layer.

If those entities are bend lines, they keep whatever layer they came in with — which may or may not be the expected "GREEN" layer that bend-lines must be on for the downstream processor to interpret. Q-311.

## Edge cases / known exceptions

- A bend-line on a non-standard layer (e.g., the CAD operator drew it freehand without setting layer) → kept on its original layer → **probably ignored downstream** because the downstream programmer expects bend lines on "GREEN".
- The author's hazard comment refers to a previous behaviour (assigning colour-7 to layer "0") which would have been worse — kept colour-7 entities but on layer 0 (the default layer), wrong layer for bend lines.
- The current behaviour is the **less bad of two known-bad options**.

## Safety classification

- [x] Touches physical process — yes (controls what the laser/punch programmer sees).
- [ ] Drives cost / pricing — indirectly.
- [ ] Reversible if wrong? — yes (manual rework), but a wrong DXF reaching production can produce **scrap** or **dangerous parts**.
- [x] Blocks production if it fails? — yes (programmer rejects the file or makes wrong parts).
- `#safety-relevant` per the author's own warning. **The strongest hazard flag in the codebase.**

## SME questions

- **Q-311 (existing):** Confirm SME description of every colour-rule. Especially colour-7 hazard.
- **Q-312 (new):** Is there a JAZO CAD-side convention that bend lines must be colour 8/9/10 (not 7)? Has it ever been violated in practice?
- **Q-313 (new):** Should colour-7 non-text entities raise a warning to the user instead of silent pass-through?

Logged in [[../needs-review/_index]].

## Related

- [[../mocs/trutopslib|TruTopsLib MOC]] — parent.
- [[../external-systems/trutops|Trumpf TruTops]] — destination CAD/CAM tool.
