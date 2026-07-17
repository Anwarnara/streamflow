# Database Architecture
**Version:** 3.1.0  
**Last Updated:** 2026-07-17  

## 1. Overview
- **Engine**: PostgreSQL 14.23
- **Port**: 5432
- **Connection Pool**: Max 25 connections

## 2. Skema Tabel Utama

### `streams`
- `id` (UUID, PK)
- `stream_key` (VARCHAR, Unique)
- `fallback_video_id` (UUID, Nullable, FK -> videos.id)
- `watermark_path` (VARCHAR, Nullable)
- `is_live` (BOOLEAN)

### `stream_destinations` (Baru v3.1)
- `id` (UUID, PK)
- `stream_id` (UUID, FK -> streams.id)
- `platform` (VARCHAR)
- `rtmp_url` (VARCHAR)
- `stream_key` (VARCHAR)

### `stream_chats` (Baru v3.1)
- `id` (UUID, PK)
- `stream_id` (UUID, FK)
- `author_name` (VARCHAR)
- `message` (TEXT)
- `timestamp` (TIMESTAMP)

## 3. Migrations & Backup
- Backup: Cron job harian pg_dump.

