---
name: bili-publish
description: Auto-publish videos to Bilibili via browser automation with playwright-cli.
allowed-tools: Bash(playwright-cli:*) Bash(npx:*)
---

# Bilibili Video Auto-Publish

Automate Bilibili login (QR code) and video publishing via browser automation.

## Usage

```
/bili login                                          # QR code login, save auth state
/bili publish <video_path> --title "Title"           # Upload and publish a video
```

## Auth State

- Saved to `.bili-auth.json` in the project root
- Contains cookies + localStorage from Bilibili login session
- **This file must never be committed to git** (already in `.gitignore`)

## Argument Parsing

When the user invokes `/bili publish`, parse the arguments as follows:

```
/bili publish <video_path> --title "Title" [--desc "Description"] [--tags "tag1,tag2"]
```

| Argument   | Required | Description                          |
|------------|----------|--------------------------------------|
| video_path | Yes      | Absolute or relative path to video   |
| --title    | Yes      | Video title                          |
| --desc     | No       | Video description                    |
| --tags     | No       | Comma-separated tags                 |

Extract these values from the user's input and use them in the publish flow.

## Sub-Commands

### `/bili login`

Follow the detailed login flow in [references/login.md](references/login.md).

Summary:
1. Open `https://passport.bilibili.com/login` with playwright-cli (headed mode)
2. Take snapshot — QR code is displayed by default, no tab switching needed
3. **Tell the user to scan the QR code with their Bilibili mobile app**
4. Poll for login success every 10-15s by checking `SESSDATA` cookie
5. After login confirmed (cookie found + URL changed to `bilibili.com`), save state
6. Close browser, report success

### `/bili publish`

Follow the detailed publish flow in [references/publish.md](references/publish.md).

Prerequisites:
- `.bili-auth.json` must exist (user must run `/bili login` first)
- Video file must exist at the specified path

Summary:
1. Pre-check: verify `.bili-auth.json` and video file both exist
2. Open blank page: `playwright-cli open --headed about:blank`
3. Load auth state: `playwright-cli state-load .bili-auth.json`
4. Navigate: `playwright-cli goto https://member.bilibili.com/platform/upload/video/frame`
5. Verify not redirected to login page (title should be "创作中心")
6. Click upload area to trigger file chooser, then `playwright-cli upload "<video_path>"`
7. Wait for upload completion (poll snapshot, look for "上传完成")
8. Fill title: click textbox, Ctrl+A, type title
9. Add tags: click tag input, type tag, press Enter (repeat for each tag)
10. Fill description: use `run-code` with `.ql-editor` selector (Quill editor)
11. Click "立即投稿" submit button
12. Wait for "稿件投递成功" confirmation
13. Save updated auth state, close browser, report success

## Error Handling

| Scenario                    | Action                                              |
|-----------------------------|-----------------------------------------------------|
| `.bili-auth.json` missing   | Tell user to run `/bili login` first                |
| Auth expired (redirect)     | Tell user to re-run `/bili login`                   |
| Video file not found        | Report error with expected path                      |
| Upload timeout              | Report timeout, suggest checking network and retry  |
| Element not found           | Take snapshot, try alternative selectors             |
| Playwright-cli not found    | Suggest: `npm install -g @playwright/cli@latest`    |

## Important Notes

- Always use **headed mode** (`--headed`) for both login and publish so the user can see the browser
- **Critical**: Must open `about:blank` first, then `state-load`, then navigate. Cannot `state-load` without an open browser, and cannot load state after navigating (redirect happens before state is applied)
- After each playwright-cli command, read the snapshot output to understand the current page state
- Use `playwright-cli snapshot` liberally to verify state before taking actions
- The Bilibili upload page may change its UI; use snapshot refs and adapt selectors accordingly
- Always clean up: close the browser when done with `playwright-cli close`
- The description field uses **Quill editor** (`.ql-editor` class), not a standard textarea. Must use `run-code` to fill it
- Some fields (partition, some tags) are auto-filled by Bilibili based on video content analysis
