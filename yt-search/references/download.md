# YouTube Video Download Flow (greenvideo.cc)

## Overview

Download YouTube videos using greenvideo.cc — a free online video parser that supports 1000+ video platforms. The entire download process is automated via playwright-cli browser automation.

## Dependencies

### playwright-cli Installation

Check if playwright-cli is available, install if missing:

```bash
npx @playwright/cli@latest --version || npm install -g @playwright/cli@latest
```

No additional tools (yt-dlp, ffmpeg) are needed — greenvideo.cc handles parsing and downloading in the browser.

## Step-by-Step Flow

### 1. Pre-Checks

```bash
# Ensure downloads directory exists
mkdir -p downloads
```

### 2. Open greenvideo.cc

```bash
playwright-cli open "https://greenvideo.cc/"
```

Wait for the page to load completely:

```bash
sleep 3
```

### 3. Paste Video URL into Input Field

Take a snapshot first to identify the input field:

```bash
playwright-cli snapshot
```

Find the URL input field (placeholder text: "请将复制的视频链接粘贴到此处") and fill in the YouTube video URL:

```bash
playwright-cli fill "请将复制的视频链接粘贴到此处" "<youtube_url>"
```

### 4. Click Parse Button

Click the "解析" / "开始" button to start parsing:

```bash
playwright-cli click "解析"
```

If the button text is "开始" instead:

```bash
playwright-cli click "开始"
```

### 5. Wait for Parsing Results

Wait for the parsing to complete (may take several seconds):

```bash
sleep 5
```

Take a snapshot to check if results have loaded:

```bash
playwright-cli snapshot
```

If results are not yet loaded, wait and retry:

```bash
sleep 3 && playwright-cli snapshot
```

Repeat up to 3 times until download options appear.

### 6. Parse Download Options

From the snapshot, look for download options. greenvideo.cc typically provides:

- **Video download buttons** — with quality labels (e.g., "720P", "1080P", "4K")
- **Audio download button** — for audio-only extraction
- **Thumbnail/Image download** — for cover images

Parse the snapshot to extract available download options and display them to the user:

```
解析完成！可用的下载选项：

| 编号 | 类型   | 质量   |
|------|--------|--------|
| 1    | 视频   | 1080P  |
| 2    | 视频   | 720P   |
| 3    | 音频   | MP3    |
```

### 7. Select and Download

Based on the user's `--format` preference, click the corresponding download button:

- `best` or `4k` → Click the highest quality video button
- `1080p` → Click "1080P" button
- `720p` → Click "720P" button
- `480p` → Click "480P" button (if available, otherwise use 720P)
- `audio` or `--audio-only` → Click audio download button

```bash
playwright-cli click "1080P"
```

### 8. Handle Download

After clicking the download button, one of two things may happen:

**Case A: Direct download starts**
- The browser initiates a file download automatically
- Wait for the download to complete

**Case B: Opens a new page/tab with video player**
- A new page opens with an embedded video player
- Look for a download button on the player or page
- Take a snapshot to find it:

```bash
playwright-cli snapshot
```

- Click the download button on the player:

```bash
playwright-cli click "下载"
```

If a new tab was opened, close it after download:

```bash
playwright-cli close
```

### 9. Wait for Download to Complete

```bash
sleep 5
```

Check the downloads directory for the new file:

```bash
ls -lt downloads/ | head -5
```

### 10. Report Download Result

On success, report:

```
下载完成！

| 项目   | 内容                                |
|--------|-------------------------------------|
| 标题   | Video Title                         |
| 格式   | 1080P (MP4)                         |
| 文件   | downloads/Video Title.mp4           |
| 大小   | 123 MB                              |
```

On failure, report the issue and suggest:
- Check the URL is correct
- Video might be private, deleted, or region-locked
- Try again — greenvideo.cc may have temporary issues
- Try a different format/quality option

### 11. Close Browser

After download is complete:

```bash
playwright-cli close
```

## Playlist Download

greenvideo.cc does not natively support batch playlist downloading. For playlists:

1. Parse the playlist URL — if greenvideo.cc shows individual video links, download each one
2. If the playlist is not supported, fall back to telling the user: "greenvideo.cc 不支持播放列表批量下载，请逐个视频下载"
3. Alternatively, suggest using yt-dlp as a fallback for playlists

## Troubleshooting

| Issue                                  | Solution                                                        |
|----------------------------------------|-----------------------------------------------------------------|
| Page doesn't load                      | Check network connection, retry                                 |
| Input field not found                  | Take snapshot, look for the correct selector/placeholder        |
| Parse button click fails               | Try alternative button text ("解析" vs "开始" vs "GO")          |
| Parsing takes too long                 | Wait longer (up to 15s), retry                                  |
| No download options appear             | Video may be unsupported/private — try a different video        |
| Download button opens player page      | Find the download icon on the player, click that instead        |
| File not in downloads/                 | Check browser's default download directory, move file            |
| Quality option not available           | The video may not have that quality — use a lower quality       |
| greenvideo.cc returns error            | Video might be region-locked, try again later                   |

## Important Notes

- greenvideo.cc is a free service with no download limits
- It supports 1000+ video platforms including YouTube, Bilibili, Douyin, Instagram, Facebook, etc.
- Downloads go through the browser, so files go to the browser's default download directory — check there if not found in `downloads/`
- The website may update its UI — if selectors break, take a snapshot and adapt
- No proxy configuration needed — greenvideo.cc handles routing
