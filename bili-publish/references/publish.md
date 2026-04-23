# Bilibili Video Publish Flow

## Overview

Upload a video to Bilibili creator studio and publish it with metadata (title, description, tags). This flow has been tested and verified on the actual Bilibili upload page.

## Prerequisites

- Auth state file `.bili-auth.json` must exist in the project root
- Video file must exist at the specified path
- `playwright-cli` must be available

## Arguments

| Argument   | Required | Description                          |
|------------|----------|--------------------------------------|
| video_path | Yes      | Absolute or relative path to video   |
| --title    | Yes      | Video title                          |
| --desc     | No       | Video description                    |
| --tags     | No       | Comma-separated tags                 |

## Step-by-Step Flow

### 1. Pre-Checks

Verify both required files exist:

```bash
ls .bili-auth.json
ls "<video_path>"
```

- If `.bili-auth.json` missing → tell user to run `/bili login` first, stop.
- If video file missing → tell user the file was not found at that path, stop.

### 2. Open Browser and Load Auth State

**CRITICAL ORDER**: Must open blank page first, then load state, then navigate. Loading state requires an open browser, and navigating without state loaded causes login redirect.

```bash
playwright-cli open --headed about:blank
```

```bash
playwright-cli state-load .bili-auth.json
```

### 3. Navigate to Upload Page

```bash
playwright-cli goto https://member.bilibili.com/platform/upload/video/frame
```

### 4. Verify Login State

```bash
playwright-cli snapshot
```

**Login valid** if:
- Page URL is `https://member.bilibili.com/platform/upload/video/frame`
- Page title contains "创作中心"
- Upload area with "点击上传或将视频拖拽到此区域" is visible

**Login expired** if:
- Page URL is `https://passport.bilibili.com/login`
- Page title is "账号登录"

If expired → close browser, tell user to re-run `/bili login`, stop.

### 5. Upload Video File

The upload area requires clicking to trigger a file chooser dialog, then uploading.

**Step A: Click the upload area to trigger file chooser**

```bash
playwright-cli click e<upload_area_ref>
```

Look for the element containing text "点击上传或将视频拖拽到此区域" with "上传视频". Clicking it triggers a file chooser modal.

**Step B: Upload the file (must be done immediately after clicking)**

```bash
playwright-cli upload "<video_path>"
```

> **IMPORTANT**: `upload` only works when a file chooser modal is active. You must click the upload area first, then immediately run `upload`. Do NOT pass extra arguments to `upload` — it only takes the file path.

### 6. Wait for Upload Completion

Poll every 5 seconds:

```bash
sleep 5 && playwright-cli snapshot
```

Look for:
- Text "上传完成" next to the video file name
- The metadata form (title, tags, description fields) becoming visible
- A section labeled "基本设置" appearing

The upload page auto-analyzes the video and may:
- Auto-fill some tags (e.g., "生活记录", "记录", "剪辑")
- Auto-select a partition/分区 (e.g., "人工智能", "娱乐")
- Generate cover frame recommendations

### 7. Fill Title

The title textbox is pre-filled with the video filename. Replace it:

```bash
# Click the title textbox (placeholder: "请输入稿件标题")
playwright-cli click e<title_textbox_ref>

# Select all existing text
playwright-cli press Control+a

# Type the new title
playwright-cli "<title>"
```

> Do NOT use `fill` — the title field may be a controlled React input. `click` + `Ctrl+A` + `type` is more reliable.

Verify: the textbox should show the new title, with a character count like "6/80".

### 8. Add Tags

The tag input has a placeholder "按回车键Enter创建标签". Tags are entered one at a time:

```bash
# Click the tag input textbox
playwright-cli click e<tag_input_ref>

# Type tag text
playwright-cli type "<tag>"

# Press Enter to create the tag
playwright-cli press Enter
```

Repeat for each tag. Tags appear as pill-shaped elements with an X button.

**Note**: Bilibili auto-adds some tags. The user's tags are added alongside these auto-generated ones. The page shows "还可以添加N个标签" to indicate remaining capacity (max ~10 tags).

**Removing auto-generated tags**: Bilibili auto-adds tags based on video content. These are often wrong. Remove them by clicking each tag pill element (the element with the tag name, NOT the X icon inside it):

```bash
# Take snapshot to find auto-generated tag refs
playwright-cli snapshot

# Click each auto-generated tag to remove it (refs change after each removal!)
playwright-cli click e<tag_pill_ref>
# Take new snapshot to get updated refs
playwright-cli snapshot
# Repeat for remaining tags
```

### 9. Fill Description (Quill Editor)

The description field (labeled "简介") uses a **Quill rich text editor**, NOT a standard `<textarea>`. There are two approaches, try them in order:

**Approach 1: Use `eval` with innerHTML on the paragraph element (RECOMMENDED)**

Find the empty `<p>` element inside `.ql-editor` from the snapshot (usually the paragraph ref inside the description area), then:

```bash
playwright-cli eval 'el => { el.innerHTML = "<p>Line 1 of description</p><p>Line 2</p><p>00:00 Opening</p><p>00:30 Summary</p>"; }' e<paragraph_ref>
```

**Approach 2: Use `run-code` with `.ql-editor` selector (fallback)**

```bash
playwright-cli run-code "async page => { const el = page.locator('.ql-editor').first(); const count = await el.count(); if (count === 0) return 'ql-editor not found'; await el.click(); await el.fill('<description>'); return 'filled'; }"
```

> **IMPORTANT**: Standard `fill` on a ref won't work for Quill editor. Use `eval` with innerHTML (Approach 1) or `run-code` with `.ql-editor` (Approach 2).

Verify: the description area should show the text, with a character count like "10/2000".

### 10. Verify Form Before Submit

```bash
playwright-cli snapshot
```

Check all required fields (marked with "*"):
- ✅ 封面 (Cover) — auto-generated from first frame
- ✅ 标题 (Title) — user-provided
- ✅ 分区 (Partition) — auto-selected
- ✅ 标签 (Tags) — user-provided + auto-generated
- 简介 (Description) — optional, user-provided

### 11. Submit / Publish

Find and click the "立即投稿" button:

```bash
playwright-cli click e<submit_button_ref>
```

The button is at the bottom of the form, labeled "立即投稿" (Submit Now).

### 12. Wait for Confirmation

```bash
sleep 5 && playwright-cli snapshot
```

**Success indicators:**
- Text "稿件投递成功" appears prominently
- Two buttons are shown: "查看进度" (View Progress) and "再投一个" (Submit Another)
- The upload form is replaced by the success screen

**Failure indicators:**
- Form validation errors highlighted in red
- Required fields still empty
- Error popup/toast message

### 13. Clean Up

```bash
# Save updated auth state (refreshes cookies)
playwright-cli state-save .bili-auth.json

# Close browser
playwright-cli close
```

### 14. Report Result

On success:
```
视频发布成功！

| 项目 | 内容 |
|------|------|
| 标题 | <title> |
| 简介 | <description> |
| 标签 | <tags> |
| 状态 | 稿件投递成功 |

查看审核进度: https://member.bilibili.com/platform/upload-manager/article
```

On failure:
```
发布失败: <reason>
浏览器仍然打开，你可以检查页面状态。
```

## Key Technical Details

| Field          | Method                                    | Notes                                          |
|----------------|-------------------------------------------|------------------------------------------------|
| Video upload   | Click upload area → `upload` file path    | Must trigger file chooser first                |
| Title          | Click + Ctrl+A + type                     | Replaces auto-filled filename                  |
| Tags           | Click input + type + Enter (per tag)      | Added alongside auto-generated tags            |
| Description    | `eval` with innerHTML on paragraph ref OR `run-code` with `.ql-editor` | Quill editor, not a standard textarea |
| Submit         | Click "立即投稿" button                    | At bottom of form                              |

## Troubleshooting

| Issue                                | Solution                                                              |
|--------------------------------------|-----------------------------------------------------------------------|
| Redirected to login page             | Auth expired → tell user to run `/bili login`                         |
| `upload` fails: "no modal state"     | Must click upload area first to trigger file chooser                  |
| `upload` fails: "too many arguments" | `upload` only takes file path, no additional arguments                |
| Title not changing                   | Use `click` + `Ctrl+A` + `type` instead of `fill`                    |
| Description not filling              | Use `eval` with innerHTML on paragraph ref (Approach 1)              |
| `run-code` syntax error for desc    | Use `eval 'el => { el.innerHTML = "..." }' e<ref>` instead          |
| Tag not creating                     | Make sure to press Enter after typing tag text                        |
| Form validation error                | Read error in snapshot, fill the missing required field               |
| Submit button not clickable          | Check for unfilled required fields (marked with "*")                  |
| Snapshot refs expired                | Take a fresh `playwright-cli snapshot` to get new refs                |
| Upload stuck at 0%                   | Network issue, suggest retry                                          |

## Notes

- The Bilibili upload page UI may change over time. Always use the current snapshot to identify elements rather than assuming fixed refs.
- Bilibili auto-analyzes uploaded videos and may pre-fill tags and partition. User-provided values are added on top.
- Large video files (>1GB) may take significant time to upload. Keep the user informed.
- Bilibili may have daily upload limits. If you see a limit message, inform the user.
- Published videos go through a review process. Inform the user if a review message appears.
- The `state-load` / `state-save` must be run from the project root directory where `.bili-auth.json` lives.
