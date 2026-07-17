# API Specification
**Version:** 3.1.0  
**Last Updated:** 2026-07-17  

## 1. Live Streaming API (Baru v3.1.0)

### `POST /api/rtmp/auth`
- Endpoint internal yang dipanggil oleh Nginx-RTMP.
- **Request**: Form data `name={streamKey}`
- **Response**: `200 OK` (Izinkan stream) atau `403 Forbidden`.

### `GET /api/streams/:id/chat`
- Endpoint SSE (Server-Sent Events) untuk real-time chat aggregator.
- **Response**: Streaming payload text/event-stream.

### `PUT /api/streams/:id/config`
- **Body**: `{"fallback_video_id": "...", "watermark_path": "..."}`
- **Response**: `200 OK`

### `POST /api/streams/:id/destinations`
- **Body**: `{"platform": "youtube", "rtmp_url": "...", "stream_key": "..."}`

## 2. Core API

### `GET /api/system/stats`
- Data hardware monitoring. Menggunakan 900ms micro-cache di backend.

### `PUT /api/videos/:id`
- Endpoint update/rename video.

