---
name: crane-image-tool
description: Use when the user wants to pull, list, inspect, or manage Docker container images without a local Docker daemon. Also use when the user mentions crane, gcrane, krane, or needs to transfer images to an air-gapped/internal network server.
---

# Crane - Container Image Management without Docker

## Overview

[crane](https://github.com/google/go-containerregistry) (go-containerregistry) is a CLI tool for managing container images without requiring Docker. Supports pull, push, list, copy, and inspect operations against any standard OCI registry.

## Pre-flight Check

Before using crane, verify the tool exists:

```bash
ls "D:/development_tool/go-containerregistry/crane.exe"
```

**If not found**, auto-download from GitHub Releases:

1. Get the latest release tag:
```bash
export HTTPS_PROXY=http://127.0.0.1:7890 && curl -sL https://api.github.com/repos/google/go-containerregistry/releases/latest | grep '"tag_name"' | sed 's/.*"v\(.*\)".*/\1/'
```

2. Download Windows amd64 zip (adjust VERSION):
```bash
export HTTPS_PROXY=http://127.0.0.1:7890 && \
mkdir -p "D:/development_tool/go-containerregistry" && \
curl -sL "https://github.com/google/go-containerregistry/releases/download/v${VERSION}/go-containerregistry_Windows_x86_64.tar.gz" -o /tmp/crane.tar.gz && \
tar -xzf /tmp/crane.tar.gz -C "D:/development_tool/go-containerregistry/"
```

Releases page: https://github.com/google/go-containerregistry/releases

## Tool Location

```
D:/development_tool/go-containerregistry/crane.exe
```

Also available: `gcrane.exe` (GCR/Artifact Registry), `krane.exe` (GitHub Container Registry) — but `crane.exe` handles all standard registries.

## Proxy Required

**All crane commands need proxy** due to GFW blocking Docker Hub. Set before every invocation:

```bash
export HTTPS_PROXY=http://127.0.0.1:7890 && export HTTP_PROXY=http://127.0.0.1:7890 && "D:/development_tool/go-containerregistry/crane.exe" <command>
```

## Quick Reference

| Command | Description |
|---------|-------------|
| `crane ls <image>` | List all available tags |
| `crane pull <image>:<tag> <output.tar>` | Pull image to tar file |
| `crane push <file.tar> <image>:<tag>` | Push tar to registry |
| `crane copy <src> <dst>` | Copy image between registries |
| `crane digest <image>:<tag>` | Get image digest (SHA256) |
| `crane manifest <image>:<tag>` | View image manifest |
| `crane config <image>:<tag>` | View image config |

## Common Workflows

### Pull image for offline/air-gapped server

```bash
# 1. Pull to tar
export HTTPS_PROXY=http://127.0.0.1:7890 && export HTTP_PROXY=http://127.0.0.1:7890 && \
"D:/development_tool/go-containerregistry/crane.exe" pull nginx:latest "C:/Users/13720/Desktop/nginx.tar"

# 2. Transfer tar to internal server (scp, USB, etc.)

# 3. On target server: docker load -i nginx.tar
```

### Use China mirror (no proxy)

If proxy is unavailable:

```bash
"D:/development_tool/go-containerregistry/crane.exe" pull docker.m.daocloud.io/library/nginx:latest nginx.tar
```

## Common Mistakes

| Issue | Fix |
|-------|-----|
| Connection timeout / i/o timeout | Set `HTTPS_PROXY` and `HTTP_PROXY` |
| crane.exe not found | Run Pre-flight Check to auto-download |
| Tag not found | Run `crane ls <image>` to verify |
| Large image slow pull | Normal. Use `--platform linux/amd64` for specific arch |

## Notes

- `crane pull` produces a tar compatible with `docker load`
- Default output directory: user's Desktop (`C:/Users/13720/Desktop/`)
- crane does NOT read system proxy settings — must set env vars explicitly
- For private registries, use `crane auth login <registry>` first
