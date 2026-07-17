# Project Handoff Guide
**Version:** 3.1.0  
**Last Updated:** 2026-07-17  

## 1. Repository
- GitHub: `Anwarnara/streamflow`

## 2. Infrastructure
- Public URL: `http://104.234.26.223:7575`
- RTMP Ingest: `rtmp://104.234.26.223/live` (Port 1935)
- Backend: systemd `streamflow-backend.service`

## 3. Fitur Utama Backend
- Modul berada di `internal/streaming/` (rtmp_auth, simulcast, chat_aggregator).

