# Slide Generation Reference

## Overview

Generate HTML slide files from script.json, then use playwright-cli to screenshot them as 1920x1080 PNG images.

**CRITICAL**: `file://` protocol is BLOCKED by playwright-cli. You MUST use an HTTP server.

## Design Principles (MUST follow)

### Page Fullness (CRITICAL — no large empty areas!)
- Every slide MUST use at least 80% of the 1920x1080 canvas — **NO large blank areas**
- Use generous padding (80-100px) but fill the inner space completely with content, cards, stats, or diagrams
- If a slide looks sparse, ADD more visual elements: background decorations, summary quotes, stat highlights, tags/badges
- Use `flex: 1` on content containers to stretch them and fill available space
- Add decorative background orbs/glows to fill corner areas (use `position: absolute` + radial gradients)
- Cards should have sufficient padding (28-32px) and descriptions to fill their boxes
- For slides with fewer than 4 content items, use LARGER elements (bigger font, more padding, extra sub-descriptions)

### Visual Richness
- Each slide MUST include visual elements beyond plain text: diagrams, icons (using CSS shapes or Unicode symbols), data visualizations, or color-coded sections
- Use multi-column layouts when comparing items (e.g., left vs right, before vs after)
- Add accent boxes, callout cards, or highlighted sections to break up text monotony
- Use color-coded tags/badges for key terms instead of plain bullet text

### Professional Layout
- Use CSS Grid or Flexbox for structured layouts (not just stacked bullet points)
- Add subtle background decorations: gradient overlays, geometric shapes, glowing orbs
- Include visual hierarchy: large heading, medium sub-points, small details
- Use number badges or icon markers for ordered items

### Content Depth
- Slides should show relationships (arrows, connectors), not just lists
- Use comparison tables with alternating row colors for multi-item comparisons
- Include visual metaphors (e.g., a formula displayed as styled equation boxes)
- Add data highlights (large numbers, percentage badges) when presenting statistics

### Forbidden Patterns
- NEVER create slides that are just a heading + 4 bullet points with no visual variety
- NEVER use identical layout for all content slides — vary between list, comparison, diagram, and stat layouts
- NEVER use only one color for all elements — use the palette (cyan #00d4ff, purple #7b2ff7, pink #ff6b6b, green #4ade80, amber #fbbf24)
- NEVER leave large blank areas on any slide — if content is sparse, enlarge it or add decorative elements

## Step 1: Read the HTML Template

Read the template file at the project root:
```
slide-template.html
```

This template contains the full CSS styling (dark tech theme) and a `{{SLIDE_CONTENT}}` placeholder in the `<body>`.

## Step 2: Generate HTML for Each Slide

For each slide in script.json, create a complete HTML file by replacing `{{SLIDE_CONTENT}}` with the slide-specific HTML.

### Title Slide HTML

```html
<div class="slide type-title">
  <h1 class="heading">{{heading}}</h1>
  <div class="divider"></div>
  <p class="subheading">{{subheading}}</p>
  <div class="brand">AI · 科技解读</div>
</div>
```

### Content Slide HTML — Choose a layout type per slide (VARY across slides!)

**Each slide HTML file MUST be a COMPLETE standalone HTML file with ALL CSS inlined.** Do NOT rely on shared styles between slides — copy the full template CSS into each file.

#### Layout 1: Feature Cards (for presenting distinct features/capabilities)

```html
<div class="slide type-content">
  <h1 class="heading">{{HEADING}}</h1>
  <div class="divider"></div>
  <div class="card-grid">
    <div class="card">
      <div class="card-icon cyan">01</div>
      <div class="card-title">Feature Name</div>
      <div class="card-desc">Brief description of this feature</div>
    </div>
    <div class="card">
      <div class="card-icon purple">02</div>
      <div class="card-title">Feature Name</div>
      <div class="card-desc">Brief description</div>
    </div>
    <!-- repeat for each feature, max 4 cards in 2x2 grid -->
  </div>
  <div class="slide-number">{{CURRENT}} / {{TOTAL}}</div>
  <div class="brand">AI · 科技解读</div>
</div>
```

#### Layout 2: Two-Column Comparison (for before/after, concept A vs B)

```html
<div class="slide type-content">
  <h1 class="heading">{{HEADING}}</h1>
  <div class="divider"></div>
  <div class="two-col">
    <div class="col-box">
      <div class="col-header cyan"><span class="icon">▶</span> Left Title</div>
      <div class="col-desc">Description text</div>
      <ul class="col-items cyan-list">
        <li>Point 1</li>
        <li>Point 2</li>
      </ul>
    </div>
    <div class="col-box">
      <div class="col-header purple"><span class="icon">⟳</span> Right Title</div>
      <div class="col-desc">Description text</div>
      <ul class="col-items purple-list">
        <li>Point 1</li>
        <li>Point 2</li>
      </ul>
    </div>
  </div>
  <div class="slide-number">{{CURRENT}} / {{TOTAL}}</div>
  <div class="brand">AI · 科技解读</div>
</div>
```

#### Layout 3: Stats + Highlight (for key numbers and data)

```html
<div class="slide type-content">
  <h1 class="heading">{{HEADING}}</h1>
  <div class="divider"></div>
  <div class="stat-grid">
    <div class="stat-item">
      <div class="stat-number cyan">10x</div>
      <div class="stat-label">Success Rate Increase</div>
    </div>
    <div class="stat-item">
      <div class="stat-number pink">61%</div>
      <div class="stat-label">Token Reduction</div>
    </div>
    <!-- max 4 stat items in 2x2 or 3 in 1x3 -->
  </div>
  <div class="slide-number">{{CURRENT}} / {{TOTAL}}</div>
  <div class="brand">AI · 科技解读</div>
</div>
```

#### Layout 4: Architecture Diagram (for system/flow diagrams)

```html
<div class="slide type-content">
  <h1 class="heading">{{HEADING}}</h1>
  <div class="divider"></div>
  <div class="arch-stack">
    <div class="arch-row">
      <div class="arch-box green">Top Layer</div>
      <span class="label-arrow">←</span>
      <span class="label-text">Description</span>
    </div>
    <div class="arch-arrow">▼</div>
    <div class="arch-box cyan">Middle Layer (Harness)</div>
    <div class="arch-arrow">▼</div>
    <div style="display: flex; gap: 60px;">
      <div class="arch-box purple">Bottom A</div>
      <div class="arch-box amber">Bottom B</div>
    </div>
  </div>
  <div class="slide-number">{{CURRENT}} / {{TOTAL}}</div>
  <div class="brand">AI · 科技解读</div>
</div>
```

#### Layout 5: Formula + Cards (for formulas with explanation)

```html
<div class="slide type-content">
  <h1 class="heading">{{HEADING}}</h1>
  <div class="divider"></div>
  <div class="formula-box">
    <span class="formula-term model">Model</span>
    <span class="formula-op">+</span>
    <span class="formula-term harness">Harness</span>
    <span class="formula-op">=</span>
    <span class="formula-term agent">Agent</span>
  </div>
  <!-- Then add card-grid or bullets below -->
  <div class="slide-number">{{CURRENT}} / {{TOTAL}}</div>
  <div class="brand">AI · 科技解读</div>
</div>
```

#### Layout 6: Basic List (use sparingly, only when other layouts don't fit)

```html
<div class="slide type-content">
  <h1 class="heading">{{heading}}</h1>
  <p class="subheading">{{subheading}}</p>
  <div class="divider"></div>
  <ul class="bullets">
    <li>{{bullet_1}}</li>
    <li>{{bullet_2}}</li>
    <li>{{bullet_3}}</li>
  </ul>
  <div class="slide-number">{{current}} / {{total}}</div>
  <div class="brand">AI · 科技解读</div>
</div>
```

If `subheading` is empty string, omit the `<p class="subheading">` line entirely.

### Layout Selection Guidelines

| Slide Content | Best Layout |
|---------------|-------------|
| Features, capabilities, tools | Feature Cards (Layout 1) |
| Comparing two concepts | Two-Column (Layout 2) |
| Numbers, metrics, experiments | Stats + Highlight (Layout 3) |
| Architecture, hierarchy, flow | Architecture Diagram (Layout 4) |
| Formulas with explanation | Formula + Cards (Layout 5) |
| Simple list (last resort) | Basic List (Layout 6) |

**Rule: At most 1 slide should use Layout 6 (Basic List). The rest must use Layouts 1-5.**

### Closing Slide HTML

```html
<div class="slide type-closing">
  <h1 class="heading">{{heading}}</h1>
  <div class="divider"></div>
  <p class="subheading">{{subheading}}</p>
  <div class="cta">
    <span class="cta-item">点赞</span>
    <span class="cta-item">关注</span>
    <span class="cta-item">转发</span>
  </div>
  <div class="brand">AI · 科技解读</div>
</div>
```

### File Naming

Save each HTML file as:
```
output/<slug>/slides/slide_000.html
output/<slug>/slides/slide_001.html
output/<slug>/slides/slide_002.html
...
```

Use zero-padded 3-digit index matching the slide's `index` field.

## Step 3: Start HTTP Server

**CRITICAL**: The background HTTP server must stay alive for ALL screenshots. The server tends to die if started as a simple background task. Follow this pattern:

### Option A: Start server inline with screenshots (RECOMMENDED)

Start the server in the same bash command chain, then take all screenshots in rapid succession:

```bash
cd "D:/Users/13720/Desktop/question/bili/output/<slug>/slides" && python -m http.server 8765 & sleep 2 && playwright-cli open "http://localhost:8765/slide_000.html" && playwright-cli resize 1920 1080 && playwright-cli screenshot --filename="output/<slug>/slides/slide_000.png" && playwright-cli goto "http://localhost:8765/slide_001.html" && playwright-cli screenshot --filename="output/<slug>/slides/slide_001.png" && ... && playwright-cli close
```

### Option B: Start server separately (if Option A is too long)

1. Start the server and immediately take the FIRST screenshot in the same command:
```bash
cd output/<slug>/slides && python -m http.server 8765 & sleep 2 && playwright-cli open "http://localhost:8765/slide_000.html" && playwright-cli resize 1920 1080 && playwright-cli screenshot --filename="output/<slug>/slides/slide_000.png"
```

2. For each subsequent slide, restart the server and take the screenshot together:
```bash
cd output/<slug>/slides && python -m http.server 8765 & sleep 1 && playwright-cli goto "http://localhost:8765/slide_001.html" && playwright-cli screenshot --filename="output/<slug>/slides/slide_001.png"
```

3. Repeat for all slides, then close browser: `playwright-cli close`

**Why restart each time?** The background `python -m http.server` process terminates unpredictably when run as a background task in the sandbox. Restarting before each slide ensures the server is always alive.

## Step 4: Screenshot All Slides

Use a SINGLE browser session. Open once, navigate between slides, close once.

### Playwright-cli correct commands:

| Command | Correct syntax | Wrong syntax |
|---------|---------------|-------------|
| Click element | `playwright-cli click e123` | ~~`playwright-cli click --ref=e123`~~ |
| Type text | `playwright-cli type "text"` | ~~`playwright-cli keyboard "text"`~~ |
| Press key | `playwright-cli press Enter` | ~~`playwright-cli keyboard Enter`~~ |
| Fill field | `playwright-cli fill e123 "text"` | - |
| Run JS | `playwright-cli eval 'fn' selector` | ~~`playwright-cli run-code 'fn'`~~ (also works but different syntax) |

### Screenshot flow:

```bash
# Open browser on first slide
playwright-cli open "http://localhost:8765/slide_000.html"

# Set viewport size
playwright-cli resize 1920 1080

# Screenshot first slide
playwright-cli screenshot --filename="output/<slug>/slides/slide_000.png"

# Navigate and screenshot each subsequent slide
playwright-cli goto "http://localhost:8765/slide_001.html"
playwright-cli screenshot --filename="output/<slug>/slides/slide_001.png"

# ... repeat for all slides

# Close browser when done
playwright-cli close
```

### Important Screenshot Notes

- Always `resize 1920 1080` AFTER opening the browser, BEFORE the first screenshot
- Use `goto` to navigate between slides (faster than opening/closing browser each time)
- The `--filename` path is relative to the project root directory
- Screenshots are saved as PNG format
- After each `goto`, check the Page Title in the output. If it says "Error response", the HTTP server died — restart it before retrying

## Step 5: Verify Screenshots

After all screenshots are taken, verify they exist:

```bash
ls -la output/<slug>/slides/*.png
```

Each PNG file should be 200-400 KB in size. If any file is 0 bytes or missing, re-take that screenshot.

## Step 6: Kill HTTP Server and Clean Up Port

Kill ALL processes on the port (Windows):

```bash
# Kill all LISTENING processes on port 8765 (Windows)
netstat -ano | grep 8765 | grep LISTENING | awk '{print $5}' | sort -u | while read pid; do taskkill //F //PID $pid 2>/dev/null; done
```

**IMPORTANT**: Before starting the server, ALWAYS clean up any stale processes on the port:
```bash
netstat -ano | grep 8765 | grep LISTENING | awk '{print $5}' | sort -u | while read pid; do taskkill //F //PID $pid 2>/dev/null; done
```

## Error Handling

| Issue | Solution |
|-------|----------|
| `file://` protocol blocked | Use HTTP server as described above |
| Page looks wrong in screenshot | Check HTML syntax, verify CSS is complete in each file |
| Blank/white screenshot | Ensure `resize 1920 1080` was called, check HTML rendering |
| Chinese text not rendering | The template uses "Microsoft YaHei" font which is built into Windows |
| Port 8765 already in use | Kill stale processes first (see Step 6), then restart server |
| Page Title shows "Error response" | HTTP server died. Restart server with `sleep 1` before `goto` |
| Server keeps dying as background task | Use Option A (inline chain) or restart server before each screenshot (Option B) |
| Multiple LISTENING processes on port | Run the `netstat + awk + taskkill` cleanup command from Step 6 |
| `playwright-cli click --ref=eX` fails | Use `playwright-cli click eX` (no `--ref` flag, just pass ref directly) |
| `playwright-cli keyboard` command not found | Use `playwright-cli type "text"` for typing, `playwright-cli press Enter` for keys |
