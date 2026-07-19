# Changelog

**Version:** 3.3.0
**Last Updated:** 2026-07-19

All notable changes to StreamFlow are documented here. Follows [Keep a Changelog](https://keepachangelog.com/) conventions.

---

## [3.3.0] — 2026-07-19

### Added

#### Feature 1 — Stream Health Monitor
- Real-time FFmpeg stderr parser in `internal/streaming/health_monitor.go`
- Parses: `fps`, `bitrate`, `speed`, `drop_frames`, `dup_frames`, `total_frames`, `out_time`
- `GET /api/streams/:id/health` — point-in-time health snapshot
- `GET /api/streams/:id/health-stream` — SSE endpoint pushing updates every 1 second
- `GET /api/streams/health/all` — all live streams health, zero DB queries (in-memory `healthStore`)
- Health stats bar rendered below each live stream card on the dashboard
- New `/stream-health` page (`templates/health.html`) polling `health/all` every 3 seconds
- Summary cards: Active Streams, Avg FPS, Total Drop Frames, Streams w/ Drops
- Per-stream table with status badge: `HEALTHY` / `WARNING` / `DEGRADED`
- Menu item added to desktop sidebar and mobile bottom nav in `layout.html`

#### Feature 2 — Auto-Reconnect Simulcast with Health Parsing
- `internal/streaming/simulcast.go` refactored
- FFmpeg stderr piped to goroutine scanner — feeds both health parsing and `LogBuffer` (FIFO, max 500 lines)
- `allCopy` detection: if all destinations are copy-compatible → FFmpeg `tee` muxer (zero CPU transcode)
- Otherwise → per-destination FFmpeg processes using `DestinationConfig` struct
- Auto-reconnect logic on FFmpeg process exit with exponential backoff

#### Feature 3 — FFmpeg Log Streaming to UI
- `GET /api/streams/:id/logs-stream` — SSE endpoint, delivers delta log lines every 500ms
- Frontend: "Logs" button on each live stream card opens a terminal-style modal
- Modal: dark background, green monospace text, auto-scroll to bottom, manual scroll lock toggle

#### Feature 5 — Scheduled Stream Auto-Start
- `internal/streaming/scheduler.go` — goroutine ticker every 30 seconds
- Queries `streams WHERE schedule_time <= NOW() AND status != 'streaming'`
- Auto-starts FFmpeg for eligible streams, clears `schedule_time` after execution
- `StartScheduler()` called from `main()` at server boot
- Frontend: `datetime-local` picker in the create/edit stream modal

#### Feature 6 — Per-destination Bitrate/Resolution Override
- DB migration: `ALTER TABLE stream_destinations ADD COLUMN` for `is_active`, `bitrate`, `resolution`, `fps`, `video_codec`, `preset`
- `internal/streaming/destinations.go` — full CRUD for destinations with encode params
- `simulcast.go` updated to use `DestinationConfig`, routing to `runSimulcastTee` or `runSimulcastPerDest`
- Frontend: collapsible "Encode Override" section in each simulcast destination form
- New routes: `PUT /api/streams/:id/destinations/:dest_id`, `DELETE /api/streams/:id/destinations/:dest_id`

### Changed
- `simulcast.go` substantially refactored for health integration and per-destination routing
- `layout.html` updated with `/stream-health` navigation entry

### Notes
- Feature 4 was intentionally descoped; feature numbering is not sequential by design.

---

## [3.2.0] — 2026-05-10

### Added
- RTMP ingest via Nginx-RTMP on port `:1935`
- Simulcast via FFmpeg `tee` muxer (multiple RTMP destinations from one encode)
- Auto-failover: fallback video loops with `-loop -1` on main video end
- Unified SSE chat aggregator pulling from YouTube and Twitch APIs
- Playlist streaming — ordered video playlists attached to streams
- Watermark overlay with configurable position (`top-right`, `top-left`, `bottom-right`, `bottom-left`)
- Chunked video upload endpoint for large files
- FFmpeg auto-thumbnail generation on video upload
- Stream analytics persisted to `stream_analytics` table
- Stream history page listing all past sessions
- Gallery with folder management (`media_folders`)
- PWA manifest and service worker for installable web app

---

## [3.1.0] — 2026-02-20

### Added
- Initial Go + Echo v4 backend
- PostgreSQL schema with `users`, `videos`, `streams` core tables
- Basic stream create/start/stop via FFmpeg
- Single-destination RTMP push
- User authentication with session tokens (72h expiry)
- Video upload and metadata extraction
- Basic stream dashboard UI
