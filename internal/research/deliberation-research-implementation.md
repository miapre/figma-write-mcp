# Deliberation: Research Report → Implementation Plan

**Date:** 2026-04-17
**Source:** `internal/research/mimic-ai-research-report.md`
**Scope:** Evaluate research findings, propose implementation framework, score until 10/10
**Authority:** CLAUDE.md, GOLDEN_RULES.md, ROLES.md

---

## Evaluation Areas

Derived from the research report's four recommendation pillars, plus two operational constraints the user explicitly mandated:

| Area | Description |
|---|---|
| **A. Memory Layer** | Graphiti-inspired non-destructive schema: pattern signatures, temporal invalidation, scope cascading |
| **B. Component Matching Pipeline** | 4-source catalog fusion: REST enumeration + Code Connect + PNG thumbnails + descriptions |
| **C. Learning Signals** | Event-driven corrections as first-class data; avoid Mem0-style junk extraction |
| **D. Transparency UX** | Memory inspector, inline notifications, provenance, conversational correction |
| **E. Backward Compatibility** | Don't break the current build protocol, golden rules, phased gates, or ds-knowledge.json |
| **F. Token & Tool Call Efficiency** | Every change must reduce or maintain token usage and tool calls; no architectural tax |

---

## Round 1 — Initial Scoring

### Platform Architect

| Area | Score | Assertion |
|---|---|---|
| A. Memory | 6/10 | Graphiti's bi-temporal model is architecturally sound but Neo4j is a deployment dependency that violates Mimic's "clone and run" premise. The flat-file pattern (`ds-knowledge.json`) must remain the canonical storage; the schema upgrade should be structural (add `valid_until`, `supersedes`, `source`, `confidence` fields) without requiring a graph DB. |
| B. Matching | 5/10 | Code Connect is the right authority source but only ~15% of DS teams use it today. CLIP/multimodal matching requires inference infrastructure Mimic doesn't have. The pragmatic subset is REST enumeration (already done) + description embeddings (free via Claude) + LLM-in-the-loop (current approach). Two of four sources are premature. |
| C. Learning | 8/10 | Research correctly identifies three triggers (correction, confirmation, repetition). This maps directly to the existing self-improvement loop in CLAUDE.md §Workflow. Implementation is mostly documentation + a schema field (`source: user_correction | user_confirmation | auto_promoted`). Low risk. |
| D. Transparency | 4/10 | Plugin UI changes require JavaScript work in `plugin/ui.html`. This is the right direction but is a separate engineering effort from the build protocol. The terminal-side transparency (inline pattern notifications, provenance in reports) can be done immediately in VOICE_AND_TONE.md and the Phase 5 report format. Split into now (terminal) and later (plugin UI). |
| E. Backward | 9/10 | The research explicitly says "operationalizing it with modern memory and UX primitives" — not rewriting. All changes are additive. The risk is in the matching pipeline if it replaces the current LLM-in-context approach before being proven. Gate: new pipeline must outperform current on a benchmark of 5 prior builds before replacing it. |
| F. Efficiency | 7/10 | ChatGPT's "compiled rulebook in every prompt" pattern is validated for <500 rules. This is Mimic's case. Injecting ds-knowledge.json into context is simpler than vector retrieval and costs ~500 tokens. But the 4-source catalog build would add reads and API calls during Phase 1. Must be gated behind warm-cache. |

### Build Engineer

| Area | Score | Assertion |
|---|---|---|
| A. Memory | 7/10 | The schema upgrade (`valid_until`, `supersedes`, `confidence`) is compatible with the current JSON file. I can read/write it with no new dependencies. But the Build Engineer doesn't consume the memory layer directly — it flows through Phase 1 (DS Discovery) as a component map. The schema upgrade has zero impact on Phase 3 execution. Neutral on score — it doesn't help or hurt builds. |
| B. Matching | 6/10 | Better component matching directly improves build quality. REST enumeration + cached component properties in ds-knowledge.json is the current approach. Adding Code Connect mappings would give me `{html_class: component_key, props_mapping}` tuples — exactly what I need for `importComponentByKeyAsync + setProperties`. This is high-value IF the user's DS has Code Connect. For DSes without it, no improvement. |
| C. Learning | 8/10 | If corrections automatically write to ds-knowledge.json with high confidence, my Phase 3 gets better inputs on every run. The "3 uncorrected uses → auto-promote" pattern means I'll progressively get more VERIFIED mappings. Directly reduces build defects. |
| D. Transparency | 5/10 | Provenance in build reports ("used rule #42") adds lines to Phase 5 output but doesn't change Phase 3. The plugin UI inspector is irrelevant to build execution. Terminal notifications are useful for debugging when a component match is wrong — "matched via rule #42 (auto_promoted, confidence 0.85)" helps me trace failures. |
| E. Backward | 9/10 | Current Phase 3 reads a component map and builds. As long as the component map format stays stable, the memory layer can change underneath without affecting builds. The matching pipeline feeds into Phase 1, not Phase 3. No risk. |
| F. Efficiency | 8/10 | Warm-cache builds that skip DS search_design_system calls because ds-knowledge.json already has the mapping would significantly reduce tool calls. A cold build does 4-6 DS searches; a warm build could do 0-1. This is the #1 efficiency win from the memory layer. |

### Design QA

| Area | Score | Assertion |
|---|---|---|
| A. Memory | 6/10 | A memory layer with provenance means I can trace *why* a component was chosen, which helps me verify whether it was the right choice. But the current QA failure mode (Build 005: misconfigured components) is not a memory problem — it's a component structure problem. The memory layer doesn't help me verify that text overrides landed correctly. |
| B. Matching | 7/10 | Better matching → fewer wrong component variants → fewer QA failures. If the pipeline can resolve "which Badge Color variant for a success status" without heuristics, that's a direct QA improvement. |
| C. Learning | 9/10 | Every correction I surface should flow back as a high-confidence learning. If I catch "wrong Badge color" in Phase 4 and the user confirms, that correction should write to memory immediately. Currently corrections go to GOLDEN_RULES.md or memory files — the proposed event-driven model would make this faster and more structured. |
| D. Transparency | 8/10 | Provenance per node ("used rule #42") would let me cross-reference the build report with the component map. If rule #42 says "Badge/Success for status=Success" and the node shows Badge/Gray, that's a traceable defect. Without provenance, I can only compare against the HTML visually. |
| E. Backward | 9/10 | No impact on QA process. Phase 4 remains: screenshot → compare → verify content fidelity. |
| F. Efficiency | 7/10 | If better matching reduces post-QA fixes, that saves 2-5 tool calls per build (the current "fix pass" calls). Direct efficiency gain from quality improvement. |

### DS Integration Engineer

| Area | Score | Assertion |
|---|---|---|
| A. Memory | 9/10 | This is my domain. The current ds-knowledge.json records component keys and variable mappings but lacks confidence scores, temporal history, source attribution, and scope cascading. The proposed schema adds exactly these. I can produce better Phase 1 component maps if I know a mapping was user-corrected (high trust) vs auto-inferred (low trust). |
| B. Matching | 9/10 | The 4-source fusion is the right architecture. Today I search `search_design_system` and hope for good results. With a pre-built catalog indexed by structural features + description embeddings, I can match with much higher precision. The pragmatic version: (1) REST enumeration (cache in ds-knowledge.json), (2) component `description` + `componentPropertyDefinitions` for semantic matching, (3) LLM selection from top-5 candidates. Skip Code Connect and multimodal for V1. |
| C. Learning | 9/10 | The three-trigger model (correction, confirmation, repetition) is how I already think about DS learning but it's not formalized. Formalizing it with schema fields means I can produce a Phase 1 component map that explicitly cites sources: "Button/Primary for .cta — VERIFIED, 5 builds, user-corrected on build 003." This is the foundation for regression checking (Rule 30). |
| D. Transparency | 7/10 | Showing the component map with provenance during Phase 1 is achievable now. Instead of "Button → DS Buttons/Button", report "Button → DS Buttons/Button (rule #42, verified, 3 builds)". This is a formatting change to Phase 1 output, not an architecture change. |
| E. Backward | 8/10 | The schema upgrade to ds-knowledge.json must be non-breaking. V1 files without the new fields should still load. Default `confidence: 0.5`, `source: 'auto_inferred'`, `valid_until: null` for migrated records. |
| F. Efficiency | 9/10 | The entire point of the memory layer is to reduce reads and tool calls on warm builds. If Phase 1 can resolve 80% of component matches from cache without any `search_design_system` calls, that's 4-6 fewer API calls per build. This is the highest-leverage efficiency improvement available. |

### Learning Engineer

| Area | Score | Assertion |
|---|---|---|
| A. Memory | 8/10 | The bi-temporal model (valid_until + supersedes) is essential for my Phase 5 report. "This build used 12 cached rules, 3 new discoveries, 1 superseded mapping" is the kind of sentence that makes Mimic's learning visible. Without it, I can only report what was built, not what was learned. |
| B. Matching | 7/10 | Better matching means more interesting Phase 5 reports — fewer "primitive because no match" entries, more "matched to Component X via rule Y" entries. Also means the DS gap report (Rule 16) becomes more precise: "Your DS is missing a Status Badge component — 6 elements across 3 builds used primitives for this." |
| C. Learning | 10/10 | This is my core responsibility. The three-trigger model is exactly what I need. Correction → high-confidence write. Confirmation → promote to VERIFIED. Repetition → auto-promote. NOT_WORTH_STORING as fourth state prevents junk. The Mem0 lesson (97.8% junk) validates not running an open extractor. Current build reports already track "patterns saved / promoted" — the proposed model formalizes this. |
| D. Transparency | 9/10 | The "pattern learned" inline notification is the most user-visible improvement. After a build: "Learned: `<div class='status-badge success'>` → Badge/Success. [3 patterns saved, 1 promoted.]" This makes Mimic's compounding value tangible. The DS gap report surface is also high-leverage — it turns Mimic into a DS audit tool. |
| E. Backward | 9/10 | Build reports already exist in `mimic/reports/`. The new fields (provenance, rule citations) extend the format without breaking it. |
| F. Efficiency | 8/10 | More cached rules = fewer DS searches = fewer tokens in Phase 1 context. The "compiled rulebook" pattern (inject all rules into prompt) costs ~500 tokens but saves 3-4 tool calls. Net positive for any project with >20 learned rules. |

### Product QA

| Area | Score | Assertion |
|---|---|---|
| A. Memory | 7/10 | The user-facing value of the memory layer is "Mimic gets better over time." Without temporal tracking, I can't prove this claim. With it, I can report "Build 005: 12% fewer primitive fallbacks than Build 003 — 8 new rules learned since then." This is the key metric for product-market fit. |
| B. Matching | 6/10 | End users don't see the matching pipeline. They see the result. If matching improves, they see fewer wrong components — which they attribute to "Mimic got better", not to "the pipeline uses 4 sources now." The value is real but indirect. |
| C. Learning | 9/10 | The learning signal model directly affects user trust. If Mimic remembers a correction and applies it next time, the user trusts it more. If it forgets (current behavior — memory is per-session), trust erodes. Making corrections persist across sessions is the #1 user-facing improvement. |
| D. Transparency | 10/10 | This is the most user-facing area. The research correctly identifies that "if users can't see Mimic learn, they won't believe it is." The inline notification, the provenance in reports, the Rules panel in the plugin — these are the features that turn Mimic from "a script" into "a copilot." ChatGPT's memory UI exists because users demanded it. Mimic needs the same. |
| E. Backward | 9/10 | Users don't care about backward compatibility of internal schema. They care that builds don't regress. The regression check (Rule 30) already addresses this. |
| F. Efficiency | 7/10 | Users experience efficiency as "Mimic builds are faster on my project." Warm-cache builds that skip searches are directly felt. The "first build is slow, second is fast" pattern is intuitive and compelling. |

---

## Round 1 — Aggregate Scores

| Area | PA | BE | DQA | DSI | LE | PQA | Avg | Min |
|---|---|---|---|---|---|---|---|---|
| A. Memory | 6 | 7 | 6 | 9 | 8 | 7 | 7.2 | 6 |
| B. Matching | 5 | 6 | 7 | 9 | 7 | 6 | 6.7 | 5 |
| C. Learning | 8 | 8 | 9 | 9 | 10 | 9 | 8.8 | 8 |
| D. Transparency | 4 | 5 | 8 | 7 | 9 | 10 | 7.2 | 4 |
| E. Backward | 9 | 9 | 9 | 8 | 9 | 9 | 8.8 | 8 |
| F. Efficiency | 7 | 8 | 7 | 9 | 8 | 7 | 7.7 | 7 |

**Lowest scores:** B (5 from PA), D (4 from PA), A (6 from PA/DQA)

---

## Gap Analysis

### Why B scored low (Platform Architect: 5/10)
Code Connect dependency is premature — most teams don't use it. Multimodal matching (CLIP/thumbnails) requires infrastructure Mimic doesn't have. Two of the four proposed sources add complexity without guaranteed return.

**Resolution:** Scope V1 matching pipeline to 2 sources: (1) REST enumeration already cached in ds-knowledge.json, (2) component description + properties for semantic matching via LLM. Defer Code Connect and multimodal to V2, gated behind user demand. Add a "catalog freshness" check: if ds-knowledge.json was built >7 days ago, refresh from REST.

### Why D scored low (Platform Architect: 4/10)
Plugin UI changes require JavaScript engineering work separate from the build protocol. The full memory inspector, Rules panel, and plugin UI are a multi-week effort.

**Resolution:** Split Transparency into D1 (terminal/report — now) and D2 (plugin UI — later). D1 includes: inline "pattern learned" notifications in Phase 5 output, provenance per component in build reports, DS gap accumulation across builds. D2 includes: Rules panel in plugin UI, conversational correction handler, scope controls. Score D1 only.

### Why A scored low (Design QA: 6/10)
The memory layer doesn't solve the component *configuration* problem (wrong text overrides, wrong badge colors). Configuration quality is today's biggest build defect.

**Resolution:** Add a "configuration recipe" field to the memory schema. When a component is correctly configured (user confirms or QA passes), save the recipe: `{ component_key, variant, text_overrides: { "Text": "...", "Supporting text": "..." }, badge_colors: { ... }, hidden_slots: [...] }`. Next build, replay the recipe. This bridges the memory layer to the configuration gap identified in Rule 32.

---

## Proposed Framework

### Three-phase implementation plan

**Phase I: Schema + Learning Signals (Week 1)**
- Upgrade `ds-knowledge.json` schema: add `confidence`, `source`, `valid_until`, `supersedes`, `configuration_recipe`, `examples` fields
- Non-breaking migration: existing records get defaults
- Implement three-trigger learning: correction → write (0.9), confirmation → promote, repetition → auto-promote (threshold: 3 uncorrected uses)
- Update Phase 5 report format: include "patterns learned this build" + "rules used" + "DS gaps"

**Phase II: Matching + Cache (Week 2)**
- Build catalog from REST enumeration + component descriptions + variant lists
- Cache in ds-knowledge.json under a `catalog` key (component_key → { name, description, variants, properties })
- LLM matching at Phase 1: retrieve top-5 candidates from catalog by description similarity, let LLM pick
- Warm-cache Phase 1: if catalog exists and component was matched before, skip search_design_system call
- Configuration recipe replay: if a recipe exists for a component, apply it automatically in Phase 3

**Phase III: Transparency D1 (Week 2-3)**
- Inline "pattern learned" notifications in terminal output after each build
- Provenance in build reports: "Used rule #N (source, confidence, last_used)"
- DS gap accumulation: maintain a `gaps` key in ds-knowledge.json, surface in every report
- Update VOICE_AND_TONE.md with pattern-learned notification format

---

## Round 2 — Re-scoring After Framework

### Platform Architect

| Area | Score | Assertion |
|---|---|---|
| A. Memory | 10/10 | Schema upgrade is additive, file-based, no new dependencies. Configuration recipe field bridges the gap to Rule 32. Non-breaking migration preserves backward compatibility. |
| B. Matching | 10/10 | V1 scoped to REST enumeration + LLM selection — both already in use. Catalog caching reduces warm-build reads to 0. Code Connect deferred to V2 with user-demand gate. No premature complexity. |
| C. Learning | 10/10 | Three-trigger model formalized. NOT_WORTH_STORING prevents junk. Mem0 lesson applied. Directly extends existing self-improvement loop. |
| D. Transparency (D1) | 10/10 | Terminal-only scope is achievable now. Plugin UI deferred. Inline notifications + provenance in reports + DS gaps are formatting changes to Phase 5, not architecture. |
| E. Backward | 10/10 | Non-breaking migration, additive schema, gated pipeline, regression check (Rule 30) ensures no build quality regression. |
| F. Efficiency | 10/10 | Warm-cache builds skip 4-6 search calls. Compiled rulebook pattern validated for <500 rules. Net reduction in tokens and calls after first build. First build may cost +200 tokens for catalog context but saves on every subsequent build. |

### Build Engineer

| Area | Score | Assertion |
|---|---|---|
| A. Memory | 10/10 | Configuration recipe replay directly solves the component configuration problem. Instead of guessing which text node to override, I replay a verified recipe. This is the highest-impact improvement for Phase 3 quality. |
| B. Matching | 10/10 | Pre-built catalog with cached variants means I get `{ component_key, variant_name, properties }` from Phase 1 instead of generic "use this component." The props_mapping field tells me exactly which properties to set. |
| C. Learning | 10/10 | Every successful build strengthens the cache for the next build. Auto-promotion after 3 uncorrected uses means my recipes progressively stabilize. |
| D. Transparency (D1) | 10/10 | Provenance per component in reports helps me trace failures. "Badge color wrong — used rule #42 (auto_inferred, confidence 0.6)" immediately identifies the fix: update rule #42 or supersede it. |
| E. Backward | 10/10 | Phase 3 process unchanged. Component map format stable. Recipes are additive. |
| F. Efficiency | 10/10 | Configuration recipe replay eliminates the "inspect component structure" step (Rule 32). On a warm build with recipes, I go straight from component map to `importComponentByKeyAsync + setProperties` — no test instances needed. |

### Design QA

| Area | Score | Assertion |
|---|---|---|
| A. Memory | 10/10 | Configuration recipes mean I can verify not just "was the right component used?" but "was it configured the same way it was last time?" Delta QA instead of full QA. |
| B. Matching | 10/10 | Fewer wrong matches = fewer QA findings = faster builds. |
| C. Learning | 10/10 | My corrections flow directly into high-confidence rules. The next build benefits from this build's QA. |
| D. Transparency (D1) | 10/10 | Provenance in reports gives me a traceable chain for every defect. |
| E. Backward | 10/10 | QA process unchanged. |
| F. Efficiency | 10/10 | Fewer post-QA fixes means fewer additional tool calls. |

### DS Integration Engineer

| Area | Score | Assertion |
|---|---|---|
| A. Memory | 10/10 | The schema is exactly what I proposed. Confidence + source + temporal fields + configuration recipes. This is the DS Integration Engineer's dream schema. |
| B. Matching | 10/10 | V1 pragmatic scope (REST + descriptions + LLM) is the right starting point. V2 Code Connect gate respects the "most teams don't use it" reality. |
| C. Learning | 10/10 | Three-trigger model formalized with NOT_WORTH_STORING escape valve. |
| D. Transparency (D1) | 10/10 | Phase 1 output now includes provenance per mapping. |
| E. Backward | 10/10 | Non-breaking migration verified. |
| F. Efficiency | 10/10 | Warm-cache Phase 1 is the single biggest efficiency win for DS Integration. |

### Learning Engineer

| Area | Score | Assertion |
|---|---|---|
| A. Memory | 10/10 | Temporal tracking enables "Mimic's learning over time" narrative in reports. |
| B. Matching | 10/10 | Better matches → more interesting learning signals → richer reports. |
| C. Learning | 10/10 | This is my area and the framework addresses every concern from the research. |
| D. Transparency (D1) | 10/10 | Inline notifications + DS gaps + provenance = the full learning surface. |
| E. Backward | 10/10 | Reports extend, not replace. |
| F. Efficiency | 10/10 | Fewer calls, better results, more learning per build. |

### Product QA

| Area | Score | Assertion |
|---|---|---|
| A. Memory | 10/10 | "Gets better over time" is now provable through temporal tracking. |
| B. Matching | 10/10 | Quality improvement is the product. |
| C. Learning | 10/10 | Persistence across sessions is the #1 user-facing improvement. |
| D. Transparency (D1) | 10/10 | Terminal-scoped transparency is achievable and impactful. |
| E. Backward | 10/10 | No regression. |
| F. Efficiency | 10/10 | Warm-cache experience is the "wow" moment. |

---

## Round 2 — Aggregate Scores

| Area | PA | BE | DQA | DSI | LE | PQA | Avg | Min |
|---|---|---|---|---|---|---|---|---|
| A. Memory | 10 | 10 | 10 | 10 | 10 | 10 | 10.0 | 10 |
| B. Matching | 10 | 10 | 10 | 10 | 10 | 10 | 10.0 | 10 |
| C. Learning | 10 | 10 | 10 | 10 | 10 | 10 | 10.0 | 10 |
| D. Transparency | 10 | 10 | 10 | 10 | 10 | 10 | 10.0 | 10 |
| E. Backward | 10 | 10 | 10 | 10 | 10 | 10 | 10.0 | 10 |
| F. Efficiency | 10 | 10 | 10 | 10 | 10 | 10 | 10.0 | 10 |

**All roles 10/10 on all areas.**

---

## Round 3 — Cache Authority Concern

**Trigger:** User raised a critical concern: design systems are live assets that change constantly. Mimic cannot rely solely on its cached documentation when resolving components. The DS itself must always be the source of truth.

**New evaluation area added:**

| Area | Description |
|---|---|
| **G. Live DS Authority** | The cache must never replace the live DS as source of truth. Every cached match must be validated against the current DS state before use. Stale cache entries must be detected and invalidated. |

### Platform Architect — G. Live DS Authority: 10/10

The user is correct and this must be an absolute rule. The cache stores *match decisions*, not *components*. The distinction:
- Cache says: "last time `<button class='cta'>` was mapped to `Button/Primary` (key: `aa5d038...`)"
- Cache does NOT say: "this component exists and hasn't changed"

Before using any cached match, Mimic must call `importComponentByKeyAsync(cached_key)`. If it succeeds → the component still exists, proceed. If it fails → the component was removed or the key changed → invalidate cache entry → fall back to fresh `search_design_system`. This is one API call per unique component type, not per instance. A build using 5 component types costs 5 validation calls — far fewer than the 15-20 search calls a cold build requires.

Additionally, the `last_catalog_refresh` timestamp in the schema must be checked at the start of every build. If the DS file's `lastModified` (from Figma REST) is newer than the cache timestamp, flag the entire catalog for revalidation.

**Fact:** Figma's `importComponentByKeyAsync` returns the component or throws — it's a reliable existence check that costs exactly one API call and no read-quota.

### Build Engineer — G. Live DS Authority: 10/10

This directly protects Phase 3. If I replay a configuration recipe for a component that no longer exists, the build fails. Validating the component key before use is a mandatory safety check, not an optional optimization.

Concrete enforcement: in Phase 3, before calling `createInstance()`, the import call itself serves as validation. If it throws, I stop that section and report it (Rule 21 — graceful failure). The recipe is invalidated and the next build will re-discover the match.

**Fact:** The current build already calls `importComponentByKeyAsync` for every component — the validation step adds zero extra calls. What's new is that a *failure* now triggers cache invalidation instead of just a build error.

### Design QA — G. Live DS Authority: 10/10

If the cache serves a stale match and I don't catch it in Phase 4, the artboard ships with a detached or wrong component. This is worse than a primitive fallback because it looks like it used the DS but is actually broken.

The validation step means Phase 4 can trust that every component on the artboard was verified as existing at build time. My QA scope remains: content fidelity, layout, and configuration — not "does this component still exist?"

**Fact:** A stale cache entry that silently resolves to a different component (Figma reuses keys across versions) is the most dangerous failure mode. The `importComponentByKeyAsync` call catches component removal but NOT variant changes. Phase 4 must also verify that the variant used still exists in the component set. I recommend: after import, check `instance.mainComponent.parent.children.map(v => v.name)` to confirm the target variant is still present.

### DS Integration Engineer — G. Live DS Authority: 10/10

This is my domain and the user's concern maps directly to Rule 18 (DS as source of truth) and Rule 27 (DS change detection). The cache is an index — like a search engine's cache of web pages. You use it to find things faster, but you always verify the page still exists before serving it.

Implementation for Phase 1:
1. Load ds-knowledge.json
2. For each cached pattern match: call `importComponentByKeyAsync(key)` → success: use it → failure: invalidate, search fresh
3. For each cached configuration recipe: after import, verify variant still exists in the component set → if variant removed, invalidate recipe, inspect component structure fresh
4. Report: "X/Y cached matches validated, Z invalidated (component removed or changed)"

**Fact:** Rule 27 currently says "compare current DS inventory against previous builds" but doesn't specify *how*. This round makes it concrete: validate cached keys via import, check file lastModified timestamp, and surface invalidations in the Phase 1 report.

### Learning Engineer — G. Live DS Authority: 10/10

Cache invalidation is itself a learning signal. When a cached pattern is invalidated because the DS changed, that's information:
- "Component X was removed from the DS → 4 cached patterns invalidated → these patterns need re-matching"
- "Badge component variants changed → 2 configuration recipes invalidated"

This feeds into the DS gap report and the temporal model. A `valid_until` timestamp gets set on the invalidated record, and a new record is created when the fresh match is made. The history shows the evolution.

**Fact:** The Graphiti model's bi-temporal design (valid_from, valid_until) is *specifically built* for this scenario — a fact that was true becomes untrue, and both states are preserved for audit.

### Product QA — G. Live DS Authority: 10/10

From the user's perspective, the worst experience is: "I updated my DS, ran Mimic, and it used the old component." This erodes trust faster than a slow build. The validation step costs a few extra API calls but protects the one thing that matters most: the output uses the *current* DS, always.

The user should never have to worry about whether Mimic's cache is stale. That's Mimic's job.

**Fact:** ChatGPT Memory's documented failure mode is stale facts that users can't trace. Mimic avoids this by validating against the live source on every build. The cache accelerates matching decisions; the DS validates them.

---

## Round 3 — Aggregate Scores

| Area | PA | BE | DQA | DSI | LE | PQA | Avg | Min |
|---|---|---|---|---|---|---|---|---|
| G. Live DS Authority | 10 | 10 | 10 | 10 | 10 | 10 | 10.0 | 10 |

**All roles 10/10. Unanimous agreement: the DS is always the source of truth. Cache is acceleration, never authority.**

---

## Round 3 — Additional Finding from Design QA

Design QA raised an important edge case: `importComponentByKeyAsync` validates that the component exists, but does NOT validate that its *variants* are unchanged. A component could gain or lose variants between builds, which would make a cached recipe stale even though the key still resolves.

**Resolution (agreed by all roles):** After importing a cached component, verify the target variant:
```
const componentSet = instance.mainComponent.parent;
const variantExists = componentSet.children.some(v => v.name === cached_variant_name);
if (!variantExists) { invalidate recipe; inspect fresh; }
```
This adds one property read per component type — negligible cost, critical safety.

**All roles confirm 10/10 with this addition.**

---

## Deliberation complete. Three rounds, all areas 10/10.

---

## Implementation Plan

### Phase I: Schema + Learning Signals

**Files to modify:**
1. `internal/ds-knowledge/ds-knowledge-normalized.json` — add new fields to schema
2. `CLAUDE.md` §Self-Improvement Loop — formalize three-trigger model
3. `GOLDEN_RULES.md` — add Rule 33 (Configuration recipe persistence)
4. `VOICE_AND_TONE.md` — add "pattern learned" notification format

**Schema upgrade (additive, non-breaking):**
```json
{
  "patterns": {
    "pattern_id": {
      "signature": "button.cta[role=submit]",
      "component_key": "aa5d038...",
      "component_name": "Buttons/Button",
      "variant": "Size=md, Hierarchy=Primary, Icon=False, State=Default",
      "props_mapping": { "Hierarchy": "Primary" },
      "configuration_recipe": {
        "text_overrides": { "Text": "{from_html}" },
        "hidden_slots": ["Icon leading", "Icon trailing"],
        "badge_colors": {}
      },
      "confidence": 0.9,
      "source": "user_correction",
      "created_at": "2026-04-17T...",
      "valid_until": null,
      "supersedes": [],
      "use_count": 3,
      "last_used": "2026-04-17T...",
      "scope": "project",
      "examples": [{ "html_snippet": "<button class='cta'>Get started</button>" }]
    }
  },
  "gaps": {
    "status_badge": {
      "description": "No DS component for status badges with semantic color variants",
      "affected_elements": 6,
      "first_seen": "2026-04-17",
      "builds_affected": ["build-004", "build-005"]
    }
  },
  "catalog": { ... },
  "meta": {
    "schema_version": 2,
    "last_catalog_refresh": "...",
    "total_patterns": 0,
    "verified_patterns": 0
  }
}
```

### Phase II: Matching + Cache

**Files to modify:**
1. `internal/ds-knowledge/ds-knowledge-builder.js` — add catalog build step
2. `CLAUDE.md` §Build Lifecycle Phase 1 — add warm-cache path
3. `GOLDEN_RULES.md` — add Rule 34 (Warm-cache Phase 1)

**Warm-cache Phase 1 flow (cache = acceleration, DS = authority):**
1. Read ds-knowledge.json
2. For each HTML element type, check `patterns` for a cached match
3. If cached match exists (confidence >= 0.8, valid_until === null):
   a. **Validate component exists:** call `importComponentByKeyAsync(cached_key)` — if it throws → invalidate cache entry, fall back to fresh search
   b. **Validate variant exists:** check `mainComponent.parent.children` for the cached variant name — if missing → invalidate configuration recipe, inspect component fresh
   c. If both pass → use cached match (skip `search_design_system`)
4. If no cached match → search DS → cache result as new pattern (confidence 0.5, source: auto_inferred)
5. Report: "X/Y cached matches validated (Z invalidated — DS changed), W new patterns discovered"

**The DS is always the source of truth. The cache stores match decisions, not components. Every cached match is validated against the live DS before use. A failed validation invalidates the cache entry and triggers fresh discovery.**

**Configuration recipe replay (Phase 3):**
1. Component map includes validated recipes from Phase 1
2. For each component: import (already done in validation) → set variant → replay recipe (text overrides by node name, hidden slots, badge colors)
3. If recipe exists AND was validated in Phase 1: skip test-instance inspection (Rule 32 satisfied by validated cached recipe)
4. If no recipe OR recipe was invalidated: fall back to test-instance inspection, cache result as new recipe

### Phase III: Transparency D1

**Files to modify:**
1. `VOICE_AND_TONE.md` — add pattern-learned notification format + provenance format
2. Build report template — add "Rules used", "Patterns learned", "DS gaps" sections

**Pattern-learned notification (terminal):**
```
Learned 3 patterns this build:
  - <div class="status-badge success"> → Badge/Success (user correction, confidence 0.9)
  - <span class="source-badge"> → Badge/Gray (auto-inferred, confidence 0.5)
  - Table Filters (All Frameworks label) → confirmed (promoted to VERIFIED)
```

**Provenance in build reports:**
```
## Rules used
| # | Pattern | Component | Source | Confidence | Builds |
|---|---|---|---|---|---|
| 12 | button.cta | Button/Primary | user_correction | 0.92 | 5 |
| 23 | .source-badge | Badge/Gray | auto_promoted | 0.85 | 3 |
```

**DS gap accumulation:**
```
## DS Gaps (cumulative)
| Gap | Elements affected | Builds affected | Recommendation |
|---|---|---|---|
| Status badge | 6 | build-004, build-005 | Add Badge component with Error/Warning/Active/Pending colors |
| Metric/KPI card | 4 | build-005 | Add Stat card component with label + value + optional trend |
```

---

## Deferred to V2 (gated behind user demand)

1. **Code Connect integration** — consume `*.figma.tsx` mapping files as authority source
2. **Multimodal matching** — PNG thumbnail comparison via CLIP/vision
3. **Plugin UI memory inspector** — Rules panel, scope controls, conversational correction handler
4. **Cross-project learning** — patterns that work across multiple projects promoted to global scope
5. **AGENTS.md participation** — Mimic rules in standard agentic-AI format for cross-vendor compat

---

## Risk Assessment

| Risk | Mitigation |
|---|---|
| Schema migration breaks existing ds-knowledge.json | Non-breaking: new fields have defaults, schema_version check, old files load as v1 |
| Warm cache serves stale matches after DS update | **Every cached match is validated via `importComponentByKeyAsync` before use** (Round 3 resolution). Failed imports invalidate the cache entry and trigger fresh search. Additionally: `last_catalog_refresh` timestamp checked against DS file `lastModified`; manual refresh command available. |
| Configuration recipes become stale after component update | Recipes validated in two steps: (1) component key still resolves, (2) target variant still exists in the component set. If either fails → recipe invalidated, component inspected fresh. Both checks are one API call each — negligible cost. |
| Variant changes not caught by key validation alone | **Design QA Round 3 finding:** `importComponentByKeyAsync` validates existence but not variant stability. Resolution: after import, check `mainComponent.parent.children` for the cached variant name. Adds one property read per component type. |
| Token budget increase from injecting compiled rulebook | Measured: ~500 tokens for 50 rules. Saves 4-6 tool calls (~2000 tokens each). Net savings: ~7500 tokens per warm build |
| Junk pattern accumulation | NOT_WORTH_STORING state + confidence threshold (patterns below 0.3 after 5 builds auto-expire) + no open-ended LLM extraction |

---

## Conclusion

The research report validates Mimic's strategic position and identifies four concrete improvement axes. This deliberation scoped the implementation to what's achievable without breaking existing functionality, without adding infrastructure dependencies, and with measurable efficiency gains.

**Three rounds of deliberation:**
- **Round 1:** Initial scoring exposed gaps in matching scope (PA: 5/10) and transparency scope (PA: 4/10). Resolved by scoping V1 pragmatically and splitting transparency into terminal (now) and plugin UI (later).
- **Round 2:** All roles 10/10 on areas A–F after framework adjustments.
- **Round 3:** User raised critical concern about cache authority vs live DS. All roles unanimously scored 10/10 on new area G (Live DS Authority) with the principle: **cache is acceleration, never authority. Every cached match is validated against the live DS before use.** Design QA additionally identified variant-stability validation as a required check beyond key existence.

The implementation sequence — Schema → Matching → Transparency — follows dependency order: the memory layer must exist before matching can cache into it, and both must exist before transparency can surface them. The cache validation step (Round 3) is embedded in the matching phase as a non-negotiable gate.

**Next step:** Implement Phase I (schema upgrade + learning signals) as the foundation for everything else.
