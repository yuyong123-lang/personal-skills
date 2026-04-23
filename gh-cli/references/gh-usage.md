# GitHub CLI (gh) Usage Guide

## Authentication

```bash
# Interactive login (browser-based)
gh auth login

# Login with token
gh auth login --with-token < token.txt

# Check auth status
gh auth status

# Logout
gh auth logout
```

## Repository Management

```bash
# Create a new repo (public)
gh repo create my-repo --public --clone

# Create from local directory and push
gh repo create my-repo --public --source=. --push

# Create a private repo
gh repo create my-repo --private --clone

# List your repos
gh repo list

# List repos for a specific user/org
gh repo list owner --limit 50

# Clone a repo
gh repo clone owner/repo

# Fork a repo
gh repo fork owner/repo --clone

# View repo info
gh repo view owner/repo

# Open repo in browser
gh repo view owner/repo --web

# Delete a repo (dangerous!)
gh repo delete owner/repo --yes

# Edit repo settings
gh repo edit --description "New description"
gh repo edit --homepage "https://example.com"
gh repo edit --visibility private
```

## Pull Requests

```bash
# Create a PR (interactive)
gh pr create

# Create with title and body
gh pr create --title "Fix bug" --body "Description here"

# Create with base branch specified
gh pr create --base main --head feature-branch --title "Title" --body "Body"

# List PRs
gh pr list
gh pr list --state open
gh pr list --author me
gh pr list --label bug

# View a PR
gh pr view 123
gh pr view 123 --web

# Checkout a PR locally
gh pr checkout 123

# Check CI status
gh pr checks 123

# Merge a PR
gh pr merge 123 --merge
gh pr merge 123 --squash
gh pr merge 123 --rebase

# Close/Reopen
gh pr close 123
gh pr reopen 123

# Add a comment
gh pr comment 123 --body "LGTM!"

# Review a PR
gh pr review 123 --approve --body "Looks good"
gh pr review 123 --request-changes --body "Please fix X"

# View diff
gh pr diff 123

# Update branch
gh pr update-branch 123
```

## Issues

```bash
# Create an issue
gh issue create --title "Bug report" --body "Description"

# Create with labels and assignees
gh issue create --title "Bug" --label bug --assignee me

# List issues
gh issue list
gh issue list --state open
gh issue list --label bug
gh issue list --assignee me

# View an issue
gh issue view 123
gh issue view 123 --web

# Close/Reopen
gh issue close 123
gh issue reopen 123

# Edit an issue
gh issue edit 123 --title "New title"

# Add a comment
gh issue comment 123 --body "Comment text"

# Pin/Unpin
gh issue pin 123
gh issue unpin 123
```

## Releases

```bash
# Create a release
gh release create v1.0.0 --title "v1.0.0" --notes "Release notes here"

# Create from a tag with auto-generated notes
gh release create v1.0.0 --generate-notes

# Upload assets to a release
gh release upload v1.0.0 ./dist/*.zip

# List releases
gh release list

# View a release
gh release view v1.0.0

# Download release assets
gh release download v1.0.0

# Delete a release
gh release delete v1.0.0 --yes
```

## GitHub Actions

```bash
# List workflow runs
gh run list

# View a specific run
gh run view 12345

# Watch a run in real-time
gh run watch

# Re-run a workflow
gh run rerun 12345

# List workflows
gh workflow list

# View workflow details
gh workflow view "CI"

# Trigger a workflow manually
gh workflow run "CI" --ref main

# Manage Actions caches
gh cache list
gh cache delete --all
```

## Gists

```bash
# Create a gist
gh gist create file.txt --public
gh gist create file.txt --desc "Description"

# List your gists
gh gist list

# View a gist
gh gist view abc123

# Edit a gist
gh gist edit abc123 file.txt

# Delete a gist
gh gist delete abc123
```

## API

```bash
# Make an API request
gh api repos/owner/repo

# Get specific fields with jq
gh api repos/owner/repo --jq '.stargazers_count'

# POST request
gh api repos/owner/repo/issues -f title="Bug" -f body="Description"

# Paginated results
gh api repos/owner/repo/issues --paginate
```

## Search

```bash
# Search repos
gh search repos "react framework" --language typescript --sort stars

# Search issues
gh search issues "bug report" --repo owner/repo --state open

# Search PRs
gh search prs --author me --state open
```

## Tips

- Use `-R owner/repo` to operate on a different repo without cd
- Use `--web` flag to open anything in the browser
- Use `--json` flag to get JSON output for scripting
- Use `--jq` to filter JSON output with jq expressions
- Most commands accept `--limit N` to control result count
- Use `gh alias set co 'pr checkout'` to create shortcuts
