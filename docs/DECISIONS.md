# Architecture Decision Records (ADR)
**Version:** 3.1.0  
**Last Updated:** 2026-07-17  

## ADR-004: FFmpeg Tee Muxer untuk Simulcast
**Konteks**: User perlu siaran ke YouTube dan Facebook bersamaan tanpa mematikan CPU server.
**Keputusan**: Menggunakan syntax FFmpeg `[tee]` untuk menduplikasi *packet* RTMP tanpa men-decode (Zero-CPU copy).
**Hasil**: Konsumsi CPU tetap ~5% walau stream di-cast ke 3 platform.

## ADR-005: SSE untuk Live Chat
**Konteks**: WebSockets terlalu berat dan overkill karena komunikasi chat stream bersifat satu arah (Server ke UI Dashboard).
**Keputusan**: Menggunakan Server-Sent Events (SSE).
**Hasil**: Koneksi lebih stabil, built-in reconnect di browser, beban sangat ringan bagi backend Go.

