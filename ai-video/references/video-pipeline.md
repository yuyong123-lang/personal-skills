# Video Pipeline Reference

## script.json Schema

The script.json file is the contract between Claude (writer) and the Python pipeline (processor).

```json
{
  "topic": "string — the topic name (Chinese)",
  "slug": "string — URL-safe identifier (lowercase, hyphens, no spaces, no Chinese)",
  "title": "string — Bilibili video title (B站风格)",
  "description": "string — Bilibili video description with timestamps",
  "tags": ["string"],
  "voice": "string — edge-tts voice name (default: zh-CN-XiaoxiaoNeural)",
  "rate": "string — speech rate (default: +15%)",
  "resolution": {"width": 1920, "height": 1080},
  "slides": [
    {
      "index": 0,
      "type": "title | content | closing",
      "heading": "string",
      "subheading": "string (can be empty)",
      "bullets": ["string"],
      "narration": "string — TTS narration text"
    }
  ]
}
```

### Field Details

| Field | Required | Description |
|-------|----------|-------------|
| topic | Yes | Topic name in Chinese |
| slug | Yes | Directory name. Lowercase + hyphens only. e.g. "gpt5-release" |
| title | Yes | Bilibili title. Catchy, 15-30 chars |
| description | Yes | Bilibili description with timestamps |
| tags | Yes | 3-5 tags for Bilibili discoverability |
| voice | No | Default: `zh-CN-XiaoxiaoNeural` |
| rate | No | Default: `+15%`. Range: `-50%` to `+100%`. Recommended: `+15%` for natural pace |
| resolution | No | Default: 1920x1080 |
| slides | Yes | 5-8 slides: title + content(s) + closing |

### Slide Types

- **title**: First slide (index 0). No bullets.
- **content**: Middle slides. Heading + bullets + narration.
- **closing**: Last slide. No bullets. Has CTA.

### Voice Options

| Voice | Gender | Style | Best For |
|-------|--------|-------|----------|
| `zh-CN-XiaoxiaoNeural` | Female | Warm, professional | Tech (DEFAULT) |
| `zh-CN-YunxiNeural` | Male | Youthful, energetic | Alternative |
| `zh-CN-YunjianNeural` | Male | Deep, authoritative | Analysis |
| `zh-CN-XiaoyiNeural` | Female | Lively, cheerful | Casual |

---

## video-pipeline.py Commands

### `check` — Dependency Verification

```bash
python video-pipeline.py check
```

Checks:
- `edge-tts` Python package (via import)
- `ffmpeg` binary (PATH + common Windows Winget locations)

Exit code 0 = all OK, 1 = missing deps.

### `tts` — TTS Audio Generation

```bash
python video-pipeline.py tts <script.json> <output-dir>
```

Input: script.json
Output:
- `output-dir/audio/slide_NNN.mp3` — per-slide audio
- `output-dir/timing.json` — timing data

**How it works:**
1. Reads script.json
2. For each slide with narration, uses edge-tts to generate MP3
3. Calculates duration from file size (edge-tts ~48kbps)
4. Adds 0.5s padding per slide
5. Writes timing.json

**timing.json format:**
```json
[
  {"slide_index": 0, "audio_file": "audio/slide_000.mp3", "duration": 8.42},
  {"slide_index": 1, "audio_file": "audio/slide_001.mp3", "duration": 15.67}
]
```

**Note:** All TTS generation runs in a single asyncio event loop to avoid connection issues.

### `compose` — Video Composition

```bash
python video-pipeline.py compose <output-dir>
```

Input:
- `output-dir/slides/slide_NNN.png` — slide images
- `output-dir/audio/slide_NNN.mp3` — audio files
- `output-dir/timing.json` — timing data

Output: `output-dir/final.mp4`

**How it works:**
1. Reads timing.json
2. For each slide: creates a video segment (static PNG + audio MP3)
   - ffmpeg encoding: `libx264 -tune stillimage -pix_fmt yuv420p`
   - Audio: `aac 192k`
   - Duration from timing.json
3. Concatenates all segments using ffmpeg concat demuxer
4. Uses absolute paths in filelist.txt to avoid path resolution issues
5. Cleans up temp segment files

**Output specs:**
- Codec: H.264 + AAC
- Resolution: 1920x1080
- Frame rate: 25 fps
- Typical size: ~1.5 MB per minute of video

### `serve` — HTTP Server for Slides

```bash
python video-pipeline.py serve <slides-dir> [port]
```

Starts an HTTP server for serving slide HTML files. Default port is 8765.

**Why use this instead of `python -m http.server`?**
- The built-in `serve` command is designed for the slide workflow
- It runs as a foreground process that stays alive (run with `&` for background)
- Use this as the PREFERRED method for serving slides during screenshot capture

**Usage:**
```bash
# Start server in background
python video-pipeline.py serve output/<slug>/slides 8765 &

# Take screenshots...

# Kill when done
kill $! 2>/dev/null || taskkill //F //PID $! 2>/dev/null
```
