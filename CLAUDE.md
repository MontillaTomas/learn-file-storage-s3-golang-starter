# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Tubely is an educational Go web application from the boot.dev "Learn File Servers and CDNs with S3 and CloudFront" course. It manages video metadata and file uploads, teaching local filesystem and AWS S3/CloudFront storage patterns.

## Setup

```bash
cp .env.example .env   # configure environment variables
go mod download        # install dependencies
./samplesdownload.sh   # download sample media files into ./samples/
```

External requirements: `ffmpeg` and `ffprobe` in PATH, AWS CLI with credentials in `~/.aws/credentials`.

## Running

```bash
go run .
# Serves at http://localhost:8091/app/
# Creates tubely.db (SQLite) and ./assets/ on first run
```

## Testing

No test suite exists in this repo (course starter code).

## Architecture

**Entry point**: `main.go` — defines `apiConfig` (holds DB, JWT secret, S3 client, env config), registers all HTTP routes, starts the server.

**Handlers** (`handler_*.go` at root): one file per feature. Each handler method is on `*apiConfig`. Authentication is done by calling `auth.GetBearerToken` + `auth.ValidateJWT` directly inside handlers.

**Internal packages**:
- `internal/auth/` — JWT creation/validation, Argon2id password hashing, bearer token extraction, refresh token generation
- `internal/database/` — SQLite auto-migration on startup, typed query methods for `users`, `videos`, `refresh_tokens` tables

**File storage**: uploads go to `./assets/` locally; S3 bucket/region/CloudFront distribution are configured via `.env` (`S3_BUCKET`, `S3_REGION`, `S3_CF_DISTRO`).

**Frontend**: static files served from `./app/` (HTML/JS/CSS). The JS client communicates with the API using JWT bearer tokens stored in localStorage.

## Key Configuration (`.env`)

| Variable | Purpose |
|---|---|
| `DB_PATH` | SQLite file path |
| `JWT_SECRET` | HS256 signing secret |
| `PLATFORM` | Set to `"dev"` to enable `POST /admin/reset` |
| `ASSETS_ROOT` | Local directory for uploaded files |
| `S3_BUCKET`, `S3_REGION`, `S3_CF_DISTRO` | AWS S3 and CloudFront config |

## Incomplete / Course Exercise Areas

- `handler_upload_thumbnail.go` — partially implemented, has TODO comments
- `handler_upload_video.go` — empty function body, course exercise to implement
- S3 integration in upload handlers is not yet wired up
