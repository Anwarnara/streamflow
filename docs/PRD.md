# Product Requirements Document (PRD)

**Version:** 3.3.0
**Last Updated:** 2026-07-19

## Product Overview

**StreamFlow** is an enterprise live streaming platform that enables operators to configure, schedule, and broadcast video content to multiple streaming platforms simultaneously. It is built for reliability, operational visibility, and deployment simplicity.

---

## Target Users

- **Broadcast Operators:** Configure streams, manage destinations, monitor health.
- **Content Managers:** Upload and organize video assets and playlists.
- **Administrators:** Manage users and platform-level settings.

---

## Goals

1. Provide a single control plane for multi-destination live streaming.
2. Minimize CPU usage through intelligent routing (tee muxer for copy-compatible streams).
3. Give operators real-time visibility into stream health without leaving the dashboard.
4. Support scheduled, hands-free stream starts.
5. Remain deployable as a single binary on a standard Linux VM.

---

## Non-Goals

- This is not a consumer-facing streaming platform (no public viewer pages in v3.x).
- This is not a video hosting or VOD platform.
- This does not manage CDN or delivery infrastructure.

---

## Feature Requirements

### FR-001: Multi-Destination Simulcast
A stream must support broadcasting to multiple RTMP destinations simultaneously. The system must use the most CPU-efficient FFmpeg muxing strategy available (tee when all outputs are copy-compatible; per-process transcode otherwise).

### FR-002: Stream Health Monitoring *(v3.3.0)*
The dashboard must display real-time health metrics (fps, bitrate, speed, drop frames) for each active stream. A dedicated `/stream-health` page must show a health summary across all active streams. Health status must be categorized as `HEALTHY`, `WARNING`, or `DEGRADED`.

### FR-003: FFmpeg Log Access *(v3.3.0)*
Operators must be able to view the raw FFmpeg output for any active stream from the dashboard UI without requiring SSH access. Logs must stream in real-time via SSE.

### FR-004: Scheduled Streaming *(v3.3.0)*
Operators must be able to set a future start time for a stream. The system must automatically start the stream at that time (within 30-second precision) without operator intervention.

### FR-005: Per-destination Encode Overrides *(v3.3.0)*
Each simulcast destination must support independent bitrate, resolution, fps, video codec, and preset overrides. When no overrides are set, the destination inherits the parent stream's settings.

### FR-006: Auto-Reconnect *(v3.3.0)*
If an RTMP destination connection drops during a simulcast, the system must attempt to reconnect automatically with exponential backoff.

### FR-007: Playlist Streaming
A stream must be able to play through an ordered playlist of videos sequentially without operator intervention between items.

### FR-008: Video Asset Management
The system must support chunked upload of large video files, automatic metadata extraction, thumbnail generation, and folder-based organization.

### FR-009: Chat Aggregation
The system must aggregate live chat messages from YouTube and Twitch into a unified SSE stream per active stream.

### FR-010: Session-Based Authentication
All API and UI access must require authentication via session tokens. Sessions expire after 72 hours.

---

## Quality Requirements

| Attribute | Requirement |
|-----------|-------------|
| Availability | Backend process managed by systemd with auto-restart on crash |
| Performance | `tee` muxer path must add <5% CPU overhead versus single-destination |
| Latency | Health metrics must update within 2 seconds of FFmpeg producing them |
| Reliability | Auto-reconnect must retry up to 5 times with exponential backoff |
| Security | All passwords bcrypt-hashed; stream keys not logged at INFO level |

---

## Out of Scope for v3.3.0

- Health alerting webhooks (planned, see `ROADMAP.md`)
- GPU transcoding
- Mobile native app
- Public viewer pages
- HLS output
