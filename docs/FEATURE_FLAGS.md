# Feature Flags

**Version:** 3.3.0
**Last Updated:** 2026-07-19

## Overview

StreamFlow does not currently use a dynamic feature flag service. Feature gates are controlled via stream-level settings stored in the database, and compile-time or environment-level toggles where applicable.

---

## Stream-Level Feature Toggles

These are per-stream settings stored in the `streams` table that act as feature flags for individual broadcasts.

| Flag / Column | Type | Default | Description |
|---------------|------|---------|-------------|
| `use_advanced_settings` | bool | false | Unlocks advanced FFmpeg parameter inputs in the stream form |
| `loop_video` | bool | false | Loops the source video continuously until manually stopped |
| `youtube_monetization` | bool | false | Passes monetization metadata to YouTube Live API |

---

## Destination-Level Overrides (v3.3.0)

Per-destination encode overrides in `stream_destinations` effectively act as feature flags for simulcast transcoding. A value of `0` or empty string means the feature is disabled (inherit from parent).

| Column | Zero/Empty = disabled | Non-zero/non-empty = enabled |
|--------|----------------------|------------------------------|
| `bitrate` | Inherit stream bitrate | Override to specific kbps |
| `fps` | Inherit stream fps | Override to specific fps |
| `resolution` | Inherit stream resolution | Override to specific resolution |
| `video_codec` | Inherit (copy or stream codec) | Transcode with specific codec |
| `preset` | Inherit FFmpeg preset | Apply specific FFmpeg preset |

When any destination has at least one non-default override, the `allCopy` path is disabled and per-destination transcoding is used for the entire simulcast session.

---

## Feature Availability by Version

| Feature | Introduced | Status |
|---------|-----------|--------|
| RTMP Ingest | v3.2.0 | Stable |
| Simulcast tee muxer | v3.2.0 | Stable |
| Auto-failover | v3.2.0 | Stable |
| Chat aggregator (SSE) | v3.2.0 | Stable |
| Playlist streaming | v3.2.0 | Stable |
| Watermark overlay | v3.2.0 | Stable |
| Chunked upload | v3.2.0 | Stable |
| Stream analytics | v3.2.0 | Stable |
| PWA support | v3.2.0 | Stable |
| Stream Health Monitor | v3.3.0 | Stable |
| Auto-Reconnect Simulcast | v3.3.0 | Stable |
| FFmpeg Log Streaming | v3.3.0 | Stable |
| Scheduled Auto-Start | v3.3.0 | Stable |
| Per-destination Encode Override | v3.3.0 | Stable |

---

## Future / Planned Flags

The following are candidates for future feature flag implementation when a dynamic flag system is introduced:

- `enable_gpu_transcoding` — Route transcode to NVENC/VAAPI when available
- `enable_health_webhooks` — Push health alerts to external webhook URLs
- `enable_mobile_hls` — Expose HLS output alongside RTMP push
