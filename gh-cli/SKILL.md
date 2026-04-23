---
name: gh-cli
description: Use when the user wants to perform GitHub operations via gh CLI — creating repos, managing PRs, issues, releases, actions, gists, API calls, or any GitHub workflow from the command line.
allowed-tools: Bash(gh:*)
---

# gh-cli

Execute GitHub CLI (`gh`) operations for repository management, pull requests, issues, releases, and more.

## When to Use

- User asks to create, clone, fork, or manage GitHub repos
- User wants to create, list, merge, or review pull requests
- User wants to create, close, or comment on issues
- User wants to create releases or upload assets
- User wants to check GitHub Actions workflow status
- User wants to make GitHub API requests
- User says "push to GitHub", "create a PR", "open an issue", etc.

## Key Guidelines

1. **Always check auth first** — run `gh auth status` if login state is uncertain.
2. **Default to `--public`** for new repos unless user specifies private.
3. **Use `--web`** when user wants to view something in the browser.
4. **Use `-R owner/repo`** to operate on a repo without being in its directory.
5. **Use `--json` + `--jq`** for scripting and extracting specific fields.
6. **For PR bodies**, use HEREDOC syntax to ensure proper formatting.
7. **Never force push to main/master** — warn the user if they request this.
8. **Confirm before destructive actions** (delete repo, delete release, etc.).
9. **Use `--yes` flag** only when the user has explicitly confirmed the action.

## Reference

See `references/gh-usage.md` for the full command reference covering:

- Authentication
- Repository management (create, clone, fork, delete, edit)
- Pull requests (create, list, merge, review, checkout, diff)
- Issues (create, list, close, comment, pin)
- Releases (create, upload, download, list)
- GitHub Actions (run, watch, workflow trigger)
- Gists (create, list, view, edit, delete)
- API requests
- Search
