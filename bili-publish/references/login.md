# Bilibili Login Flow (QR Code)

## Overview

Open Bilibili login page in headed mode. The QR code is displayed by default — no tab switching needed. Wait for the user to scan it with the Bilibili mobile app, detect successful login via cookie check, and save the auth state.

## Prerequisites

- `playwright-cli` must be available (check with `playwright-cli --version`)
- If not available, install: `npm install -g @playwright/cli@latest`

## Step-by-Step Flow

### 1. Open Login Page (Headed)

```bash
playwright-cli open --headed https://passport.bilibili.com/login
```

> **IMPORTANT**: Always use `--headed` mode so the user can see the QR code in the browser window.

### 2. Verify QR Code Is Visible

```bash
playwright-cli snapshot
```

The Bilibili login page shows the QR code **by default**. In the snapshot, look for:
- An `img` element with text "Scan me!" — this is the QR code
- The element is inside an `iframe` or `generic` container
- Alternative login tabs ("密码登录", "短信登录") are visible but **do not need to be clicked** — QR code is the default view

**If the QR code is NOT visible** (rare), look for a tab/button labeled with QR code icon to switch back.

### 3. Tell User to Scan

Output a clear message to the user:

```
请用B站手机APP扫描浏览器中显示的二维码登录：

1. 打开手机上的哔哩哔哩APP
2. 进入 扫一扫 功能
3. 扫描浏览器窗口中显示的二维码
4. 在手机上确认登录

等待扫码中...
```

### 4. Poll for Login Success

Poll every **10-15 seconds** using the SESSDATA cookie check:

```bash
sleep 10 && playwright-cli cookie-get SESSDATA
```

**Success indicators (both should be true):**
- `SESSDATA` cookie is found (domain: `.bilibili.com`, httpOnly, secure, sameSite: Lax)
- Page URL has changed to `https://www.bilibili.com/` (auto-redirect after login)
- Page title shows `哔哩哔哩 (゜-゜)つロ 干杯~-bilibili`

**Polling strategy:**
- First check: 10 seconds after telling user
- Subsequent checks: every 15 seconds
- Maximum wait: ~180 seconds (QR code expiry time)
- If after 60 seconds no login, remind user the QR code may expire soon

**If QR code expired**, reload the page to generate a new QR code:

```bash
playwright-cli reload
playwright-cli snapshot
# Tell user: "QR code refreshed, please scan the new one."
```

### 5. Save Auth State

Once login is confirmed (SESSDATA cookie found):

```bash
cd <project_root> && playwright-cli state-save .bili-auth.json
```

> Make sure to run from the project root directory so `.bili-auth.json` is saved in the correct location.

### 6. Verify Saved State

```bash
ls -la .bili-auth.json
```

The file should be ~100KB+ and contain `cookies` (including `SESSDATA`) and `origins` with localStorage data.

### 7. Close Browser

```bash
playwright-cli close
```

### 8. Report Success

```
登录成功！认证状态已保存到 .bili-auth.json
现在可以使用 /bili publish 来发布视频了。
```

## Troubleshooting

| Issue                              | Solution                                                        |
|------------------------------------|-----------------------------------------------------------------|
| QR code not visible                | Take snapshot, look for QR tab icon/button and click it         |
| Page loads slowly                  | Wait and retry snapshot                                         |
| QR code expired (user didn't scan) | `playwright-cli reload` to refresh QR code, tell user to rescan |
| Login fails after scan             | Try again from step 1, may be a temporary issue                 |
| Cookie not found after scan        | Try `playwright-cli cookie-list --domain=bilibili.com`          |
| Browser not opening                | Check playwright-cli is installed and not blocked               |
| `state-save` path error            | Ensure you're in the project root directory before saving       |

## Key Facts

- **QR code is the default login method** — no need to switch tabs
- Login redirects to `https://www.bilibili.com/` (homepage)
- `SESSDATA` cookie is the primary login indicator
- Auth state file is ~100KB+ (contains many Bilibili cookies and storage)
- QR code expires in ~180 seconds; page must be reloaded for a new one
