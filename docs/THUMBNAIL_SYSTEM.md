# Thumbnail System

**Version:** 3.3.0
**Last Updated:** 2026-07-19

## Overview

StreamFlow automatically generates thumbnail images for uploaded videos using FFmpeg. Thumbnails are stored on disk and their paths are recorded in the `videos.thumbnail_path` column.

---

## Generation Process

When a video is uploaded and processed, the backend:

1. Receives the completed file path after chunked upload finishes.
2. Runs FFmpeg metadata extraction to populate `duration`, `resolution`, `bitrate`, `fps`, `format`.
3. Calculates a seek position at 10% of the video duration (ensures we avoid black frames at the start).
4. Runs FFmpeg to extract a single JPEG frame:

```
/usr/local/bin/ffmpeg -ss <seek_time> -i <input_path> -vframes 1 -q:v 2 -f image2 <output_path>
```

5. The `thumbnail_path` column in `videos` is updated with the generated file path.

---

## Storage

- Thumbnails are stored alongside or adjacent to the video file.
- Naming convention: `<video_uuid>_thumb.jpg`
- The path stored in `thumbnail_path` is relative or absolute depending on configuration — API consumers should use the `/api/videos/:id/thumbnail` endpoint rather than constructing paths manually.

---

## API Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/videos/:id/thumbnail` | Serve the video's thumbnail image |
| POST | `/api/videos/:id/thumbnail/generate` | Re-trigger thumbnail generation for an existing video |

The `generate` endpoint is useful when a video was uploaded before thumbnail support was available, or when the auto-generated thumbnail is unsatisfactory.

---

## FFmpeg Command Details

- `-ss <seek>` is placed before `-i` (input seeking) for performance — avoids decoding the entire video to reach the seek point.
- `-vframes 1` extracts exactly one frame.
- `-q:v 2` sets high JPEG quality (scale 1–31, lower is better).
- Output format is `image2` (single image file).

---

## Error Handling

- If FFmpeg fails during thumbnail generation, the upload is not aborted. The video record is saved with `thumbnail_path = null`.
- The frontend should handle a null thumbnail gracefully (show a placeholder icon).
- Manual re-generation via `POST /api/videos/:id/thumbnail/generate` can recover missing thumbnails.

---

## Watermark vs Thumbnail

The thumbnail system is distinct from the watermark overlay system:

- **Thumbnail:** A static image extracted from the video for display in the UI. Does not affect the stream output.
- **Watermark:** An image composited over the live video output during streaming via FFmpeg's `overlay` filter. Configured per stream in the `streams` table (`watermark_path`, `watermark_position`).
