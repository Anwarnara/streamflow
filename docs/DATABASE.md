# Database Reference

**Version:** 3.3.0
**Last Updated:** 2026-07-19

## Overview

StreamFlow uses PostgreSQL 14. Database name: `streamflow`. All primary keys are UUIDs. All timestamps are `TIMESTAMPTZ`. Migrations are idempotent.

---

## Tables (14)

### 1. `users`

Core user accounts. Default role is `admin` for all users in the current single-tenant deployment.

| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID | PK |
| `username` | TEXT | Unique |
| `password` | TEXT | bcrypt hash |
| `avatar_path` | TEXT | Relative path to avatar file |
| `gdrive_api_key` | TEXT | Google Drive integration key |
| `user_role` | TEXT | Default: `admin` |
| `status` | TEXT | Default: `active` |
| `youtube_access_token` | TEXT | OAuth access token |
| `youtube_refresh_token` | TEXT | OAuth refresh token |
| `youtube_token_expiry` | TIMESTAMPTZ | Token expiry |
| `youtube_channel_id` | TEXT | Channel ID |
| `created_at` | TIMESTAMPTZ | Default: `NOW()` |
| `updated_at` | TIMESTAMPTZ | Updated on change |

---

### 2. `videos`

Uploaded video files with extracted metadata.

| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID | PK |
| `title` | TEXT | Display title |
| `filepath` | TEXT | Absolute path to file |
| `thumbnail_path` | TEXT | Path to auto-generated thumbnail |
| `file_size` | BIGINT | Bytes |
| `duration` | FLOAT | Seconds |
| `format` | TEXT | e.g. `mp4`, `mkv` |
| `resolution` | TEXT | e.g. `1920x1080` |
| `bitrate` | INT | kbps |
| `fps` | FLOAT | Frames per second |
| `user_id` | UUID | FK → `users(id)` |
| `folder_id` | UUID | FK → `media_folders(id)`, nullable |
| `upload_date` | TIMESTAMPTZ | Upload timestamp |
| `created_at` | TIMESTAMPTZ | Default: `NOW()` |
| `updated_at` | TIMESTAMPTZ | Updated on change |

---

### 3. `streams`

Stream configurations. A stream links a video (or playlist) to one or more RTMP destinations.

| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID | PK |
| `title` | TEXT | Display title |
| `video_id` | UUID | FK → `videos(id)`, nullable |
| `rtmp_url` | TEXT | Primary RTMP URL |
| `stream_key` | TEXT | Primary stream key |
| `platform` | TEXT | e.g. `youtube`, `twitch`, `custom` |
| `bitrate` | INT | Default: `2500` kbps |
| `resolution` | TEXT | e.g. `1920x1080` |
| `fps` | INT | Default: `30` |
| `orientation` | TEXT | Default: `horizontal` |
| `loop_video` | BOOL | Loop source video |
| `schedule_time` | TIMESTAMPTZ | Scheduled auto-start time; cleared after execution |
| `duration` | INT | Max duration seconds, 0 = unlimited |
| `status` | TEXT | Default: `offline`; values: `offline`, `streaming`, `scheduled`, `error` |
| `start_time` | TIMESTAMPTZ | When stream started |
| `end_time` | TIMESTAMPTZ | When stream ended |
| `use_advanced_settings` | BOOL | Enable advanced FFmpeg settings |
| `youtube_monetization` | BOOL | Enable YT monetization flag |
| `user_id` | UUID | FK → `users(id)` |
| `is_live` | BOOL | Currently live flag |
| `fallback_video_id` | UUID | FK → `videos(id)`, nullable — loops if main ends |
| `watermark_path` | TEXT | Path to watermark image |
| `watermark_position` | TEXT | Default: `top-right` |
| `playlist_id` | UUID | FK → `playlists(id)`, nullable |
| `created_at` | TIMESTAMPTZ | Default: `NOW()` |
| `updated_at` | TIMESTAMPTZ | Updated on change |

---

### 4. `stream_destinations`

Simulcast destinations for a stream. Each destination can override encode parameters. Added columns in v3.3.0: `is_active`, `bitrate`, `resolution`, `fps`, `video_codec`, `preset`.

| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID | PK |
| `stream_id` | UUID | FK → `streams(id)` ON DELETE CASCADE |
| `rtmp_url` | TEXT | Destination RTMP URL |
| `stream_key` | TEXT | Destination stream key |
| `platform` | TEXT | Platform name |
| `platform_icon` | TEXT | Icon filename |
| `is_active` | BOOL | Default: `true` — include in simulcast |
| `bitrate` | INT | Default: `0` — `0` means inherit from stream |
| `resolution` | TEXT | Default: `''` — empty means inherit |
| `fps` | INT | Default: `0` — `0` means inherit |
| `video_codec` | TEXT | Default: `''` — empty means inherit |
| `preset` | TEXT | Default: `''` — FFmpeg preset, empty means inherit |
| `created_at` | TIMESTAMPTZ | Default: `NOW()` |

---

### 5. `media_folders`

User-organized folders for video library management.

| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID | PK |
| `name` | TEXT | Folder name |
| `user_id` | UUID | FK → `users(id)` |
| `created_at` | TIMESTAMPTZ | Default: `NOW()` |
| `updated_at` | TIMESTAMPTZ | Updated on change |

---

### 6. `playlists`

Named, ordered collections of videos for use in streams.

| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID | PK |
| `name` | TEXT | Playlist name |
| `user_id` | UUID | FK → `users(id)` |
| `created_at` | TIMESTAMPTZ | Default: `NOW()` |
| `updated_at` | TIMESTAMPTZ | Updated on change |

---

### 7. `playlist_items`

Ordered items within a playlist.

| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID | PK |
| `playlist_id` | UUID | FK → `playlists(id)` ON DELETE CASCADE |
| `video_id` | UUID | FK → `videos(id)` |
| `position` | INT | Sort order |
| `created_at` | TIMESTAMPTZ | Default: `NOW()` |

---

### 8. `playlist_videos`

Junction table linking playlists to videos (many-to-many).

| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID | PK |
| `playlist_id` | UUID | FK → `playlists(id)` |
| `video_id` | UUID | FK → `videos(id)` |
| `created_at` | TIMESTAMPTZ | Default: `NOW()` |

---

### 9. `playlist_audios`

Audio tracks associated with playlists (background audio support).

| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID | PK |
| `playlist_id` | UUID | FK → `playlists(id)` |
| `audio_path` | TEXT | Path to audio file |
| `title` | TEXT | Display title |
| `created_at` | TIMESTAMPTZ | Default: `NOW()` |

---

### 10. `stream_analytics`

Per-stream performance metrics collected during streaming sessions.

| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID | PK |
| `stream_id` | UUID | FK → `streams(id)` |
| `viewer_count` | INT | Concurrent viewers |
| `bytes_sent` | BIGINT | Total bytes pushed |
| `duration` | INT | Session duration seconds |
| `recorded_at` | TIMESTAMPTZ | Metric timestamp |

---

### 11. `stream_chats`

Chat messages aggregated from platform chat APIs (YouTube, Twitch, etc.).

| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID | PK |
| `stream_id` | UUID | FK → `streams(id)` |
| `platform` | TEXT | Source platform |
| `author` | TEXT | Display name |
| `message` | TEXT | Chat message text |
| `received_at` | TIMESTAMPTZ | When message was received |

---

### 12. `stream_history`

Historical record of completed streaming sessions.

| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID | PK |
| `stream_id` | UUID | FK → `streams(id)` |
| `title` | TEXT | Stream title at time of session |
| `platform` | TEXT | Target platform |
| `start_time` | TIMESTAMPTZ | Session start |
| `end_time` | TIMESTAMPTZ | Session end |
| `duration` | INT | Duration in seconds |
| `created_at` | TIMESTAMPTZ | Default: `NOW()` |

---

### 13. `stream_playlist_items`

Tracks playback position within a playlist during an active stream.

| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID | PK |
| `stream_id` | UUID | FK → `streams(id)` |
| `playlist_item_id` | UUID | FK → `playlist_items(id)` |
| `played_at` | TIMESTAMPTZ | When this item started playing |

---

### 14. `user_sessions`

Session tokens for authenticated users. Sessions expire after 72 hours.

| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID | PK |
| `user_id` | UUID | FK → `users(id)` ON DELETE CASCADE |
| `token` | TEXT | Unique session token |
| `expires_at` | TIMESTAMPTZ | `NOW() + interval '72 hours'` |
| `created_at` | TIMESTAMPTZ | Default: `NOW()` |

---

## Migration Notes

- All `ALTER TABLE` migrations use `ADD COLUMN IF NOT EXISTS` for idempotency.
- v3.3.0 migration: `stream_destinations` gained `is_active`, `bitrate`, `resolution`, `fps`, `video_codec`, `preset`.
- Never use `DROP COLUMN` or `DROP TABLE` without explicit approval and a prior backup.
