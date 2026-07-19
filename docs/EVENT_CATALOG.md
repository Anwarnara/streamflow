# Event Catalog

**Version:** 3.3.0
**Last Updated:** 2026-07-19

## Overview

This document catalogs all internal events, SSE event types, and system-level state transitions in StreamFlow. Events are either delivered via SSE to the frontend or represent internal Go channel/goroutine signals.

---

## SSE Event Streams

### `/api/streams/:id/health-stream`
**Interval:** 1 second
**Consumer:** Dashboard stream card health bar, `/stream-health` page

| Field | Type | Description |
|-------|------|-------------|
| `stream_id` | string | UUID of the stream |
| `fps` | float | Current frames per second from FFmpeg |
| `bitrate` | float | Current output bitrate in kbps |
| `speed` | float | FFmpeg processing speed (1.0 = real-time) |
| `drop_frames` | int | Cumulative dropped frames |
| `dup_frames` | int | Cumulative duplicate frames |
| `total_frames` | int | Total frames processed |
| `out_time` | string | Stream elapsed time (HH:MM:SS.ss) |
| `status` | string | `HEALTHY` / `WARNING` / `DEGRADED` / `OFFLINE` |
| `last_updated` | string | ISO 8601 timestamp |

**Status thresholds:**
- `HEALTHY`: speed >= 0.95, drop_frames < 1% of total
- `WARNING`: speed 0.80–0.95, or drop_frames 1–5% of total
- `DEGRADED`: speed < 0.80, or drop_frames > 5% of total, or bitrate = 0

---

### `/api/streams/:id/logs-stream`
**Interval:** 500ms delta
**Consumer:** Terminal-style log modal on stream card (triggered by "Logs" button)

Delivers raw lines from the stream's `LogBuffer` (FIFO, max 500 lines). Each SSE event carries one log line as plain text.

---

### `/api/streams/:id/chat-stream`
**Interval:** Event-driven (pushed on new message arrival)
**Consumer:** Chat panel on stream detail view

| Field | Type | Description |
|-------|------|-------------|
| `platform` | string | Source platform (youtube, twitch) |
| `author` | string | Display name |
| `message` | string | Chat message text |
| `received_at` | string | ISO 8601 timestamp |

---

## Stream Lifecycle State Transitions

```
         ┌──────────┐
    ┌───►│ offline  │◄─────────────┐
    │    └────┬─────┘              │
    │         │ start / schedule   │
    │         ▼                    │
    │    ┌──────────┐             │
    │    │scheduled │             │ stop / error
    │    └────┬─────┘             │
    │         │ schedule_time met  │
    │         ▼                    │
    │    ┌──────────┐             │
    └────│streaming │─────────────┘
         └──────────┘
```

| Transition | Trigger | Actor |
|------------|---------|-------|
| `offline` → `streaming` | Manual start via `POST /api/streams/:id/start` | User |
| `offline` → `scheduled` | `schedule_time` set on stream | User |
| `scheduled` → `streaming` | Scheduler goroutine tick finds `schedule_time <= NOW()` | System (scheduler.go) |
| `streaming` → `offline` | Manual stop, FFmpeg natural exit, or error | User / System |
| `streaming` → `error` | FFmpeg crash, all reconnect attempts exhausted | System |
| `error` → `offline` | User acknowledges / resets | User |

---

## Internal Goroutine Events

These are not exposed over HTTP but represent internal signals in the Go runtime.

| Event | File | Description |
|-------|------|-------------|
| FFmpeg stderr line received | `health_monitor.go` | Triggers health metric parse and `LogBuffer` append |
| FFmpeg process exit | `simulcast.go` | Triggers auto-reconnect logic with exponential backoff |
| Scheduler tick | `scheduler.go` | 30s tick; queries DB for streams due to start |
| healthStore write | `health_monitor.go` | Updates in-memory map under `sync.RWMutex` |

---

## Platform Integration Events

| Platform | Event Type | Description |
|----------|-----------|-------------|
| YouTube | Chat message poll | `chat_aggregator.go` polls YouTube Live Chat API |
| Twitch | Chat message | IRC-over-WebSocket connection in `chat_aggregator.go` |
| RTMP ingest | `on_publish` | Nginx-RTMP calls `POST /api/rtmp/auth` when a publisher connects |
