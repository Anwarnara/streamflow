# Work Log

**Version:** 3.3.0
**Last Updated:** 2026-07-19

---

## 2026-07-19 — v3.3.0 Release

### Summary
Shipped five new features completing the "operational visibility and control" milestone. All features are live in production.

### Feature 1 — Stream Health Monitor
- Implemented `internal/streaming/health_monitor.go`
- Real-time FFmpeg stderr parser: extracts `fps`, `bitrate`, `speed`, `drop_frames`, `dup_frames`, `total_frames`, `out_time` using regex against FFmpeg's `stats` output line format
- In-memory `healthStore` (`sync.RWMutex`-protected map) — no DB queries on health reads
- Status classification logic: `HEALTHY` / `WARNING` / `DEGRADED` / `OFFLINE`
- `GET /api/streams/:id/health` — point-in-time snapshot
- `GET /api/streams/:id/health-stream` — SSE, 1-second push interval
- `GET /api/streams/health/all` — all active streams, zero DB queries
- Frontend: health stats bar added below each live stream card
- New page: `/stream-health` (`templates/health.html`) polling `health/all` every 3s
- Summary cards: Active Streams, Avg FPS, Total Drop Frames, Streams w/ Drops
- Per-stream table with color-coded status badges
- Navigation: added "Stream Health" to desktop sidebar and mobile bottom nav in `layout.html`

### Feature 2 — Auto-Reconnect Simulcast with Health Parsing
- `internal/streaming/simulcast.go` substantially refactored
- FFmpeg `stderr` pipe now feeds a goroutine scanner that simultaneously: (a) appends lines to `LogBuffer` FIFO (max 500 lines), (b) passes stats lines to health parser
- `allCopy` detection at stream start: checks if all `DestinationConfig` entries have zero/empty overrides
- If `allCopy` → routes to `runSimulcastTee` (FFmpeg `tee` muxer, one encode pipeline)
- Else → routes to `runSimulcastPerDest` (separate FFmpeg processes per destination)
- Auto-reconnect goroutine: watches for FFmpeg process exit, retries with exponential backoff, up to 5 attempts

### Feature 3 — FFmpeg Log Streaming to UI
- `GET /api/streams/:id/logs-stream` SSE endpoint, registered in `main.go`
- Delivers delta log lines from `LogBuffer` every 500ms (tracks last-sent index)
- Frontend: "Logs" button on live stream card triggers JavaScript modal
- Modal: `#1a1a1a` dark background, green monospace terminal font, auto-scroll to latest line, scroll-lock toggle on manual scroll

### Feature 5 — Scheduled Stream Auto-Start
- `internal/streaming/scheduler.go` — new file
- `StartScheduler()` function: launches goroutine with `time.NewTicker(30 * time.Second)`
- On each tick: `SELECT * FROM streams WHERE schedule_time <= NOW() AND status != 'streaming'`
- For each result: calls stream start logic, sets `schedule_time = NULL` in DB
- `StartScheduler()` called from `cmd/backend/main.go` at server boot
- Frontend: `datetime-local` input added to create/edit stream modal

### Feature 6 — Per-destination Bitrate/Resolution Override
- DB migration: `ALTER TABLE stream_destinations ADD COLUMN IF NOT EXISTS is_active BOOLEAN DEFAULT true, ADD COLUMN IF NOT EXISTS bitrate INT DEFAULT 0, ...` (all 5 new columns)
- `internal/streaming/destinations.go` updated: CRUD handlers read and write all new encode fields
- `simulcast.go` reads `DestinationConfig` struct for each destination, drives `allCopy` check and per-dest encoding params
- Frontend: collapsible "Encode Override" accordion section in each destination form row
- Routes added: `PUT /api/streams/:id/destinations/:dest_id`, `DELETE /api/streams/:id/destinations/:dest_id`

### Documentation Updated
- All 22 docs in `/var/www/streaming/docs/` updated to v3.3.0 / 2026-07-19

---

## 2026-05-10 — v3.2.0 Release

### Features Delivered
- RTMP Ingest via Nginx-RTMP on port `:1935`
- Simulcast via FFmpeg `tee` muxer
- Auto-failover with fallback video and `-loop -1`
- Unified SSE chat aggregator (YouTube + Twitch)
- Playlist streaming with `stream_playlist_items` tracking
- Watermark overlay (configurable position)
- Chunked video upload endpoint
- FFmpeg auto-thumbnail generation on upload
- `stream_analytics` table + analytics API
- Stream history page
- Gallery with `media_folders`
- PWA manifest + service worker

---

## 2026-02-20 — v3.1.0 Initial Release

### Foundation
- Go 1.25 + Echo v4 backend bootstrapped
- PostgreSQL schema: `users`, `videos`, `streams` core tables
- Basic CRUD for users, videos, streams
- Single-destination RTMP push via FFmpeg
- Session-based auth (`user_sessions`, 72h expiry)
- Basic dashboard UI with stream create/start/stop
- Video upload + FFmpeg metadata extraction
