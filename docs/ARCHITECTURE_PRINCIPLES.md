# Architecture Principles

**Version:** 3.3.0
**Last Updated:** 2026-07-19

## Overview

StreamFlow is an enterprise live streaming platform built for reliability, low-latency output, and operational simplicity. This document captures the architectural principles that guide all design and implementation decisions.

---

## Stack

| Layer | Technology |
|-------|------------|
| Backend | Go 1.25 + Echo v4 |
| Database | PostgreSQL 14 (`streamflow`) |
| Reverse Proxy | Nginx (:7575) |
| RTMP Ingest | Nginx-RTMP (:1935) |
| Streaming Engine | FFmpeg (custom AVX-512 build at `/usr/local/bin/ffmpeg`) |
| Service Manager | systemd (`streamflow-backend.service`) |

---

## Core Principles

### 1. FFmpeg as the Streaming Engine

FFmpeg is the single source of truth for all media operations: encoding, muxing, thumbnail generation, and health metrics. The Go backend orchestrates FFmpeg processes — it does not reimplement media logic.

- Always invoke `/usr/local/bin/ffmpeg` explicitly.
- Parse FFmpeg stderr in real-time via goroutine scanners.
- Maintain a `LogBuffer` FIFO (max 500 lines) per active process for UI log streaming.

### 2. In-Process State for Hot Paths

Health metrics are kept in an in-memory `healthStore` (map protected by `sync.RWMutex`). This eliminates DB round-trips on every SSE tick. The store is populated by the FFmpeg stderr parser goroutine and read by health endpoints.

Acceptable trade-off: metrics are lost on process restart, but they are ephemeral by nature and repopulated within seconds of stream start.

### 3. SSE over WebSockets for Server Push

StreamFlow uses Server-Sent Events (SSE) for all server-to-client push:
- Health metrics (1s interval)
- FFmpeg log streaming (500ms delta)
- Chat aggregation

SSE is simpler to implement, requires no upgrade handshake, works through standard HTTP proxies, and is sufficient for unidirectional push. WebSockets are reserved for cases requiring bidirectional communication (none currently exist in StreamFlow).

### 4. Simulcast Routing: tee vs. per-destination

When starting a simulcast stream, `simulcast.go` evaluates all destinations:

- **allCopy path:** If every destination is copy-compatible (no transcode overrides), use FFmpeg's `tee` muxer. One decode, zero CPU transcoding, multiple outputs.
- **per-destination path:** If any destination has encode overrides (bitrate, resolution, fps, codec, preset), launch separate FFmpeg processes per destination via `DestinationConfig`.

This decision is made at stream start time and logged. See `DECISIONS.md` ADR-003.

### 5. Scheduled Execution via Goroutine Ticker

The scheduler (`internal/streaming/scheduler.go`) runs a goroutine with a 30-second ticker. It queries `streams WHERE schedule_time <= NOW() AND status != 'streaming'`, starts each eligible stream, and clears `schedule_time`. No external job queue or cron daemon is required.

### 6. Stateless HTTP, Stateful Streaming

HTTP handlers are stateless and context-free. Streaming state (running FFmpeg PIDs, health data, log buffers) is held in package-level maps in `internal/streaming/`. The DB is the durable record; in-memory maps are the hot cache.

### 7. Single Binary Deployment

The entire backend compiles to a single binary at `streaming-go/bin/backend`. Templates are embedded or served from the `templates/` directory. No container runtime is required. Deployment is: build → copy binary → `systemctl restart`.

### 8. Database Conventions

- All PKs are UUIDs.
- All timestamps are `TIMESTAMPTZ`.
- Migrations are idempotent (`ADD COLUMN IF NOT EXISTS`, `CREATE TABLE IF NOT EXISTS`).
- No ORM — raw `database/sql` with `pgx` driver for predictable query behavior.

### 9. Security Boundaries

- RTMP stream keys validated via Nginx `on_publish` callback to `/api/rtmp/auth`.
- Session tokens (72h) stored in `user_sessions`, checked on every authenticated route.
- YouTube OAuth tokens stored per-user; never logged or returned in list responses.

### 10. Observability

- FFmpeg stderr → `LogBuffer` → `/api/streams/:id/logs-stream` SSE
- FFmpeg stderr → `healthStore` → `/api/streams/:id/health` + `/api/streams/health/all`
- Stream analytics persisted to `stream_analytics` table
- Service logs via `journalctl -u streamflow-backend.service`

---

## Component Diagram

```
Browser
  │
  ├─ HTTP/HTTPS ──► Nginx :7575 ──► Go Backend :8100
  │                                   │
  │                                   ├─ PostgreSQL :5432
  │                                   │
  │                                   └─ FFmpeg processes (per stream)
  │                                        │
  │                                        └─ RTMP out → platforms
  │
  └─ RTMP ──► Nginx-RTMP :1935
                  │
                  └─ on_publish auth ──► Go Backend :8100/api/rtmp/auth
```

---

## Key Files

| File | Responsibility |
|------|---------------|
| `cmd/backend/main.go` | Server init, route registration, scheduler start |
| `internal/streaming/handlers.go` | HTTP handlers for stream CRUD and control |
| `internal/streaming/simulcast.go` | Multi-destination FFmpeg orchestration |
| `internal/streaming/health_monitor.go` | FFmpeg stderr parsing, healthStore, SSE |
| `internal/streaming/scheduler.go` | Scheduled stream auto-start |
| `internal/streaming/destinations.go` | Destination CRUD with encode overrides |
| `internal/streaming/chat_aggregator.go` | Unified chat SSE from YouTube/Twitch |
| `internal/streaming/playlist_stream.go` | Playlist-based streaming logic |
| `internal/streaming/rtmp_auth.go` | RTMP publish auth callback |
| `internal/streaming/rtmp_stats.go` | RTMP stats endpoint |
