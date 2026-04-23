---
name: ai-video
description: Generate AI/tech explainer videos with TTS voiceover and publish to Bilibili.
allowed-tools: Bash(python:*) Bash(playwright-cli:*) Bash(pip:*) Bash(mkdir:*) Bash(npx:*)
---

# AI Video Auto-Generation Pipeline

Automatically generate explainer videos about trending AI/tech topics, with TTS voiceover, slideshow visuals, and Bilibili publishing.

## Usage

```
/ai-video create [--topic "TOPIC"]    # Full pipeline: topic → script → slides → audio → video → publish
/ai-video topic                       # Step 1 only: discover trending topics
/ai-video script --topic "TOPIC"      # Step 2 only: generate script.json
/ai-video render <output-dir>         # Steps 3-5: generate slides + TTS + compose video
```

## Argument Parsing

### `/ai-video create`

```
/ai-video create [--topic "TOPIC"]
```

| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| --topic  | No       | (auto)  | Topic to cover. If omitted, discover trending topics first |

If `--topic` is provided, skip topic discovery and go directly to script generation.

### `/ai-video topic`

No arguments. Searches for trending AI/tech topics and presents choices.

### `/ai-video script`

```
/ai-video script --topic "TOPIC"
```

| Argument | Required | Description |
|----------|----------|-------------|
| --topic  | Yes      | The topic to write a script about |

### `/ai-video render`

```
/ai-video render <output-dir>
```

| Argument   | Required | Description |
|------------|----------|-------------|
| output-dir | Yes      | Path to the output directory (e.g. output/my-topic) containing script.json |

---

## Sub-Commands

### `/ai-video create` — Full Pipeline

Execute the complete pipeline in order. Follow each step precisely.

#### Step 0: Pre-check

Run dependency check:
```bash
python video-pipeline.py check
```

If `edge-tts` is missing: `pip install edge-tts`
If `ffmpeg` is missing: inform user to run `winget install Gyan.FFmpeg`

#### Step 1: Topic Discovery (skip if --topic provided)

Follow the detailed flow in [references/topic-discovery.md](references/topic-discovery.md).

Summary:
1. Use WebSearch to find trending AI/tech topics (Chinese + English queries)
2. Present 3-5 options to the user as a numbered list
3. Wait for user to pick one

#### Step 2: Script Generation

Follow the detailed flow in [references/script-generation.md](references/script-generation.md).

Summary:
1. Generate script.json following the schema in [references/video-pipeline.md](references/video-pipeline.md)
2. Create output directory: `output/<slug>/`
3. Create subdirectories: `output/<slug>/slides/` and `output/<slug>/audio/`
4. Write script.json to `output/<slug>/script.json`

**Script writing guidelines:**
- Title: B站风格，吸引眼球 (e.g. "GPT-5来了！这5个变化将改变一切")
- 5-8 slides: 1 title + 3-6 content + 1 closing
- narration: conversational spoken Chinese, 2-4 sentences per slide, each sentence < 60 chars
- bullets: concise visual points (different from narration)
- tags: 3-5 Bilibili-relevant tags

#### Step 3: Slide Generation

Follow the detailed flow in [references/slide-generation.md](references/slide-generation.md).

**IMPORTANT**: `file://` protocol is blocked by playwright-cli. Must use an HTTP server.

Summary:
1. Start a local HTTP server on port 8765:
   ```bash
   python -m http.server 8765 --directory output/<slug>/slides &
   ```
2. For each slide, read the template from [slide-template.html](../../slide-template.html)
3. Replace `{{SLIDE_CONTENT}}` with the appropriate HTML (see slide HTML templates below)
4. Write each slide HTML to `output/<slug>/slides/slide_NNN.html`
5. Use playwright-cli to screenshot ALL slides in one browser session:
   ```bash
   playwright-cli open "http://localhost:8765/slide_000.html"
   playwright-cli resize 1920 1080
   playwright-cli screenshot --filename="output/<slug>/slides/slide_000.png"
   # Then for each subsequent slide:
   playwright-cli goto "http://localhost:8765/slide_001.html"
   playwright-cli screenshot --filename="output/<slug>/slides/slide_001.png"
   # ... repeat for all slides
   playwright-cli close
   ```
6. Kill the HTTP server when done

#### Step 4: TTS Generation

```bash
python video-pipeline.py tts output/<slug>/script.json output/<slug>
```

This generates `audio/slide_NNN.mp3` for each slide and `timing.json`.

#### Step 5: Video Composition

```bash
python video-pipeline.py compose output/<slug>
```

This produces `output/<slug>/final.mp4`.

#### Step 6: Publish

Invoke the bili-publish skill:
```
/bili publish output/<slug>/final.mp4 --title "<title>" --desc "<description>" --tags "<tag1,tag2,tag3>"
```

---

### `/ai-video topic` — Topic Discovery Only

Follow [references/topic-discovery.md](references/topic-discovery.md).

### `/ai-video script` — Script Generation Only

Follow [references/script-generation.md](references/script-generation.md).
Requires --topic argument.

### `/ai-video render` — Render Only

Prerequisite: `output/<slug>/script.json` must already exist.

Execute Steps 3-5 from the create pipeline:
1. Slide generation (Step 3)
2. TTS generation (Step 4)
3. Video composition (Step 5)

---

## Slide Design Requirements

### Quality Standards (MUST follow)
- **Visual Richness**: Every slide MUST include visual elements beyond plain text: diagrams, icons, color-coded sections, stat highlights, or comparison layouts
- **Layout Variety**: Do NOT use identical layout for all content slides. Alternate between: list layout, comparison (2-column), diagram with arrows, stat cards, and table layouts
- **Color Palette**: Use the theme colors intentionally — cyan `#00d4ff` for key terms, purple `#7b2ff7` for emphasis, pink `#ff6b6b` for warnings/stats, green `#4ade80` for positive, amber `#fbbf24` for highlights
- **Visual Hierarchy**: Large heading → medium sub-points → small details. Use font-size and color to create clear levels
- **Forbidden**: NEVER create slides that are just a heading + 4 plain bullet points with no visual variety

### Content Slide Layout Types (vary across slides)

1. **Feature Cards**: Use `.card-grid` with individual cards for each point (icon + title + desc)
2. **Comparison Table**: Use `.compare-table` for before/after, left/right comparisons
3. **Stats Layout**: Use `.stat-grid` with large numbers and labels for data-heavy slides
4. **Diagram Layout**: Use CSS arrows/boxes for flow diagrams and architecture illustrations
5. **List with Icons**: Enhanced bullets with colored icon markers and descriptions

### Slide HTML Templates

These replace `{{SLIDE_CONTENT}}` in the template. Read the full template from [slide-template.html](../../slide-template.html).

**IMPORTANT**: Each slide HTML file must be a COMPLETE standalone HTML file with ALL CSS inlined. Do NOT rely on shared styles between slides.

#### Title Slide (index 0, type "title")

```html
<div class="slide type-title">
  <h1 class="heading">{{HEADING}}</h1>
  <div class="divider"></div>
  <p class="subheading">{{SUBHEADING}}</p>
  <div class="brand">AI · 科技解读</div>
</div>
```

#### Content Slide — Feature Cards Layout

Use for slides presenting distinct features or capabilities.

```html
<div class="slide type-content">
  <h1 class="heading">{{HEADING}}</h1>
  <div class="divider"></div>
  <div class="card-grid">
    <div class="card">
      <div class="card-icon cyan">01</div>
      <div class="card-title">Feature Name</div>
      <div class="card-desc">Brief description</div>
    </div>
    <!-- repeat for each feature -->
  </div>
  <div class="slide-number">{{CURRENT}} / {{TOTAL}}</div>
  <div class="brand">AI · 科技解读</div>
</div>
```

#### Content Slide — Comparison Layout

Use for slides comparing two concepts side by side.

```html
<div class="slide type-content">
  <h1 class="heading">{{HEADING}}</h1>
  <div class="divider"></div>
  <div class="compare-table">
    <div class="compare-col">
      <div class="compare-header cyan">Left Label</div>
      <div class="compare-item">Point 1</div>
      <div class="compare-item">Point 2</div>
    </div>
    <div class="compare-divider">VS</div>
    <div class="compare-col">
      <div class="compare-header purple">Right Label</div>
      <div class="compare-item">Point 1</div>
      <div class="compare-item">Point 2</div>
    </div>
  </div>
  <div class="slide-number">{{CURRENT}} / {{TOTAL}}</div>
  <div class="brand">AI · 科技解读</div>
</div>
```

#### Content Slide — Stats Layout

Use for slides with key metrics or impressive numbers.

```html
<div class="slide type-content">
  <h1 class="heading">{{HEADING}}</h1>
  <div class="divider"></div>
  <div class="stat-grid">
    <div class="stat-item">
      <div class="stat-number cyan">10x</div>
      <div class="stat-label">成功率提升</div>
    </div>
    <!-- repeat for each stat -->
  </div>
  <div class="slide-number">{{CURRENT}} / {{TOTAL}}</div>
  <div class="brand">AI · 科技解读</div>
</div>
```

#### Content Slide — Basic List (use sparingly)

```html
<div class="slide type-content">
  <h1 class="heading">{{HEADING}}</h1>
  <p class="subheading">{{SUBHEADING}}</p>
  <div class="divider"></div>
  <ul class="bullets">
    <li>{{BULLET_1}}</li>
    <li>{{BULLET_2}}</li>
    ...
  </ul>
  <div class="slide-number">{{CURRENT}} / {{TOTAL}}</div>
  <div class="brand">AI · 科技解读</div>
</div>
```

If `subheading` is empty, omit the `<p class="subheading">` line entirely.

#### Closing Slide (last index, type "closing")

```html
<div class="slide type-closing">
  <h1 class="heading">{{HEADING}}</h1>
  <div class="divider"></div>
  <p class="subheading">{{SUBHEADING}}</p>
  <div class="cta">
    <span class="cta-item">点赞</span>
    <span class="cta-item">关注</span>
    <span class="cta-item">转发</span>
  </div>
  <div class="brand">AI · 科技解读</div>
</div>
```

---

## Output Directory Structure

```
output/<slug>/
├── script.json          # Generated script data
├── timing.json          # TTS timing data (auto-generated by tts command)
├── slides/
│   ├── slide_000.html   # HTML source for each slide
│   ├── slide_000.png    # Screenshot of each slide (1920x1080)
│   ├── slide_001.html
│   ├── slide_001.png
│   └── ...
├── audio/
│   ├── slide_000.mp3    # TTS audio for each slide
│   ├── slide_001.mp3
│   └── ...
└── final.mp4            # Final composed video
```

---

## Error Handling

| Scenario | Action |
|----------|--------|
| `edge-tts` not found | Auto-install: `pip install edge-tts` |
| `ffmpeg` not found | Tell user: `winget install Gyan.FFmpeg`, then restart shell |
| TTS generation fails (NoAudioReceived) | Check network/proxy. Retry once. |
| `file://` blocked by playwright-cli | Use HTTP server (`python -m http.server`) instead |
| ffmpeg concat path error | Script uses absolute paths — should not happen. If it does, check the filelist.txt |
| Slide image missing | Check that playwright-cli screenshots succeeded before running compose |
| Bilibili auth expired | Tell user to run `/bili login` first |
| Video file too large (>500MB) | Reduce slide count or narration length, or increase CRF value |

---

## Important Notes

- **CRITICAL**: Never use `file://` URLs with playwright-cli — it is blocked. Always use `python -m http.server` to serve slides.
- Always `playwright-cli resize 1920 1080` before taking screenshots.
- Open ONE browser session, navigate between slides with `goto`, take all screenshots, then `close` once.
- Each slide's narration and bullets are DIFFERENT content: narration is conversational, bullets are concise visual points.
- Default voice: `zh-CN-XiaoxiaoNeural` (female, warm, professional). Default rate: `+15%` (natural conversational pace). Can override in script.json.
- Write narration in colloquial Chinese (说白了, 打个比方, 关键来了), avoid formal AI-sounding language (此外, 因此, 综上所述).
- The HTTP server should be started in the `slides/` directory, serving on port 8765.
- Kill the HTTP server process after all screenshots are taken.
- The publish step delegates to the existing `/bili publish` skill.
