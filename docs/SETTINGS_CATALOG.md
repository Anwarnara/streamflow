# Settings Catalog

**Version:** 3.3.0
**Last Updated:** 2026-07-19

## Overview

This document catalogs all configurable settings in StreamFlow, covering stream-level parameters, destination overrides, system defaults, and environment-level configuration.

---

## Stream Settings

Stored in the `streams` table. Configurable per stream via the UI or `PUT /api/streams/:id`.

| Setting | Column | Default | Description |
|---------|--------|---------|-------------|
| Title | `title` | — | Display name for the stream |
| Source Video | `video_id` | null | Single video source (mutually exclusive with playlist) |
| Source Playlist | `playlist_id` | null | Ordered playlist source |
| Primary RTMP URL | `rtmp_url` | — | Main destination RTMP endpoint |
| Stream Key | `stream_key` | — | Primary stream key |
| Platform | `platform` | — | e.g. `youtube`, `twitch`, `custom` |
| Bitrate | `bitrate` | 2500 | Output bitrate in kbps |
| Resolution | `resolution` | — | e.g. `1920x1080`, `1280x720` |
| FPS | `fps` | 30 | Output frames per second |
| Orientation | `orientation` | `horizontal` | `horizontal` or `vertical` |
| Loop Video | `loop_video` | false | Loop source video indefinitely |
| Fallback Video | `fallback_video_id` | null | Loops when main video ends |
| Watermark | `watermark_path` | null | Path to overlay image |
| Watermark Position | `watermark_position` | `top-right` | `top-right`, `top-left`, `bottom-right`, `bottom-left` |
| Schedule Time | `schedule_time` | null | ISO 8601 datetime for auto-start; cleared after execution |
| Max Duration | `duration` | 0 | Max stream duration in seconds; `0` = unlimited |
| Advanced Settings | `use_advanced_settings` | false | Unlock advanced FFmpeg parameter fields in UI |
| YouTube Monetization | `youtube_monetization` | false | Pass monetization flag to YouTube Live API |

---

## Destination Override Settings (v3.3.0)

Stored in `stream_destinations`. Per-destination encode overrides added in v3.3.0.

| Setting | Column | Default | Behavior when default |
|---------|--------|---------|----------------------|
| Active | `is_active` | true | Include in simulcast |
| Bitrate Override | `bitrate` | 0 | Inherit from stream |
| Resolution Override | `resolution` | `''` | Inherit from stream |
| FPS Override | `fps` | 0 | Inherit from stream |
| Video Codec | `video_codec` | `''` | Copy or inherit stream codec |
| Preset | `preset` | `''` | Inherit FFmpeg preset |

When any destination has a non-default override, the `allCopy` fast path is disabled and per-destination transcoding is used.

---

## User Settings

Stored in the `users` table.

| Setting | Column | Default | Description |
|---------|--------|---------|-------------|
| Username | `username` | — | Login credential |
| Avatar | `avatar_path` | null | Relative path to avatar image |
| Google Drive Key | `gdrive_api_key` | null | API key for Google Drive integration |
| Role | `user_role` | `admin` | User permission role |
| Status | `status` | `active` | `active` or `suspended` |
| YouTube OAuth | `youtube_*` | null | OAuth tokens and channel info |

---

## Server / Environment Settings

Passed as environment variables or compiled defaults in `main.go`.

| Variable | Default | Description |
|----------|---------|-------------|
| `DB_DSN` | — | PostgreSQL connection string |
| `PORT` | `8100` | Backend HTTP listen port |
| `TEMPLATE_DIR` | `./templates` | Path to HTML templates directory |
| `FFMPEG_PATH` | `/usr/local/bin/ffmpeg` | FFmpeg binary path |
| `UPLOAD_DIR` | — | Directory for video file uploads |

---

## Health Thresholds (v3.3.0)

Used by `health_monitor.go` to classify stream status. Currently compile-time constants.

| Status | Condition |
|--------|-----------|
| `HEALTHY` | speed ≥ 0.95 AND drop_frames < 1% of total_frames |
| `WARNING` | speed 0.80–0.95 OR drop_frames 1–5% of total_frames |
| `DEGRADED` | speed < 0.80 OR drop_frames > 5% of total_frames OR bitrate = 0 |
| `OFFLINE` | No active FFmpeg process |

---

## Timing Constants (compile-time)

| Constant | Value | Location |
|----------|-------|----------|
| Scheduler tick | 30 seconds | `scheduler.go` |
| Max reconnect retries | 5 | `simulcast.go` |
| LogBuffer max lines | 500 | `health_monitor.go` |
| Health SSE push interval | 1 second | `health_monitor.go` |
| Log SSE delta interval | 500ms | `handlers.go` |
| Health page poll interval | 3 seconds | `templates/health.html` |
| Session expiry | 72 hours | `user_sessions` table default |
