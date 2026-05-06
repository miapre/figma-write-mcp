# Research dossier for mimic-ai: MCP UX, voice, progress, and strategy

**Bottom line up front.** Across four research threads, one pattern dominates: **named, falsifiable labor earns trust** — in the MCP protocol, in designer-facing copy, and in progress UX. mimic-ai's biggest leverage points are (1) embedding rich status inside tool results because most MCP clients render progress notifications poorly, (2) writing like Linear — terse, verb-first, craft-aware — never like GitHub Copilot's default "too polite to be useful," (3) narrating specific counts ("Scanning 1,284 layers, 42 Button variants clustered") rather than spinners, and (4) positioning as an **analytical/critic MCP** in a market crowded with generative DS tools (Anima, Figma Make, Supernova AI, Storybook MCP). The repo itself — `miapre/mimic-ai`, an HTML-to-Figma bridge with a learning cache — already embodies the "compounds over time" thesis; the research below is about how to make that invisible work feel visible, credible, and respectful to senior designers.

---

## 1. Customizing MCP tool messages across clients

### Protocol mechanics: three channels, widely varied support

The MCP spec (current stable revisions 2025-03-26 and 2025-06-18) gives servers three ways to communicate mid-call, each with different fate in different clients.

**`notifications/progress`** — the client opts in by attaching a `progressToken` in the request's `_meta`; the server streams zero or more progress notifications. As of rev 2025-03-26 each notification can carry a human-readable `message` field alongside the numeric `progress` / `total`:

```json
{"jsonrpc":"2.0","method":"notifications/progress",
 "params":{"progressToken":"abc123",
           "progress":50,"total":100,
           "message":"Clustering 42 Button variants"}}
```

Rules that matter in practice: `progress` **must strictly increase** (a common bug is resetting the counter per sub-task — use a cumulative float 0.0→1.0), `total` is optional, and notifications must stop after the final response. Spec: [modelcontextprotocol.io/specification/2025-03-26/basic/utilities/progress](https://modelcontextprotocol.io/specification/2025-03-26/basic/utilities/progress).

**`notifications/message`** (the logging channel) — server declares `logging` capability, the client may set a level via `logging/setLevel` (RFC 5424 levels from debug to emergency), and the server emits arbitrary JSON in `data`. The spec is explicit that "implementations are free to expose logging through any interface pattern" — meaning **you cannot rely on users seeing log messages**. Spec: [modelcontextprotocol.io/specification/2025-03-26/server/utilities/logging](https://modelcontextprotocol.io/specification/2025-03-26/server/utilities/logging).

**Elicitation** (`elicitation/create`, added 2025-06-18) lets a server ask the user a mid-call question via a JSON-Schema-described form; response actions are `accept` / `decline` / `cancel`. The 2025-11-25 revision added a URL mode for OAuth-style browser flows. **Sampling** (`sampling/createMessage`) lets the server borrow the client's LLM — useful for agentic MCPs that don't want their own API keys. Both are powerful but **inconsistently supported**.

### Client rendering — what actually happens

| Client | `notifications/progress` | `logging/message` | Elicitation | Sampling | Markdown in tool result |
|---|---|---|---|---|---|
| **Claude Code** | Received but **not visibly rendered** — tracked in [issue #4157](https://github.com/anthropics/claude-code/issues/4157) and [#3174](https://github.com/anthropics/claude-code/issues/3174) | Received, not displayed | ✅ form + URL mode | ❌ | ✅ |
| **Claude Desktop** | Partial; surfaces status text in recent builds | Silently consumed | ✅ | ✅ (since Dec 2024) | ✅ |
| **Cursor** | Shows a numeric counter ("5, 6, 7…") — the `message` field is **not visibly rendered** | Not surfaced | ❌ ([forum #116516](https://forum.cursor.com/t/mcp-elicitation-support-immediate-need/116516)) | ❌ ([forum #149604](https://forum.cursor.com/t/149604)) | ✅ |
| **VS Code (Copilot Chat)** | ✅ progress indicator | ✅ surfaced | ✅ form UI (v1.102+) | ⚠️ inconsistent | ✅ |
| **JetBrains Copilot** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Windsurf (Cascade)** | No documented progress UI | Undocumented | Undocumented | Undocumented | ✅ (incl. mermaid) |
| **Cline** | ❌ [discussion #3544](https://github.com/cline/cline/discussions/3544); **injects its own `task_progress` param into your tool call** ([#8256](https://github.com/cline/cline/issues/8256)) which can break strict schemas | Partial | ❌ [#4522](https://github.com/cline/cline/discussions/4522) | ❌ | ✅ |
| **Zed** | ❌ docs explicitly list Sampling / Elicitation / Discovery as "welcome contributions" | ❌ | ❌ | ❌ | Limited |
| **MCP Inspector** | ✅ | ✅ | ✅ (v0.16.2+) | ✅ | ✅ |

**The brutal truth:** the client mimic-ai is built against — Claude Code — currently drops progress notifications on the floor. Cursor displays a counter but not the message. Cline can actively break your server. Only VS Code, JetBrains, and MCP Inspector are fully wired for rich mid-call UX today.

### How real MCP servers handle this — none of them rely on progress notifications

Scanning the field: **GitHub MCP** ([github/github-mcp-server](https://github.com/github/github-mcp-server)), **Linear MCP**, **Sentry MCP**, and **Figma Dev Mode MCP** (HTTP-only, `localhost:3845`) all return synchronously and embed any status in the tool result text. Playwright-style MCPs use `logging/message` per navigation step for debug, not user UX. The reference [`server-everything`](https://github.com/modelcontextprotocol/servers/tree/main/src/everything) exercises every capability but exists mostly to test clients. The de facto pattern in FastMCP and the C# SDK is `ctx.report_progress(current, total, message)` — emit it and let capable clients render it.

### Workarounds, in order of ROI

The dual-emit pattern is the right default: **always** emit `notifications/progress` (for Inspector, VS Code, JetBrains, and future Claude Code) **and** return a markdown status block in the final tool result so every client surfaces something. A plain-text progress ladder works everywhere:

```
## Scan complete

✅ Read 1,284 layers across 3 pages
✅ Detected 42 Button variants in 4 clusters
✅ Matched 18 tokens to published `color/semantic` scale
⚠️ 2 components not published — skipped
⏱ 47s total · see `.mimic/run-2026-04-17.json`
```

Other patterns worth knowing: **chunked tools** (`start_job` → `job_id` → `poll_job`) let the host LLM loop and surface status in chat naturally; **keep-alive ticks** (emit an empty progress notification every ~10s) prevent some clients from timing out; **elicitation as progress** (firing an empty elicitation to force a UI update) is a known hack but fragile and not recommended.

### Gotchas: things you cannot customize

You cannot brand the server name shown to users — clients display whatever key appears in the user's `mcp.json` / `mcpServers` config. Tool names get client-specific prefixes (Claude Code renders them as `mcp__<server>__<tool>`). There is **no icon or color field in the spec**. The 2025-late "MCP Apps" extension adds rich UI to VS Code and Claude but is not universal. Tool-result size limits differ per client (Claude Code warns at 10k tokens, defaults to 25k, honors a `_meta["anthropic/maxResultSizeChars"]` override; others silently truncate). Markdown rendering is per-client, not negotiable per-message.

---

## 2. Voice and tone for a design system copilot

### Start from what designers already reject

The loudest warning from the field, from a community developer writing about Copilot: **"Most GitHub Copilot setups are too polite to be useful. By default, Copilot tries to agree, avoid criticism, and keep answers 'safe'… weak suggestions, missed problems, and bad decisions slipping through"** ([dev.to/playfulprogramming/github-copilot-is-too-nice](https://dev.to/playfulprogramming/github-copilot-is-too-nice-fix-it-with-a-tone-of-voice-file-39ij)). Senior designers react the same way to Notion-AI effusiveness. Your voice ceiling is set by the most senior, most skeptical designer in the room; your floor is set by not patronizing juniors.

Two guiding quotes worth internalizing. Karri Saarinen (Linear) on craft: *"With craft it always starts with people need to care about it… to design with craft, you must develop and trust your intuition"* ([linear.app/now/why-is-quality-so-rare](https://linear.app/now/why-is-quality-so-rare)). Julie Zhuo on taste: *"Taste becomes the critical differentiator. When AI can do many things well, your ability to recognize exceptional work versus average output becomes your most valuable skill"* — with her "conviction + humility" balance. Translation for mimic-ai: **be confident about facts, humble about judgment, never effusive**.

### Eight voice principles for mimic-ai

**Quiet confidence, not cheerleading.** No "Great question!", "Sure! I'd be happy to help!", or "🎉". Open with the artifact, not the pleasantry. Borrow Linear's tone — "short, direct, practices what it preaches."

**Craft-first vocabulary.** Use the words designers already use: token, variant, spec, spacing scale, contrast ratio, ramp, semantic role. Torrey Podmajersky's voice chart calls this "Vocabulary" — *"Key terms enhance personality"* ([Strategic Writing for UX](https://www.oreilly.com/library/view/strategic-writing-for/9781098174323/)).

**Suggest, don't prescribe.** Reserve "you should" for accessibility and irreversible actions. Default to "Consider," "Try," "One option," "Here's a draft." Shopify Polaris phrases it precisely: *"Put your merchant at the center and in control… Give merchants enough information to make the right decision on their own"* ([shopify.dev/docs/apps/design/content](https://shopify.dev/docs/apps/design/content)).

**Show work, state limits.** Expose reasoning briefly ("Matched 12 tokens from your `color/semantic` scale"). When confidence is low, say so plainly. Claude's default behavior is the model here — it acknowledges uncertainty and outlines assumptions rather than confidently guessing.

**Brevity over warmth.** Polaris fundamentals: *"Approach content like Jenga. What's the most you can take away before things fall apart?"* ([polaris-react.shopify.com/content/fundamentals](https://polaris-react.shopify.com/content/fundamentals)). Start sentences with verbs. Skip "You can."

**Never patronize the senior; never strand the junior.** Default copy is terse enough for seniors; an "Explain" or "Why?" affordance expands rationale for juniors on demand. Never pad the primary string with teaching.

**Own errors without apologizing.** State what failed, what's known, what to try. No "Oops!" or "Sorry about that." NN/g's chatbot guidance: *"Be transparent. Let users know you're having trouble and provide a clear path to resolution"* ([nngroup.com/articles/3-is-of-microcopy](https://www.nngroup.com/articles/3-is-of-microcopy/)).

**Treat the designer as the author.** mimic-ai drafts; the designer approves. Frame outputs as drafts, options, or starting points — never finished answers.

### Benchmark microcopy from peer products

| Product | State | Quoted copy | Source |
|---|---|---|---|
| **Linear Method** | Principles | "Aim for brevity. Short specs are more likely to be read." / "Opinionated defaults." / "Make the best thing the easiest thing." | [linear.app/method/introduction](https://linear.app/method/introduction) |
| **Figma AI / First Draft** | Prompt help | "Enter a prompt, like: 'A pricing page for a developer tools startup'… Click Make it." | [help.figma.com](https://help.figma.com/hc/en-us/articles/23955143044247-Use-First-Draft-with-Figma-AI) |
| **GitHub Copilot** | Inline suggestion | Pure ghost text — no label, no preamble, just the code. `Tab` accepts. | [code.visualstudio.com/docs/copilot](https://code.visualstudio.com/docs/copilot/reference/copilot-vscode-features) |
| **Vercel v0** | Positioning | "v0 is your always-on pair-programmer… each message refines the output, so you're working iteratively rather than hoping the first prompt lands perfectly." | [v0.dev/faq](https://v0.dev/faq) |
| **Raycast** | Changelog voice | "Fixed the empty floating bar problem when using Toggle Focus Session before starting any session." — declarative, subject-less, past tense. | [raycast.com/changelog](https://www.raycast.com/changelog/page/2) |
| **Notion AI** | Post-generation options | "Keep" · "Try again" · "Discard" — two words, verb-first. | [thomasjfrank.com/notion-ai-guide](https://thomasjfrank.com/notion-ai-guide/) |
| **Shopify Polaris** | Microcopy rules | "Be direct ('add apps' not 'you can add apps')." / "Start sentences with verbs." | [polaris-react.shopify.com](https://polaris-react.shopify.com/content/fundamentals) |

### Do / don't phrasings for mimic-ai

**Loading** — ✅ "Scanning your component library…" / "Reading `tokens.json` (237 tokens)" ❌ "Hold tight! I'm working my magic ✨"

**Completion** — ✅ "Drafted 3 button variants using your `color/action` tokens." / "Done. 4 components updated, 1 skipped (locked)." ❌ "All done! 🎉 Hope this helps!"

**Suggestion** — ✅ "Consider `spacing/200` here — it matches the surrounding grid." / "One option: extract this into a `Card.Header` variant." ❌ "You should definitely use spacing/200."

**Error** — ✅ "Couldn't parse `tokens.json` — line 42, trailing comma." / "I don't have access to the Figma file yet. Connect it, or paste the frame." ❌ "Oops! Something went wrong 😬 Please try again."

**Low confidence** — ✅ "Two reads on this. Which did you mean — the nav header or the page header?" ❌ Silently picking and proceeding.

**Critique** — ✅ "The contrast ratio is 3.8:1. WCAG AA for text requires 4.5:1." ❌ "This looks pretty good, but you might want to maybe consider…"

**Resolving the core tensions:** default to efficient (one line, verb-first); reserve conversational warmth for ambiguity; be authoritative about system facts, humble about judgment calls; for "I don't know," follow Claude's pattern — state the limit, state what you *can* do, ask one specific question.

---

## 3. Progress indicators that build trust

### The psychology floor: three numbers and one principle

Nielsen's canonical thresholds ([nngroup.com/articles/response-times-3-important-limits](https://www.nngroup.com/articles/response-times-3-important-limits/)): **0.1s** is instantaneous, **1s** preserves flow of thought, **10s is the attention cliff** — beyond which you need a percent-done indicator and a visible cancel, or users mentally leave. Figma design system analysis is almost always past 10s, so **a determinate bar plus a live step line plus a cancel is mandatory**, not optional.

Don Norman's synthesis of Maister's queueing psychology gives the design lever: **uncertain waits feel longer than known waits; unexplained waits feel longer than explained waits; anxious waits feel longer than occupied waits.** Reduce uncertainty, not duration.

Luke Wroblewski on skeleton screens: *"We had made people watch the clock… as a result, time went slower and so did our app. We focused on the indicator and not the progress"* ([lukew.com/ff/entry.asp?1797](https://www.lukew.com/ff/entry.asp?1797)). The same logic argues against bare spinners and for **previewing the shape of the answer**.

### The labor illusion — the most important finding for mimic-ai

Buell and Norton's 2011 *Management Science* paper, *"The Labor Illusion: How Operational Transparency Increases Perceived Value"* ([HBS PDF](https://www.hbs.edu/ris/Publication%20Files/Norton_Michael_The%20labor%20illusion%20How%20operational_f4269b70-3732-4fc4-8113-72d0c47533e0.pdf)), is the single most relevant research for this product. Across five experiments in travel and dating search, **users preferred sites that showed labor with a wait over sites that returned identical results instantly**. The causal chain: transparency → perceived effort (β=0.23) → reciprocity (β=0.58) → perceived value (β=0.68), with effort and reciprocity fully mediating. Even when participants were explicitly told how many sites would be searched, the transparent condition still made them *estimate* more sites searched (M=11.88 vs M=8.71, p<0.01) — transparency shapes perception beyond information. Satisfaction rose significantly (F=5.52, p<0.05) and repurchase intent rose more (F=8.85, p<0.01).

**The boundary condition matters:** if results are bad, transparency *amplifies* dissatisfaction (users blame you harder for "working hard" and delivering poor output). And per Buell's follow-up work and NN/g's 2025–26 guidance on "Magic-8-Ball Thinking," dressed-up "thinking" UIs can **over-inflate trust** in unreliable reasoning. Transparency must be curated — real labor, not stack-trace theater.

### Progress-bar design — Chris Harrison's CMU research

Two findings are directly actionable:

Harrison, Amento, Kuznetsov, Bell, *"Rethinking the Progress Bar"* (UIST 2007, [PDF](https://www.chrisharrison.net/projects/progressbars/ProgBarHarrison.pdf)) tested 9 bar behaviors at identical 5.5s durations: **pauses are perceived most negatively**; bars that **accelerate at the end** are preferred; the **peak-and-end effect dominates** — the last ~20% disproportionately shapes the memory of duration. Harrison, Yeo, Hudson, *"Faster Progress Bars"* (CHI 2010, [PDF](https://www.chrisharrison.net/projects/progressbars2/ProgressBarsHarrison.pdf)) found **ribbing animating backwards against the fill direction** produced the strongest perceived speedup — users rated a solid 5.0s bar as equivalent in duration to a 5.61s ribbed bar, a **~11% perceived compression** (p<0.001). Design consequences: never pause the bar, ease the curve to accelerate at the end, reserve ≥10% of fill for the long-tail final step so 99% doesn't stick.

### How AI products actually build trust during generation

| Product | UX pattern | Trust mechanism |
|---|---|---|
| **ChatGPT** | Token streaming + collapsible "Thinking…" for reasoning models | Output *is* the progress bar; value arrives while you read. |
| **Claude (extended thinking)** | Structured, bulleted thinking block streamed before the answer ([anthropic.com/news/visible-extended-thinking](https://www.anthropic.com/news/visible-extended-thinking)) | Summarized thinking keeps latency-to-first-token low while signaling deliberation. |
| **Perplexity** | "Searching… Reading 6 sources…" with live source chips that populate before the answer | **Named sources = falsifiable progress.** Users can verify the labor is real. |
| **Cursor (Agent mode)** | Inline expandable tool-call cards with diffs | Every tool call is auditable and approvable before continuing. |
| **v0** | Plan statement first, then streamed JSX with live preview | Commits to a plan (reduces misinterpretation anxiety), then streams value into a visible preview. |
| **Devin** | Editable plan pane + scrolling shell/browser panel + `notes.txt` memory file | Trust via delegation-with-checkpoints rather than continuous exposure. |

**The specificity principle** unites them: trust comes from **named, falsifiable labor**. *"Analyzing your design system"* ⟶ *"Scanning 1,284 layers in 3 pages"* ⟶ *"Found 42 Button instances — clustering by fill, radius, and label typography."* Each step up is more trustworthy because each is more falsifiable.

### Pitfalls

**Fake progress** (random-walk bars, fabricated steps) collapses trust faster than no indicator once users notice retries produce different "steps." **Stalled bars** violate Harrison's pause-aversion finding. The **"90% stuck" phenomenon** happens when the last unbounded step is allocated too little of the bar. **Wrong ETAs** damage trust more than missing ETAs — only show estimates bounded within ~30% error. **Silent retries** read as suspicious pauses; surface the retry. And **firehose logs** violate Buell & Norton's "curated effort" — labor illusion works on selected effort, not stack traces.

### Concrete status-message rewrites

- "Loading…" → **"Scanning 1,284 layers across 3 pages (page 2 of 3)"**
- "Analyzing your file" → **"Found 42 Button instances — clustering by fill, radius, and label typography"**
- "Almost done… 99%" (for 40s) → **"Finalizing token names (this step can take up to 30s)"**
- "Processing…" (static) → **"Step 3/5 · Matching components to Material tokens · 1,024 / 1,284 layers"**
- "Error, retrying" → **"Figma rate limit hit — waiting 4s then resuming at layer 812/1,284"**
- "2 minutes remaining" (inaccurate) → **"Elapsed 1:12 · Typically 2–4 min for files this size"**

---

## 4. Other considerations the user didn't ask about

### Position as a critic, not a generator

The 2025–26 DS tool landscape is **crowded with generative tools**: Anima Agent ("God Mode"), Figma Make, Supernova AI, Storybook MCP. **mimic-ai's repo tagline — "translates HTML into Figma using published components and design tokens" — is partly generative, but its compounding cache and DS-awareness are really analytical.** The gap in the market is the **reviewer/linter** role, modeled on Grammarly, Hemingway, ESLint, and Axe. Their shared DNA: **passive surfacing, severity taxonomies, "why" as a first-class affordance, no auto-apply by default.** Hemingway famously highlights without rewriting — reviewers noted it "restricted your options" and functions as diagnostic rather than fix engine. mimic-ai should default to **description + linked rationale + cheap dismiss**, not mutation.

This matches current designer sentiment. Brad Frost (the canonical voice here) frames it as *"DS+AI" not "AI+DS"*: *"the AI is deliberately constrained to using the high-quality design system materials to ensure what's being generated adheres to the organization's established standards"* ([bradfrost.com/blog/post/agentic-design-systems-in-2026](https://bradfrost.com/blog/post/agentic-design-systems-in-2026/)). Sparkbox's 2022 Design Systems Survey ([designsystemssurvey.sparkbox.com/2022](https://designsystemssurvey.sparkbox.com/2022/)) has flagged **adoption** as the top challenge every year since 2018; only 16% of teams track metrics; 35% of in-house teams have considered starting over. A tool that measures adoption, surfaces debt, and teaches the system — without overwriting it — lands directly on unmet demand.

### Present recommendations as questions, with evidence

The Google PAIR Guidebook ([pair.withgoogle.com](https://pair.withgoogle.com/)) and Microsoft HAX Toolkit ([microsoft.com/haxtoolkit](https://www.microsoft.com/en-us/haxtoolkit/)) converge: when AI augments rather than automates, **offer control**, **make clear what the system can and cannot do** (HAX G1/G2), **make clear why it did what it did** (G11), and **support efficient correction** (G9). Frame findings as hypotheses plus evidence:

> *"Should `Button/Primary` and `Button/Main` be merged? (87% visual overlap across fill, radius, and label typography; found in 14 frames.)"*

…beats *"Merge these components."* Research on advice-giving shows people resist evaluative feedback unless it is contextualized, offers choice rather than command, and preserves authorship.

### Trust calibration: categorical, not percentage

Li et al. (arXiv [2402.07632](https://arxiv.org/abs/2402.07632)) show that **miscalibrated AI confidence actively impairs user reliance, and miscalibration is difficult for users to detect** — users over-rely on overconfident AI and under-rely on underconfident AI. A Berkeley iSchool capstone found **numeric percentages are routinely misinterpreted**; categorical bands (high/medium/low) performed better. Showing *what data the AI used* had a stronger borderline-positive effect on calibration than confidence scores alone. Vasconcelos et al. add a counterintuitive finding: **explanations can increase over-reliance** when users engage shallowly, via the "explainability = trustworthiness" heuristic.

For mimic-ai: use **categorical confidence with evidence counts** — *"Strong pattern (found in 12/14 components)"* — never "87%." Every rec should cite its evidence. Override must be **one click**.

### Onboarding: one finding, not a dashboard

Raycast's CEO on Linear's onboarding: *"We didn't need to explain how to use Linear. We added them to the tool and they figured out what they needed to do"* ([linear.app/customers/raycast](https://linear.app/customers/raycast)). Raycast itself front-loads the single critical friction point (Spotlight shortcut collision) explicitly. For data-heavy first-run UX, the pattern is: **stream progress with meaningful labels, offer quick-scan vs deep-scan, and end with one actionable win** rather than a full audit dump. mimic-ai's first run should: connect → scan with live narrated status → surface 1–3 top findings → let the user act on one. Save the full audit for run 2.

### Open-source distribution for designers

The README hero matters disproportionately. Designer-facing OSS tools that work ([Tokens Studio](https://github.com/tokens-studio/sd-transforms), [Style Dictionary](https://github.com/amzn/style-dictionary), [Storybook](https://github.com/storybookjs/storybook)) lead with a GIF or demo, state the problem in the first 50 words, and reduce install to one command. mimic-ai's current install flow (`bash <(curl -fsSL .../install.sh)` + manual Figma plugin import + running the bridge + enabling library) is **four steps and a persistent terminal** — friction that costs non-technical designers. Simplify where possible; script what can be scripted; explain the bridge requirement visually.

**Trust signals designers specifically scan for:** an explicit "runs locally, your design data never leaves your machine" claim on the README hero (Lighthouse uses this phrasing), an OSS license badge, a screenshot of the tool working against a recognizable DS, contributor count, last-commit badge, a CHANGELOG, and — for designers — a Figma Community listing even if distribution is MCP-native.

### "Learn from usage": make memory legible

The successful pattern from Cursor and Claude Code is **memory as a human-readable, editable file** — `.cursor/rules/*.mdc`, `CLAUDE.md`, `notes.txt`. Not a black box. mimic-ai's README already describes this implicitly ("the knowledge belongs to your team's specific DS, your specific naming conventions, and your specific corrections over time") — surface it as `.mimic/learned.md` or similar, with human sentences like *"I've noticed you treat `Btn/*` and `Button/*` as equivalent"* and a one-click "forget this." Hierarchy matters: **team-level patterns** (committed to git) vs **personal** (gitignored). No telemetry by default — all learning in inspectable files.

### Information architecture for recommendations

Borrow from Lighthouse, Axe DevTools, and ESLint: **primary axis is severity** (critical / important / suggestion / info), **secondary is type** (duplicate component, unused token, naming inconsistency, missing variant, spacing drift), **tertiary is scope** ("affects 14 frames"). A 0–100 DS-health score with per-category sub-scores is controversial but sticky — Lighthouse's score is imperfect but unforgettable. Treat recs as **triageable inbox items** (open / dismissed / fixed / snoozed), matching Linear's mental model.

### Handling bad recommendations

Two escape hatches per rec — **"Not now"** (single dismiss) and **"Teach me this isn't a bug"** (writes a rule into `.mimic/rules.md`). Copilot's biggest complaint is the absence of reliable Esc dismissal ([orgs/community/discussions/157778](https://github.com/orgs/community/discussions/157778)) — don't repeat that. Granular dismiss (rule / component / project / forever) matches how designers actually triage. An optional micro-survey after dismiss ("Wrong / Not a priority / Already known") is cheap if never blocking. Feedback data ≠ auto-training per PAIR — always allow users to see and edit what was learned.

### Competitive landscape

| Tool | Category | Local-first | AI angle | Gap |
|---|---|---|---|---|
| **Specify** | Token pipeline | Cloud | Limited | No analysis |
| **Supernova** | Tokens + docs | Cloud | AI prototyping added | Setup "days not hours" |
| **Zeroheight** | Docs/governance | Cloud | Minor | Not analytical |
| **Knapsack** | Enterprise DS | Cloud | "AI-ready data" | Not designer-first |
| **Tokens Studio** | Figma plugin + platform | Plugin + cloud | Limited | Tokens only |
| **Anima** | Design→code (Anima Agent "God Mode") | Cloud | Heavy generative | Not analytical |
| **Figma Code Connect** | Design↔code linking | Hybrid | Feeds agents | Not a reviewer |
| **FigmaLint / Design System Auditor / onBeacon** | Figma plugins | Plugin | Yes | Not MCP-native |
| **Storybook MCP** | Code-side generation | Local | Generative | Not analytical |

**The unoccupied quadrant: MCP-native + local-first + analytical + designer-facing.** FigmaLint is closest but plugin-only; Storybook MCP is code-only; Anima and Figma Make are generative. mimic-ai can credibly own "the DS audit layer for agentic workflows" — complementary to generative MCPs, not competing.

---

## Implications for mimic-ai

**Ship progress as text blocks, not just notifications.** Claude Code (your current primary target) drops `notifications/progress` silently. The highest-ROI move is returning a markdown status ladder inside the tool result on every call — capable clients still get rich rendering, incapable ones still show the work. Dual-emit. Never depend on `logging/message` being visible. Avoid elicitation and sampling on critical paths until Cursor and Cline catch up, or ship them with a documented fallback (tool argument + API-key mode).

**Write like Linear, not Copilot.** Defaults should be terse, verb-first, and craft-aware. The copy test: would a senior DS lead keep reading, or close the tab after the third emoji? "Drafted 3 button variants using your `color/action` tokens" beats "All done! 🎉 Hope this helps!" Reserve warmth for ambiguity, not success.

**Make the labor specific and falsifiable.** The Buell & Norton result is the load-bearing finding for this product. *"Scanning 1,284 layers in 3 pages · 42 Button variants detected · clustering by fill, radius, typography"* is not cosmetic — it is a causal driver of perceived value, reciprocity, and tolerance for wait. Combine with Harrison's rules: never pause, ease the bar to accelerate at the end, reserve ≥10% for the long tail.

**Frame recommendations as questions with evidence.** *"Should `Button/Primary` and `Button/Main` be merged? (87% overlap, 14 frames)"* — not *"Merge these."* Use categorical confidence bands, cite evidence counts, make dismiss one click, and store learned patterns in a readable `.mimic/learned.md` file the designer can edit. This is the combined lesson from PAIR, HAX, and the AI-reliance literature, and it matches how designers react to unsolicited advice.

**Position against the generative pack.** The 2025–26 DS tool landscape is crowded with generators (Anima, Figma Make, Supernova AI, Storybook MCP). The analytical/critic/reviewer slot is open. Lean into Brad Frost's "DS-as-guardrail" thesis; market mimic-ai as the audit layer inside agentic design workflows, complementing generative MCPs rather than competing. The repo's current "translates HTML into Figma using your DS" framing is the wedge; the longer-term brand is **"the reviewer for your design system — local, legible, and it gets smarter about your conventions."**

**Reduce install friction before marketing.** The current flow is four steps plus a persistent terminal. Every step is a drop-off for non-technical designers. The README needs a GIF of mimic-ai finding a real issue in a recognizable DS within the first fold, an explicit "runs locally, your design data never leaves your machine" claim (Lighthouse phrasing), and a Figma Community listing as a trust artifact even if distribution stays MCP-native.

**One finding on first run, not a dashboard.** Stream narrated status, end with a single actionable insight, save the full audit for run 2. Linear's "figure out on your own" ethos applies; Raycast's pattern of explicitly naming the critical friction point (the bridge requirement) applies too. The compound-learning thesis in your README — "by run 3, familiar patterns require no lookups; by run 10, most builds are nearly free" — is a *narrative* users need to feel, not just read. Show them the cache hit count, surface the patterns learned, make the curve visible.