---
name: quick-research-card
description: Triggered when the user types "QRC [company name or ticker]". Generates a structured company research flashcard covering daily price catalyst, industry & business overview, technology moat, AI stack position, financials, recent developments, competitors & risks, and insider activity, then renders it as a downloadable PNG image. Must start immediately when the user says "QRC" — do not wait for additional confirmation.
---

# Quick Research Card (QRC) Skill

## Trigger

The user types `QRC [company name or ticker]` to trigger this skill. Examples:
- `QRC NVDA`
- `QRC TSMC`
- `QRC Palantir`
- `QRC LWLG`

---

## Execution Steps

### Step 1 — Parallel Search (4 searches, run in parallel when possible)

Search in the following order, keeping each query concise (1–6 words):

1. **Daily price action** — `[TICKER] stock news today [current year]`
2. **Company fundamentals** — `[Company name] business overview technology AI [current year]`
3. **Financials** — `[TICKER] revenue EPS earnings quarterly [current year]`
4. **Insider activity** — `[TICKER] insider trading buying selling SEC [current year]`

> If the user provides price context when triggering (e.g. "QRC NVDA up 15% today"), narrow the first search to that direction.

---

### Step 2 — Render HTML Flashcard

Use the template below to generate HTML, populating it with search results.

**Color scheme:**
- Daily move is **up** → catalyst box uses green background `#EAF3DE`, border `#C0DD97`, title color `#27500A`
- Daily move is **down / flat** → catalyst box uses red background `#FCEBEB`, border `#F7C1C1`, title color `#A32D2D`
- Middle three columns (industry / tech moat / AI position) + financial metrics → gray background `#f5f5f3`
- Bottom three sections (recent news / competitors & risks / insider activity) → light amber background `#faf8d7`, title color `#854F0B`

**Section title styling:**
- Middle section titles: `font-weight: 600; color: #185FA5` (blue, bold)
- Bottom section titles: `font-weight: 600; color: #854F0B` (amber, bold)
- Catalyst box title: `font-weight: 600`, color changes with up/down (see above)

**Section content requirements:**

| Section | Content |
|---|---|
| ⚡/▼ Daily Price Catalyst | Date, price range, % change, 1–3 numbered catalysts with specific explanations |
| Industry · Business · Customers | One sentence describing the core business model, then customer-type tags |
| Technology Moat & Scarcity | Core technology moat, list 2–4 validated metric tags (green) |
| AI Stack Position | Identify the company's position in the AI infrastructure chain (e.g. optical interconnect, chip testing, model layer, compute layer), use purple tags |
| Financial Metrics (2 rows × 4 cells) | Most recent 3 quarters revenue + EPS, plus gross margin / cash / market cap and other key metrics |
| Recent Developments | 2–3 recent news items, each with bold title + one-line explanation + date |
| Competitors & Risks | Left column: 2–4 main competitors; Right column: 3–4 key risks |
| Insider Activity | One paragraph summary, distinguish voluntary open-market sales vs. RSU tax-related sales; add green tag if neutral |

**Header specification:**
- Top-left: Full company name (e.g. "Teradyne Inc.", "NVIDIA Corporation") + ticker badge (blue)
- Subtitle: Industry · Headquarters · Year founded · Employee count
- Top-right: Market cap + 52-week range

---

### Step 3 — Export to PNG

Use Playwright to render the HTML to PNG:

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page(
        viewport={"width": 908, "height": 1200},
        device_scale_factor=3
    )
    # Write HTML file to /home/claude/[ticker]_flashcard.html
    page.goto("file:///home/claude/[ticker]_flashcard.html")
    page.wait_for_timeout(500)
    height = page.evaluate("document.body.scrollHeight")
    page.set_viewport_size({"width": 908, "height": height + 48})
    page.screenshot(
        path="/mnt/user-data/outputs/[TICKER]_QRC.png",
        full_page=True
    )
    browser.close()
```

Output filename convention: `[TICKER]_QRC.png` (e.g. `NVDA_QRC.png`)

---

### Step 4 — Present Results

Use the `present_files` tool to deliver the PNG to the user for download.

Then add **1–3 sentences** briefly summarizing the key catalyst — do not repeat information already shown on the flashcard.

---

## Important Notes

- **Company name**: The header must always show the company's **full legal name** (e.g. "Lightwave Logic Inc."), not just the ticker or abbreviation
- **Date**: The catalyst section date must be today's actual date, never a placeholder
- **Data freshness**: Financial data should prioritize the most recent quarterly report, labeled by quarter (e.g. Q1'26)
- **Insider sales**: Distinguish RSU tax-related sales (neutral) vs. voluntary open-market sales (flag separately)
- **No daily catalyst**: If no clear daily catalyst is found, change the section title to "Recent Price Context" and describe the recent trend background
- **Search failure**: If any data cannot be found, mark the corresponding cell "Data unavailable" — never fabricate
- **Language**: Output in Chinese by default, retaining English for financial terms (EPS, PDK, BEOL, etc.). Switch to English if the user requests it.

---

## Full HTML Template

The HTML file structure follows this framework — populate with real data:

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    background: #f5f5f3;
    padding: 24px;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Helvetica Neue", Arial, sans-serif;
    color: #1a1a1a;
  }
  .fc {
    background: #ffffff;
    border: 0.5px solid rgba(0,0,0,0.12);
    border-radius: 12px;
    padding: 16px 18px;
    font-size: 12px;
    color: #555;
    line-height: 1.5;
    max-width: 860px;
    margin: 0 auto;
  }
  /* Header */
  .hdr { display:flex; justify-content:space-between; align-items:flex-start; padding-bottom:10px; border-bottom:1px solid rgba(0,0,0,0.1); margin-bottom:10px; }
  .title { font-size:20px; font-weight:600; color:#111; display:flex; align-items:center; gap:8px; margin-bottom:4px; }
  .badge { font-size:11px; font-weight:500; padding:2px 8px; border-radius:4px; background:#E6F1FB; color:#185FA5; border:0.5px solid #B5D4F4; }
  .sub { font-size:11px; color:#888; }
  .hdr-right { text-align:right; flex-shrink:0; }
  .hdr-right .val { font-size:18px; font-weight:600; color:#111; }
  .hdr-right .lbl { font-size:10px; color:#888; }
  /* Grids */
  .grid2 { display:grid; grid-template-columns:1fr 1fr; gap:8px; margin-bottom:8px; }
  .grid3 { display:grid; grid-template-columns:1fr 1fr 1fr; gap:8px; margin-bottom:8px; }
  .grid4 { display:grid; grid-template-columns:repeat(4,1fr); gap:6px; margin-bottom:8px; }
  /* Sections */
  .sec { background:#f5f5f3; border-radius:8px; padding:8px 10px; }
  .sec-amber { background:#faf8d7; border-radius:8px; padding:8px 10px; }
  /* Catalyst box — color set inline based on up/down */
  .catalyst { border-radius:8px; padding:8px 10px; margin-bottom:8px; }
  /* Titles */
  .sec-title { font-size:11px; font-weight:600; color:#185FA5; margin-bottom:5px; }
  .sec-title-amber { font-size:11px; font-weight:600; color:#854F0B; margin-bottom:5px; }
  .sec-title-catalyst { font-size:11px; font-weight:600; margin-bottom:6px; display:flex; align-items:center; justify-content:space-between; }
  /* Body text */
  .body { font-size:11px; color:#555; line-height:1.55; }
  .body-amber { font-size:11px; color:#633806; line-height:1.55; }
  /* Tags */
  .tags { display:flex; flex-wrap:wrap; gap:3px; margin-top:5px; }
  .tag { font-size:10px; padding:1px 6px; border-radius:3px; white-space:nowrap; }
  .tag-green { background:#EAF3DE; color:#3B6D11; border:0.5px solid #C0DD97; }
  .tag-blue { background:#E6F1FB; color:#185FA5; border:0.5px solid #B5D4F4; }
  .tag-purple { background:#EEEDFE; color:#3C3489; border:0.5px solid #AFA9EC; }
  /* Metrics */
  .metric { background:#fff; border-radius:6px; padding:6px 8px; border:0.5px solid rgba(0,0,0,0.1); }
  .metric .lbl { font-size:10px; color:#888; }
  .metric .val { font-size:13px; font-weight:600; color:#111; }
  .metric .sub { font-size:10px; color:#888; }
  /* Catalyst rows */
  .catalyst-row { display:flex; gap:6px; margin-bottom:5px; align-items:flex-start; }
  .catalyst-row:last-child { margin-bottom:0; }
  .cnum { width:16px; height:16px; border-radius:50%; font-size:10px; font-weight:600; display:flex; align-items:center; justify-content:center; flex-shrink:0; margin-top:1px; }
  .ctext { font-size:11px; line-height:1.5; }
  /* News & list items */
  .news-row { font-size:11px; color:#633806; padding:4px 0; border-bottom:0.5px solid #e8e4a0; line-height:1.45; }
  .news-row:last-child { border-bottom:none; padding-bottom:0; }
  .news-date { font-size:10px; color:#854F0B; margin-top:1px; }
  .row-item { font-size:11px; color:#633806; padding:2px 0 2px 10px; position:relative; line-height:1.4; }
  .row-item::before { content:'—'; position:absolute; left:0; color:#BA7517; font-size:10px; }
  .col-sub { font-size:10px; color:#888; margin-bottom:3px; }
  .strong { font-weight:600; color:#111; }
  .strong-amber { font-weight:600; color:#412402; }
</style>
</head>
<body>
<div class="fc">
  <!-- HEADER -->
  <div class="hdr">
    <div>
      <div class="title">[Full Company Name] <span class="badge">NASDAQ: [TICKER]</span></div>
      <div class="sub">[Industry] · [Headquarters] · Founded [Year] · [Employee count] employees</div>
    </div>
    <div class="hdr-right">
      <div class="lbl">Market Cap ([Today's date])</div>
      <div class="val">[Market cap]</div>
      <div class="lbl">52-week range [range]</div>
    </div>
  </div>

  <!-- DAILY CATALYST (green example for up; switch to red for down) -->
  <div class="catalyst" style="background:#EAF3DE; border:0.5px solid #C0DD97;">
    <div class="sec-title-catalyst" style="color:#27500A;">
      <span>⚡ Daily Price Catalyst · [Date]</span>
      <span style="display:flex;align-items:center;gap:6px;font-size:11px;">
        <span>[Price range or move description]</span>
        <span style="font-size:12px;font-weight:600;padding:2px 8px;border-radius:4px;background:#C0DD97;color:#27500A;">[% change]</span>
      </span>
    </div>
    <div class="catalyst-row">
      <div class="cnum" style="background:#C0DD97;color:#27500A;">1</div>
      <div class="ctext" style="color:#3B6D11;"><span class="strong" style="color:#27500A;">[Catalyst title]</span> — [Detailed explanation]</div>
    </div>
    <!-- Repeat for 2–3 catalysts -->
  </div>

  <!-- MIDDLE THREE COLUMNS -->
  <div class="grid3">
    <div class="sec">
      <div class="sec-title">Industry · Business · Customers</div>
      <div class="body">[Content]</div>
      <div class="tags">
        <span class="tag tag-blue">[Customer type]</span>
      </div>
    </div>
    <div class="sec">
      <div class="sec-title">Technology Moat & Scarcity</div>
      <div class="body">[Content]</div>
      <div class="tags">
        <span class="tag tag-green">[Validated metric]</span>
      </div>
    </div>
    <div class="sec">
      <div class="sec-title">AI Stack Position</div>
      <div class="body">Positioned in the <span class="tag tag-purple" style="display:inline-block;margin:2px 0;">[Layer name]</span> of the AI infrastructure stack. [Explanation]</div>
    </div>
  </div>

  <!-- FINANCIAL METRICS (two rows) -->
  <div class="grid4">
    <div class="metric"><div class="lbl">Revenue [Quarter]</div><div class="val">[Amount]</div><div class="sub">YoY [change]</div></div>
    <div class="metric"><div class="lbl">Revenue [Quarter]</div><div class="val">[Amount]</div><div class="sub">YoY [change]</div></div>
    <div class="metric"><div class="lbl">Revenue [Quarter]</div><div class="val">[Amount]</div><div class="sub">YoY [change]</div></div>
    <div class="metric"><div class="lbl">EPS / Net Income</div><div class="val">[Value]</div><div class="sub">[Note]</div></div>
  </div>
  <div class="grid4" style="margin-bottom:8px;">
    <div class="metric"><div class="lbl">Gross Margin</div><div class="val">[Value]</div><div class="sub">[Note]</div></div>
    <div class="metric"><div class="lbl">Operating Margin</div><div class="val">[Value]</div><div class="sub">[Note]</div></div>
    <div class="metric"><div class="lbl">Cash / FCF</div><div class="val">[Value]</div><div class="sub">[Note]</div></div>
    <div class="metric"><div class="lbl">P/E</div><div class="val">[Value]</div><div class="sub">Mkt Cap [value]</div></div>
  </div>

  <!-- BOTTOM TWO COLUMNS (amber) -->
  <div class="grid2">
    <div class="sec-amber">
      <div class="sec-title-amber">Recent Developments</div>
      <div class="news-row"><span class="strong-amber">[Title]</span> — [Explanation]<div class="news-date">[Date]</div></div>
      <div class="news-row"><span class="strong-amber">[Title]</span> — [Explanation]<div class="news-date">[Date]</div></div>
      <div class="news-row"><span class="strong-amber">[Title]</span> — [Explanation]<div class="news-date">[Date]</div></div>
    </div>
    <div>
      <div class="sec-amber" style="margin-bottom:8px;">
        <div class="sec-title-amber">Competitors & Risks</div>
        <div style="display:grid;grid-template-columns:1fr 1fr;gap:4px;">
          <div>
            <div class="col-sub">Key Competitors</div>
            <div class="row-item">[Competitor]</div>
          </div>
          <div>
            <div class="col-sub">Key Risks</div>
            <div class="row-item">[Risk item]</div>
          </div>
        </div>
      </div>
      <div class="sec-amber">
        <div class="sec-title-amber">Insider Activity</div>
        <div class="body-amber">[Summary paragraph distinguishing voluntary sales vs. RSU tax sales]</div>
      </div>
    </div>
  </div>

</div>
</body>
</html>
```
