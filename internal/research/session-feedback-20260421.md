# Session Feedback Compilation — 2026-04-20/21

Every piece of user feedback from the validation gauntlet + demo build session, mapped to tool changes.

---

## RULES ADDED THIS SESSION

| Rule | What | File |
|---|---|---|
| 35 | Sequential component imports only | GOLDEN_RULES.md |
| 36 | No overlapping components (auto-HUG on insert) | GOLDEN_RULES.md + plugin/code.js |
| 37 | Hide ALL icon slots on EVERY component (automated in plugin) | GOLDEN_RULES.md + plugin/code.js |
| 38 | Zero raw values — absolute, no exceptions | GOLDEN_RULES.md |
| 39 | Text styles are the only way to style text (never detach, never bind individual props) | GOLDEN_RULES.md |
| 40 | No default text on any component (override ALL text after insertion) | GOLDEN_RULES.md |
| 41 | Read the HTML before writing any text (no paraphrasing, no inventing) | GOLDEN_RULES.md |
| 42 | No hardcoded line breaks (container width controls wrapping) | GOLDEN_RULES.md |

## CODE CHANGES MADE THIS SESSION

| Change | File | Status |
|---|---|---|
| Component import timeout 15s→60s | plugin/code.js | Shipped |
| Parallel import race (component + set simultaneously) | plugin/code.js | Shipped |
| Variable resolution timeout 5s→30s | plugin/code.js | Shipped |
| Parenthetical variable name normalization | plugin/code.js | Shipped |
| Style import timeout 8s→20s | plugin/code.js | Shipped |
| Style preload timeout 10s→30s | plugin/code.js | Shipped |
| Bridge timeouts (45→90s components, 120→300s preload) | bridge.js | Shipped |
| Frame default to HUG on both axes | plugin/code.js | Shipped |
| Auto-HUG on component insertion into AL parents | plugin/code.js | Shipped |
| Progressive notifications at 5s/15s/30s | plugin/code.js | Shipped |
| Auto-hide icon booleans on every insert_component | plugin/code.js | Shipped |
| `figma_preload_variables` new tool | plugin/code.js + mcp.js + bridge.js | Shipped |
| `fontFamily` param on set_session_defaults | plugin/code.js + mcp.js | Shipped (reverted from hot path) |

## FEEDBACK THAT STILL NEEDS TOOL-LEVEL FIXES

### 1. Build communication completely absent
**Feedback:** "I saw no message at all here regarding this, is this a miss from the tool?"
**Issue:** Phase 5 (Report & Communicate) never executed during any build. No learning patterns saved, no gap recommendations surfaced, no build report generated.
**Root cause:** The build protocol requires Claude to call `mimic_ai_knowledge_write` after each build and generate a Phase 5 report. But nothing enforces this — it's just documented in CLAUDE.md. The builder can skip it without error.
**Fix needed:** Either (a) make `mimic_ai_knowledge_write` automatic after every build via the bridge/plugin, or (b) add a MANDATORY Phase 5 gate that blocks the build from being reported as "done" until knowledge is written and a report is produced.

### 2. DS component text never overridden (Rule 40)
**Feedback:** "Why did you use header links that say 'Products, Services, Resources, About'? This can't happen."
**Issue:** Every component was inserted with default text showing — header nav, tabs, footer, badges, inputs. The builder had to be reminded on EVERY component.
**Root cause:** Component insertion and text override are separate steps. The builder inserts the component, then may or may not remember to override text.
**Fix needed:** The build protocol should require a TEXT OVERRIDE PASS immediately after component insertion. Rule 40 was added but needs enforcement in the tool — perhaps `insert_component` could return a warning if text layers contain default content.

### 3. HTML content fidelity violations (Rule 41)
**Feedback:** "I see 'UPTIME GUARANTEE' vs 'Uptime' in the HTML. DIFFERENT CONTENT!"
**Issue:** Text was written from memory instead of read from the HTML source. Metric labels, badge text, pricing headers — all invented.
**Root cause:** The builder writes text based on what it remembers, not what the HTML says. No verification step.
**Fix needed:** Before writing ANY text to a Figma node, the builder must have the HTML text in context. The build protocol should include "read the HTML section → extract text → write exact text" as a mandatory sequence.

### 4. Text styles detached and individual properties bound (Rule 39)
**Feedback:** "Instead of this [raw properties] you should do this [text style]. How many times do I have to tell you?"
**Issue:** Text styles were detached and fontSize/lineHeight bound as individual variables. This is NOT DS-compliant — the text style IS the variable.
**Root cause:** The builder tried to "fix" the line height by detaching the style and binding individual properties, which is worse than the original.
**Fix needed:** Rule 39 added. The plugin could also enforce: if a text node has a textStyleId, refuse to allow setBoundVariable on fontSize/lineHeight.

### 5. Hardcoded line breaks (Rule 42)
**Feedback:** "There was a line break there that breaks the design."
**Issue:** `\n` inserted in text content to control wrapping, instead of letting the container width handle it.
**Root cause:** The builder added `\n` to make text wrap where the HTML had `<br>` or where it "looked better."
**Fix needed:** Rule 42 added. The plugin's `create_text` handler could strip `\n` unless explicitly passed as a parameter.

### 6. Component hidden container layers
**Feedback:** "The container layer is hidden" (on header and footer)
**Issue:** After inserting DS components (header nav, footer), key internal layers were hidden — causing the component to appear blank.
**Root cause:** When modifying nested elements (hiding links, changing text), parent frames were accidentally hidden. The `showAll` approach then revealed everything including items that should stay hidden.
**Fix needed:** Never hide/show parent frames when trying to modify child content. Only modify the specific property you need (text content, boolean properties). If a child needs to be hidden, hide the child directly, not its parent.

### 7. Table column width misalignment
**Feedback:** "Table headings aren't aligned with the rows content"
**Issue:** Header cells were 80px while data cells were 160px for the same columns.
**Root cause:** Different code paths created header row and data rows with different width constants.
**Fix needed:** Define column widths ONCE and reuse for both header and data rows. The build code should have a single `COL_WIDTHS` array used by all rows.

### 8. Bar chart bars bunched up
**Feedback:** "Events processed chart is still bad"
**Issue:** All 12 bars crammed into the left portion of the chart instead of distributed evenly.
**Root cause:** Bars had `layoutGrow=0` and fixed widths, so they stacked on the left. Later fix set `layoutGrow=1` but the bars frame itself wasn't stretching.
**Fix needed:** When building charts, bars must use `layoutGrow=1` AND the bars container must be `layoutAlign="STRETCH"` with `primaryAxisSizingMode="FIXED"`.

### 9. Metrics too wide
**Feedback:** "Metrics below hero shouldn't be so wide"
**Issue:** Metrics stretched edge-to-edge across 1440px.
**Root cause:** Metric items used `layoutGrow=1` distributing across the full parent width.
**Fix needed:** Use a centered inner container with max-width, not full-width distribution. Match the HTML's layout approach.

### 10. Footer variant choice
**Feedback:** "Why did you use Large06? That has a subscription newsletter."
**Issue:** Chose a footer variant with newsletter/subscription that doesn't match the HTML's simple footer.
**Root cause:** Didn't inspect the footer variants to find the best match. Just picked the first one.
**Fix needed:** Before inserting ANY component variant, inspect all available variants and pick the one that best matches the HTML's structure. This is Rule 32 (component structure inspection) applied to variant selection.

---

## RECOMMENDATIONS FOR NEXT SESSION

1. **Start with Phase 5 output** — before building anything, design the learning communication that should appear after the build. Work backward from the desired output.

2. **Read the entire HTML first** — extract all text content into a reference list before any Figma operations. Every text override comes from this list, not from memory.

3. **One component at a time** — insert, inspect, configure ALL properties (text, icons, variants, booleans), verify visually, then move to the next. No batch inserts without configuration.

4. **Column widths as constants** — define once, reuse everywhere. No different widths for header vs data rows.

5. **Plugin restart needed** — the icon auto-hide fix in plugin/code.js needs a plugin restart to take effect.
