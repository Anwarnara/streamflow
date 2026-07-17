# StreamFlow Changelog

## Version 3.0.3 (2026-07-17)

**PRODUCTION READY**

### 🎉 New Features & Improvements

#### UI/UX Enhancements
- **✅ Gallery Dropdown Fix**: Fixed 3-dot menu position to open upward (`bottom-8`) with higher z-index (`z-50`)
  - Prevents occlusion by bottom navigation bars
  - Improved accessibility on mobile devices
  - Menu now opens above the button instead of below

#### Features
- **✅ Video Rename**: Full-featured rename via 3-dot menu with real-time database update
- **✅ Thumbnail System**: Auto-generation during upload using FFmpeg (640px JPEG at 1s mark)
- **✅ Manual Thumbnail Generation**: Batch script for existing videos (`scripts/generate_missing_thumbnails.sh`)
- **✅ Gallery Thumbnails**: Display in both grid and list views with fallback icon

### Technical Details
- **Backend**: Go 1.25.0 + Echo Framework v4.12.0
- **Database**: PostgreSQL 14 (local, port 5432)
- **FFmpeg**: 6.1.1 AVX-512 custom build at `/usr/local/bin/ffmpeg`
- **Ports**: 8100 (internal), 7575 (Nginx public)
- **Session**: 72-hour cookie expiry

### Documentation
- Complete system documentation (11 files, 184KB)
- Thumbnail system guide (THUMBNAIL_SYSTEM.md, 399 lines)
- All core docs aligned to v3.0.3 (2026-07-17)

### Files Modified
- `/var/www/streaming-go/templates/gallery.html` (line 226)
  - Changed: `top-8 z-20` → `bottom-8 z-50`

---

**Deployment:** http://104.234.26.223:7575  
**Repository:** https://github.com/Anwarnara/streamflow  
**Tech Stack:** Go + Echo + PostgreSQL + Nginx + FFmpeg  
**Last Updated:** 2026-07-17 08:05 WIB
