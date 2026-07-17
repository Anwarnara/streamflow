# System Event Catalog
**Version:** 3.0.3  
**Last Updated:** 2026-07-17  

## 1. Internal Events
Saat ini StreamFlow menggunakan synchronous processing, namun event log dicatat untuk audit:
- `USER_LOGIN_SUCCESS`
- `VIDEO_UPLOAD_COMPLETED`
- `STREAM_STARTED`
- `STREAM_STOPPED`
- `ROTATION_TRIGGERED`

## 2. External Webhooks (Planned)
Fase mendatang akan mendukung webhooks untuk integrasi Discord/Slack:
- `ON_STREAM_START`
- `ON_STREAM_ERROR`

