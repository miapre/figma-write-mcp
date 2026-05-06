# Deliberation: Voice, Tone, Progress UX & Positioning

**Date:** 2026-04-17
**Source:** `internal/research/voice-and-tone.md`
**Scope:** Evaluate research findings → implementation plan → score until 10/10

---

## Evaluation Areas

| Area | Description |
|---|---|
| **A. Progress UX** | Ship progress as markdown text blocks in tool results; dual-emit with notifications/progress; never depend on logging |
| **B. Voice & Tone** | Rewrite VOICE_AND_TONE.md to align with Linear-style brevity, verb-first craft vocabulary, Buell & Norton labor illusion |
| **C. Recommendation Framing** | Questions + evidence, not commands. Categorical confidence (strong/moderate/weak), one-click dismiss, cite evidence counts |
| **D. Status Message Specificity** | Named, falsifiable labor in all progress output. "Scanning 1,284 layers" not "Loading..." |
| **E. Positioning** | Analytical/critic MCP, not generator. Brad Frost "DS+AI" framing. Complementary to Anima/Make/Storybook |
| **F. Backward Compatibility** | Don't break existing build protocol, tool results, or knowledge system |
| **G. Token Efficiency** | Status blocks add tokens to tool results — must stay within 10k char limit, net positive for user comprehension |

---

## Round 1 — Scoring

### Platform Architect

| Area | Score | Assertion |
|---|---|---|
| A | 9/10 | Research proves Claude Code drops progress notifications (issue #4157). Markdown in tool results is the only reliable channel. The dual-emit pattern (notifications + result text) costs nothing — emit both, the capable client renders whichever works. But: tool result size limits (Claude Code 10k default) mean status blocks must be compact. Cap at ~500 chars. |
| B | 10/10 | We already created VOICE_AND_TONE.md this session. The research provides the authoritative benchmarks (Linear, Polaris, Copilot-too-polite) to strengthen it. The eight principles from the research map directly to our existing document structure. This is a rewrite, not a rearchitecture. |
| C | 9/10 | "Questions + evidence" is a report format change. The DS gap section in VOICE_AND_TONE.md already exists — upgrade it from declarative statements to questions. Categorical confidence is implementable now (replace numeric 0.9 with "Strong pattern (12 uses)"). One-click dismiss requires plugin UI (deferred to V2). Terminal equivalent: "dismiss" as a conversational command. |
| D | 10/10 | This is the load-bearing finding (Buell & Norton). Every `mimic_status` response and every Phase completion message should include specific counts. "Phase 1 complete: 8 DS components matched, 2 primitive fallbacks, 0 searches skipped (cold build)" — not "Phase 1 complete." The change is in message formatting, not architecture. |
| E | 8/10 | Positioning is a README + marketing concern, not a code change. But it shapes how we write tool descriptions, build reports, and error messages. The "reviewer for your design system" framing should be reflected in `mimic_status` output and report headers. Low risk, high clarity. |
| F | 10/10 | All changes are additive — rewording messages, adding counts to existing outputs. No API changes, no schema changes, no new dependencies. |
| G | 9/10 | A 500-char status block in tool results adds ~125 tokens. Build reports already cost 500+ tokens. The net effect is positive because specific status reduces user follow-up questions (which cost 1000+ tokens each). Cap status blocks strictly. |

### Build Engineer

| Area | Score | Assertion |
|---|---|---|
| A | 10/10 | Tool result text is what I already return. Adding a markdown status block to my Phase 3 output means the user sees progress without polling. Zero extra tool calls. |
| B | 10/10 | Voice changes don't affect build execution. |
| C | 9/10 | Recommendations in build reports are Phase 5 (Learning Engineer's domain). My only concern: if recommendation questions are verbose, they add to report length. Keep each to one line. |
| D | 10/10 | I can count frames created, components inserted, and recipes replayed during Phase 3. Surfacing these counts is trivial: "Built 47 frames, 12 DS component instances, 3 recipes replayed." |
| E | 10/10 | Positioning doesn't affect build code. |
| F | 10/10 | No build process changes. |
| G | 10/10 | Counts are already tracked internally. Formatting them into a status string costs zero extra computation. |

### Design QA

| Area | Score | Assertion |
|---|---|---|
| A | 10/10 | Status blocks in tool results mean I can see what was built without taking a screenshot first. "47 frames, 12 components, 3 text style mismatches" tells me where to focus QA. |
| B | 10/10 | Voice changes in error messages help me understand failures faster. "Couldn't parse line 42" beats "Something went wrong." |
| C | 10/10 | Questions + evidence is how I already think about QA findings. "Is this Badge/Gray intentional? (HTML shows status='success', expected Badge/Success)" — actionable, verifiable. |
| D | 10/10 | Specific counts help me verify. "12 DS component instances" — I can count them in the artboard. |
| E | 10/10 | The "critic/reviewer" framing matches QA's role perfectly. |
| F | 10/10 | No QA process changes. |
| G | 10/10 | More information in fewer messages = fewer QA back-and-forth cycles. |

### DS Integration Engineer

| Area | Score | Assertion |
|---|---|---|
| A | 10/10 | Phase 1 discovery is the longest step. A status block showing "Searched: buttons ✅, tabs ✅, badges ✅, pagination ⏳" gives the user live visibility into what I'm doing. |
| B | 10/10 | Craft vocabulary ("token", "variant", "semantic role") is how I already communicate. The research validates this and provides concrete phrasings. |
| C | 10/10 | DS gap recommendations as questions is the right frame: "Should your DS include a Status Badge component? (6 elements across 3 builds used primitives for this.)" |
| D | 10/10 | "Matched 42 components from cache (3 invalidated — Badge component updated)" is the kind of falsifiable labor that builds trust. |
| E | 10/10 | The analytical positioning is my entire job. |
| F | 10/10 | No discovery process changes. |
| G | 10/10 | Cache status reporting adds ~100 tokens but eliminates "how many searches did you do?" follow-ups. |

### Learning Engineer

| Area | Score | Assertion |
|---|---|---|
| A | 10/10 | Pattern-learned notifications (already in VOICE_AND_TONE.md) are the progress equivalent for Phase 5. The research validates this with Buell & Norton's labor illusion. |
| B | 10/10 | The "Linear, not Copilot" principle is already in VOICE_AND_TONE.md. The research provides the benchmark copy to strengthen it. |
| C | 10/10 | "Questions + evidence" is how recommendations should be phrased. The current format says "Add Badge component." The new format: "Should your DS include a Status Badge? (6 elements, 3 builds, all primitives.)" This is a one-line template change in VOICE_AND_TONE.md. |
| D | 10/10 | The specificity principle ("named, falsifiable labor") is the single most important finding for build reports. Every Phase 5 report should lead with specific counts. |
| E | 10/10 | The "compounds over time" narrative is already our thesis. The research says to make it *felt*: "Build 005: 12 patterns from cache. Build 001 had 0. Your DS knowledge grew by 12 rules in 4 builds." |
| F | 10/10 | Report format changes are additive. |
| G | 10/10 | Specific counts replace vague descriptions — same token count, more information. |

### Product QA

| Area | Score | Assertion |
|---|---|---|
| A | 10/10 | The user sees progress in the tool result. This is the #1 UX improvement possible without client-side changes. |
| B | 10/10 | The "too polite to be useful" finding is decisive. Our existing VOICE_AND_TONE.md already avoids this, but the research provides concrete do/don't phrasings to add. |
| C | 10/10 | Questions + evidence matches the HAX Guidelines (G11 "make clear why"). One-click dismiss deferred to plugin UI. Terminal dismiss via conversation is sufficient for V1. |
| D | 10/10 | Buell & Norton is the load-bearing finding. Every message that says "Loading..." should say what is being loaded and how much. |
| E | 10/10 | "The reviewer for your design system" is a market position no one occupies. It should be in the tool's description field. |
| F | 10/10 | No user-facing breaking changes. |
| G | 10/10 | Specific messages reduce user anxiety and follow-up questions. Net token savings. |

---

## Round 1 — Aggregate

| Area | PA | BE | DQA | DSI | LE | PQA | Min |
|---|---|---|---|---|---|---|---|
| A | 9 | 10 | 10 | 10 | 10 | 10 | 9 |
| B | 10 | 10 | 10 | 10 | 10 | 10 | 10 |
| C | 9 | 9 | 10 | 10 | 10 | 10 | 9 |
| D | 10 | 10 | 10 | 10 | 10 | 10 | 10 |
| E | 8 | 10 | 10 | 10 | 10 | 10 | 8 |

### Gap resolution

**A (PA: 9/10):** Concern about tool result size limits. **Resolution:** Cap status blocks at 500 chars. Use a compact format: `✅ Phase 1: 8 matched, 2 fallbacks | ✅ Phase 2: 12 styles, 16 vars | ⏳ Phase 3: building...` — fits in one line per phase. **PA raises to 10/10.**

**C (PA: 9/10, BE: 9/10):** One-click dismiss deferred to plugin UI. **Resolution:** Terminal dismiss is a conversational command: user says "dismiss that recommendation" → Mimic writes a NOT_WORTH_STORING rule. Verbal one-click equivalent. Both raise to 10/10. **Recommendation questions capped at one sentence + one parenthetical evidence clause.** BE raises to 10/10.

**E (PA: 8/10):** Positioning is marketing, not code. **Resolution:** Add the "reviewer for your design system" framing to: (1) `mimic_status` tool description, (2) build report headers, (3) VOICE_AND_TONE.md identity section. Three string changes. **PA raises to 10/10.**

---

## Round 2 — All 10/10. Deliberation complete.

---

## Implementation Plan

### 1. Rewrite VOICE_AND_TONE.md identity section

**Current:** "Mimic is a design system copilot."
**New:** "Mimic is the reviewer for your design system. It translates HTML into Figma using your components and tokens, gets smarter about your conventions over time, and tells you where your DS has gaps."

Add the eight voice principles from the research:
- Quiet confidence, not cheerleading
- Craft-first vocabulary (token, variant, spec, spacing scale, ramp)
- Suggest, don't prescribe (reserve "you should" for a11y and irreversibles)
- Show work, state limits
- Brevity over warmth (Polaris Jenga test)
- Never patronize the senior; never strand the junior
- Own errors without apologizing
- Treat the designer as the author

### 2. Add do/don't microcopy examples to VOICE_AND_TONE.md

From the research's concrete phrasings — loading, completion, suggestion, error, low confidence, critique.

### 3. Update status message format for labor illusion

Replace generic messages with specific, falsifiable ones:
- "Phase 1 complete" → "Phase 1: 8 DS components matched, 2 fallbacks (searched: buttons, tabs, badges, table cells, pagination)"
- "Building..." → "Phase 3: 47/52 frames built · 12 DS components inserted · 3 recipes replayed"
- "Build complete" → "Done. 52 frames, 12 DS components, 0 raw hex, 0 non-DS fonts. 3 patterns learned."

### 4. Upgrade recommendation format to questions + evidence

- "Add Badge component" → "Should your DS include a Status Badge? (6 elements across 3 builds used primitives — the pattern is consistent.)"
- "Add filter trigger component" → "Would a Filter Dropdown Trigger component help? (Used in 4 of 5 table screens, always built as primitive.)"

### 5. Add categorical confidence to pattern-learned notifications

- "Strong pattern (12 uses, user-verified)" instead of "confidence: 0.92"
- "New pattern (first use, auto-inferred)" instead of "confidence: 0.5"
- "Weak — verify next build" instead of "confidence: 0.3"

### 6. Update mcp.js mimic_status tool description

Add "the reviewer for your design system" to the tool's description string.

---

## Files to modify

| File | Change |
|---|---|
| `VOICE_AND_TONE.md` | Rewrite identity, add 8 principles, add do/don't microcopy, update recommendation format, add categorical confidence bands |
| `mcp.js` | Update `mimic_status` description; format status messages with specific counts |
| `internal/learning/build-learning.js` | Update report template with questions-as-recommendations, categorical confidence, specific labor counts |

---

## What we're NOT implementing (deferred)

- Plugin UI for dismiss/snooze (V2)
- .mimic/learned.md human-readable file (V2 — currently using ds-knowledge.json)
- DS health score (0-100) — controversial, needs validation
- Severity taxonomy for recommendations (V2)
- README GIF/install simplification (marketing task, not code)
- Figma Community listing (distribution task)
- Dual-emit progress notifications (low ROI until Claude Code renders them — issue #4157)
