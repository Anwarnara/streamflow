# StreamFlow - Thumbnail System Documentation

**Version:** 3.0.1  
**Last Updated:** 2026-07-17  
**Status:** Fully Operational

---

## Overview

StreamFlow automatically generates video thumbnails during upload using FFmpeg. Thumbnails are displayed in the Gallery view (both grid and list layouts) and Stream modal video selector.

---

## Architecture

### Components

```
Video Upload Flow
    ↓
Chunked Upload Complete
    ↓
FFmpeg Thumbnail Generation
    ↓
Database Entry (thumbnail_path)
    ↓
Frontend Display (gallery.html)
```

### File Locations

```
/var/www/streaming-go/
├── uploads/
│   └── videos/
│       ├── *.mp4                    # Video files
│       └── thumb_*.jpg              # Thumbnail files
├── internal/
│   └── handlers/
│       └── chunked_upload.go        # Auto-generates thumbnails
└── templates/
    └── gallery.html                 # Displays thumbnails
```

---

## Thumbnail Generation

### Automatic Generation (on Upload)

Location: `/var/www/streaming-go/internal/handlers/chunked_upload.go` (line 263-294)

**Process:**
1. Video upload completes via chunked upload
2. FFmpeg extracts frame at 00:00:01
3. Thumbnail saved as `thumb_<filename>_<timestamp>.jpg`
4. Database `thumbnail_path` column updated
5. If FFmpeg fails, video still saved (thumbnail NULL)

**FFmpeg Command:**
```bash
/usr/local/bin/ffmpeg \
  -y \
  -i <video.mp4> \
  -ss 00:00:01.000 \
  -vframes 1 \
  -vf "scale=640:-1" \
  -q:v 2 \
  <thumb.jpg>
```

**Parameters:**
- `-ss 00:00:01.000` - Extract frame at 1 second
- `-vframes 1` - Extract only 1 frame
- `-vf "scale=640:-1"` - Width 640px, height auto (maintain aspect ratio)
- `-q:v 2` - JPEG quality level 2 (high quality, ~150-200KB)

### Manual Generation (for existing videos)

Script: `/var/www/streaming-go/scripts/generate_missing_thumbnails.sh`

**Usage:**
```bash
cd /var/www/streaming-go
./scripts/generate_missing_thumbnails.sh
```

**What it does:**
1. Queries database for videos with `thumbnail_path IS NULL`
2. Generates thumbnails using FFmpeg
3. Updates database with thumbnail path
4. Reports progress and errors

**Check missing thumbnails:**
```bash
sudo -u postgres psql -d streamflow -c \
  "SELECT COUNT(*) - COUNT(thumbnail_path) as missing FROM videos;"
```

---

## Database Schema

### Column: `thumbnail_path`

```sql
ALTER TABLE videos ADD COLUMN thumbnail_path TEXT;
```

**Type:** TEXT (nullable)  
**Format:** `/uploads/videos/thumb_<name>_<timestamp>.jpg`  
**Example:** `/uploads/videos/thumb_1080p_30_Fps_V_3_1721174425.jpg`

**Query videos without thumbnails:**
```sql
SELECT id, title, filepath 
FROM videos 
WHERE thumbnail_path IS NULL;
```

**Update thumbnail path:**
```sql
UPDATE videos 
SET thumbnail_path = '/uploads/videos/thumb_example.jpg', 
    updated_at = NOW() 
WHERE id = '<video-id>';
```

---

## Frontend Display

### Gallery Template

Location: `/var/www/streaming-go/templates/gallery.html`

**Grid View (line 207-216):**
```javascript
<div class="aspect-video bg-dark-900 flex items-center justify-center">
    ${video.thumbnail_path ? 
        `<img src="${video.thumbnail_path}" alt="${video.title}" class="w-full h-full object-cover">` :
        `<i class="ti ti-video text-4xl text-gray-600"></i>`
    }
</div>
```

**List View (line 246-250):**
```javascript
<div class="w-16 h-16 bg-dark-900 rounded flex items-center justify-center">
    ${video.thumbnail_path ? 
        `<img src="${video.thumbnail_path}" alt="${video.title}" class="w-full h-full object-cover rounded">` :
        `<i class="ti ti-video text-2xl text-gray-600"></i>`
    }
</div>
```

**Behavior:**
- If `thumbnail_path` exists → Display `<img>` tag
- If `thumbnail_path` is NULL → Display video icon placeholder

### Static File Serving

Location: `/var/www/streaming-go/cmd/backend/main.go` (line 95)

```go
e.Static("/uploads", "uploads")
```

**Access URL:**
```
http://104.234.26.223:7575/uploads/videos/thumb_example.jpg
```

---

## API Response

### GET /api/videos

**Response includes `thumbnail_path`:**
```json
[
  {
    "id": "81a48206-43e3-47a2-acfb-3d1c9ffa2d58",
    "title": "1080p 30 Fps V_3.mp4",
    "filepath": "/uploads/videos/1080p 30 Fps V_3_1784243984.mp4",
    "thumbnail_path": "/uploads/videos/thumb_1080p_30_Fps_V_3_manual.jpg",
    "file_size": 130457973,
    "user_id": "22b2c732-f548-4fdf-9e5e-7538e3354a5f",
    "upload_date": "2026-07-16T23:19:44.508784Z"
  }
]
```

---

## Troubleshooting

### Issue: Thumbnails not appearing

**1. Check if thumbnail file exists:**
```bash
ls -lh /var/www/streaming-go/uploads/videos/thumb_*.jpg
```

**2. Check database entries:**
```bash
sudo -u postgres psql -d streamflow -c \
  "SELECT id, title, thumbnail_path FROM videos LIMIT 5;"
```

**3. Test static file access:**
```bash
curl -I http://localhost:8100/uploads/videos/test_thumb.jpg
# Should return: HTTP/1.1 200 OK
```

**4. Check FFmpeg availability:**
```bash
/usr/local/bin/ffmpeg -version
# Should show: ffmpeg version 6.1.1 (AVX-512)
```

**5. Review backend logs:**
```bash
journalctl -u streamflow-backend -f | grep -i thumbnail
```

### Issue: FFmpeg fails during upload

**Symptoms:**
- Upload succeeds but thumbnail is NULL
- Backend logs show: `Warning: Failed to generate thumbnail`

**Causes:**
1. Video codec not supported by FFmpeg
2. Video file corrupted
3. FFmpeg binary path incorrect
4. Insufficient disk space

**Solution:**
```bash
# Manually generate thumbnail
/usr/local/bin/ffmpeg -i <video.mp4> -ss 00:00:01 -vframes 1 -vf "scale=640:-1" test.jpg

# If successful, update database
sudo -u postgres psql -d streamflow -c \
  "UPDATE videos SET thumbnail_path = '/uploads/videos/thumb_<name>.jpg' WHERE id = '<id>';"
```

### Issue: Thumbnails too large (>500KB)

**Adjust JPEG quality in `chunked_upload.go`:**
```go
// Change from -q:v 2 to -q:v 4 (lower quality)
cmd := exec.Command("/usr/local/bin/ffmpeg", 
    "-y", "-i", finalFilePath, 
    "-ss", "00:00:01.000", 
    "-vframes", "1", 
    "-vf", "scale=640:-1", 
    "-q:v", "4",  // Changed from 2 to 4
    thumbPath)
```

---

## Performance

### Thumbnail Specs

- **Resolution:** 640px width (auto height, maintains aspect ratio)
- **Format:** JPEG
- **Quality:** Level 2 (high quality)
- **Typical Size:** 150-200 KB
- **Generation Time:** ~1-2 seconds per video

### Optimization

**Current:**
- Thumbnails generated synchronously during upload
- One thumbnail per video (at 1 second mark)

**Future Enhancements (v3.2):**
- Multiple thumbnails (sprite sheet for hover preview)
- Background thumbnail generation queue
- WebP format support (smaller file size)
- Lazy loading thumbnails in gallery

---

## Security

### File Access

- Static files served via Echo's `e.Static()` middleware
- No authentication required for thumbnails (same as videos)
- Files stored outside public web root

### Path Injection Prevention

```go
// Thumbnail paths are generated server-side, not user-controlled
thumbFileName := fmt.Sprintf("thumb_%s_%d.jpg", base, time.Now().Unix())
```

---

## Maintenance

### Clean up orphaned thumbnails

Thumbnails without corresponding database entries:

```bash
# List all thumbnail files
ls /var/www/streaming-go/uploads/videos/thumb_*.jpg

# List thumbnails in database
sudo -u postgres psql -d streamflow -c \
  "SELECT thumbnail_path FROM videos WHERE thumbnail_path IS NOT NULL;"

# Manual cleanup (compare and remove orphans)
```

### Regenerate all thumbnails

```bash
# Drop all thumbnails from database
sudo -u postgres psql -d streamflow -c \
  "UPDATE videos SET thumbnail_path = NULL;"

# Run generation script
cd /var/www/streaming-go
./scripts/generate_missing_thumbnails.sh
```

---

## Testing

### Test auto-generation on new upload

1. Upload a new video via `/gallery` page
2. Wait for upload to complete
3. Check backend logs for FFmpeg execution
4. Verify thumbnail appears in gallery
5. Check database for `thumbnail_path` value

### Test manual generation script

```bash
# Create a test video without thumbnail
sudo -u postgres psql -d streamflow -c \
  "UPDATE videos SET thumbnail_path = NULL WHERE id = '<test-video-id>';"

# Run script
./scripts/generate_missing_thumbnails.sh

# Verify thumbnail regenerated
sudo -u postgres psql -d streamflow -c \
  "SELECT thumbnail_path FROM videos WHERE id = '<test-video-id>';"
```

---

## Statistics

**Current Status (2026-07-17):**
```
Total Videos:        1
With Thumbnails:     1 (100%)
Missing Thumbnails:  0 (0%)
```

**Query:**
```sql
SELECT 
    COUNT(*) as total_videos,
    COUNT(thumbnail_path) as with_thumbnails,
    COUNT(*) - COUNT(thumbnail_path) as missing_thumbnails
FROM videos;
```

---

## Version History

- **v3.0.1** (2026-07-17): Thumbnail system fully operational
  - FFmpeg integration complete
  - Auto-generation on upload
  - Manual generation script added
  - Frontend display working (grid + list views)

---

**Document Owner:** Development Team  
**Last Verified:** 2026-07-17 00:33 WIB  
**Status:** ✅ PRODUCTION READY
