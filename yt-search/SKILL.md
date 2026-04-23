---
name: yt-search
description: Search YouTube videos and download them locally using playwright-cli via greenvideo.cc.
allowed-tools: Bash(playwright-cli:*) Bash(npx:*)
---

# YouTube Video Search & Download

Search YouTube videos via browser automation, then download selected videos using greenvideo.cc online parser.

## Usage

```
/yt search <keyword> [--count N]                          # Search YouTube and list results
/yt download <url> [--format FORMAT] [--audio-only]       # Download a specific video
/yt playlist <url> [--format FORMAT]                      # Download an entire playlist
```

## Output

All downloaded files are saved to the `downloads/` directory in the project root.

## Argument Parsing

### `/yt search`

```
/yt search <keyword> [--count N]
```

| Argument  | Required | Default | Description                          |
|-----------|----------|---------|--------------------------------------|
| keyword   | Yes      | -       | Search query keywords                |
| --count   | No       | 10      | Number of results to display (1-50)  |

### `/yt download`

```
/yt download <url_or_id> [--format FORMAT] [--audio-only]
```

| Argument     | Required | Default | Description                                      |
|--------------|----------|---------|--------------------------------------------------|
| url_or_id    | Yes      | -       | YouTube video URL or video ID                     |
| --format     | No       | best    | Video quality: `4k`, `1080p`, `720p`, `480p`, `best` |
| --audio-only | No       | false   | Download audio only (MP3)                          |

### `/yt playlist`

```
/yt playlist <playlist_url> [--format FORMAT]
```

| Argument      | Required | Default | Description                                      |
|---------------|----------|---------|--------------------------------------------------|
| playlist_url  | Yes      | -       | YouTube playlist URL                              |
| --format      | No       | best    | Video quality: `4k`, `1080p`, `720p`, `480p`, `best` |

## Sub-Commands

### `/yt search`

Follow the detailed search flow in [references/search.md](references/search.md).

Summary:
1. Install playwright-cli if needed
2. Open YouTube search page: `https://www.youtube.com/results?search_query=<keyword>`
3. Take snapshot, parse video results (title, channel, duration, views, URL)
4. Display numbered list to user
5. Ask user to pick a video by number
6. Once user picks → extract video URL → proceed to download

After user picks a video, automatically invoke the download flow with the selected video's URL.

### `/yt download`

Follow the detailed download flow in [references/download.md](references/download.md).

Summary:
1. Install playwright-cli if missing
2. Open greenvideo.cc in browser
3. Paste the YouTube URL into the input field
4. Click "解析" button to parse the video
5. Select the appropriate quality option
6. Click download button to save the video
7. Report download result (file path, size, format)

### `/yt playlist`

greenvideo.cc does not support batch playlist downloading. For playlists:
1. Open the playlist URL in greenvideo.cc to check if it shows individual videos
2. If individual videos are shown, download each one separately
3. If not supported, inform the user and suggest downloading videos one by one

## Format Mapping

| Format   | Action on greenvideo.cc              | Description            |
|----------|--------------------------------------|------------------------|
| best     | Click highest quality video button   | Best available quality |
| 4k       | Click "4K" button (if available)     | Up to 4K resolution    |
| 1080p    | Click "1080P" button                 | Up to 1080p            |
| 720p     | Click "720P" button                  | Up to 720p             |
| 480p     | Click "480P" button (fallback 720P)  | Up to 480p             |
| audio    | Click audio download button          | Audio only (MP3)       |

## Dependencies

| Tool           | Install Command                        | Purpose                          |
|----------------|----------------------------------------|----------------------------------|
| playwright-cli | `npm install -g @playwright/cli@latest` | Browser automation for search & download |

Only playwright-cli is needed — greenvideo.cc handles all video parsing and downloading in the browser.

## Error Handling

| Scenario                       | Action                                                    |
|--------------------------------|-----------------------------------------------------------|
| playwright-cli not found       | Auto-install via `npm install -g @playwright/cli@latest`  |
| greenvideo.cc page fails to load | Check network, retry                                     |
| Parsing fails or no results    | Retry, suggest trying a different video                   |
| Download button not found      | Take snapshot to inspect page, adapt selectors            |
| Video unavailable/private      | Report error, suggest trying a different video            |
| Download fails (network)       | Suggest retry, check network connection                   |
| Playlist not supported         | Inform user, suggest downloading videos one by one        |
| No search results              | Suggest different keywords                                |

## Important Notes

- YouTube search uses `playwright-cli` browser automation to browse YouTube and extract results
- Video downloading uses greenvideo.cc — a free online video parser supporting 1000+ platforms
- No need for yt-dlp or ffmpeg — greenvideo.cc handles everything in the browser
- Downloaded files may go to the browser's default download directory; check there if not found in `downloads/`
- The website UI may change over time — if selectors break, take a snapshot and adapt
- The `--audio-only` flag is a shortcut for `--format audio`
