# Handoff Document

**Version:** 3.3.0
**Last Updated:** 2026-07-19

## Purpose

This document provides a complete state snapshot of the StreamFlow project for handoff between engineers, agents, or sessions. Anyone picking up this project should be able to reach operational status by reading this document and the linked references.

---

## Current State — v3.3.0 (2026-07-19)

### What Works
- Full stream lifecycle: create → configure → schedule or start → monitor → stop
- Simulcast to multiple RTMP destinations simultaneously
- Per-destination encode overrides (bitrate, resolution, fps, codec, preset)
- Real-time stream health monitoring via SSE and the `/stream-health` dashboard
- FFmpeg log streaming to the UI terminal modal
- Scheduled stream auto-start (30s precision)
- Auto-reconnect on simulcast destination drop
- Playlist streaming (ordered video sequences)
- Chat aggregation from YouTube and Twitch
- Stream analytics and history
- Video gallery with folder management
- Chunked file upload with FFmpeg metadata extraction and thumbnail generation
- Watermark overlay with configurable position
- RTMP ingest via Nginx-RTMP (`:1935`)
- PWA installable web app

### Known Limitations / Open Issues
- Health webhook alerts not yet implemented (see `IDEAS.md`, `ROADMAP.md`)
- No GPU transcoding support (all CPU via x264)
- Mobile native app does not exist; PWA is the mobile interface
- Chat aggregation only supports YouTube and Twitch; other platforms are manual RTMP only
- Scheduler precision is ~30 seconds (acceptable, by design per ADR-005)

---

## Development Environment

### Prerequisites
- Go 1.25
- PostgreSQL 14 (`streamflow` database)
- Nginx + Nginx-RTMP module
- FFmpeg custom build at `/usr/local/bin/ffmpeg`

### Build and Run
```bash
cd /var/www/streaming/streaming-go
go build -o bin/backend ./cmd/backend/
systemctl restart streamflow-backend.service
```

### Check Status
```bash
systemctl status streamflow-backend.service
journalctl -u streamflow-backend.service -f
```

### Access
- Web UI: `http://<host>:7575`
- API: `http://<host>:8100/api/`
- RTMP ingest: `rtmp://<host>:1935/live/<stream_key>`

---

## Key Files Quick Reference

| File | Purpose |
|------|---------|
| `cmd/backend/main.go` | Entry point, route registration, scheduler start |
| `internal/streaming/handlers.go` | HTTP handlers |
| `internal/streaming/simulcast.go` | Multi-destination FFmpeg, allCopy routing, auto-reconnect |
| `internal/streaming/health_monitor.go` | Stderr parser, healthStore, health SSE |
| `internal/streaming/scheduler.go` | Scheduled auto-start goroutine |
| `internal/streaming/destinations.go` | Destination CRUD with encode overrides |
| `internal/streaming/chat_aggregator.go` | YouTube + Twitch chat SSE |
| `streaming-go/templates/health.html` | /stream-health dashboard page |
| `streaming-go/templates/layout.html` | Base layout with nav (updated for health link) |

---

## Documentation Index

| Document | Purpose |
|----------|---------|
| `AGENT_RULES.md` | Rules for AI agents working on this codebase |
| `API_SPEC.md` | All API endpoints |
| `ARCHITECTURE_PRINCIPLES.md` | System design principles |
| `CHANGELOG.md` | Version history |
| `DATABASE.md` | Full schema for all 14 tables |
| `DECISIONS.md` | Architecture Decision Records (ADRs) |
| `DOMAIN_MODEL.md` | Entity relationships and domain concepts |
| `ERROR_STATES.md` | Error handling and common failure modes |
| `EVENT_CATALOG.md` | SSE events and lifecycle transitions |
| `FEATURE_FLAGS.md` | Feature toggles and availability matrix |
| `HANDOFF.md` | This document |
| `IDEAS.md` | Backlog of future ideas |
| `PERMISSIONS.md` | Roles and access control |
| `PRD.md` | Product Requirements Document |
| `PROJECT_INIT_CHECKLIST.md` | Bootstrap checklist |
| `PROJECT_VISION.md` | High-level vision and goals |
| `ROADMAP.md` | Feature roadmap with status |
| `SETTINGS_CATALOG.md` | All configurable settings |
| `THUMBNAIL_SYSTEM.md` | Thumbnail generation system |
| `UI_UX.md` | UI/UX patterns and conventions |
| `USER_FLOW.md` | Key user journey flows |
| `WORKLOG.md` | Chronological work log |
