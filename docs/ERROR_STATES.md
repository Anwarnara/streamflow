# Error Handling & States
**Version:** 3.1.0  
**Last Updated:** 2026-07-17  

## 1. RTMP Ingest Errors
- `403 Forbidden` pada auth: Jika stream key OBS salah atau tidak terdaftar. Nginx otomatis memutus koneksi OBS.

## 2. Simulcast Error Handling (Failover)
- Jika proses FFmpeg simulcast terputus atau OBS mati, backend tidak membiarkan stream YouTube mati. Go akan men-trigger function `triggerFallback()` yang melooping video siaga (Stand By).

