# Domain Model & Business Rules
**Version:** 3.1.0  
**Last Updated:** 2026-07-17  

## 1. Entitas Utama
- **User**: Pengguna sistem (Admin, Member).
- **Video**: Aset media fisik yang diunggah.
- **Folder**: Struktur organisasi logis untuk Video.
- **Stream**: Konfigurasi transmisi, menyimpan key RTMP, failover video, dan setting watermark.
- **Stream Destination**: Target platform untuk simulcast (YouTube, Twitch, dll).
- **Stream Chat**: Pesan dari penonton yang ditarik via webhook.

## 2. Business Rules
- **Ownership**: Member hanya dapat mengelola Video/Stream miliknya sendiri.
- **Simulcast**: Satu stream masuk (ingest) dapat diteruskan ke N destinasi via `[tee]` muxer.
- **Failover**: Jika ingest RTMP terputus, backend langsung mengeksekusi FFmpeg loop dari video standby agar koneksi destinasi tidak mati.

