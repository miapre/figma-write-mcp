# Mimic AI v2 — Pre-Mortem Transcript

**Date:** 2026-05-06
**Subject:** Design spec stress test before implementation planning
**Context:** v1.8.0 archived after weeks of fix-on-fix cycles. v2 spec written. Pre-mortem run to catch architectural blind spots before committing to implementation.

---

## Context

- **What:** Mimic AI v2 — MCP server + Figma plugin that translates HTML into Figma using the user's DS, learns from every build, acts as DS copilot
- **Who:** Designers who maintain design systems (not developers)
- **Success:** Designers give Mimic any HTML, get Figma output using only their DS. Learning loop makes it faster. Build reports recommend DS improvements.

---

## Raw Failure Reasons (7 identified)

1. Plugin enforcement gate too rigid for imperfect real-world DSs
2. Component matching unreliable due to inconsistent naming conventions
3. LLM orchestrator can't reliably execute 6 phases + 45 tools + 7-step component config
4. WebSocket bridge drops silently during long builds
5. Setup too complex for designer audience
6. Learning loop never reaches critical mass — users don't build enough
7. Native chart building requires math that LLMs get wrong

---

## Agent Deep-Dives

### 1. Plugin Enforcement Too Rigid

**Story:** First users have community libraries with partial token coverage. Plugin rejects every body text node and background fill. Zero output. Word spreads: "Mimic doesn't work."

**Assumption:** Target DSs are structurally complete enough to cover every token category.

**Warnings:** Zero-output first builds with 10+ errors. Support clusters around "my DS has X but not Y."

### 2. Component Matching Unreliable

**Story:** Button named "CTA/Primary" under "Interactions/Actions" — search for "button" returns wrong match. Cache compounds the error across builds. Agency designers with inherited DSs hit this hardest.

**Assumption:** Component names are predictable enough for keyword search.

**Warnings:** High confidence + high correction rate simultaneously. Agency users significantly lower satisfaction.

### 3. LLM Orchestration Collapse

**Story:** 12-section dashboard. LLM skips Style Inventory, jumps to Build. 7-step component config falls apart under working memory pressure. "Always incomplete in unpredictable ways."

**Assumption:** Tool hints substitute for state tracking, but working memory pressure defeats advisory guidance.

**Warnings:** Phase skip rate >15% on 6+ section builds. Component config completeness <70% on complex dashboards.

### 4. WebSocket Bridge Silent Failure

**Story:** WebSocket drops at 2-3 minute mark. MCP server thinks it's connected. No reconnect. Embedded bridge means no separate restart. Partial builds leave orphaned frames.

**Assumption:** WebSocket remains stable for full build duration without keepalive.

**Warnings:** Success rate drops sharply at >150 tool calls. "Restart plugin" pattern on complex builds.

### 5. First-Time Experience Too Complex

**Story:** Terminal → npx → JSON config → plugin manifest → library enable. Each step filters out non-technical designers. 607 downloads were self-selected developers. "Incredible once it works" vs "I gave up."

**Assumption:** Designers will complete developer-grade setup if the promise is compelling enough.

**Warnings:** Support requests at JSON config step, not tool behavior. Completion rate differs by OS/technical background.

### 6. Learning Loop Never Reaches Critical Mass

**Story:** 2 builds in week 1, then dormant. DS changes invalidate cache. Auto-promote threshold of 3 builds never reached. Gap recommendations never actioned.

**Assumption:** Designers return repeatedly within a timeframe where knowledge remains valid.

**Warnings:** Median 1-2 builds/user/month. Gap recommendations never forwarded to DS owners.

### 7. Native Chart Math Failures

**Story:** Bars with wrong proportions. Donuts summing to 340° or 390°. Radar polygons lopsided. SVG paths off-canvas. Each broken chart costs 25 tool calls. "100% DS compliant" is a joke when output is geometrically wrong.

**Assumption:** LLMs can reliably do floating-point geometry at production quality.

**Warnings:** >30% donut correction rate. 3x estimated token consumption due to retries.

---

## Synthesis

**Most Likely Failure:** LLM orchestration collapse on complex builds. Simple builds work. Dashboards break in unpredictable ways.

**Most Dangerous Failure:** Plugin enforcement kills adoption. 80% of real DSs have token gaps. Zero output on first build = users never return.

**Hidden Assumption:** The spec assumes the user's DS is structurally complete. Every enforcement decision was designed for the ideal DS, not the real one.

**Revised Plan:**
1. Add chart calculation layer in MCP server — math in Node.js, not prompts
2. Make enforcement graduated, not binary — adapt per-category based on what DS provides
3. Add phase enforcement to MCP layer — mechanical sequencing, not advisory hints
4. Add WebSocket keepalive + auto-reconnect with operation queue
5. First build succeeds on any DS — demonstrate value first, recommend improvements second

**Pre-Build Checklist:**
1. Chart math validation with known data, all chart types
2. Imperfect DS test suite (4 configurations)
3. Complex build stress test (12 sections, 3 chart types, 3 runs)
4. WebSocket durability test (5+ minutes, 200+ calls, simulated drop)
5. Non-technical designer onboarding test (time each step)
