# Topic Discovery Reference

## Overview

Find trending AI/tech topics from multiple sources and present options to the user.

## Step 1: Web Search Queries

Use the WebSearch tool with these queries (mix of English and Chinese). **Replace the date/month with the CURRENT date dynamically** — do NOT hardcode specific months:

**English queries:**
- `AI news this week [current month year]`
- `latest AI breakthrough [current year]`
- `new AI model release [current year]`
- `tech industry news this week`
- `AI trending topic this week`

**Chinese queries:**
- `AI最新进展 [当前年月]`
- `人工智能 热点新闻 本周`
- `大模型最新发布`
- `科技热点 本周`
- `AI领域 重大突破 最新`

**Also search Bilibili hot topics (use mcp__web-search-prime):**
- `B站 科技 热门 本周`
- `人工智能 热搜话题`

## Step 2: Browse News Sources (Optional)

If WebSearch doesn't return good results, use playwright-cli to browse:

### Hacker News
```bash
playwright-cli open "https://news.ycombinator.com"
playwright-cli snapshot
# Read the front page titles, look for AI/ML related posts
playwright-cli close
```

### 36kr AI Section
```bash
playwright-cli open "https://www.36kr.com/information/AI"
playwright-cli snapshot
# Read article titles from the AI section
playwright-cli close
```

## Step 3: Present Options

Format the discovered topics as a numbered list:

```
以下是近期热门 AI/科技话题：

1. **[Topic 1]** — [1-2 sentence description of why it's interesting]
2. **[Topic 2]** — [description]
3. **[Topic 3]** — [description]
4. **[Topic 4]** — [description]
5. **[Topic 5]** — [description]

请选择一个话题编号，或输入自定义话题：
```

## Step 4: Wait for Selection

Wait for the user to pick a topic number or provide a custom topic. Use the selected topic for script generation.

## Topic Selection Criteria

When filtering topics, prefer:
- **Popularity FIRST**: Topics must be widely discussed (trending on Weibo/Bilibili/Twitter/Hacker News). Check search result volume and engagement
- **Mass appeal**: Topics should be understandable by general tech audience, not niche specialists
- **High search volume**: Prefer topics people are actively searching for (check Bilibili search trends)
- Recent news (< 1 week old)
- Topics with clear educational value (can be explained in 3-5 minutes)
- Topics with concrete facts/data points to discuss
- Topics interesting to Bilibili's tech audience
- Avoid: purely speculative topics, topics requiring deep domain expertise, political topics, overly niche topics

### Tag Selection Criteria

Tags must be **popular and discoverable**:
- Always include "人工智能" as base tag
- Use tags that are commonly searched on Bilibili (check Bilibili search suggestions)
- Prefer short, broad tags over long specific ones
- Include at least one trending/hot tag if available
- Example good tags: "人工智能", "AI", "ChatGPT", "Claude", "科技解读", "编程"
- Example bad tags: "Harness Engineering", "oh-my-claudecode", "Hashline" (too niche)
