# Validation Ground Truth — Component Maps per DS

Generated from Figma MCP search on 2026-04-20. These are the expected component matches for each HTML test fixture, per design system.

---

## DS 1: LayerLens Theme (Rich — closest to "mature DS")

| HTML Pattern (Fixture) | Expected DS Component | Component Key | Notes |
|---|---|---|---|
| `<button class="btn-primary">` (F1-F5) | Buttons/Button | `aa5d03...` | Primary variant |
| `<button class="btn-secondary">` (F1-F5) | Buttons/Button | `aa5d03...` | Secondary/outline variant |
| `<span class="badge">` (F1, F3) | Badge | `be5842...` | Semantic color variants (success/warning/neutral) |
| `<span class="status-badge">` (F3, F5) | Badge | `be5842...` | Status-specific color |
| `<span class="role-badge">` (F3) | Badge | `be5842...` | Neutral/gray variant |
| `<input type="text/email">` (F2) | Input field | `79a3ea...` | Default state |
| `<select>` (F2, F5) | Input dropdown | `4254d8...` | Or Dropdown `e140be...` |
| Checkbox + label (F2) | Checkbox group item | `9a9288...` | Checked/unchecked states |
| `<table>` header cells (F3, F5) | Table header cell | `fc351f...` | Per column |
| `<table>` data cells (F3, F5) | Table cell | `309ce5...` | Multiple variant types |
| Filter bar (F3, F5) | Table filters | `5cda64...` | With dropdowns + search |
| Tabs (F5) | Horizontal tabs | `18b942...` | Active/inactive states |
| Page header (F4, F5) | Page header | `8ad1c8...` | Title + description + actions |
| Card header (F4 stat cards) | Card header | `5c432a...` | If applicable |
| Divider (F2 form actions border) | Content divider | `d4ab45...` | |
| **EXPECTED GAPS** | | | |
| Sidebar navigation (F4) | — | — | No sidebar nav component; primitive expected |
| Pagination (F3, F5) | — | — | No pagination component found; primitive expected |
| KPI/Stat card (F4, F5) | — | — | No stat card component; primitive expected |
| Activity feed item (F4) | — | — | No activity list component; primitive expected |
| Avatar circle (F3, F4) | — | — | Search needed — may exist |

**Summary: ~14 component matches, ~4-5 expected gaps. Rich DS with good coverage.**

---

## DS 2: Untitled UI PRO STYLES (Comprehensive — "enterprise DS")

| HTML Pattern (Fixture) | Expected DS Component | Component Key | Notes |
|---|---|---|---|
| `<button class="btn-primary">` (F1-F5) | Buttons/Button | `916550...` | Primary variant |
| `<span class="badge">` (F1, F3) | Badge | `31eccb...` | Color variants |
| `<input type="text/email">` (F2) | Input field | `4cd5ec...` | Default state |
| Textarea (if needed) | Textarea input field | `970cd0...` | |
| `<table>` (F3, F5) | Table | `0ea67b...` | Full table component |
| `<table>` header (F3, F5) | Table header | `f71171...` | |
| `<table>` cells (F3, F5) | Table cell | `a5a2b3...` | |
| Blog/content card (F1) | Blog post card | `33a529...` | May match card pattern |
| **EXPECTED GAPS** | | | |
| Sidebar navigation (F4) | — | — | Untitled UI has nav patterns but need to search |
| Tabs (F5) | — | — | Need to search specifically |
| Pagination (F3, F5) | — | — | Need to search |
| Checkbox (F2) | — | — | Need to search |
| Select/dropdown (F2, F5) | — | — | Need to search |
| KPI/Stat card (F4, F5) | — | — | Likely primitive |
| Form structure (F2) | — | — | Likely primitive |

**Summary: ~8 strong matches confirmed, several need deeper search. Large component library — likely more matches than shown.**

---

## DS 3: Material 3 Design Kit (Different paradigm — "mobile-first DS")

| HTML Pattern (Fixture) | Expected DS Component | Component Key | Notes |
|---|---|---|---|
| `<button>` (F1-F5) | Connected button group | `ab725a...` | M3 uses "Filled button," "Outlined button" etc. — need specific search |
| Card (F1) | Stacked card / Horizontal card | `a79bb6...` / `78d3f1...` | M3 card paradigm |
| Tabs (F5) | tab (individual) | `77b591...` | M3 tab component |
| `<input>` (F2) | Docked input date picker | `a4270e...` | M3 has text fields — need specific search |
| **EXPECTED GAPS** | | | |
| Table (F3, F5) | — | — | M3 doesn't have a web table component — significant gap |
| Pagination (F3, F5) | — | — | No pagination in M3 |
| Sidebar navigation (F4) | — | — | M3 has Navigation rail/bar (mobile), not web sidebar |
| Badge (F1, F3) | — | — | M3 has Badge but need to verify it's available |
| Status badges (F3, F5) | — | — | Likely no direct match |
| Filter bar (F3, F5) | — | — | No web filter component |
| KPI card (F4, F5) | — | — | No stat card |

**Summary: Material 3 is mobile-first. Web patterns (tables, pagination, sidebar nav) will mostly be primitives. This DS tests Mimic's ability to gracefully fall back and still produce useful output. Expect 3-5 component matches and 8+ gaps — the gap report will be rich.**

---

## Test Value per DS

| DS | Component richness | Expected matches | Expected gaps | What it validates |
|---|---|---|---|---|
| **LayerLens** | High (web-focused) | 14+ | 4-5 | Best-case: rich DS, high component reuse, good learning opportunity |
| **Untitled UI** | Very high (comprehensive) | 8-12+ | 3-5 | External DS: proves DS-agnosticism, tests discovery on unfamiliar DS |
| **Material 3** | Medium (mobile-first) | 3-5 | 8+ | Worst-case: DS mismatch, tests graceful fallback, rich gap report |

This gradient (rich → medium → sparse-for-web) is ideal for testing the full range of Mimic's behavior.
