# User Flows

**Version:** 3.3.0
**Last Updated:** 2026-07-19

## Overview

This document describes the primary user journeys through the StreamFlow interface, covering the most common workflows an operator performs.

---

## Flow 1: Upload a Video and Start a Stream

1. Navigate to **Gallery** (`/gallery`).
2. Click **Upload**. Select a video file. The chunked upload starts.
3. When upload completes, FFmpeg extracts metadata and generates a thumbnail. The video appears in the gallery.
4. Navigate to **Dashboard** (`/`).
5. Click **New Stream**. Fill in title, select the uploaded video, add RTMP URL and stream key for the target platform.
6. Click **Start**. FFmpeg launches. The stream card updates to `LIVE`. The health stats bar begins showing metrics within 2 seconds.

---

## Flow 2: Create a Simulcast with Per-destination Overrides (v3.3.0)

1. Navigate to **Dashboard** and click **New Stream**.
2. Configure the primary stream (title, source video, base bitrate/resolution).
3. In the **Simulcast Destinations** section, click **Add Destination**.
4. Enter RTMP URL and stream key for the first platform (e.g. YouTube at 4500kbps).
5. Expand the **Encode Override** section. Set `bitrate: 4500`, `resolution: 1920x1080`.
6. Click **Add Destination** again. Enter details for a second platform (e.g. Twitch).
7. Set `bitrate: 3000`, `resolution: 1280x720` for the Twitch destination.
8. Click **Start**. Since destinations have overrides, per-destination FFmpeg processes launch. Both platforms receive their respective quality levels.

---

## Flow 3: Schedule a Stream (v3.3.0)

1. Navigate to **Dashboard**, click **New Stream**.
2. Configure stream source and destinations as normal.
3. In the form, set the **Schedule Time** field using the datetime picker (e.g. tomorrow at 9:00 AM).
4. Click **Save** (not Start). The stream card shows status `SCHEDULED` with the scheduled time.
5. At the scheduled time (within 30 seconds), the system auto-starts the stream. Status changes to `LIVE`.

---

## Flow 4: Monitor Stream Health (v3.3.0)

1. Navigate to **Stream Health** (`/stream-health`) from the sidebar/bottom nav.
2. The page shows summary cards at top: Active Streams, Avg FPS, Total Drop Frames, Streams w/ Drops.
3. The table below shows per-stream metrics refreshed every 3 seconds.
4. A stream showing `WARNING` status indicates elevated drop frames or speed < 0.95. Click through to its dashboard card for more detail.
5. Click the **Logs** button on a live stream card to open the terminal modal and inspect raw FFmpeg output in real time.

---

## Flow 5: Create a Playlist Stream

1. Navigate to **Playlists** (`/playlists`). Click **New Playlist**. Name it.
2. Add videos to the playlist in desired order. Drag to reorder.
3. Navigate to **Dashboard**, click **New Stream**.
4. Select **Playlist** as the source instead of a single video. Pick the playlist.
5. Enable **Loop** if you want the playlist to repeat indefinitely.
6. Start the stream. FFmpeg plays through playlist items sequentially. `stream_playlist_items` tracks the current position.

---

## Flow 6: Debug a Failing Stream (v3.3.0)

1. A stream card on the dashboard shows a `DEGRADED` or `ERROR` status badge.
2. Click **Logs** on the stream card.
3. The terminal modal opens, showing the raw FFmpeg stderr output from the `LogBuffer` (last 500 lines).
4. Identify the error in the log (e.g. "Connection refused" for a dead RTMP endpoint, or codec mismatch).
5. Stop the stream, fix the configuration (e.g. update the RTMP URL), and restart.

---

## Flow 7: Manage the Video Library

1. Navigate to **Gallery**.
2. Create folders via **New Folder** to organize content (e.g. by show, date, or client).
3. Upload videos to specific folders.
4. Edit video title, or trigger thumbnail regeneration via the video card menu.
5. Delete videos that are no longer needed (system checks that the video is not attached to an active stream before allowing deletion).
