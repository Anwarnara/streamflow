# Agent Rules

**Version:** 3.3.0
**Last Updated:** 2026-07-19

## Purpose

This document defines the rules, constraints, and operating procedures for AI agents (Kiro, Hermes, or similar) working on the StreamFlow codebase. Any agent modifying this project must follow these rules.

---

## 1. Project Identity

- **App Name:** StreamFlow
- **Type:** Enterprise Live Streaming Platform
- **Backend:** Go 1.25 + Echo v4, listening on `:8100`
- **Database:** PostgreSQL 14, database name `streamflow`
- **Reverse Proxy:** Nginx on `:7575`
- **RTMP Ingest:** Nginx-RTMP on `:1935`
- **FFmpeg:** `/usr/local/bin/ffmpeg` (custom AVX-512 build — do not replace or alias)
- **Binary:** `/var/www/streaming/streaming-go/bin/backend`
- **Service:** `streamflow-backend.service` (systemd)

---

## 2. Directory Structure

```
/var/www/streaming/
├── streaming-go/
│   ├── cmd/backend/main.go
│   ├── internal/streaming/
│   │   ├── handlers.go
│   │   ├── simulcast.go
│   │   ├── health_monitor.go
│   │   ├── scheduler.go
│   │   ├── destinations.go
│   │   ├── chat_aggregator.go
│   │   ├── rtmp_auth.go
│   │   ├── rtmp_stats.go
│   │   └── playlist_stream.go
│   ├── templates/
│   └── bin/backend
└── docs/
```

---

## 3. Coding Rules

### Go Backend
- All new handlers must be registered in `cmd/backend/main.go` under the appropriate route group.
- Use Echo v4 context (`echo.Context`) for all handlers.
- All DB queries must use parameterized queries — no string interpolation.
- UUIDs for all primary keys; use `github.com/google/uuid`.
- Errors must be logged with context before returning HTTP error responses.
- Goroutines must have clear lifecycle management (context cancellation or stop channels).
- New streaming features go in `internal/streaming/`.

### FFmpeg
- Always use `/usr/local/bin/ffmpeg` — never rely on PATH resolution.
- Parse stderr for health metrics in real-time using a goroutine scanner.
- The `-re` flag must be used for file-based input to avoid CPU spikes.
- The `LogBuffer` FIFO (max 500 lines) must be maintained for all FFmpeg processes.

### Database
- Migrations must be idempotent (`IF NOT EXISTS`, `IF NOT EXISTS` column checks).
- Never DROP columns without explicit user approval and a backup step.
- All new tables require `created_at TIMESTAMPTZ DEFAULT NOW()`.
- Foreign keys must reference `users(id)` or the appropriate parent table with `ON DELETE CASCADE` where appropriate.

### Frontend / Templates
- Templates live in `/var/www/streaming/streaming-go/templates/`.
- Use `layout.html` as the base layout — add new pages by extending it.
- New menu items must be added to both the desktop sidebar AND the mobile bottom nav in `layout.html`.
- SSE endpoints must be consumed with `EventSource` API; do not use polling where SSE is available, unless the endpoint is a snapshot (e.g., `/health/all` polled at 3s is acceptable).

---

## 4. API Rules

- All API routes are prefixed with `/api/`.
- Stream-specific routes follow the pattern `/api/streams/:id/...`.
- New endpoints must be documented in `docs/API_SPEC.md` before or immediately after implementation.
- SSE endpoints must set headers: `Content-Type: text/event-stream`, `Cache-Control: no-cache`, `Connection: keep-alive`.

---

## 5. Service Management

- After rebuilding the binary, restart the service: `systemctl restart streamflow-backend.service`
- Check service status: `systemctl status streamflow-backend.service`
- View live logs: `journalctl -u streamflow-backend.service -f`
- Do NOT restart the service mid-stream without confirming with the user — it will kill active FFmpeg processes.

---

## 6. Documentation Rules

- All docs live in `/var/www/streaming/docs/`.
- Every doc must carry `**Version:** x.y.z` and `**Last Updated:** YYYY-MM-DD` at the top.
- `CHANGELOG.md` must be updated for every version bump.
- `WORKLOG.md` must be updated with a dated entry for every significant change session.
- `API_SPEC.md` must reflect the current state of all routes.
- `DATABASE.md` must reflect the current schema after any migration.

---

## 7. Security Rules

- Passwords must be hashed with bcrypt (cost ≥ 12) — never stored in plaintext.
- Session tokens expire after 72 hours (`user_sessions` table).
- RTMP stream keys must not be logged at INFO level.
- YouTube OAuth tokens must not be logged or exposed in API responses.
- All file upload paths must be validated to prevent path traversal.

---

## 8. Version Tracking

- Current version: **v3.3.0**
- Version must be updated in: `CHANGELOG.md`, `WORKLOG.md`, `ROADMAP.md`, and all doc headers.
- Feature numbering skipped 4 by design (Feature 4 was descoped); do not renumber existing features.

---

## 9. Prohibited Actions

- Do NOT use `os/exec` to run arbitrary user-supplied shell commands.
- Do NOT expose internal FFmpeg process PIDs in public API responses.
- Do NOT delete stream history records automatically.
- Do NOT modify `nginx.conf` or `nginx-rtmp.conf` without explicit instruction.
- Do NOT commit `.env` files or secrets to version control.
- Do NOT run `DROP TABLE` without explicit user confirmation and backup.
