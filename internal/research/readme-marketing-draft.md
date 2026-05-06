# Mimic AI

**HTML to Figma — using your design system's real components and tokens.**

Give Mimic an HTML prototype. It searches your design system, matches components and tokens, and builds the equivalent in Figma — using your real library assets, not approximations. Same HTML, different DS = different output. Every build, it learns your conventions and tells you what your DS is missing.

![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)
![Node.js: v20.6+](https://img.shields.io/badge/node-%3E%3D20.6-brightgreen)
![Platform: macOS / Windows](https://img.shields.io/badge/platform-macOS%20%7C%20Windows-lightgrey)
[![Glama](https://glama.ai/mcp/servers/@miapre/mimic-ai/badge)](https://glama.ai/mcp/servers/@miapre/mimic-ai)
[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_MCP-0078d4?logo=visualstudiocode&logoColor=white)](vscode:mcp/install?%7B%22name%22%3A%22mimic-ai%22%2C%22type%22%3A%22stdio%22%2C%22command%22%3A%22npx%22%2C%22args%22%3A%5B%22-y%22%2C%22%40miapre%2Fmimic-ai%22%5D%7D)
[![Install in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Install_MCP-24bfa5?logo=visualstudiocode&logoColor=white)](vscode-insiders:mcp/install?%7B%22name%22%3A%22mimic-ai%22%2C%22type%22%3A%22stdio%22%2C%22command%22%3A%22npx%22%2C%22args%22%3A%5B%22-y%22%2C%22%40miapre%2Fmimic-ai%22%5D%7D)

> Open-source MCP server. Works with Claude Code, Cursor, VS Code, Windsurf, JetBrains. Not a Figma product.

---

<!-- TODO: Replace with actual screenshot/GIF of same HTML → 3 different DS outputs side by side -->
*Demo: same landing page HTML built with three different design systems — LayerLens (team library), Material UI (community), HeroUI (community). Same structure. Different tokens. Different output.*

---

## What makes it different

| | Generic AI | Mimic |
|---|---|---|
| **Components** | Draws rectangles that look like buttons | Inserts your published Button with the correct variant, size, and color |
| **Tokens** | Hardcodes hex values | Binds your variables — colors update when you re-publish |
| **Typography** | Raw font size and weight | Applies your text styles — your typeface, your scale |
| **Spacing** | Raw pixel values | Binds your spacing variables where available |
| **Layout** | Fixed frames, manual positioning | Everything uses auto-layout — resizable, responsive, maintainable |
| **Learning** | Starts fresh every time | Remembers your corrections. Build 10 is faster than build 1. |

---

## Works with any design system

| DS type | Support |
|---|---|
| **Team library** (components + tokens) | Full — components, text styles, color/spacing/radius variables |
| **Team library** (components only) | Components used, raw fallback for tokens. Report recommends adding variables. |
| **Community library** (Material UI, HeroUI, iOS kits) | Full — components import normally, variables via key-based import |

---

## How it learns

Mimic keeps a local knowledge file. Each build loads what it knows and saves what it discovered.

| Build | What happens |
|---|---|
| **1st** | Scans your library. Caches every component mapping. |
| **3rd** | Uncorrected patterns promoted to VERIFIED — skipped on future builds. |
| **10th+** | Most patterns cached. Builds are nearly instant. |

Correct it once: *"That's wrong — use Button/Primary."* The mapping updates permanently.

Your DS changes? Mimic detects it. New components surface. Removed components fall back gracefully. The DS is always the source of truth.

After every build: what was used, what was learned, what's missing from your DS — framed as questions, not commands.

---

## Try it

> **Requirements:** [Node.js](https://nodejs.org/) v20.6+, [Figma desktop](https://www.figma.com/downloads/), Professional plan or above.

### 1. Install

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/miapre/mimic-ai/main/install.sh)
```

Downloads Mimic, installs dependencies, asks for your Figma token, registers the tool. About a minute.

### 2. Install the Figma plugin

**Plugins → Development → Import plugin from manifest** → select `~/mimic-ai/plugin/manifest.json`.

### 3. Start (each session)

```bash
cd ~/mimic-ai && npm run bridge
```

Then in Figma: **Plugins → Development → Mimic AI → Run**. Badge shows **● ready**.

### 4. Enable your design system

Assets panel → Team library icon → toggle your DS on. Once per file.

> **Community libraries** (Material UI, HeroUI, iOS kits) work out of the box. Enable the library and Mimic handles the rest.

### 5. Build

Ask your AI assistant:

> *"Build this HTML in Figma on the 'Screens' page. Use my design system."*

Or describe what you want:

> *"Dashboard: 4 KPI cards, data table, donut chart. Use my nav shell and spacing-xl gaps."*

---

## MCP client setup

Mimic uses MCP (Model Context Protocol). The build protocol is optimized for **Claude Code** — other clients work but may not follow the full governance lifecycle.

<details>
<summary><strong>Claude Code</strong></summary>

```json
{
  "mcpServers": {
    "mimic-ai": {
      "command": "npx",
      "args": ["-y", "@miapre/mimic-ai"]
    }
  }
}
```

</details>

<details>
<summary><strong>Cursor</strong></summary>

Add to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "mimic-ai": {
      "command": "npx",
      "args": ["-y", "@miapre/mimic-ai"]
    }
  }
}
```

</details>

<details>
<summary><strong>VS Code</strong></summary>

Click the install badge above, or add to settings:

```json
{
  "mcp": {
    "servers": {
      "mimic-ai": {
        "command": "npx",
        "args": ["-y", "@miapre/mimic-ai"]
      }
    }
  }
}
```

</details>

<details>
<summary><strong>Windsurf</strong></summary>

```json
{
  "mcpServers": {
    "mimic-ai": {
      "command": "npx",
      "args": ["-y", "@miapre/mimic-ai"]
    }
  }
}
```

</details>

<details>
<summary><strong>JetBrains</strong></summary>

Settings → Tools → AI Assistant → MCP Servers → Add:

```json
{
  "mimic-ai": {
    "command": "npx",
    "args": ["-y", "@miapre/mimic-ai"]
  }
}
```

</details>

All clients need the bridge running (Step 3) and the Figma plugin active.

---

## How it works

You talk to your AI assistant. Mimic talks to Figma. Everything happens locally — your design data never leaves your machine.

- **Creating is unlimited** — frames, components, token bindings have no rate limit.
- **Inspecting is limited** — reading your library uses Figma's daily quota (200 Professional / 600 Enterprise). Mimic caches aggressively to stay well under.
- **Token bindings are real** — nodes use your actual variables. Update a token, re-publish, nodes update.
- **Everything uses auto-layout** — frames resize, content reflows, nothing is absolute-positioned. The same build quality you'd produce by hand.

---

## What gets checked

Every build runs through automated checks before it's reported as done:

- Every text node uses a text style from your DS
- Every color is bound to a variable — not hardcoded
- Every frame uses auto-layout — resizable, not static
- Spacing bound to your spacing variables where available
- Content matches the source exactly — nothing paraphrased or invented
- Your DS components used wherever the library has a match
- Charts built with real data, not placeholder shapes
- Build report tells you what was used, what was learned, and what your DS is missing

46 rules enforced automatically. Full spec: [`GOLDEN_RULES.md`](GOLDEN_RULES.md).

---

<details>
<summary><strong>Available tools (35)</strong></summary>

**Status & learning:** `mimic_status`, `mimic_discover_ds`, `mimic_ai_knowledge_read`, `mimic_ai_knowledge_write`

**DS setup:** `figma_preload_styles`, `figma_preload_variables`, `figma_set_session_defaults`, `figma_list_text_styles`, `figma_read_variable_values`

**Build:** `figma_create_frame` (with maxWidth), `figma_create_text`, `figma_create_rectangle`, `figma_create_chart`, `figma_insert_component`, `figma_batch`

**Edit:** `figma_set_component_text`, `figma_set_text`, `figma_set_node_fill`, `figma_set_layout_sizing` (with maxWidth/minWidth), `figma_set_variant`, `figma_set_visibility`, `figma_swap_main_component`, `figma_replace_component`, `figma_move_node`, `figma_delete_node`

**Inspect & QA:** `figma_get_node_props`, `figma_get_node_children`, `figma_get_node_parent`, `figma_get_text_info`, `figma_get_component_variants`, `figma_get_selection`, `figma_select_node`, `figma_get_page_nodes`, `figma_get_pages`, `figma_change_page`, `figma_validate_ds_compliance`

</details>

<details>
<summary><strong>Figma setup details</strong></summary>

### Desktop app required
Browser Figma won't work — the bridge needs a local network connection. [Download Figma desktop](https://www.figma.com/downloads/)

### Personal Access Token
The bridge needs a token to look up published component keys.

1. Figma → **Profile** → **Settings** → **Personal access tokens** → **Generate new token**
2. Read access is enough
3. Copy immediately — Figma only shows it once
4. Paste when the install script asks

### Publishing your DS
Components and tokens must be in a separate file, published as a team library. Re-publish after changes.

### Figma plan
Professional or above. Free plan can't publish libraries.

</details>

---

## Privacy

Runs entirely on your machine. No design data leaves your machine. The only outbound call is to the Figma REST API for published component keys — the same call Figma's own plugins make.

---

## Known constraints

- **Figma Professional plan required** — free plan can't publish libraries
- **First-build font caching** — if text styles don't apply on the first build, re-run. Second attempt succeeds.
- **npx mode limitations** — `npx -y @miapre/mimic-ai` doesn't set `FIGMA_ACCESS_TOKEN`. Use the full installer for team library support.
- **Claude-optimized governance** — 46 rules and learning reports work best with Claude Code. Other clients get the tools but may not follow the full protocol.
- **Chart arcs use absolute positioning** — donut slices and line paths use pixel math. Everything else in charts (bars, labels, legends) uses auto-layout.

---

## License

MIT
