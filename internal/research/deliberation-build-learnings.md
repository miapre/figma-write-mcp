# Deliberation: Build Session Learnings (Builds 004–006)

**Date:** 2026-04-17
**Builds:** 004 (Trace Explorer List — primitives only), 005 v1/v2 (Trace Explorer List — misconfigured components), Detail View, Premium Dashboard
**Trigger:** User corrections across 3 builds revealed 9 systematic failure patterns not covered by existing rules

---

## Proposed Changes

### A. Spacing variable binding at creation time (not as fix pass)
**What:** Every `figma.createFrame()` that sets padding or gap must immediately follow with `setBoundVariable` calls using DS spacing variables. Raw numeric padding/gap is never acceptable.
**Where:** GOLDEN_RULES.md Rule 3 (already mentions spacing), CLAUDE.md Phase 3, ROLES.md Build Engineer gate

### B. Component icon 3-layer model
**What:** Button components have: (1) VARIANT `Icon` for slot type, (2) BOOLEAN `Icon leading`/`Icon trailing` for visibility, (3) INSTANCE_SWAP for icon selection. All three layers must be configured.
**Where:** GOLDEN_RULES.md Rule 12, CLAUDE.md Phase 3, ROLES.md Build Engineer checklist

### C. Badge Color must be explicitly set
**What:** DS Badge has a `Color` property. When inserting Badges (especially inside Table cells), Color must be set to match HTML intent. Default colors are never correct for semantic use.
**Where:** GOLDEN_RULES.md Rule 12, ROLES.md Build Engineer checklist, ROLES.md Design QA gate

### D. Text overrides by node.name, never by index
**What:** Already documented in Rule 32. Needs enforcement emphasis in Phase 3 and QA.
**Where:** ROLES.md Build Engineer checklist (already present but needs stronger language)

### E. Table sizing protocol
**What:** Header row: `counterAxisSizingMode='AUTO'` (hug height). Table area: `layoutSizingVertical='HUG'` (not FILL). At least one column: `layoutSizingHorizontal='FILL'` to stretch table to full width.
**Where:** New sub-section in GOLDEN_RULES.md Rule 5, CLAUDE.md Phase 3, ROLES.md Build Engineer gate

### F. Multi-item component cleanup (tabs, etc.)
**What:** When using a component with N default items but needing M < N, hide extras with `visible=false`. Don't just override the first M labels.
**Where:** GOLDEN_RULES.md Rule 12, ROLES.md Build Engineer checklist

### G. Page header: explicitly disable unused boolean properties
**What:** DS Page header has many boolean features. If HTML doesn't use them, set ALL to false. Don't rely on defaults.
**Where:** GOLDEN_RULES.md Rule 12

### H. No text characters as icons (reinforcement)
**What:** Already Rule 12. Reinforce: when icon needed and DS icon not found, use placeholder box (empty frame with border), never text character.
**Where:** GOLDEN_RULES.md Rule 12 (strengthen), ROLES.md Build Engineer checklist (already present)

### I. Connection line coordinate calculation
**What:** Lines in graph visualizations must calculate from actual node positions. Use center-x, bottom-edge, etc. Bezier via vectorNetwork.
**Where:** GOLDEN_RULES.md new sub-section under Rule 25 (Charts), or new rule

---

## 6-Role Deliberation

### Round 1

#### Platform Architect
**Score: 8/10**
- Changes A–H are tool-level patterns that apply to any DS. Good.
- Change I (connection lines) is borderline — it's a rendering technique, not a DS-agnostic principle. Should be framed as "graph visualization coordinate protocol" to keep it generic.
- Concern: Badge Color (C) mentions specific Color property values (Success/Error/Orange) — these are DS-specific. The rule should say "set the component's color/variant property to match HTML semantic intent" without naming specific values.
- Missing: the icon 3-layer model (B) names specific properties (Icon leading swap). Rephrase as generic pattern: "components with icon slots may have up to 3 layers of configuration."

#### Build Engineer
**Score: 7/10**
- All changes directly address Phase 3 execution failures. Strong.
- Change E (table sizing) needs more specificity: which frame is the "table area"? Is it the parent of header + rows, or each row individually? Need: the table wrapper frame gets `layoutSizingVertical='HUG'`, each row gets `counterAxisSizingMode='AUTO'`, and the first data column (or a designated flex column) gets `layoutSizingHorizontal='FILL'`.
- Change A (spacing binding) needs a concrete code pattern, not just prose. Should reference the canonical `setBoundVariable('paddingTop', spacingVar)` pattern.
- Missing from Phase 3 gate: explicit check for "Badge Color set on every Badge instance" and "table fills parent width."

#### Design QA
**Score: 8/10**
- Changes C (Badge Color) and E (table sizing) directly address visual QA failures.
- Phase 4 should add: "verify Badge colors match HTML intent" and "verify table fills available width."
- Change F (multi-item cleanup) affects QA — hidden items should not appear in screenshots.
- Missing: Phase 4 should verify no placeholder text ("Label", "Olivia Rhye") remains in any component instance.

#### DS Integration Engineer
**Score: 9/10**
- Changes A, C, D, F, G are all DS configuration depth. Correct scope.
- Rule 32 (component structure inspection) already covers D. The new emphasis on per-build enforcement is good.
- Change B (icon 3-layer model) should be documented as a configuration recipe pattern in Rule 33, not just Rule 12.
- Concern: Rule 12 is getting long. Consider splitting into 12a (variant selection), 12b (text overrides), 12c (icon configuration), 12d (property completeness). Or keep consolidated but add a numbered checklist.

#### Learning Engineer
**Score: 8/10**
- Changes produce learnable patterns that can be cached in ds-knowledge.json.
- Badge Color mapping per context (framework=Gray, status=semantic, tag=colored) is a cacheable recipe.
- Icon 3-layer model is a cacheable recipe per component type.
- Table sizing protocol is a cacheable recipe.
- Missing: should the build report explicitly track "spacing variables bound vs raw" as a quality metric? Yes.

#### Product QA
**Score: 8/10**
- All changes improve output quality from user's perspective.
- The user caught every one of these failures manually. That means Phase 4 QA is too shallow.
- Phase 4 must add content-level verification: every text node in a component instance must show HTML content, not placeholder content.
- Missing: the build communication template should include "spacing compliance: X/Y frames bound to DS variables."

### Round 1 Summary
Average: 8.0/10. Key gaps identified:
1. DS-specific values in generic rules (Badge Color names, icon property names)
2. Table sizing needs more structural specificity
3. Phase 4 QA needs placeholder text check
4. Build report needs spacing compliance metric
5. Rule 12 is overloaded

---

### Round 2 (addressing gaps)

#### Platform Architect — Revised: 10/10
- Badge Color rule now says "set the component's semantic color property to match HTML intent" — DS-agnostic.
- Icon model now says "components with icon slots may require up to 3 configuration layers: slot type selection (variant), visibility toggle (boolean), and content swap (instance swap)" — DS-agnostic.
- Connection lines framed as "graph visualization coordinate protocol" under charts rule.

#### Build Engineer — Revised: 10/10
- Table sizing protocol now specifies: table wrapper = `layoutSizingVertical='HUG'`, row = `counterAxisSizingMode='AUTO'`, designated flex column = `layoutSizingHorizontal='FILL'`.
- Spacing binding at creation time: prose references the canonical `setBoundVariable('paddingTop', var)` and `setBoundVariable('itemSpacing', var)` pattern.
- Phase 3 gate now includes: Badge Color check, table width check, spacing variable check.

#### Design QA — Revised: 10/10
- Phase 4 gate now includes: "verify no placeholder text in component instances" and "verify Badge colors match HTML" and "verify table fills width."
- Placeholder text check specifically calls out: "Label", "Olivia Rhye", default avatar names, default badge text.

#### DS Integration Engineer — Revised: 10/10
- Rule 12 consolidated with numbered sub-items rather than split into separate rules. Keeps the rule set compact.
- Icon 3-layer model documented as a recipe pattern in Rule 33 note.

#### Learning Engineer — Revised: 10/10
- Build report quality section now includes: "Spacing compliance: X/Y frames bound to DS spacing variables."
- Badge Color, icon model, and table sizing are documented as cacheable recipe types.

#### Product QA — Revised: 10/10
- Build communication template updated to include spacing compliance.
- Phase 5 explicitly states: no build is declared done if placeholder text exists anywhere.

### Round 2 Summary
All roles: 10/10. Proceed to implementation.

---

## Implementation Plan

1. **GOLDEN_RULES.md**
   - Rule 3: add spacing binding mandate (at creation time, not as fix pass)
   - Rule 5: add table sizing sub-protocol
   - Rule 12: add icon 3-layer model, Badge Color, multi-item cleanup, page header boolean completeness, icon placeholder box fallback
   - Rule 25: add graph visualization coordinate protocol

2. **ROLES.md**
   - Build Engineer Phase 3 gate: add spacing variable check, Badge Color check, table width check
   - Build Engineer checklist: add icon 3-layer model, Badge Color, multi-item cleanup, placeholder text prohibition
   - Design QA Phase 4 gate: add placeholder text check, Badge Color verification, table width verification, spacing compliance verification

3. **CLAUDE.md**
   - Phase 3 Build section: add spacing binding protocol, icon 3-layer model, table sizing protocol, Badge Color protocol

4. **Observation log**: append observations 79–84
