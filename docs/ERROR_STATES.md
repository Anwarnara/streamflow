# Error States & Handling

**Version:** 3.3.0
**Last Updated:** 2026-07-19

## Overview

This document outlines common error states in StreamFlow, how they are detected, and how the system recovers or reports them.

---

## Stream Execution Errors

### 1. FFmpeg Fails to Start
- **Symptom:** Stream status rapidly flips from `streaming` back to `error` or `offline`.
- **Detection:** `os/exec.Cmd.Start()` returns an error, or the process exits immediately with a non-zero code.
- **Handling:** Error is logged. Status updated in DB. If this was a scheduled start, the `schedule_time` is cleared so it doesn't infinite loop.
- **Troubleshooting:** Check the stream's log buffer via `/api/streams/:id/logs-stream` or the server logs. Often caused by an invalid video file, unreachable RTMP destination, or invalid custom FFmpeg arguments.

### 2. RTMP Destination Connection Drop (Simulcast)
- **Symptom:** One destination drops, but others might stay up.
- **Detection:** FFmpeg stderr parser catches connection reset/timeout logs. In `tee` muxer mode, FFmpeg may exit entirely depending on the `drop_pkts_on_overflow` flag.
- **Handling:** Feature 2 (v3.3.0) auto-reconnect logic kicks in. The `simulcast.go` watcher detects the premature exit and attempts to restart the process with exponential backoff (up to 5 retries).

### 3. Stream Health Degradation
- **Symptom:** UI shows a yellow `WARNING` or red `DEGRADED` badge.
- **Detection:** Feature 1 (v3.3.0) parses real-time metrics.
  - `WARNING`: High CPU usage indicated by `speed < 0.95`, or occasional dropped frames.
  - `DEGRADED`: Severe dropped frames (>5% of total), `speed < 0.8`, or zero bitrate output.
- **Handling:** Currently informational. Alerts the user via the UI (`/stream-health`). Future roadmap includes webhook alerting.

---

## API Error Responses

All API errors return a standard JSON structure:
```json
{
  "error": "Human readable message describing what went wrong",
  "code": "ERR_SPECIFIC_CODE"
}
```

### Common Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `ERR_UNAUTHORIZED` | 401 | Missing or invalid session token. |
| `ERR_NOT_FOUND` | 404 | The requested resource (stream, video, user) does not exist. |
| `ERR_VALIDATION` | 400 | Invalid input (e.g., negative bitrate, missing RTMP URL). |
| `ERR_STREAM_RUNNING`| 409 | Attempted to start a stream that is already active. |
| `ERR_FILE_TOO_LARGE`| 413 | Upload exceeds the chunk or total size limit. |
| `ERR_FFMPEG_CRASH` | 500 | FFmpeg process crashed unexpectedly. |

---

## Database Errors

- **Constraint Violations:** Handled gracefully by the backend. E.g., attempting to delete a video that is currently attached to a stream will return a 409 Conflict.
- **Connection Loss:** The `pgx` driver handles connection pooling and automatic reconnections to PostgreSQL.

---

## Log Analysis

When investigating an error, always check:
1. Stream-specific logs: `GET /api/streams/:id/logs-stream` (UI terminal modal)
2. Backend service logs: `journalctl -u streamflow-backend.service -n 100 --no-pager`
3. Nginx error logs (if API is unreachable): `/var/log/nginx/error.log`
