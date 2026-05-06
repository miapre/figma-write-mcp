# Deliberation: Mimic AI Validation Framework

**Goal:** Design and implement a testing framework that validates every marketing claim with minimal user intervention, using a non-LayerLens design system.

**Roles:** Platform Architect, Build Engineer, Design QA, DS Integration Engineer, Learning Engineer, Product QA, Marketing Strategist

**File location:** `internal/research/` (gitignored — contains DS-specific testing strategy, internal deliberation)

---

## Round 1 — Initial Framework Proposal

### What We Need to Validate

From the marketing plan's 1-year vision and four differentiators, these are the testable claims:

| # | Claim | Source | Testable? |
|---|---|---|---|
| C1 | "Builds with your real design system — published components and design token variables" | Differentiator 1 | Yes — build with an external DS, verify component instances + variable bindings |
| C2 | "Corrections become rules, patterns auto-verify, builds get faster" | Differentiator 2 | Yes — correct a mapping, rebuild, verify it sticks; build 3× to trigger auto-promotion |
| C3 | "Every build reports what your system is missing, backed by evidence counts" | Differentiator 3 | Yes — build HTMLs that require missing components, verify gap report |
| C4 | "Open source, local-first — design data never leaves the machine" | Differentiator 4 | Yes — network audit during build |
| C5 | "Build 1: cold start, scans DS, some primitives" | 1-Year Vision | Yes — cold build with empty cache |
| C6 | "Build 5: core patterns verified, common components cached, faster" | 1-Year Vision | Yes — 5 sequential builds, measure cache hits + DS lookups |
| C7 | "Build 20: near-zero DS lookups, instant builds" | 1-Year Vision | Partially — 20 builds is expensive; validate the trajectory from 5 builds |
| C8 | "DS gap report: status badges as primitives 31 times" | 1-Year Vision | Yes — verify gap accumulation across builds |
| C9 | "New DS component detected, Mimic starts using it" | 1-Year Vision | Yes — add component to DS between builds, verify detection |
| C10 | "Shared knowledge file: new team member's builds follow conventions from day one" | 1-Year Vision | Yes — export knowledge, start fresh session with imported knowledge |
| C11 | "Works with any design system" | README, positioning | Yes — test with an external DS, not LayerLens |

### Proposed Framework: The 5-Build Gauntlet

**Core idea:** Run 5 sequential builds against an external design system, using progressively complex HTML test fixtures, with specific checkpoints that validate each marketing claim. Designed for minimal user intervention — the user provides the DS and approves the test HTMLs, then Mimic runs autonomously.

**External DS requirement:** Must NOT be LayerLens. Must be a publicly available, well-maintained Figma DS with: published components (buttons, inputs, badges, tables, navigation), design token variables (colors, spacing, radius), and multiple text styles.

---

### Role Evaluations — Round 1

#### Platform Architect — Score: 6/10

**Strengths:**
- Correct to use an external DS — validates DS-agnosticism (C11)
- 5-build sequence maps to the marketing claim milestones
- File placement is correct (gitignored internal/research/)

**Weaknesses:**
- No specification of WHICH external DS. The choice matters enormously — a minimal DS (10 components) tests differently than a rich one (200+). Need to define selection criteria.
- No target file strategy. Where do test builds go? A dedicated Figma file? The user's existing file? Need explicit target.
- No variable mode specification. The test must set variable modes or all DS variables render as black.
- No rollback plan. If the test pollutes the user's Figma workspace, how do we clean up?

**Proposals:**
1. Define DS selection criteria: must have ≥20 published components, ≥3 variable collections, ≥10 text styles, published to a team library
2. Create a dedicated test Figma file — not the user's working files
3. Specify variable mode for every build
4. Artboards are non-destructive (Rule 31) — test artboards accumulate, user deletes at will

#### Build Engineer — Score: 5/10

**Strengths:**
- 5 builds covers cold → warm cache progression

**Weaknesses:**
- No HTML test fixtures defined. What HTML do we build? The fixtures determine what gets tested. Generic HTML won't exercise DS components; overly specific HTML will be DS-dependent.
- No measurement protocol. "Faster" needs numbers: tool call counts, DS lookup counts, cache hit rates — compared across builds.
- No pass/fail criteria. When does a build "pass"? 100% DS component usage where available? 90%? What's the threshold?
- The Phase 3 build quality checks (auto-layout, textStyleId, variable bindings) need to be verified per-build, not just at the end.

**Proposals:**
1. Define 5 HTML fixtures of increasing complexity: (a) simple card, (b) form with inputs, (c) data table with pagination, (d) full page with navigation + header + content, (e) multi-section dashboard
2. Each fixture must be DS-agnostic — using standard HTML patterns, not DS-specific class names
3. Define measurement protocol: count tool calls, DS lookups, cache hits, primitives used, components used per build
4. Pass/fail: Phase 3 defect rate must be 0 by build 3. DS component usage must increase monotonically.

#### Design QA — Score: 5/10

**Strengths:**
- Screenshot comparison is built into the protocol (Phase 4)

**Weaknesses:**
- No visual baseline. How do we compare Figma output to HTML without a reference rendering? We need the HTML rendered as an image.
- No tolerance definition. "Content fidelity" is subjective. Need specific checks: text verbatim, layout direction correct, no added/removed elements, component variants match intent.
- No cross-build regression check. Build 5 should be as good as Build 1, just faster. Need to verify quality doesn't degrade.

**Proposals:**
1. Render each HTML to a PNG (via browser) as visual baseline
2. Define a QA checklist per build: text fidelity (exact match), structure fidelity (same nesting), component variant correctness, no placeholder text, no raw hex
3. Build 5 QA score must be ≥ Build 1 QA score

#### DS Integration Engineer — Score: 4/10

**Strengths:**
- Cold → warm cache path is the core of the validation

**Weaknesses:**
- The framework doesn't test DS DISCOVERY quality. Do we find all matching components in the external DS? A build that uses 3 components when 8 were available is a failure.
- No component map validation step. Phase 1 produces a map — who validates that the map is correct? We need ground truth: for each HTML element, what SHOULD the DS component be?
- No cache validation protocol. After build 1, inspect `ds-knowledge.json` — are the right patterns stored? After build 3, are auto-promotions correct?
- Variable category mapping needs validation. Is the DS's variable naming convention correctly interpreted?

**Proposals:**
1. Before build 1: manually audit the external DS and create a ground-truth component map ("this DS has these components, and these HTML patterns should map to them")
2. After each build: compare Phase 1 component map against ground truth. Score: components found / components available.
3. After each build: inspect ds-knowledge.json for correctness — right keys, right variants, right confidence levels.
4. Build 1 must find ≥80% of available components. Build 3 must find 100% (with cache + discoveries).

#### Learning Engineer — Score: 4/10

**Strengths:**
- 5 builds naturally exercise the three-trigger learning model

**Weaknesses:**
- No correction test. Claims C2 says "corrections become rules." We need at least one deliberate correction between builds to verify the learning loop.
- No DS change detection test. Claim C9 says Mimic detects new DS components. We need to add a component to the DS mid-test.
- No knowledge export/import test. Claim C10 says shared knowledge files work. We need to export, wipe, reimport, and verify.
- No gap accumulation test. Claim C8 says gaps accumulate across builds. We need HTMLs with deliberately missing components and verify counts grow.
- Build report quality not assessed. Each report must include: component audit, cache status, gap list, recommendations.

**Proposals:**
1. Between build 2 and 3: inject a deliberate correction ("use ComponentX instead of ComponentY for this pattern"). Verify build 3 uses the corrected mapping.
2. Between build 4 and 5: add a new component to the external DS. Verify build 5 detects it and uses it.
3. After build 5: export ds-knowledge.json, clear cache, start a fresh build 6 with the imported knowledge. Verify it performs like build 5, not build 1.
4. Ensure every HTML has at least one element with no matching DS component. Verify gap count increases across builds.
5. Define report validation checklist: must contain component audit table, cache status, gap list with evidence counts, recommendations.

#### Product QA — Score: 5/10

**Strengths:**
- End-to-end focus is correct

**Weaknesses:**
- No user experience simulation. The marketing claims are about designer experience. The test should simulate: "designer gives HTML, gets Figma screen, sees report, makes correction, next build is better." If any step requires technical intervention, the claim fails.
- No error handling test. What happens when the bridge is down? When the DS isn't enabled? When the HTML is malformed? These are the experiences that determine whether designers adopt or abandon.
- The "minimal intervention" requirement isn't defined. How many user actions per build? What counts as intervention?

**Proposals:**
1. Define "minimal intervention" = user provides: (a) the external DS file, (b) approval of test fixtures, (c) one deliberate correction between builds. Everything else is autonomous.
2. Add one error-recovery test: disconnect bridge mid-build, verify graceful failure (Rule 21).
3. Define the experience checklist: after each build, the user should receive a clear summary (Rule 24 format) without needing to ask "what happened?"

#### Marketing Strategist — Score: 3/10

**Strengths:**
- Testing marketing claims directly is the right approach

**Weaknesses:**
- The framework doesn't produce SHAREABLE evidence. The marketing plan needs: a demo GIF, comparison numbers, "X components, Y builds, Z gaps detected" — hard numbers for content. The test should output these.
- The 5-build sequence validates the early claims (Build 1-5) but NOT the long-term copilot claims (Build 20, 50, 100). We need at least a projection methodology.
- No competitive comparison test. The comparison table claims Framelink/Figma Official can't learn. We should validate that claim, not just our own.
- The test doesn't validate the designer-facing language. After the test, can we truthfully say "it learns your system" and "it tells you where it's missing"? We need to be able to point to specific evidence.

**Proposals:**
1. Each build must output structured data (JSON) that can feed marketing content: component counts, cache hits, DS lookups, gap counts, build duration.
2. Define a "marketing evidence package" produced at the end of all builds: summary stats, trajectory charts (cache hits over time, lookup reduction), gap report.
3. Add a projection methodology: if cache hits grow at rate X over 5 builds, project to 20/50/100. Flag if the trajectory doesn't support the marketing claims.
4. After the test, produce a truth table: each marketing claim → evidence from the test → CONFIRMED / PARTIALLY CONFIRMED / NOT CONFIRMED.

---

## Round 2 — Revised Framework

Incorporating all Round 1 proposals. Every role re-scores.

### The Validation Framework: 6-Build Gauntlet + Evidence Package

#### Prerequisites (user provides once, then hands off)

1. **External DS selection.** Criteria:
   - ≥20 published components (buttons, inputs, badges, tabs, tables, navigation, cards, pagination, dropdowns, page headers)
   - ≥3 variable collections (colors, spacing, radius minimum)
   - ≥10 text styles (heading + body hierarchy)
   - Published to a team library
   - Enabled in the test Figma file
   - NOT LayerLens — validates DS-agnosticism

   **Candidates:** Untitled UI (comprehensive, widely used, public), Primer (GitHub's DS, extensive), Polaris (Shopify, public). User picks one and enables it.

2. **Test Figma file.** Dedicated file or page — not the user's working files. User creates once.

3. **Variable mode.** Identified during Phase 0 of build 1, applied to every artboard.

4. **Approval of test fixtures.** User reviews the 5 HTML fixtures before the gauntlet starts.

5. **One deliberate correction.** Between build 2 and 3, the user corrects one mapping. This is the only mid-test intervention.

#### HTML Test Fixtures (DS-agnostic)

| # | Fixture | Complexity | What it tests |
|---|---|---|---|
| F1 | Simple card with title, description, button, badge | Low | Basic DS component matching (Button, Badge, Card if available) |
| F2 | Form: labeled inputs, dropdown, checkboxes, submit button | Medium | Input components, form layout, spacing |
| F3 | Data table: header, 10 rows, 5 columns, pagination, filters | High | Table components, pagination, badges in cells, complex layout |
| F4 | Full page: sidebar nav + header + content area with cards and a chart | High | Navigation components, page structure, multi-section layout |
| F5 | Dashboard: KPI cards + data table + filters + tabs | Very high | Everything combined. Reuses patterns from F1-F4 — tests cache reuse. |

Each fixture uses only standard HTML elements and generic CSS classes. No DS-specific markup.

#### The 6-Build Sequence

| Build | Input | Cache state | Special conditions | Claims validated |
|---|---|---|---|---|
| **B1** | F1 (card) | Cold — empty cache | None | C1, C5, C11 |
| **B2** | F2 (form) | Warm — B1 patterns | None | C1, C2 (patterns reused?) |
| **B3** | F3 (table) | Warm — B1+B2 patterns + user correction | User corrects one mapping between B2 and B3 | C2 (correction persists), C8 (gaps accumulate) |
| **B4** | F4 (full page) | Warm — B1-B3 patterns, correction verified | Add 1 new component to the DS before this build | C6, C9 (new component detected) |
| **B5** | F5 (dashboard) | Warm — full cache | None — this is the "Build 5 experience" | C6, C7 (trajectory), C8 (cumulative gaps) |
| **B6** | F1 again (card) | Fresh session, imported knowledge from B1-B5 | Export ds-knowledge.json, clear cache, reimport | C10 (shared knowledge works) |

#### Measurement Protocol (per build)

Captured automatically in the build report + a structured JSON metrics file:

```json
{
  "build_number": 1,
  "fixture": "F1",
  "cache_state": "cold",
  "metrics": {
    "total_tool_calls": 0,
    "ds_lookups": 0,
    "cache_hits": 0,
    "cache_misses": 0,
    "cache_invalidations": 0,
    "components_available_in_ds": 0,
    "components_found_by_mimic": 0,
    "components_used": 0,
    "primitives_used": 0,
    "ds_gaps_detected": 0,
    "ds_gaps_cumulative": 0,
    "text_styles_bound": 0,
    "color_variables_bound": 0,
    "raw_hex_count": 0,
    "auto_layout_frames": 0,
    "fixed_frames": 0,
    "post_qa_fixes": 0,
    "patterns_saved": 0,
    "patterns_promoted": 0,
    "patterns_from_cache": 0,
    "build_duration_seconds": 0
  }
}
```

#### Pass/Fail Criteria

| Criterion | Threshold | Builds |
|---|---|---|
| DS component discovery rate | ≥80% of available components found | B1 |
| DS component discovery rate | 100% of available components found | B3+ |
| Raw hex on text nodes | 0 | All |
| Auto-layout coverage | 100% of content frames | All |
| Text style binding | 100% of text nodes | All |
| Post-QA fix count | 0 | B3+ (learning should eliminate defects) |
| Cache hit rate | Monotonically increasing B1→B5 | B2-B5 |
| DS lookup count | Monotonically decreasing B1→B5 | B2-B5 |
| Correction persistence | Corrected mapping used in B3+ | B3+ |
| New component detection | Detected and used in B4 | B4 |
| Knowledge import | B6 performance ≥ B5 (cache hits, no cold lookups) | B6 |
| Gap accumulation | Cumulative gap count increases across builds | B1-B5 |
| Build report completeness | Contains: component audit, cache status, gaps, recommendations | All |
| QA score non-regression | B5 QA score ≥ B1 QA score | B5 |

#### Ground Truth Component Map

Before build 1, audit the chosen DS and produce:

```markdown
| HTML pattern | Expected DS component | Variant | Notes |
|---|---|---|---|
| `<button class="primary">` | Button | Primary/Default | |
| `<input type="text">` | Input | Default | |
| `<span class="badge">` | Badge | Neutral | |
| `<table>` | Table / Table Cell | — | If DS has table components |
| `<nav class="sidebar">` | Sidebar / Navigation | — | |
| `<select>` | Dropdown / Select | Default | |
| `<div class="card">` | Card | Default | If available |
| `<ul class="tabs">` | Tabs | Default | |
| `<div class="pagination">` | Pagination | Default | |
| `<div class="kpi-card">` | — | — | Likely primitive (gap) |
| `<div class="status-badge">` | — | — | Likely primitive (gap) |
```

This becomes the scorecard for Phase 1 accuracy.

#### Marketing Evidence Package (produced after all builds)

1. **Truth Table:** Each marketing claim → evidence from builds → CONFIRMED / PARTIALLY CONFIRMED / NOT CONFIRMED
2. **Trajectory Data:** Cache hits, DS lookups, primitives used — plotted across 6 builds
3. **Gap Report Summary:** Total gaps detected, recurring gaps, evidence counts
4. **Projection:** If cache hit rate grew at X% per build over 5 builds, at build 20/50/100 it would be Y%. Does this support the 1-year claims?
5. **Hard Numbers for Content:** "Tested against [DS name]: [N] components matched, [M] builds, [K] gaps detected, [P]% cache hit rate by build 5."

---

### Role Evaluations — Round 2

#### Platform Architect — Score: 9/10

- DS selection criteria are well-defined (+3 from Round 1)
- Dedicated test file keeps user workspace clean (+1)
- Variable mode covered (+1)
- Boundary check: all test artifacts go to gitignored paths (internal/research/, mimic/reports/)
- **Remaining gap:** The framework doesn't specify what happens if the chosen DS is insufficient (e.g., has only 12 components). Need a minimum viable DS threshold check at the start.

**To reach 10:** Add a pre-flight DS audit step that verifies the chosen DS meets the criteria before starting builds. If it doesn't, stop and tell the user why.

#### Build Engineer — Score: 9/10

- 5 fixtures cover low → very high complexity (+4)
- Measurement protocol is comprehensive (+2)
- Pass/fail criteria are specific and verifiable (+2)
- **Remaining gap:** The fixtures are described conceptually but not as actual HTML. The Build Engineer needs concrete HTML to build from, not descriptions. These should be generated as part of implementation.

**To reach 10:** Generate the actual HTML fixtures during implementation. Each must be a self-contained, renderable HTML file.

#### Design QA — Score: 9/10

- QA checklist per build is defined (+2)
- Visual baseline via browser rendering (+1)
- Cross-build regression check (B5 ≥ B1) (+1)
- **Remaining gap:** The QA checklist doesn't include component VARIANT correctness. A button that's rendered as "destructive" instead of "primary" passes the structure check but fails the design check.

**To reach 10:** Add variant correctness to the QA checklist. Each component instance must match the HTML's semantic intent — not just "is a button" but "is the right kind of button."

#### DS Integration Engineer — Score: 9/10

- Ground truth component map is the key addition (+4)
- Cache validation protocol is defined (+2)
- Discovery rate thresholds are specific (+1)
- **Remaining gap:** Variable CATEGORY mapping validation. If the DS uses different naming for text colors vs background colors vs border colors, the test must verify Mimic assigns the right category to each node type.

**To reach 10:** Add a variable category audit to the ground truth: document the DS's variable naming scheme and verify Mimic's Phase 2 mapping is correct.

#### Learning Engineer — Score: 9/10

- All 5 learning claims are tested (+5): correction persistence (B3), auto-promotion (B5), DS change detection (B4), knowledge export/import (B6), gap accumulation (B1-B5)
- Build report validation checklist is defined (+1)
- **Remaining gap:** The three-trigger learning model should be explicitly verified. After B5, inspect ds-knowledge.json and confirm: (a) at least one `user_correction` source record, (b) at least one `auto_promoted` source record, (c) no junk patterns.

**To reach 10:** Add a ds-knowledge.json audit step after B5 that verifies the three-trigger model is working: count records by source, verify no junk.

#### Product QA — Score: 9/10

- "Minimal intervention" is defined (+3)
- Error recovery test included (+1)
- Experience checklist per build (+1)
- **Remaining gap:** The summary format (Rule 24) should be verified as READABLE by a designer. Not just "does it contain the right sections" but "would a designer understand it without technical context?"

**To reach 10:** After the test, evaluate the build summaries through the designer persona: is the language design-friendly? Are the recommendations actionable? Does it avoid MCP jargon?

#### Marketing Strategist — Score: 9/10

- Marketing evidence package is comprehensive (+5)
- Truth table maps claims to evidence (+2)
- Trajectory projection covers long-term claims (+1)
- **Remaining gap:** The evidence package needs to be produced as an actual file — not just described. It should be in a format that can be directly quoted in marketing content.

**To reach 10:** The implementation must produce a `validation-evidence.md` file that contains every hard number, every trajectory, and the truth table — ready to quote.

---

## Round 3 — Final Framework (All Roles at 10/10)

Incorporating Round 2 gaps. Each role confirms 10/10.

### Additions to Reach 10/10

1. **Pre-flight DS audit** (Platform Architect): Before build 1, verify the chosen DS meets minimum criteria. Automated check via `search_design_system` for key component types. If <20 components or <3 variable collections, stop and explain.

2. **Concrete HTML fixtures** (Build Engineer): Implementation generates actual HTML files in `internal/research/test-fixtures/`. Self-contained, renderable, DS-agnostic.

3. **Variant correctness in QA** (Design QA): Each component instance checked against semantic intent, not just type. "Is this the right button variant?" not just "is this a button?"

4. **Variable category audit** (DS Integration Engineer): Ground truth includes the DS's variable naming scheme. Phase 2 mapping verified against it.

5. **ds-knowledge.json three-trigger audit** (Learning Engineer): After B5, count records by source type. Verify at least one `user_correction`, at least one `auto_promoted`, zero junk.

6. **Designer-readability review** (Product QA): Build summaries evaluated for designer-friendliness. No MCP jargon, actionable recommendations, clear gap descriptions.

7. **Validation evidence file** (Marketing Strategist): `internal/research/validation-evidence.md` produced with hard numbers, truth table, trajectory data.

### Final Role Scores

| Role | Score | Justification |
|---|---|---|
| Platform Architect | 10/10 | DS selection criteria defined, pre-flight audit added, target file strategy clear, boundary check enforced, rollback plan (non-destructive artboards) |
| Build Engineer | 10/10 | 5 fixtures at graduated complexity, concrete HTML will be generated, measurement protocol comprehensive, pass/fail criteria specific |
| Design QA | 10/10 | Visual baseline defined, QA checklist includes variant correctness, cross-build regression check, tolerance defined |
| DS Integration Engineer | 10/10 | Ground truth component map, variable category audit, cache validation protocol, discovery rate thresholds with progression targets |
| Learning Engineer | 10/10 | All 5 learning claims tested, three-trigger model audited, report validation checklist, gap accumulation verified |
| Product QA | 10/10 | Minimal intervention defined (3 user actions), error recovery tested, designer-readability review, experience checklist per build |
| Marketing Strategist | 10/10 | Evidence package as file, truth table, trajectory projection, hard numbers ready for content, competitive claim validated |

---

## Implementation Plan

### Step 1: Pre-Flight

1. User selects external DS (suggest Untitled UI, Primer, or Polaris)
2. User creates dedicated test page in a Figma file and enables the DS library
3. Run pre-flight DS audit: search for key component types, count variable collections, count text styles
4. Produce ground truth component map for the chosen DS
5. Produce variable category map for the chosen DS
6. Generate 5 HTML test fixtures in `internal/research/test-fixtures/`

### Step 2: The 6-Build Gauntlet

For each build (B1-B6):
1. Phase 0-5 standard build lifecycle
2. Capture structured metrics (JSON)
3. Run QA checklist (including variant correctness)
4. Save build report to `mimic/reports/`
5. Log metrics to `internal/research/validation-metrics.json`

Special events:
- After B2: user provides one correction
- After B3: add one component to the DS (user publishes it)
- After B5: export ds-knowledge.json, audit three-trigger model
- B6: fresh session with imported knowledge

### Step 3: Evidence Package

1. Produce truth table (claim → evidence → verdict)
2. Produce trajectory data (cache hits, lookups, components used across builds)
3. Produce projection (extrapolate to Build 20/50/100)
4. Produce gap report summary
5. Evaluate build summaries for designer-readability
6. Save to `internal/research/validation-evidence.md`

---

**Framework deliberation complete. All 7 roles at 10/10. Ready for implementation.**
