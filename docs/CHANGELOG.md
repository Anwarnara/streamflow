# Changelog
**Version:** 3.1.0  
**Last Updated:** 2026-07-17  

## Version 3.1.0 (2026-07-17)

**PRODUCTION READY**

### 🎉 New Features: Enterprise Live Streaming
- **RTMP Ingest Server**: Nginx-RTMP module integration on port 1935 with Go-backed authentication.
- **Simulcast Engine**: FFmpeg `[tee]` pseudo-muxer integration for zero-CPU overhead multi-platform restreaming.
- **Auto-Failover**: Automatic fallback to "Stand By" video loops when RTMP input disconnects, keeping destination streams alive.
- **Dynamic Watermark**: Real-time overlay capabilities for live streams.
- **Unified Live Chat**: SSE (Server-Sent Events) endpoint for real-time chat aggregation.
- **Live Dashboard**: New UI for stream monitoring, RTMP key management, and chat.

### Documentation
- Restructured documentation into 20 standard files using `web-dokumen` spec.

