# API Specification
**Version:** 3.0.3  
**Last Updated:** 2026-07-17  

## 1. Authentication
Semua endpoint (kecuali `/api/auth/login` dan `/api/auth/signup`) membutuhkan cookie `streamflow_session`.

## 2. Core Endpoints

### `POST /api/auth/login`
- **Body**: `{"username": "...", "password": "..."}`
- **Response**: `200 OK`, Sets HTTPOnly Cookie.

### `GET /api/videos`
- **Response**: `200 OK`, `[{"id": "...", "title": "...", "thumbnail_path": "..."}]`

### `PUT /api/videos/:id`
- **Body**: `{"title": "New Title"}` (Partial update)
- **Response**: `200 OK`, `{"message": "Video updated"}`

### `GET /api/system/stats`
- **Response**: `200 OK`, `{"cpu": 45.2, "memory_used": 1024, "disk_used": 50.5}`
- **Note**: 900ms micro-cache di backend untuk mencegah race condition flicker.

## 3. Chunked Upload Flow
- `POST /api/videos/chunk/init` -> `uploadId`
- `POST /api/videos/chunk/upload?uploadId=X&chunkIndex=N` (Binary body)
- `POST /api/videos/chunk/complete?uploadId=X`

