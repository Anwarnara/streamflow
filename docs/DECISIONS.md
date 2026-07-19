# Architectural Decisions

**Version:** 3.3.0
**Last Updated:** 2026-07-19

This document logs significant architectural decisions made during the development of StreamFlow, following the ADR (Architecture Decision Record) format.

---

## ADR-001: FFmpeg as the Core Streaming Engine

**Date:** 2026-02-20
**Context:** We need a reliable way to encode, mux, and push video to RTMP destinations. Options included using GStreamer bindings, wrapping libavcodec directly in Cgo, or orchestrating the FFmpeg CLI.
**Decision:** We will use the FFmpeg CLI binary (`/usr/local/bin/ffmpeg`) and orchestrate it from Go via `os/exec`.
**Consequences:** 
- Faster development time and access to all FFmpeg features out-of-the-box.
- Requires parsing stderr for health and status metrics (implemented in v3.3.0).
- We must handle zombie processes and graceful shutdowns explicitly in Go.

---

## ADR-002: Server-Sent Events (SSE) vs WebSockets for Server Push

**Date:** 2026-05-10
**Context:** We need to push real-time data (chat messages, logs, health metrics) from the Go backend to the frontend UI.
**Decision:** Use Server-Sent Events (SSE) via the standard `text/event-stream` format instead of WebSockets.
**Consequences:**
- Unidirectional communication only (Server -> Client), which perfectly fits our use cases.
- Simpler implementation in both Go (standard HTTP handlers with flusher) and JS (`EventSource`).
- No upgrade handshake required; works seamlessly through Nginx and standard HTTP proxies.
- Handlers must explicitly set headers: `Content-Type: text/event-stream`, `Cache-Control: no-cache`, `Connection: keep-alive`.

---

## ADR-003: Simulcast Routing (allCopy Detection)

**Date:** 2026-07-19
**Context:** In v3.3.0 (Feature 6), we added per-destination encode overrides (bitrate, resolution, etc.). Previously, simulcast always used FFmpeg's `tee` muxer, which requires all outputs to use the exact same encoded stream (zero extra CPU cost).
**Decision:** Implement `allCopy` detection in `simulcast.go`. 
- If all destinations inherit stream settings (or explicitly request `copy`), we route to `runSimulcastTee` (1 decode, 1 encode, N outputs).
- If *any* destination has an override, we route to `runSimulcastPerDest`, which launches separate FFmpeg encoding pipelines per destination based on the `DestinationConfig` struct.
**Consequences:**
- Maintains the extreme efficiency of the `tee` muxer for standard use cases.
- Provides flexibility for advanced users who need a lower-bitrate stream for a specific platform.
- Backend must carefully parse the `DestinationConfig` to make the routing decision at stream start time.

---

## ADR-004: In-Memory `healthStore` for Stream Metrics

**Date:** 2026-07-19
**Context:** Feature 1 requires real-time health metrics parsed from FFmpeg. To populate the `/stream-health` dashboard, the UI needs to poll the health of *all* active streams frequently (every 3 seconds). Querying the database every 3 seconds for transient metric data is inefficient.
**Decision:** Store parsed health metrics in an in-memory `healthStore` (a Go map protected by a `sync.RWMutex`) located in `internal/streaming/health_monitor.go`.
**Consequences:**
- Zero database queries required to serve the `/api/streams/health/all` endpoint.
- Metrics are ephemeral; if the backend process restarts, the metrics are lost.
- This trade-off is acceptable because FFmpeg starts generating new stderr metrics within 1-2 seconds of being launched, rapidly repopulating the cache.

---

## ADR-005: Scheduled Execution via Goroutine Ticker

**Date:** 2026-07-19
**Context:** Feature 5 requires streams to start automatically at a scheduled time. We evaluated using a persistent queue (RabbitMQ/Redis), a cron daemon, or an internal Go scheduler.
**Decision:** Implement a simple internal scheduler (`scheduler.go`) using a Go ticker that fires every 30 seconds. It queries the DB for `streams WHERE schedule_time <= NOW() AND status != 'streaming'`, starts them, and clears the `schedule_time`.
**Consequences:**
- Avoids the operational complexity of adding an external dependency (Redis/Cron) to our single-binary deployment.
- Precision is limited to ~30 seconds, which is acceptable for live streaming start times.
- If the server is down exactly when a stream is scheduled, it will start on the next tick after boot.
