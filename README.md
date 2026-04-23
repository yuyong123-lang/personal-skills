# Personal Skills

A collection of custom [Claude Code](https://claude.ai/code) skills — reusable slash commands that extend Claude's capabilities for video production, browser automation, document processing, code development, and GitHub workflows.

Each skill is a self-contained directory with a `SKILL.md` definition and optional reference docs. Drop them into your `.claude/skills/` folder and invoke with `/<skill-name>`.

## What's Inside

- **Video & Media** — AI explainer video generation, Bilibili publishing, YouTube search & download
- **Office Documents** — Create and edit .pptx, .docx, .xlsx with rich style templates
- **Development** — Structured coding workflow, design document generation, docs management
- **GitHub** — Full CLI workflow for repos, PRs, issues, releases, and Actions
- **Browser Automation** — Playwright-based web interaction and testing

## Skills

| Skill | Description |
|-------|-------------|
| **ai-video** | Generate AI/tech explainer videos with TTS voiceover and publish to Bilibili |
| **bili-publish** | Auto-publish videos to Bilibili via browser automation |
| **code-development** | Structured code development workflow — requirement analysis, implementation, testing |
| **gh-cli** | GitHub CLI operations — repos, PRs, issues, releases, actions, API calls |
| **detailed-design-doc** | Generate comprehensive detailed design documents for modules |
| **docs-management** | Create, update, and organize project documentation |
| **md-to-docx** | Convert and merge Markdown content into existing Word documents |
| **officecli** | Create, analyze, and modify Office documents (.docx, .xlsx, .pptx) |
| **playwright-cli** | Automate browser interactions and test web pages with Playwright |
| **yt-search** | Search YouTube videos and download them locally |

## Usage

Copy any skill directory into your `.claude/skills/` folder, then invoke it in Claude Code with `/<skill-name>`.
