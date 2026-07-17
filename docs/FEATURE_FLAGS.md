# Feature Flags & Toggles
**Version:** 3.0.3  
**Last Updated:** 2026-07-17  

## 1. Active Flags
- `ENABLE_CHUNKED_UPLOAD`: `true` (Menggunakan sistem chunked Go baru)
- `ENABLE_AUTO_THUMBNAIL`: `true` (Eksekusi FFmpeg pada upload)
- `ENABLE_YOUTUBE_OAUTH`: `false` (Dalam tahap pengembangan)
- `ENABLE_PWA_INSTALL`: `true` (Service worker aktif)

## 2. Environment Variables
Feature flags umumnya dikendalikan via `.env` file di production.

