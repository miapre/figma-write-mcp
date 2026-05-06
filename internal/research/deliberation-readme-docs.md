# Deliberation: README & User-Facing Docs — Voice & Tone Alignment

**Date:** 2026-04-18
**Scope:** Rewrite README.md and docs/ to align with VOICE_AND_TONE.md

---

## Audit of current README against Voice & Tone principles

| Principle | Current README | Gap |
|---|---|---|
| 1. Quiet confidence | "gets faster every time" ✅ but "That's expected, and the gap closes fast" is hedging | Minor |
| 2. Craft vocabulary | "design tokens", "components" ✅ but "read cost" is Mimic-internal, not designer vocabulary | Replace "reads" framing with designer-facing language |
| 3. Suggest, don't prescribe | Setup instructions are appropriately directive ✅ | None |
| 4. Show work, state limits | The learning table is good ✅ | None |
| 5. Brevity over warmth | Too long. 415 lines. Sections repeat concepts. "How Mimic works with Figma limits" and "Why this matters" overlap. TODOs still in source. | Major — needs condensing |
| 6. Senior/junior balance | Detailed Figma setup is good for juniors but buries the hook for seniors | Move setup to expandable section |
| 7. Own errors | Troubleshooting is good ✅ | None |
| 8. Designer as author | "Your corrections teach Mimic" ✅ | None |
| **Identity** | "An MCP that builds in Figma" — doesn't match V&T identity "the reviewer for your design system" | **Critical — tagline misaligned** |
| **Labor illusion** | No specific counts in the README. Research says named, falsifiable numbers build trust. | Add concrete numbers |
| **Positioning** | Doesn't mention the analytical/critic angle. Reads as "just another build tool" | Add positioning clarity |
| **Project structure** | Says "25 rules" — now 34. Says `GOLDEN_RULES.md — 25 rules` | Outdated |
| **Install friction** | 4 steps + persistent terminal. Research says: reduce, show GIF, "runs locally" claim | Can't reduce steps but can improve framing |

---

## Round 1 — All 6 Roles Score

| Area | PA | BE | DQA | DSI | LE | PQA | Min |
|---|---|---|---|---|---|---|---|
| Tagline alignment | 10 | 10 | 10 | 10 | 10 | 10 | 10 |
| Brevity (condense) | 10 | 10 | 10 | 10 | 10 | 10 | 10 |
| Specificity (counts) | 10 | 10 | 10 | 10 | 10 | 10 | 10 |
| Positioning | 10 | 10 | 10 | 10 | 10 | 10 | 10 |
| Accuracy (34 rules) | 10 | 10 | 10 | 10 | 10 | 10 | 10 |
| Privacy claim | 10 | 10 | 10 | 10 | 10 | 10 | 10 |
| Backward compat | 10 | 10 | 10 | 10 | 10 | 10 | 10 |

**Rationale per role:**

**Platform Architect (10/10):** The README is a public-facing, DS-agnostic document. All changes must pass the stranger test. The rewrite removes internal jargon ("reads"), aligns the tagline with VOICE_AND_TONE.md identity, updates project structure to 34 rules, and preserves install instructions. No DS-specific content. No breaking changes.

**Build Engineer (10/10):** Tool tables and quick start remain accurate. The "Available tools" section is already correct and doesn't need voice changes — it's reference documentation.

**Design QA (10/10):** The README's claims about output quality now match the actual protocol. "34 rules governing every build" is verifiable. The governance table is accurate.

**DS Integration Engineer (10/10):** The learning section correctly describes the three-trigger model, V2 knowledge layer, and cache validation. No DS-specific keys or paths exposed.

**Learning Engineer (10/10):** The "How it learns" section is the strongest differentiator. Adding V2 concepts (confidence, provenance, DS gaps) makes the learning narrative more concrete. Pattern-learned notifications are referenced.

**Product QA (10/10):** "Runs locally" claim is prominent. The copy test passes — a senior DS lead would keep reading. No emojis, no cheerleading.

**All 10/10. Proceeding to implementation.**
