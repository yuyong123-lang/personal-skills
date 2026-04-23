# Script Generation Reference

## Overview

Generate a `script.json` file that defines the entire video content. This file is consumed by `video-pipeline.py` for TTS and video composition.

## Step 1: Create Output Directory

```bash
mkdir -p output/<slug>
mkdir -p output/<slug>/slides
mkdir -p output/<slug>/audio
```

Where `<slug>` is derived from the topic:
- Lowercase English letters, digits, hyphens only
- No spaces, no Chinese characters
- Example: "GPT-5 发布" → "gpt5-release", "Claude 4.6" → "claude-4-6"

## Step 2: Generate script.json

Write a JSON file following this exact schema:

```json
{
  "topic": "Original topic name in Chinese",
  "slug": "url-safe-slug",
  "title": "B站风格视频标题",
  "description": "视频描述，包含时间戳和内容概要",
  "tags": ["人工智能", "tag2", "tag3"],
  "voice": "zh-CN-XiaoxiaoNeural",
  "rate": "+0%",
  "resolution": {"width": 1920, "height": 1080},
  "slides": [...]
}
```

## Step 3: Write Slides

### Required Slide Structure

A video MUST have exactly this structure:
1. **Slide 0**: Title slide (type "title")
2. **Slides 1 to N-1**: Content slides (type "content"), 3-6 slides
3. **Last slide**: Closing slide (type "closing")

Total: 5-8 slides.

### Title Slide (first slide)

```json
{
  "index": 0,
  "type": "title",
  "heading": "简短有力的标题",
  "subheading": "一句话概括视频内容",
  "bullets": [],
  "narration": "大家好，今天我们来聊聊[topic]。"
}
```

- heading: 4-10 Chinese characters, punchy
- subheading: 8-15 characters, describes what this video covers
- narration: 1-2 sentences greeting the audience and introducing the topic

### Content Slides (middle slides)

```json
{
  "index": 1,
  "type": "content",
  "heading": "本页标题",
  "subheading": "可选副标题（可为空字符串）",
  "bullets": [
    "简洁要点一",
    "简洁要点二",
    "简洁要点三"
  ],
  "narration": "这一页的口播内容，用自然口语化的方式讲解这些要点。每句话不要太长。"
}
```

- heading: 4-8 characters, describes this section
- subheading: optional, can be empty string ""
- bullets: 3-5 items, each 8-20 characters, concise visual points
- narration: 2-4 sentences of conversational Chinese, each sentence < 60 characters
- **CRITICAL**: bullets and narration must be DIFFERENT content
  - bullets = concise, keyword-style, for reading
  - narration = conversational, explanatory, for listening

### Closing Slide (last slide)

```json
{
  "index": N,
  "type": "closing",
  "heading": "感谢观看",
  "subheading": "关注我，获取更多AI前沿解读",
  "bullets": [],
  "narration": "以上就是[topic]的核心解读。如果觉得有帮助，请点赞关注，我们下期再见！"
}
```

## Step 4: Title and Tags Guidelines

### Bilibili Title Tips
- Use numbers: "5个变化", "3大突破"
- Use emotional triggers: "将改变一切", "颠覆认知"
- Keep under 30 characters
- Example good titles:
  - "GPT-5来了！这5个变化将改变一切"
  - "AI Agent爆火背后，你不知道的3个真相"
  - "Sora开源了！AI视频生成进入新纪元"

### Tag Selection
- 3-5 tags
- Always include "人工智能" as base tag
- Include topic-specific tags
- Include trending tags when relevant
- Example: ["人工智能", "GPT-5", "AI前沿", "科技解读", "OpenAI"]

## Step 5: Voice and Speed Options

### Voice Options

| Voice | Gender | Style | Best For |
|-------|--------|-------|----------|
| `zh-CN-XiaoxiaoNeural` | Female | Warm, professional | Tech explainers (DEFAULT) |
| `zh-CN-YunxiNeural` | Male | Youthful, energetic | Alternative male voice |
| `zh-CN-YunjianNeural` | Male | Deep, authoritative | Serious analysis |
| `zh-CN-XiaoyiNeural` | Female | Lively, cheerful | Casual, engaging topics |

### Speed Settings

The default `rate` is `+0%` which can sound slow and overly deliberate. To make the voice sound more natural and engaging:

| Rate | Effect | Recommendation |
|------|--------|---------------|
| `+0%` | Default speed, can sound robotic | Only use for serious/dramatic topics |
| `+10%` | Slightly faster, more conversational | Good default for most videos |
| `+15%` | Natural conversational pace | **RECOMMENDED for tech explainers** |
| `+20%` | Energetic, brisk | For fast-paced topics |
| `+25%` | Very fast | Only for short, punchy content |

**Default rate should be `+15%`** — this gives a natural, engaging pace that doesn't sound AI-generated.

### Narration Writing Tips (to sound less AI-like)

- Use **colloquial Chinese**: 说白了, 简单来讲, 换句话说, 打个比方
- Add **rhetorical questions**: 你猜怎么着？ 问题出在哪？
- Use **short sentences**: Each sentence should be under 40 characters
- Avoid **formal/written style**: 不要使用"此外"、"因此"、"综上所述"
- Add **transitions**: 这个有意思了, 关键来了, 重点来了
- Sound like a **knowledgeable friend**, not a news anchor

## Step 6: Save File

Write the JSON to `output/<slug>/script.json`. Verify:
- Valid JSON (no trailing commas)
- All slide indices are sequential starting from 0
- First slide is type "title", last is type "closing"
- Every slide has non-empty narration
