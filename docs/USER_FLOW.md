# User Journey Maps
**Version:** 3.1.0  
**Last Updated:** 2026-07-17  

## 1. Live Streaming Flow (OBS)
1. Buka halaman Live Control.
2. Dapatkan RTMP URL (`rtmp://ip/live`) and Stream Key dari UI.
3. Masukkan ke OBS dan klik "Start Streaming".
4. Nginx mengotentikasi Stream Key via `/api/rtmp/auth`.
5. FFmpeg mem-broadcast simulcast secara transparan di background.
6. Chat penonton mengalir secara otomatis ke UI via SSE.

## 2. Video Upload & Chunking
1. User upload file GB+.
2. Klien memotong jadi chunks 50MB.
3. Server menggabungkan dan memanggil FFmpeg untuk membuat thumbnail.

