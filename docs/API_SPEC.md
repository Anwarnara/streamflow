# API Specification

**Version:** 3.3.0
**Last Updated:** 2026-07-19

## Overview

StreamFlow backend exposes a REST API on port `:8100` via Echo v4. All endpoints are prefixed with `/api/`. Authentication uses session tokens stored in `user_sessions` (72-hour expiry).

---

## Authentication

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/auth/login` | Login, returns session token |
| POST | `/api/auth/logout` | Invalidate session |
| GET | `/api/auth/me` | Current user info |

---

## Users

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/users` | List users |
| POST | `/api/users` | Create user |
| GET | `/api/users/:id` | Get user |
| PUT | `/api/users/:id` | Update user |
| DELETE | `/api/users/:id` | Delete user |
| POST | `/api/users/:id/avatar` | Upload avatar |

---

## Videos

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/videos` | List videos (paginated) |
| POST | `/api/videos/upload` | Chunked upload |
| GET | `/api/videos/:id` | Get video metadata |
| PUT | `/api/videos/:id` | Update video |
| DELETE | `/api/videos/:id` | Delete video |
| GET | `/api/videos/:id/thumbnail` | Get thumbnail |
| POST | `/api/videos/:id/thumbnail/generate` | Trigger FFmpeg thumbnail gen |

---

## Streams

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/streams` | List streams |
| POST | `/api/streams` | Create stream |
| GET | `/api/streams/:id` | Get stream |
| PUT | `/api/streams/:id` | Update stream |
| DELETE | `/api/streams/:id` | Delete stream |
| POST | `/api/streams/:id/start` | Start stream (launch FFmpeg) |
| POST | `/api/streams/:id/stop` | Stop stream (kill FFmpeg) |

---

## Stream Health (v3.3.0 — Feature 1)

### GET `/api/streams/:id/health`

Returns a real-time health snapshot for a single stream parsed from FFmpeg stderr.

**Response:**
```json
{
  "stream_id": "uuid",
  "fps": 29.97,
  "bitrate": 2487.5,
  "speed": 1.0,
  "drop_frames": 0,
  "dup_frames": 0,
  "total_frames": 18432,
  "out_time": "00:10:14.23",
  "status": "HEALTHY",
  "last_updated": "2026-07-19T12:34:56Z"
}
```

**Status values:** `HEALTHY` | `WARNING` | `DEGRADED` | `OFFLINE`

---

### GET `/api/streams/:id/health-stream`

SSE endpoint. Pushes health updates every 1 second.

**Headers:**
- `Content-Type: text/event-stream`
- `Cache-Control: no-cache`
- `Connection: keep-alive`

**Event format:** `data: {json payload}\n\n`

---

### GET `/api/streams/health/all`

Snapshot of health metrics for ALL currently live streams. Zero DB queries — served from in-memory `healthStore`.

**Response:**
```json
[
  {
    "stream_id": "uuid",
    "fps": 29.97,
    "bitrate": 2490.0,
    "speed": 1.0,
    "drop_frames": 2,
    "dup_frames": 0,
    "total_frames": 54210,
    "out_time": "00:30:10.44",
    "status": "WARNING",
    "last_updated": "2026-07-19T12:34:56Z"
  }
]
```

---

## FFmpeg Log Streaming (v3.3.0 — Feature 3)

### GET `/api/streams/:id/logs-stream`

SSE endpoint. Streams delta lines from the stream's `LogBuffer` (FIFO, max 500 lines) every 500ms.

**Headers:**
- `Content-Type: text/event-stream`
- `Cache-Control: no-cache`
- `Connection: keep-alive`

**Event format:** `data: <log line>\n\n`

Used by the frontend terminal-style log modal on live stream cards.

---

## Stream Destinations (v3.3.0 — Feature 6)

### GET `/api/streams/:id/destinations`

List all simulcast destinations for a stream.

**Response:**
```json
[
  {
    "id": "uuid",
    "stream_id": "uuid",
    "rtmp_url": "rtmp://live.twitch.tv/app",
    "stream_key": "live_xxx",
    "platform": "twitch",
    "platform_icon": "twitch.svg",
    "is_active": true,
    "bitrate": 0,
    "resolution": "",
    "fps": 0,
    "video_codec": "",
    "preset": "",
    "created_at": "2026-07-19T00:00:00Z"
  }
]
```

`bitrate: 0`, `fps: 0`, empty `resolution`/`video_codec`/`preset` all mean "inherit from parent stream."

---

### POST `/api/streams/:id/destinations`

Create a new simulcast destination.

**Body:** Same fields as response above (excluding `id`, `created_at`).

---

### PUT `/api/streams/:id/destinations/:dest_id`

Update a simulcast destination including per-destination encode overrides.

**Body:**
```json
{
  "rtmp_url": "rtmp://...",
  "stream_key": "...",
  "platform": "youtube",
  "is_active": true,
  "bitrate": 4500,
  "resolution": "1920x1080",
  "fps": 60,
  "video_codec": "libx264",
  "preset": "veryfast"
}
```

---

### DELETE `/api/streams/:id/destinations/:dest_id`

Delete a simulcast destination.

---

## Scheduled Streams (v3.3.0 — Feature 5)

Scheduling is done via the `schedule_time` field on stream create/update. The scheduler goroutine (30s tick) auto-starts streams when `schedule_time <= NOW()`.

| Method | Path | Description |
|--------|------|-------------|
| PUT | `/api/streams/:id` | Set `schedule_time` (ISO 8601 datetime) |

No dedicated schedule endpoint — pass `schedule_time` in the stream update body.

---

## Playlists

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/playlists` | List playlists |
| POST | `/api/playlists` | Create playlist |
| GET | `/api/playlists/:id` | Get playlist with items |
| PUT | `/api/playlists/:id` | Update playlist |
| DELETE | `/api/playlists/:id` | Delete playlist |
| POST | `/api/playlists/:id/items` | Add item to playlist |
| DELETE | `/api/playlists/:id/items/:item_id` | Remove item |
| PUT | `/api/playlists/:id/items/reorder` | Reorder items |

---

## Media Folders

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/folders` | List folders |
| POST | `/api/folders` | Create folder |
| PUT | `/api/folders/:id` | Rename folder |
| DELETE | `/api/folders/:id` | Delete folder |

---

## Stream Analytics

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/streams/:id/analytics` | Get analytics for a stream |
| GET | `/api/analytics/summary` | Platform-wide summary |

---

## Stream Chat

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/streams/:id/chat` | Get recent chat messages |
| GET | `/api/streams/:id/chat-stream` | SSE: live chat aggregator |

---

## Stream History

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/streams/history` | List past stream records |
| GET | `/api/streams/history/:id` | Get single history record |

---

## RTMP

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/rtmp/auth` | Nginx-RTMP on_publish auth callback |
| GET | `/api/rtmp/stats` | Current RTMP publisher stats |

---

## Error Responses

All errors follow:

```json
{
  "error": "human-readable message",
  "code": "MACHINE_CODE"
}
```

| HTTP Status | Meaning |
|-------------|---------|
| 400 | Bad request / validation error |
| 401 | Unauthenticated |
| 403 | Forbidden |
| 404 | Resource not found |
| 409 | Conflict (e.g., stream already live) |
| 500 | Internal server error |
