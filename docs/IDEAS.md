# Ideas & Backlog

**Version:** 3.3.0
**Last Updated:** 2026-07-19

## Overview

This document tracks future ideas, potential features, and exploratory concepts that have not yet been formally scheduled. Items here are not committed to any roadmap milestone.

---

## High Priority Ideas

### Health Alerting Webhooks
When a stream's health status transitions to `WARNING` or `DEGRADED`, fire an HTTP POST to a user-configured webhook URL. Payload would include stream ID, current metrics, and status. This enables integration with PagerDuty, Slack, Discord, or custom monitoring systems without polling the API.

- Implementation path: extend `health_monitor.go` to check status transitions and trigger webhook dispatch in a goroutine.
- Per-stream webhook URL stored in `streams` table (new column: `health_webhook_url`).
- Cooldown period (e.g., max 1 alert per 5 minutes per stream) to avoid flooding.

### GPU Transcoding Support
Currently all transcoding runs on CPU via `libx264`. On hosts with NVIDIA GPUs, `h264_nvenc` would dramatically reduce CPU load and allow more simultaneous streams. On Intel/AMD hosts, `h264_vaapi` or `h264_qsv` are options.

- Implementation path: detect available hardware encoders at startup (run `ffmpeg -encoders | grep nvenc`), store result in a server capability struct.
- `DestinationConfig` gains a `use_gpu` flag that swaps the codec and adds the appropriate device flags.
- UI: show "GPU Available" badge in settings if detected.

### Mobile Native App
The current PWA covers basic mobile use, but a native iOS/Android app would enable push notifications for stream health alerts, background monitoring, and camera-based mobile streaming input.

- Short-term: React Native or Flutter wrapper around the existing web UI + API.
- Long-term: native camera capture → RTMP push from mobile device into the Nginx-RTMP ingest.

---

## Medium Priority Ideas

### HLS Output alongside RTMP
In addition to pushing to RTMP destinations, generate an HLS playlist (`.m3u8` + `.ts` segments) locally. This would enable an embedded player on the StreamFlow web UI to monitor the stream output directly, without needing a separate playback service.

- FFmpeg `-hls_time 2 -hls_list_size 5` output to a local directory.
- Nginx serves the HLS files statically.

### Stream Preview Thumbnail (Live)
Periodically capture a JPEG frame from the active FFmpeg output (every 30 seconds) and expose it via `GET /api/streams/:id/preview`. Display in the dashboard stream card.

- Use FFmpeg `-vframes 1 -f image2` pipe output to a temp file.

### Multi-tenant / Organization Support
Currently `user_role` defaults to `admin` for all users, reflecting a single-tenant deployment. A multi-tenant mode would add an `organizations` table, ownership scoping, and viewer/operator/admin role distinctions per org.

### Stream Clipping
Allow users to mark a start and end time during a live session and export that segment as an MP4 using `ffmpeg -ss -to -c copy`.

---

## Low Priority / Exploratory Ideas

### Transcription / Captions
Run Whisper (OpenAI or local) on the audio stream to generate live captions. Expose via SSE for overlay in the web player.

### Analytics Dashboard with Charts
Upgrade the current `stream_analytics` table display into time-series charts (viewer count, bitrate over time) using a lightweight charting library (Chart.js or uPlot).

### RTMP Pull (Re-stream Ingest)
Instead of local file input, pull an existing RTMP/HLS stream as the FFmpeg source and re-stream it to destinations. Useful for relay/re-broadcasting.

### Kubernetes / Docker Deployment
Package the binary, Nginx, and PostgreSQL into a `docker-compose.yml` for reproducible deployments. Current assumption is bare-metal/VM.

### Public Stream Page
A public-facing, embeddable page per stream with a live player, chat widget, and viewer count — shareable without requiring login.
