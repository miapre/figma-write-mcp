# Zero-budget growth plan for Mimic AI

Mimic AI's honest path to 1,000+ stars is a 90-day campaign anchored on three moves: **reposition the README around the one thing competitors can't copy (design-system-aware writes with a compounding cache), launch on Show HN + r/LocalLLaMA + r/ClaudeAI in the same week, and PR into every MCP awesome-list in one sitting.** Everything else is support. Framelink (GLips/Figma-Context-MCP) went 0 → ~10k stars in seven months doing essentially this, in the same niche, and Mimic has a genuinely differentiated angle — HTML **into** Figma with your real components, while Framelink goes the other direction. The realistic ceiling in 12 months at ~5 hours/week is **1,000–3,000 stars**, with a top-decile chance at Framelink-like 5k–10k if the launch GIF is strong and you're still shipping at month 9.

Budget is zero, time is ~5 hrs/week during work — so every tactic below is scored by impact-per-hour. Skip anything that looks fun but doesn't convert (LinkedIn for MCP, YouTube long-form, TikTok). Concentrate on the three highest-leverage surfaces: the README itself, Hacker News, and the MCP directory ecosystem.

## What Mimic AI actually is (and how to position it)

Mimic is an **open-source MCP server + Figma desktop plugin + Node bridge** that lets Claude Code build real Figma designs from HTML, using your published component library and design token variables. The key technical insight — and your entire marketing hook — is that **Figma's read API is rate-limited (200/day Pro, 600/day Enterprise), but the plugin write channel isn't**. Mimic minimizes reads, caches what it learns about your design system in a local JSON file, and by run 10 most builds require almost no API calls. Requirements: Figma desktop app, Figma Professional plan or higher, a personal access token, and Claude Code.

The competitive landscape is already crowded with Figma-MCP tools but **none of them occupy Mimic's niche**: Framelink (14k stars) reads Figma and writes code, Figma's official MCP reads designs (and just added `use_figma` writes, but they're metered against the read quota), cursor-talk-to-figma-mcp writes but doesn't understand design systems, html.to.design imports web pages but is paid and generic. **Mimic's one-sentence moat**: *"Most Figma MCPs read. Mimic writes — using your real design system components, with a learning cache that makes repeat builds nearly free."* That exact framing should appear in your README hero, your Show HN title, your X bio, and every post. Own it before Figma's official server matures into the same space.

The brutally honest constraint: your audience is narrow — *people who use Claude Code **and** have a Figma Pro plan **and** maintain a design system worth using*. That's tens of thousands of people globally, not millions. This is fine for stars and reputation, but it caps realistic monetization.

## Fix the repo first — this is 40% of the work

Before any promotion, the README must convert visitors in under five seconds. Studying Framelink's 95-line README (the direct competitor), Ollama, Aider, and top MCP servers reveals a consistent pattern: **logo → H3 tagline → 5-badge row → demo GIF above the fold → 30-second pitch → three-link nav (Quick start / Comparison / FAQ) → clear requirement disclosure**. Mimic's current README has the right content but buries the hook in prose.

Your hero section should look like this (copy-paste ready):

```markdown
<p align="center"><img src="./assets/logo.png" width="480"></p>
<h1 align="center">Mimic AI</h1>
<h3 align="center">The Figma MCP that uses your design system —<br/>and gets faster every run.</h3>

<p align="center">
  <img alt="npm" src="https://img.shields.io/npm/v/@miapre/mimic-ai.svg">
  <img alt="downloads" src="https://img.shields.io/npm/dm/@miapre/mimic-ai.svg">
  <img alt="MIT" src="https://img.shields.io/github/license/miapre/mimic-ai">
  <img alt="Glama" src="https://glama.ai/mcp/servers/@miapre/mimic-ai/badge">
  <img alt="stars" src="https://img.shields.io/github/stars/miapre/mimic-ai?style=social">
</p>

<p align="center"><img src="./assets/demo.gif" width="100%"></p>

<p align="center"><b>HTML in Claude Code → real Figma frames using your published components and design tokens. Writes bypass Figma's REST rate limits. A learning cache makes repeat builds nearly free.</b></p>

<p align="center"><a href="#quick-start"><b>Quick start →</b></a> · <a href="#comparison"><b>Compare vs Framelink / Figma official →</b></a> · <a href="#faq"><b>FAQ →</b></a></p>

> **Not a Figma product.** Independent, open-source MCP server for Claude Code. Requires Figma Professional plan and Figma desktop.
```

The **demo GIF is your single highest-leverage asset** — it's the thumbnail on HN, Reddit, X, and Product Hunt. Make it 15–20 seconds, 1200px wide, under 10 MB, showing a split screen: HTML prompt in Claude Code on the left, Figma canvas filling with real DS components on the right, with "Run 1: 5 reads → Run 3: 0 reads" as a terminal overlay at the end. Tools that work for free: **Kap** (macOS) or **ScreenToGif** (Windows) for capture, **Gifski** for compression. Check the raw video + GIF into `/assets` in the repo — don't rely on GitHub's opaque user-attachments URLs.

Add a **comparison table** right after the pitch. This is the single biggest conversion move Framelink didn't make, and it flips the "is this just another Figma MCP?" question:

| | Mimic | Framelink | Figma Official | html.to.design |
|---|:---:|:---:|:---:|:---:|
| Writes to Figma canvas | ✅ unlimited | ❌ | ⚠️ metered | ✅ paid |
| Uses your real components | ✅ native | ⚠️ agent infers | ✅ w/ Code Connect | ⚠️ partial |
| Binds to design token variables | ✅ | ❌ | ✅ | ❌ |
| Bypasses REST rate limits | ✅ plugin | ❌ | ❌ | ❌ |
| Learns over time | ✅ | ❌ | ❌ | ❌ |
| Open source | ✅ MIT | ✅ MIT | ❌ | ❌ |

Add an **FAQ** that answers the three questions that will otherwise become angry issues: *Why Figma Pro? Why Claude Code only? How is this different from Figma's official MCP?* Add a **star-history chart** at the bottom (`api.star-history.com/svg?repos=miapre/mimic-ai`). Enable **GitHub Discussions** (indexed by Google, becomes long-tail SEO). Add **20 repository topics** — the keyword slots that drive in-GitHub search: `mcp`, `model-context-protocol`, `mcp-server`, `figma`, `figma-plugin`, `figma-api`, `figma-mcp`, `claude-code`, `claude`, `anthropic`, `design-systems`, `design-tokens`, `html-to-figma`, `design-to-code`, `ai-agent`, `ai-tools`, `design-automation`, `websocket`, `developer-tools`, `nodejs`. Upload a **1280×640 social preview PNG** in Settings → General; this triples link-preview CTR on X, Slack, Discord, and iMessage.

Rewrite the GitHub "About" description to be keyword-dense: *"MCP server for Claude Code that builds Figma designs from HTML using your published components and design tokens. Learns your design system and gets faster every run."* That single field is the second-heaviest ranking signal in GitHub search after the repo name.

## Week 1 — the launch week

Launching all at once on a single Tuesday creates the wave; scattering over weeks wastes each moment. Here's the minute-by-minute plan.

**Tuesday 8:30 AM Pacific (15:30 UTC)** — Post to Hacker News. Title exactly: `Show HN: Mimic AI – HTML to Figma via Claude Code, using your design system`. No superlatives, no "introducing," no "first." Submit at https://news.ycombinator.com/submit. Within 60 seconds, post a first comment explaining why you built it, your stack, the two or three real limitations, and that you'd like to be roasted. Tone is engineer-to-engineer, humble, specific — this is the Fly.io / Aider formula. Reply to every comment within 5–10 minutes for the next four hours. Do **not** message friends to upvote — HN's vote-ring detection will kill the post. If it doesn't catch, you're allowed one resubmission in 24–48 hours.

**Tuesday 11 AM PT** — Post to **r/ClaudeAI** (https://reddit.com/r/ClaudeAI, ~747k subs). Title: *"I built an MCP that turns HTML into Figma using your real design system components — open source, would love feedback."* Self-post with embedded GIF, GitHub link in the body, "why I built this" narrative in 150 words. Flair: project/showcase if available.

**Tuesday 6 PM PT** — Post to **r/LocalLLaMA** (https://reddit.com/r/LocalLLaMA, ~688k subs). This is the single most important sub for MCP tools. Lead with the local/no-cloud angle: *"Open-source MCP server: HTML → Figma via Claude Code, runs 100% locally, learns your design system across runs."* Late-night/early-AM PT posting gets the European morning wave.

**Wednesday** — Post to **r/mcp** (small but on-topic, highest conversion per reader), **r/SideProject** (~131k, promo-friendly), **r/coolgithubprojects** (purpose-built for GitHub repos, use format `[Node.js] Mimic AI — HTML to Figma via Claude Code`). Share in the **Anthropic Discord** (https://discord.com/invite/6PPFFzqPDZ, ~85k members, `#showcase` or `#mcp` channel — read pinned rules first).

**Thursday** — Post a 5-tweet thread on X with the demo GIF as the first tweet. Tag `#buildinpublic` and `#MCP`. Reply to any tweet from `@alexalbert__` (Anthropic DevRel), `@swyx`, `@simonw`, or `@mckaywrigley` in the last 24 hours with a genuine reply — don't pitch Mimic, just engage. If Graham Lipsman (`@glipsman`, Framelink creator) tweets that week, reply with a "love your work, built the inverse direction here's how it differs" note. He will likely amplify. Post a shorter version on LinkedIn with the image and GitHub link in the first comment.

**Thursday–Friday** — Submit to every MCP directory in one sitting. This takes about 90 minutes and seeds discoverability for the next 12 months: **Official MCP Registry** via `mcp-publisher` CLI (https://github.com/modelcontextprotocol/registry), **mcp.so** (GitHub issue at https://github.com/chatmcp/mcp-directory/issues), **Smithery** (https://smithery.ai/new), **PulseMCP** (https://www.pulsemcp.com/submit), **MCPHub** (https://mcphub.tools/), **Cursor Directory** (https://cursor.directory/plugins/new), **mcpservers.org** (https://mcpservers.org/submit). Mimic is already on Glama (https://glama.ai/mcp/servers/miapre/mimic-ai) — claim the listing via the admin panel to update metadata.

**Friday–Sunday** — Open one PR each to the three canonical awesome lists: **punkpeye/awesome-mcp-servers** (84k stars, the canonical list — read CONTRIBUTING.md and submit under "Design Tools"), **appcypher/awesome-mcp-servers**, **modelcontextprotocol/servers** (third-party section). Also: **jqueryscript/awesome-claude-code**, **e2b-dev/awesome-ai-agents**, **Shubhamsaboo/awesome-llm-apps**, and **react-figma/awesome-figma**. One PR with one-line description matching the list's style, alphabetical placement, no self-promo language in the PR description itself. wong2's list only accepts form submissions at https://mcpservers.org/submit, not PRs.

Expected Week 1 outcome if the GIF is good and the README converts: **100–500 stars, 10–50 real users, one or two maintainers of other Figma/MCP tools will find you**.

## Month 1 — consolidate and compound

After launch week, the star curve flattens hard. The job now is to build recurring surfaces that catch long-tail traffic.

**Ship v0.2** with one genuinely new capability and write it up. This gives you a second HN submission window (new feature = not a dupe) and new Reddit angles. Candidate features with high marketing leverage: Cursor support (doubles your addressable audience overnight and triggers the Cursor subreddit + cursor.directory listing), a diagram of the learning-cache stats ("my design system went from 23 reads/build to 0 by run 8"), or a bundled Figma Community plugin companion (unlocks the entire Figma Community as a funnel).

**Publish one long-form blog post per week** on your own domain (or a `github.io` Pages site — worth standing up; Framelink does this with framelink.ai and it's where all GitHub link traffic funnels), then syndicate to **dev.to** with `canonical_url` set to your original, then to **Hashnode** a few days later with the same canonical. Never publish to Medium first. Tag heavily: `#showdev`, `#ai`, `#opensource`, `#figma`, `#claude`, `#mcp` on dev.to. The posts that will rank in 3–6 months and feed a steady star drip:

1. *"How to hit Figma's MCP rate limit in 10 minutes (and what to do about it)"* — targets a fast-growing pain point
2. *"Framelink vs Figma's official MCP vs Mimic: which Figma MCP server should you use?"* — comparison posts rank forever
3. *"Teaching an LLM your design system: persistent knowledge for Figma agents"* — the learning-cache angle
4. *"From v0/Lovable prototype to Figma: reverse-engineering AI prototypes back into your design system"* — rides the AI prototyping trend
5. *"Every way to get HTML into Figma in 2026, compared"* — head-term roundup

**Build in public on X daily** with a 2-minute micro-post (shipped a fix, weird bug, small win). One longer thread per week. One reply per day on a high-reach account in AI dev tools (`@alexalbert__`, `@swyx`, `@simonw`, `@mckaywrigley`, `@jmorganca`, `@rauchg`, `@paulgauthier`) or design-systems (`@mrmrs_`, `@jsngr`, `@meganleq`). The 80/20 rule applies: 80% of your replies add value without mentioning Mimic; 20% surface it when genuinely relevant. Use Typefully or similar to batch-schedule — total weekly X time should be ~3 hours, not 10.

**Extend directory submissions** through the launch-platform tier for steady trickle traffic: **Peerlist Launchpad** (free, weekly Monday launches, https://peerlist.io/launchpad), **Microlaunch** (https://microlaunch.net/submit), **Launching Next** (https://www.launchingnext.com/submit/), **SaaSHub** (https://www.saashub.com/submit-product), **DevHunt** (https://devhunt.org/submit, 6-week free queue), **Side Projectors**, **OpenAlternative** (https://openalternative.co/submit, OSS-focused). Skip **BetaList** (effectively paid-gated now) and **Uneed/Fazier** unless you're willing to engage in their badge-exchange or 30-ish-dollar skip-queue mechanics. Also submit to **AlternativeTo** (https://alternativeto.net/new-app/) as an alternative to Framelink / html.to.design / Figma official MCP — the backlink and category placement pay off. **Libraries.io**, **Socket.dev**, **Openbase**, and **LibHunt** auto-index from npm, so just make sure your `package.json` has a strong description and keywords.

**Product Hunt launch** in week 3 or 4, not week 1. Submit at https://www.producthunt.com/posts/new, launch Tuesday–Thursday at 12:01 AM PT. Tagline (60 chars): *"HTML to Figma, using your actual design system."* Don't pay for anything. Ask five engaged GitHub issue/PR contributors to be your first hunters — no link-sharing in DMs or your post gets deranked for vote manipulation. A realistic PH outcome for a niche MCP tool: 100–300 upvotes, ~50 extra stars, and a long-tail SEO boost from the PH page itself.

## Quarter 1 — content flywheels and community presence

By month 3, one-shot launches stop working. Growth now depends on **recurring content** (benchmarks, release notes, tutorial series) and **compound community presence**.

**Second HN submission** around week 6–8 — allowed as long as it's substantively new. Aider's Paul Gauthier built a habit of posting a new Show HN every time SWE-bench scores improved; Mimic's equivalent is: *"Show HN: Mimic AI v0.3 — Cursor support and 10× faster repeat builds."* Or a benchmark post: *"I built 50 real landing pages with Mimic — here's the fidelity, failure modes, and cache stats."* Benchmark posts are HN catnip and give you shareable numbers that live in Twitter threads for months.

**Join Discords and lurk before posting**. Highest-signal: **Friends of Figma** (https://discord.com/invite/figma, ~33k), **MCP community Discord** (linked from modelcontextprotocol/servers README), **r/LocalLLaMA Discord** (via sub sidebar), **Indie Hackers** (indiehackers.com). Answer three to five questions in each before mentioning Mimic. Post only in designated `#show-and-tell` or `#projects` channels. Total Discord time: 30 minutes/week max — it's low-ROI compared to Reddit and HN but pays off when someone asks "what's the best HTML-to-Figma tool?" and your name is top of mind.

**Designer community requires translation.** Mimic solves a designer problem but speaks developer. When posting to r/Figma, r/FigmaDesign, r/UXDesign, r/DesignSystems, or the Figma Forum at https://forum.figma.com/, lead with the designer pain (*"I built a Figma that reads from my coded prototypes instead of requiring me to rebuild them by hand"*), not the tech stack. On Designer News (https://www.designernews.co/stories/new), Sidebar.io (https://sidebar.io/submit), and Prototypr.io (https://prototypr.io/toolbox/submit), submit the blog post or landing page, not the raw GitHub repo — designers bounce off `.md` files.

**Consider shipping a thin Figma Community plugin companion** (free to publish at https://www.figma.com/community). It would wrap the install flow or act as a browser for the learned-knowledge JSON. This unlocks Figma's own ~1M+ plugin-directory traffic funnel and becomes a trust badge ("published on Figma Community"). One week of effort, very high leverage.

**Recurring content cadence** (~5 hrs/week total):
- 1 blog post every 10 days, cross-posted dev.to + Hashnode with canonical
- 1 X thread per week (usually a summary of the blog post with GIFs)
- 1 Reddit post per 2 weeks in a different sub, framed for that community
- 1 benchmark or release note every 3–4 weeks
- Daily 2-minute X micro-posts

## Antipatterns — what will actively hurt you

- **Upvote manipulation on HN/Reddit**: messaging friends to upvote, posting the URL in Slack/Discord with an implicit ask. Vote-ring detection is sophisticated and the penalty is shadow-removal. Tell people the project exists; let them find it on `/newest` themselves.
- **Cross-posting the same text to multiple subs in one day**: Reddit suspends accounts for this. Rewrite each post for that sub's culture, space by 24–48 hours.
- **Superlatives in titles** ("best", "fastest", "world's first"): auto-suppressed by HN and downvoted by r/programming on sight.
- **Hiding the Figma Pro requirement**: users who hit the paywall leave angry issues and one-star comments. State it in the README, the install script, every post. Mimic's current README buries this — move it to the top.
- **Posting in r/LLMDevs, r/MachineLearning without reading rules**: r/LLMDevs prohibits self-promo; r/MachineLearning restricts project posts to weekend self-promo threads. You will be banned.
- **Engagement-farming replies**: spamming "🔥" or "This" on every influencer post is transparently self-interested and gets muted.
- **Ignoring the first critical HN comment**: silence doubles down the critic. Reply fast and humbly.

## Monetization reality check

You can't meaningfully monetize Mimic for a while, and maybe never — but that's the honest expectation, not a failure. Real earnings data from GitHub Sponsors disclosures and from maintainers like Caleb Porzio, azu, and Sindre Sorhus suggest hobby OSS projects at 1k–5k stars typically earn **$50–$500/month**, at 5k–10k stars **$500–$3k/month**, and past that require a paid SaaS layer or Caleb Porzio's "Sponsorware" model (specific features gated to paying sponsors, merged back when funding goal hits).

Your audience already pays for Figma Pro ($15/mo) and Claude Code ($20/mo minimum) — they can afford a $5–$20/month supporter tier. Realistic path: **GitHub Sponsors and Ko-fi set up day one** (zero downside), **Sponsorware feature** after 1k stars (e.g., enterprise design-token adapters or bulk file regeneration gated behind a $14/mo tier), **consulting at $150–300/hr** once the reputation exists ("I'll set up your design-system-aware AI pipeline"), and **a hosted tier in year 2** if the architecture shifts (a web service that takes HTML + Figma token and produces a Figma file, $10–$20/mo). Dual-license and enterprise contracts are unrealistic until 5k+ stars and kill contributor momentum. **Do not quit your day job for this.** Realistic combined year-1 income: $0–$2k. Realistic year-2 if it works: $500–$2k/month.

The deeper truth: for this specific tool, **reputation and career capital are the real payoff**, not Sponsor revenue. A 5k-star OSS project on your GitHub profile has outsized value for design-engineering roles, speaking slots, and future co-founding opportunities — orders of magnitude more than the Sponsor income.

## Realistic expectations, by month

Base rates for solo OSS AI tools launching in a competitive-but-not-saturated niche (MCP is hot but narrow) with ~5 hrs/week of promotion, assuming a strong launch week and consistent shipping: **Month 1: 100–500 stars, 10–50 users. Month 3: 300–1,500 stars, 50–300 users, first $0–100/mo on Sponsors. Month 6: 1k–5k stars top-decile, 300–1k typical; first plausible Sponsorware revenue. Month 12: 3k–10k stars if it clicked and you kept shipping (Framelink's trajectory), 500–3k stars typical, under 500 if you stopped shipping before month 6.** The single strongest predictor of where you land isn't code quality or marketing cleverness — it's **whether you're still shipping at month 9**. Eighty percent of solo OSS projects are abandoned by then. Survive that and you're top-quintile by default.

## The prioritized 90-day action list

For the free-time-during-work constraint, here's the exact order of operations, ranked by impact-per-hour:

**This week (5 hrs total):** Rewrite the README hero with the structure above (1.5 hr). Record and compress the demo GIF (1.5 hr). Add the 20 repo topics, social preview image, comparison table, FAQ, and Discussions toggle (1 hr). Write the Show HN first-comment draft and the r/LocalLLaMA/r/ClaudeAI post drafts (1 hr).

**Launch week (5 hrs):** Tuesday 8:30 AM PT Show HN, 11 AM r/ClaudeAI, 6 PM r/LocalLLaMA. Wednesday r/mcp + r/SideProject + r/coolgithubprojects + Anthropic Discord. Thursday X thread + LinkedIn. Thursday/Friday 90-min directory submission sprint (MCP Registry, mcp.so, Smithery, PulseMCP, MCPHub, Cursor Directory, mcpservers.org). Weekend: three awesome-list PRs.

**Weeks 2–4 (5 hrs/wk):** First long-form blog post + dev.to/Hashnode cross-post. Daily X micro-posts. Second tier of directory submissions (Peerlist, Microlaunch, Launching Next, AlternativeTo, SaaSHub, DevHunt). Ship v0.2 with one meaningful feature. Product Hunt launch end of week 3 or 4.

**Months 2–3 (5 hrs/wk):** One blog post every 10 days. Second HN submission tied to a new benchmark or feature. Design-community outreach (r/Figma, r/DesignSystems, Designer News, Sidebar.io, Figma Forum). Consider shipping the thin Figma Community plugin companion. Join and lurk in Friends of Figma, MCP Discord, r/LocalLLaMA Discord. Month 3: publish a comparison benchmark post ("50 pages, 4 tools") — the single most shareable piece of content you'll create.

The whole plan is designed so **missing any single post or directory doesn't break it**. The README rewrite plus the launch-week three-Reddit-HN trifecta plus the MCP-directory sweep deliver maybe 70% of the total value. Everything else compounds on top. Focus on the 70%; let the rest happen when time permits.

## The one thing that matters most

Ship a genuinely great 15-second demo GIF before you do anything else. Framelink's entire trajectory traces to one clean before/after clip that made people stop scrolling on X in February 2025. Your technical work already supports a better demo than Framelink's (you have the visual payoff of Figma filling in live with real components, which is viscerally impressive). If the GIF doesn't make a design-engineer lean toward their screen in the first three seconds, no amount of Reddit cross-posting will save it — and if it does, the rest of this plan is a formality.