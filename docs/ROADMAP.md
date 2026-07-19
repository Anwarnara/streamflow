# Roadmap

**Version:** 3.3.0
**Last Updated:** 2026-07-19

## Status Key

| Symbol | Meaning |
|--------|---------|
| ✅ DONE | Implemented and shipped |
| 🔄 IN PROGRESS | Currently being built |
| 📋 PLANNED | Scheduled for a future milestone |
| 💡 IDEA | Exploratory, not yet committed |

---

## v3.1.0 — Foundation ✅ DONE

- ✅ Go + Echo v4 backend
- ✅ PostgreSQL schema (users, videos, streams)
- ✅ Basic stream create/start/stop via FFmpeg
- ✅ Single-destination RTMP push
- ✅ User authentication (session tokens, 72h expiry)
- ✅ Video upload and metadata extraction

---

## v3.2.0 — Simulcast & Production Features ✅ DONE

- ✅ RTMP Ingest via Nginx-RTMP (:1935)
- ✅ Simulcast tee muxer (multi-destination)
- ✅ Auto-failover (fallback video with loop)
- ✅ Unified SSE chat aggregator (YouTube + Twitch)
- ✅ Playlist streaming
- ✅ Watermark overlay with configurable position
- ✅ Chunked video upload
- ✅ FFmpeg auto-thumbnail generation
- ✅ Stream analytics
- ✅ Stream history page
- ✅ Gallery with folder management
- ✅ PWA support

---

## v3.3.0 — Operational Visibility & Control ✅ DONE (2026-07-19)

- ✅ **Feature 1 — Stream Health Monitor**
  - Real-time FFmpeg stderr parsing (fps, bitrate, speed, drop_frames, dup_frames, out_time)
  - `GET /api/streams/:id/health` and `/health-stream` (SSE 1s)
  - `GET /api/streams/health/all` — zero DB queries, in-memory healthStore
  - `/stream-health` dashboard page, health bar on stream cards

- ✅ **Feature 2 — Auto-Reconnect Simulcast with Health Parsing**
  - Stderr piped to goroutine: feeds health parsing + LogBuffer (max 500 lines FIFO)
  - allCopy detection → tee muxer (zero CPU) or per-destination transcode
  - Exponential backoff auto-reconnect on FFmpeg exit

- ✅ **Feature 3 — FFmpeg Log Streaming to UI**
  - `GET /api/streams/:id/logs-stream` (SSE 500ms delta)
  - Terminal-style modal in dashboard (dark bg, green mono, auto-scroll)

- ✅ **Feature 5 — Scheduled Stream Auto-Start**
  - `scheduler.go` goroutine ticker every 30s
  - Auto-starts eligible streams, clears `schedule_time`
  - `datetime-local` picker in create/edit stream modal

- ✅ **Feature 6 — Per-destination Bitrate/Resolution Override**
  - DB: `stream_destinations` extended with encode override columns
  - `destinations.go` CRUD with encode params
  - `simulcast.go` routes via `DestinationConfig` (tee vs per-dest)
  - Collapsible "Encode Override" UI per destination

---

## v3.4.0 — Alerting & Observability 📋 PLANNED

- 📋 Health alerting webhooks (see `IDEAS.md`)
  - Trigger on `WARNING` / `DEGRADED` transitions
  - Per-stream configurable webhook URL
  - Cooldown period to prevent alert flooding
- 📋 Stream preview thumbnail (live frame capture every 30s)
- 📋 Analytics time-series charts (Chart.js / uPlot)

---

## v3.5.0 — Performance & Scale 📋 PLANNED

- 📋 GPU transcoding support (NVENC, VAAPI, QSV)
  - Auto-detect available hardware encoders at startup
  - `use_gpu` flag per DestinationConfig
- 📋 HLS output alongside RTMP push
  - In-browser live preview without external player
- 📋 RTMP pull / re-stream ingest (pull existing stream as source)

---

## Future / Exploratory 💡

- 💡 Mobile native app (iOS / Android)
- 💡 Multi-tenant / organization support
- 💡 Stream clipping (export segment as MP4)
- 💡 Live transcription / captions (Whisper integration)
- 💡 Public stream embed page
- 💡 Docker / docker-compose packaging
