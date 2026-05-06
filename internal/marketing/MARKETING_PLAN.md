# Mimic AI — Marketing Plan

**Owner:** Marketing Strategist
**Constraint:** ~5 hrs/week, zero budget
**Authoritative sources:** `VOICE_AND_TONE.md` (copy standards), growth plan (`internal/research/zero-budget-gowth-plan.md`), research report (`internal/research/mimic-ai-research-report.md`), marketing governance (`internal/marketing/MARKETING_GOVERNANCE.md`)

---

## The Core Thesis

Mimic is not an HTML-to-Figma converter that happens to learn. **Mimic is a design system copilot that enters through HTML-to-Figma builds.** The builds are the entry point — the compounding knowledge is the product.

After 1 year of usage, Mimic:
- **Knows your DS deeply.** Every component, every variant, every token mapping — learned from hundreds of builds and corrections. Near-zero lookups. Instant builds.
- **Remembers your conventions.** "This team always uses `Button/Primary` for CTAs, `Tag/sm` for metadata, `spacing-xl` between sections." Not because someone documented it — because Mimic observed it across 200 builds and you confirmed it.
- **Surfaces what your DS is missing.** "You've built 47 screens. Status badges appeared in 31 of them, always as primitives. Your DS doesn't have a Status Badge component. Should it?" — backed by evidence counts, not opinions.
- **Catches regressions.** "Last quarter you used `Card/Outlined` for settings panels. This build used a primitive. Your DS still has the component. Was this intentional?"
- **Gets proactive.** "Your DS has 3 new components since last month. Two of them match patterns I've been building as primitives. Using them now."
- **Becomes institutional memory.** Team members leave, conventions drift — Mimic is the reviewer that never forgets. The knowledge file is inspectable JSON on your machine, sharable with your team.

This is the story. Not "HTML to Figma." That's the feature. The story is: **Mimic becomes the teammate who knows your design system better than anyone on the team, and it gets better every time you use it.**

---

## Positioning

### One-Sentence Moat (Designer-Facing)
> *"The DS copilot that learns your design system from every build — and tells you where it's missing."*

### One-Sentence Moat (Technical / Mixed)
> *"Most Figma MCPs read. Mimic writes — using your real design system components, with a learning cache that makes repeat builds nearly free."*

### Four Differentiators (always present)
1. **Builds with your real design system** — published components and design token variables, not inferred or generic
2. **Compounds over time** — corrections become rules, patterns auto-verify, builds get faster, recommendations get sharper
3. **Surfaces DS gaps** — every build reports what your system is missing, backed by evidence counts across builds
4. **Open source, local-first** — MIT licensed, design data never leaves the machine, knowledge file is inspectable and sharable

### The Designer's Problem (lead with this)

Designers have always faced two painful realities:

1. **The prototype-to-Figma gap.** AI makes it trivially easy to generate HTML prototypes now (Claude, v0, Lovable, Bolt). Business stakeholders hand off HTML mockups. Designers iterate on coded prototypes because it's fast. But getting those back into Figma — using your real DS components, your tokens, your conventions — is still manual, tedious work. Rebuilding by hand. Mimic closes that gap.

2. **DS maintenance is the hardest job in design.** The Sparkbox Design Systems Survey has flagged adoption as the top challenge every year since 2018. Only 16% of teams track metrics. 35% have considered starting over. Components drift, conventions get lost, new team members don't know the rules. Mimic becomes the reviewer that never forgets — it surfaces gaps, catches regressions, and builds institutional memory that survives team turnover.

### Competitive Frame
| | Mimic | Framelink | Figma Official MCP | html.to.design |
|---|:---:|:---:|:---:|:---:|
| Writes to Figma canvas | ✅ unlimited | ❌ | ⚠️ metered | ✅ paid |
| Uses your real components | ✅ native | ⚠️ agent infers | ✅ w/ Code Connect | ⚠️ partial |
| Binds to design token variables | ✅ | ❌ | ✅ | ❌ |
| Learns over time | ✅ | ❌ | ❌ | ❌ |
| DS gap detection | ✅ | ❌ | ❌ | ❌ |
| Open source | ✅ MIT | ✅ MIT | ❌ | ❌ |

### Audience Segments (in priority order)

**1. Designers who create screens using a design system (Primary)**
UX/UI designers, product designers who work in Figma with a published DS. They get HTML prototypes — from AI tools, from stakeholders, from their own quick iterations — and need to bring them into Figma as DS-compliant screens. They care about: speed, fidelity to their DS, not having to rebuild by hand.

*Their language:* components, tokens, variants, spacing scale, design specs, handoff. They don't say "MCP server" or "rate limit bypass." They say "it builds with my components" and "it learns my system."

**2. Design system leads (Primary)**
The people responsible for maintaining, evolving, and driving adoption of the DS. Often senior designers, sometimes dedicated DS roles. The hardest job in design — keeping a system alive, consistent, and adopted. They care about: DS coverage, gap detection, consistency enforcement, institutional memory.

*Their hook:* "Mimic tells you where your DS is missing — backed by evidence from real builds, not guesswork." The DS copilot angle is built for them.

**3. Developers (Tertiary)**
Developers using Claude Code or Cursor who occasionally need Figma output. They care about: the technical architecture, MCP integration, rate-limit bypass. They'll find Mimic through developer channels (GitHub, MCP directories, HN) and appreciate the technical rigor. But they're not the primary adopter — they're the referral path. A developer discovers Mimic, shows it to their design team, and the designers adopt it.

*The developer's role in the funnel:* discovery → referral. Not primary usage.

---

## The 1-Year Vision (Use This in Content)

| Timeframe | What Mimic knows | What the designer experiences |
|---|---|---|
| **Build 1** | Nothing. Cold start. Scans your entire DS. | "It found my components and used them. Some primitives where my DS has gaps. Took a few minutes." |
| **Build 5** | Core patterns verified. Common components cached. | "It's faster. It remembered that I use `Button/Primary` for CTAs. It didn't ask about tabs this time." |
| **Build 20** | Deep knowledge. Recipes for every common component. Most builds need zero DS lookups. | "I just point it at an HTML and it builds a pixel-perfect Figma screen in seconds. It knows my DS better than the new hire." |
| **Build 50** | Comprehensive DS audit data. Recurring gaps visible. Pattern library covers most HTML patterns. | "The gap report says I've used status badges as primitives 31 times across 6 months. I finally built the component. Mimic started using it immediately." |
| **Build 100+** | Institutional memory. Team conventions encoded. Regression detection active. | "A junior designer joined the team. I shared the knowledge file. On day one, their builds follow all our conventions. Mimic is the style guide that actually enforces itself." |

This table is the marketing narrative. It should appear (adapted) in: the README, the first blog post, the Product Hunt pitch, conference talks, the Figma Community listing.

---

## Strategic Objectives

1. **Establish Mimic as a DS copilot, not an HTML converter.** Every piece of content leads with the compounding value, not the one-time build.
2. **Reach designers where they are.** Designer channels first, developer channels for discovery/referral. The GIF and the 1-year narrative are the primary assets.
3. **Build the demo GIF around the copilot narrative.** Not just "HTML → Figma." Show: Build 1 (slow, lookups) → Build 5 (fast, from cache) → the gap report. The learning is the payoff, not the build.
4. **Validate with different design systems.** Before major push, test with 2-3 external DSs to prove DS-agnosticism. The claim "works with any design system" must be true.
5. **Seed every discovery surface** — Figma Community (highest priority), design directories, MCP directories, awesome lists.

---

## Channel Priority (Reordered)

### Tier 1 — Designer Channels (Primary)

| Channel | Audience | When | Approach |
|---|---|---|---|
| **Figma Community plugin** | Designers | After stable version | Biggest designer funnel (~1M+ users). Thin companion plugin or full listing. |
| **r/FigmaDesign** (105K) | Designers | Can post now | Lead with pain: "I kept rebuilding prototypes by hand." GIF. Designer vocabulary. |
| **r/DesignSystems** | DS leads | Can post now | Lead with DS copilot angle: "It tells you where your DS is missing." |
| **Friends of Figma** Discord (~33K) | Designers | Lurk first, then #projects | 3-5 genuine answers before mentioning Mimic. |
| **Into Design Systems** community | DS leads | Can post now | Exact audience match. Personal pitch. |
| **Figma Forum** (forum.figma.com) | Designers | Can post now | Designer vocabulary. Blog post link, not GitHub. |
| **Designer News** | Designers | Can post now | Submit blog post, not raw repo. |
| **Sidebar.io / Prototypr.io** | Designers | After blog post | Submit the post with visuals. |
| **Design newsletters** (Figmalion, Muzli, Dense Discovery) | Designers | After stable + demo video | Earned. Need a launch moment. |

### Tier 2 — Developer Channels (Discovery + Referral)

| Channel | When | Approach |
|---|---|---|
| **Hacker News** (Show HN) | Launch week | Engineer-to-engineer. Technical differentiators. Limitations upfront. |
| **r/ClaudeAI** (~747K) | Launch week | Technical but accessible. Demo GIF. |
| **r/LocalLLaMA** (~688K) | Launch week | Local/no-cloud angle. |
| **MCP directories** (Glama, mcp.so, Smithery, etc.) | Ongoing | Technical listing. Already partially done. |
| **awesome-mcp-servers** (84K) | After Glama badge | One-line description. |
| **r/mcp** | Launch week | On-topic, high conversion per reader. |

### Tier 3 — Mixed Channels

| Channel | When | Approach |
|---|---|---|
| **Product Hunt** | Week 3-4, after README converts | GIF + 1-year narrative. Tagline: "The DS copilot that learns your design system." |
| **X** | Ongoing | Build in public. GIF-first. Design systems hashtags. |
| **LinkedIn** | Ongoing | Shorter version. GitHub link in first comment. |

---

## Roadmap

### Phase 0: Pre-Launch (This Week — 5 hrs total)

**Prerequisite: nothing ships until these are done.**

| Action | Time | Status |
|---|---|---|
| Rewrite README hero — lead with DS copilot narrative, not MCP technical details | 1.5 hr | ☐ |
| Record demo GIF — show the learning arc (Build 1 → Build 5 → gap report), not just one build | 1.5 hr | ☐ |
| Add 20 repo topics (include `design-system-copilot`, `design-system-audit`, `figma-design-system`) | 15 min | ☐ |
| Upload 1280x640 social preview PNG | 15 min | ☐ |
| Enable GitHub Discussions | 5 min | ☐ |
| Rewrite GitHub "About" — designer-facing: "DS copilot that builds Figma designs from HTML using your published components. Learns your system, flags your gaps." | 10 min | ☐ |
| Add comparison table + FAQ to README | 20 min | ☐ |
| Write launch post drafts (designer-facing for r/FigmaDesign, technical for HN) | 30 min | ☐ |

### Phase 1: Launch Week (5 hrs)

**Designer channels launch alongside developer channels — not after.**

**Tuesday:**
- Show HN (8:30 AM PT) — technical angle, engineer-to-engineer
- r/ClaudeAI (11 AM PT) — demo GIF, GitHub link
- r/FigmaDesign — designer pain angle, same day

**Wednesday:**
- r/DesignSystems — DS copilot angle, gap detection narrative
- r/mcp, r/SideProject
- Anthropic Discord (#showcase)
- Into Design Systems community — personal pitch

**Thursday:**
- X thread with demo GIF — tag design systems people, not just AI people
- LinkedIn — shorter version
- Friends of Figma Discord (if lurk period done)

**Thursday-Friday:**
- MCP directory sprint (Glama ✅, mcp.so, Smithery, PulseMCP, MCPHub, Cursor Directory, mcpservers.org)

**Friday-Sunday:**
- Awesome list PRs (mcp-servers, awesome-figma, awesome-ai-for-design, Awesome-Design-Tools)

### Phase 2: Month 1 — Consolidate (5 hrs/wk)

| Week | Action |
|---|---|
| 2 | First blog post: designer-facing, "How I stopped rebuilding prototypes by hand" or the 1-year narrative. Cross-post dev.to/Hashnode. |
| 2-3 | Second-tier directories: Peerlist, AlternativeTo, TOOOLS.design (submitted ✅), Sidebar.io |
| 3 | Validate with 2-3 external DSs. Document results. |
| 3-4 | Product Hunt launch. Tagline: "The DS copilot that learns your design system." |

### Phase 3: Quarter 1 — Content Flywheels (5 hrs/wk)

**Blog post queue (reordered — designer-first):**
1. "How Mimic learned my design system in 20 builds (and what it told me was missing)" — the 1-year narrative, compressed
2. "From v0/Lovable prototype to Figma: getting AI prototypes back into your design system" — rides the AI prototyping wave, speaks to designers
3. "Every way to get HTML into Figma in 2026, compared" — head-term SEO, designer search query
4. "Design system maintenance is broken. Here's what 50 builds taught me about mine." — DS leads, audit angle
5. "Framelink vs Figma's official MCP vs Mimic: which Figma MCP should you use?" — comparison, ranks forever
6. "Teaching an LLM your design system: persistent knowledge for Figma agents" — technical, developer audience

**Recurring cadence:**
- 1 blog post every 10 days, cross-posted dev.to + Hashnode with canonical
- 1 X thread per week
- 1 Reddit post per 2 weeks — alternate between designer subs and developer subs
- 1 benchmark or DS audit case study every 3-4 weeks
- Daily 2-minute X micro-posts

**Community presence:**
- Friends of Figma Discord, Into Design Systems — answer DS questions, build credibility
- MCP Discord, r/LocalLLaMA Discord — developer discovery channel
- 30 min/week total

**Figma Community plugin companion** — high priority. Unlocks Figma's 1M+ plugin directory. Even a thin version (knowledge browser, DS gap viewer) is enough.

---

## The Demo GIF (Revised)

**The GIF must tell the copilot story, not just the build story.**

- **Format:** 20-30 seconds, 1200px wide, under 10MB
- **Structure:**
  1. (5s) HTML in Claude Code → "Build this in Figma using my DS"
  2. (8s) Figma canvas fills with real DS components — the visual payoff
  3. (5s) Terminal shows: "12 DS components used. 2 primitives. 1 DS gap detected: Status Badge."
  4. (5s) Cut to Build 5: same flow but faster. Terminal: "15 patterns from cache. 0 DS lookups."
  5. (3s) End card: "Mimic AI — the DS copilot that learns your system."
- **Key:** The transition from Build 1 to Build 5 is the moment. That's the learning. That's the product.
- **Tools:** Kap (macOS) for capture, Gifski for compression
- **Storage:** `/assets` in the repo

---

## Key Accounts to Engage (Rebalanced)

**Design Systems people (primary):**
`@brad_frost` (Atomic Design, the canonical DS voice), `@jina` (Design Tokens spec), `@mrmrs_` (Tachyons, design systems), `@jsngr` (design engineering), `@meganleq` (design systems), `@danmall` (design systems consulting), `@nathanacurtis` (EightShapes, DS methodology)

**AI + Design people:**
`@riaborisov` (AI design), `@tabordasilva` (Figma), `@glipsman` (Framelink — adjacent, may amplify)

**Developer/AI people (referral):**
`@alexalbert__` (Anthropic DevRel), `@swyx`, `@simonw`, `@mckaywrigley`, `@paulgauthier` (Aider)

---

## Monetization Path (Unchanged)

| Milestone | Action | Expected |
|---|---|---|
| Day 1 | GitHub Sponsors + Ko-fi | $0 (zero downside) |
| 1k stars | Sponsorware feature (DS audit dashboard, team knowledge sync) at $14/mo | $50-500/mo |
| 2k+ stars | Consulting: "I'll set up your DS-aware AI pipeline" at $150-300/hr | Reputation-based |
| Year 2 | Hosted tier or premium features | $500-2k/mo |

**The real payoff is reputation and career capital.** A 5k-star DS copilot positions the creator as a design-engineering thought leader.

---

## Anti-Patterns

- No upvote manipulation
- No cross-posting identical text
- No superlatives ("best," "fastest," "world's first")
- No hiding the Figma Pro requirement
- No engagement farming
- No ignoring critical comments
- No posting without reading sub rules
- No generic AI hype language
- **No leading with "MCP server" in designer channels** — designers don't know or care what MCP is. Lead with what it does for them.
- **No leading with rate-limit bypass for designers** — that's a developer concern. Lead with "it builds with your real components and learns your system."

---

## The One Thing That Matters Most

**Ship before month 9.** 80% of solo OSS projects are abandoned by then. Survive that and you're top-quintile by default.

Second most important: **validate with different DSs before the big push.** The claim "works with any design system" is the promise. If it breaks on someone's DS during launch week, that's the story — not the copilot narrative.

---

## Tracking

Living document. Updated every marketing session:
- Mark completed items with date
- Update channel statuses from distribution tracker
- Surface next highest-impact action
- Flag deviations from the plan

**Last updated:** 2026-04-20
