# YouTube Search Flow (playwright-cli)

## Overview

Use playwright-cli to browse YouTube, search for videos by keyword, extract search results, and display them as a numbered list for the user to pick from.

## Step-by-Step Flow

### 1. Open YouTube Search Page

```bash
playwright-cli open https://www.youtube.com/results?search_query=<URL_ENCODED_KEYWORD>
```

> Encode the keyword for URL (replace spaces with `+`, special characters with percent encoding).

Example:
```bash
playwright-cli open "https://www.youtube.com/results?search_query=AI+tutorial"
```

### 2. Wait for Results to Load

```bash
sleep 3 && playwright-cli snapshot
```

YouTube search results typically take 2-3 seconds to fully render. Wait briefly then take a snapshot.

### 3. Parse Search Results

The YouTube search results page has a structure like:

```yaml
- generic (video renderer):
  - link (video title) [ref=eN]:
    - text: "Video Title Here"
    - /url: /watch?v=VIDEO_ID
  - generic:
    - link "Channel Name" [ref=eN]
    - text: "123K views · 2 days ago"
  - generic:
    - text: "10:30"  # duration
```

**For each video result, extract:**
- **Title**: text content of the video title link
- **URL**: `https://www.youtube.com` + the `/url` attribute (e.g., `/watch?v=dQw4w9WgXcQ`)
- **Channel**: text of the channel link
- **Views & Age**: text like "123K views · 2 days ago"
- **Duration**: text like "10:30" (usually in a separate overlay element)

### 4. Display Results to User

Format the results as a numbered list:

```
YouTube 搜索结果 "AI tutorial":

 #  | 标题                                    | 频道         | 时长   | 播放量
----|-----------------------------------------|-------------|--------|--------
 1  | AI Tutorial for Beginners              | TechChannel | 15:30  | 1.2M views
 2  | Learn AI in 10 Minutes                  | CodeAcademy | 10:22  | 500K views
 3  | Complete AI Course 2024                 | FreeCodeCamp| 3:45:00| 2.1M views
 ...

请输入要下载的视频编号（输入 0 取消）：
```

### 5. Handle User Selection

When the user picks a number:
1. Extract the corresponding video URL from the parsed results
2. Automatically proceed to the download flow with that URL

If the user wants to see more results:
- Scroll down to load more: `playwright-cli press End`
- Take a new snapshot
- Parse and display additional results

### 6. Close Browser After Selection

```bash
playwright-cli close
```

Close the browser once the user has selected a video and we have the URL.

## Parsing Tips

### Identifying Video Results

Video results are the main items on the search page. Look for:
- Links with `/watch?v=` in the URL — these are video links
- Each video result is usually inside a container with the video title, channel info, and metadata

### Filtering Out Non-Video Results

YouTube search also shows:
- **Ads** (sponsored videos) — usually marked with "Ad" or "Sponsored"
- **Shorts** — video duration shows as "0:XX" and may have different layout
- **Playlists** — URL contains `/playlist?list=`
- **Channels** — URL contains `/c/` or `/@`
- **Movie rentals** — marked with "Movie" badge

**Only include standard video results** (`/watch?v=` URLs) in the numbered list. You can optionally note if a result is a Short or ad.

### Handling Long Result Pages

YouTube uses infinite scroll. To see more results:

```bash
playwright-cli press End
sleep 2
playwright-cli snapshot
```

Each scroll loads ~10 more results. Repeat as needed based on the `--count` argument.

### Extracting Video ID

The video URL format is `/watch?v=VIDEO_ID`. Extract the `VIDEO_ID` part:
- It's an 11-character string (e.g., `dQw4w9WgXcQ`)
- The full URL is `https://www.youtube.com/watch?v=VIDEO_ID`

## Search Page URL Patterns

| Search Type         | URL Pattern                                                    |
|---------------------|----------------------------------------------------------------|
| Basic search        | `https://www.youtube.com/results?search_query=KEYWORD`         |
| With filters        | `https://www.youtube.com/results?search_query=KEYWORD&sp=FILTER` |
| Sort by upload date | `&sp=CAI%253D` (append to search URL)                          |
| Sort by view count  | `&sp=CAM%253D` (append to search URL)                          |
| Sort by rating      | `&sp=CAE%253D` (append to search URL)                          |

## Troubleshooting

| Issue                           | Solution                                                    |
|---------------------------------|-------------------------------------------------------------|
| Page loads blank/white          | Wait longer (5s), then retry snapshot                       |
| Consent/cookie dialog appears   | Click "Accept all" or "I agree" button in snapshot          |
| Results not loading             | Check network, reload page                                  |
| No video results found          | Try different keywords, check if YouTube is accessible       |
| Snapshot too large              | Use `--depth` parameter to limit snapshot depth              |
| Video titles truncated          | Use `playwright-cli eval` to get full title from element     |
| Region block / captcha          | May need to handle bot detection, try again later            |
