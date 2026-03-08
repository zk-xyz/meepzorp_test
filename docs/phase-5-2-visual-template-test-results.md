# Phase 5.2 — Visual Template Test Results & Amendment Notes

**Date:** March 7, 2026
**Tests completed:** 2, 3 (A/B), 4
**System versions:** Brand OS v1.5, VIM v1.8, Bridge Spec v2.0
**Production method:** PIL composite (confirmed from Test 1 as reliable path)
**Font stack used:** Inter Bold/Medium/Regular (converted from @fontsource/inter woff2 via fonttools)

-----

## Test Results Summary

### Test 2: Blog Hero — "AEO Is Built for Decisions"

- **File:** `test2_blog_hero.png` (1344×576)
- **Result:** PASS — all compliance checks clear
- **Ghost text:** "AEO" at 280px Inter Bold, alpha 40/255 = 15.7%
- **Elements:** Ghost text + data flow lines (8% opacity) + "Built for Decisions" accent (24px) + "AEO vs SEO" category tag (16px tertiary)
- **Notes:** Clean execution. The cinematic 21:9 format gives the ghost text room to breathe. Data flow lines add texture without competing. No foreground headline — blog hero sits behind the article title on the page, so the ghost text IS the primary visual.

### Test 3: Quote Card — "The Story Gap"

- **Files:** `test3_quote_card_a_with_ghost.png` and `test3_quote_card_b_no_ghost.png` (1080×1080)
- **Result:** PASS (both versions) with one decision point
- **Ghost text (Version A):** "CLARITY" at 200px Inter Regular, alpha 38/255 = 14.9%
- **Decision required:** v1.8 Section 2a says "Never in 1:1 formats without a dominant headline." The quote (30px Medium) is NOT a headline — it's body-scale text. Version B omits ghost text per strict reading. Version A treats it as pure background texture. **Recommendation: Version A is stronger for LinkedIn feed presence. Propose adding a carve-out to the ghost text conditions: "Quote cards with a single dominant text block may use ghost text in 1:1 format when the ghost word reinforces the article's core concept."**
- **C12 edge case:** Quote at 30px Medium. C12 says body ≤24px, above that needs heading weight. At 1080×1080, 30px is proportionally equivalent to ~16px on a standard viewport. The quote IS the primary content element — it's not really "body copy." Flag for clarification: **Does C12 apply to the primary content element on a standalone card, or only to supporting body text within a multi-element layout?**

### Test 4: LinkedIn Square — "The Story Gap"

- **File:** `test4_linkedin_square.png` (1200×1200)
- **Result:** PASS — all compliance checks clear
- **Ghost text:** "STORY" at 280px Inter Regular (reduced from 300px to stay under 80% frame width — was at 82.9%, now 77.3%), alpha 45/255 = 17.6%
- **Headline:** "The Story" (white) + "Gap" (orange accent) at 150px Bold per Figma spec
- **C5 mark:** Text "C5" placeholder in orange, bottom-right. Production needs the actual SVG logomark.
- **Ghost text condition check:** All five conditions met. This is the cleanest ghost text use case — dominant headline, text-primary, single hierarchy level.

-----

## Template Amendment Notes (for `visual-content-prompt-templates-c5.md` v1.0 → v2.0)

These changes are needed to align templates with VIM v1.8.

### 1. Compliance Spec References

**All templates:** Update compliance check references from C1–C8 to C1–C13. Add to every template's compliance section:

- C9: Ghost text opacity ≤25%
- C10: No glass-on-glass stacking
- C11: Monogram rotation 45°/90° only
- C12: Body copy ≤24px
- C13: Wordmark contrast (text.primary or text.secondary only)

### 2. Background Color Expansion

**All templates:** C1 check must include `#ffffff`, `#f2f2f2`, `#faf9f5` as approved light-mode surfaces, and `#fbdbf2` as approved campaign hero background (requires active campaign).

### 3. Typography Usage System

**All templates using ghost text:** Add formal conditions check:

- Apply when: single dominant headline, text-primary, ≥3:1 aspect ratio OR ≥60% content area, ≤2 hierarchy levels
- Never when: data viz primary, ghost word >80% frame width, >2 hierarchy levels, 1:1 without dominant headline
- Weight: Regular 400 only for ghost text; Bold 700 for foreground headlines; Medium 500 for everything else
- Opacity: 15–25% hard range

### 4. Social Template Figma Specs

**LinkedIn landscape (1200×628):** Add Figma-confirmed spec: heading Bold 119px, leading 0.91, tracking -2.38px, wordmark bottom-right at 38px
**LinkedIn/Instagram square (1200×1200):** Add Figma-confirmed spec: heading Bold 150px, leading 1.0, tracking -3px, C5 logomark bottom-right in orange

### 5. Campaign Hero Pattern (NEW)

**Add new template:** `campaign-hero-01` for the full-width pink section pattern. Light Pink (#fbdbf2) background, dark text, "Best Story Wins" badge, two-column body copy. Requires active campaign.

### 6. Wordmark Contrast Rule (NEW)

**All templates with wordmark:** Add C13 check. Wordmark must render at text.primary (nav/header) or text.secondary (footer). Never tertiary or lower.

### 7. Anti-Pattern Checklist

**All templates:** Add pre-generation checklist derived from Section 4f:

- No Regular 400 for foreground text
- No body text above 24px
- No center-aligned body copy
- No more than 65% non-dark fill
- No more than 3 text blocks
- No glass-on-glass

-----

## VIM Amendment Notes (for v1.8 → v1.9)

### 1. C7/5f Inconsistency

C7 in the compliance table lists wordmark backgrounds as `#1a171a` or `#ffffff` only. Section 5f adds `#fbdbf2` (campaign hero) as approved. **Fix:** Update C7 to include `#fbdbf2` with the qualifier "(campaign hero — requires active campaign)".

### 2. Ghost Text 1:1 Condition Refinement

Current: "Never in 1:1 formats without a dominant headline."
**Proposed addition:** "Quote cards and single-message cards with one dominant text element may use ghost text in 1:1 format when the ghost word directly reinforces the content's core concept and the ghost text occupies ≤60% of frame width."

### 3. C12 Scope Clarification

Current: "Body copy ≤24px. Text above this threshold must use heading weight and hierarchy treatment."
**Proposed clarification:** "Applies to supporting body text in multi-element layouts. The primary content element on a standalone card (e.g., a pull quote on a quote card) is not subject to C12 when it is the only text element and is sized proportionally to the card format."

### 4. PIL Implementation Spec (NEW — for Section 4d or companion doc)

Add production-tested PIL recipe to the generation policy or a companion implementation guide:

- Font stack: Inter (preferred, via @fontsource/inter + fonttools woff2→ttf conversion) → Poppins (fallback) → DejaVu Sans (emergency)
- Ghost text alpha range: 33–64/255 (13–25%). Start at 38–45, hard ceiling at 64
- Gradient fade: numpy array multiplication of ghost alpha channel × gradient mask
- RGBA layer compositing: separate layer per element type, alpha_composite stack

### 5. Font Acquisition Note (NEW — for Section 2 or Known Gaps)

Inter is not pre-installed in most headless environments. Documented working method: `npm install @fontsource/inter` → convert woff2 to TTF via `fonttools`. Add to Known Gaps or to a companion "Environment Setup" doc.

-----

## Phase 5.2 Status Update

| Task                     | Status                                                            |
| ------------------------ | ----------------------------------------------------------------- |
| Test 2: Blog Hero        | ✅ Complete — PASS                                                |
| Test 3: Quote Card       | ✅ Complete — PASS (A/B, decision needed on ghost text convention) |
| Test 4: LinkedIn Square  | ✅ Complete — PASS (ghost text width caught and fixed)             |
| Template Amendment Notes | ✅ Complete (this document)                                       |
| VIM Amendment Notes      | ✅ Complete (this document)                                       |

## Next Steps

- Jason decision on Test 3 ghost text convention (Version A vs B)
- Jason decision on C12 scope clarification
- Apply template amendments → `visual-content-prompt-templates-c5.md` v2.0
- Apply VIM amendments → VIM v1.9
- Phase 5.3: Notion DB for Prompt Library
