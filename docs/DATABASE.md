# Database Architecture
**Version:** 3.0.3  
**Last Updated:** 2026-07-17  

## 1. Overview
- **Engine**: PostgreSQL 14.23
- **Port**: 5432
- **Connection Pool**: Max 25 connections

## 2. Skema Tabel Utama

### `users`
- `id` (UUID, PK)
- `username` (VARCHAR, Unique)
- `password_hash` (VARCHAR)
- `role` (VARCHAR: 'admin', 'member')

### `videos`
- `id` (UUID, PK)
- `title` (VARCHAR)
- `filepath` (VARCHAR)
- `thumbnail_path` (VARCHAR, Nullable)
- `user_id` (UUID, FK -> users.id)
- `folder_id` (UUID, FK -> folders.id)

### `streams`
- `id` (UUID, PK)
- `name` (VARCHAR)
- `platform` (VARCHAR)
- `stream_key` (VARCHAR)
- `user_id` (UUID, FK -> users.id)

## 3. Migrations & Backup
- Database berjalan di bare-metal VPS (bukan dockerized untuk fase ini).
- Backup: Cron job harian pg_dump.

