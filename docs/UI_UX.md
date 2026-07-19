# UI/UX Patterns & Conventions

**Version:** 3.3.0
**Last Updated:** 2026-07-19

## Overview

This document describes the UI/UX patterns, conventions, and component standards used across the StreamFlow web interface.

---

## Layout

- All pages extend `layout.html` in `streaming-go/templates/`.
- `layout.html` provides: top header bar, desktop left sidebar nav, mobile bottom nav, and the main content area.
- New pages must be added to both the desktop sidebar AND the mobile bottom nav.

### Navigation Items (as of v3.3.0)

| Label | Route | Added |
|-------|-------|-------|
| Dashboard | `/` | v3.1.0 |
| Gallery | `/gallery` | v3.2.0 |
| Playlists | `/playlists` | v3.2.0 |
| History | `/history` | v3.2.0 |
| Stream Health | `/stream-health` | v3.3.0 |
| Settings | `/settings` | v3.1.0 |

---

## Stream Cards (Dashboard)

Each active or configured stream is displayed as a card on the main dashboard.

### Live Stream Card Elements (v3.3.0)

- Stream title and status badge (`LIVE`, `OFFLINE`, `SCHEDULED`)
- Destination platform icons
- Action buttons: Start / Stop / Edit / Logs
- **Health stats bar** (added v3.3.0): fps, bitrate, speed, drop_frames — shown only when stream is live
- Status indicator: colored dot (green = HEALTHY, yellow = WARNING, red = DEGRADED)

### Logs Modal (v3.3.0)

Triggered by the "Logs" button on a live stream card.

- Dark background (`#1a1a1a`), green monospace font (`#00ff41` or similar terminal green)
- Fixed-height scrollable container
- Auto-scrolls to the latest line
- Manual scroll lock toggle (click to pause auto-scroll)
- Consumes `GET /api/streams/:id/logs-stream` SSE endpoint
- Closes `EventSource` connection on modal dismiss

---

## Stream Health Page (`/stream-health`)

Template: `templates/health.html`

- Polls `GET /api/streams/health/all` every 3 seconds
- Summary row at top: 4 stat cards
  - Active Streams (count)
  - Avg FPS (across all live streams)
  - Total Drop Frames (sum)
  - Streams w/ Drops (count where drop_frames > 0)
- Main content: per-stream table
  - Columns: Stream ID, FPS, Bitrate, Speed, Total Frames, Drop Frames, Dup Frames, Elapsed Time, Status, Last Updated
  - Status badge: green `HEALTHY`, yellow `WARNING`, red `DEGRADED`
- Auto-refreshes table rows in place without full page reload

---

## Forms

### Create / Edit Stream Modal

- Title input
- Source selector: Video (single) or Playlist
- Platform + RTMP URL + Stream Key
- Bitrate / Resolution / FPS inputs
- Orientation toggle (horizontal / vertical)
- Loop toggle
- Fallback video selector
- Watermark upload + position picker
- **Schedule Time** (v3.3.0): `datetime-local` input for auto-start
- Advanced settings toggle (hidden by default)

### Simulcast Destination Form

- Platform icon selector
- RTMP URL + Stream Key inputs
- Active toggle
- **Encode Override** collapsible section (v3.3.0):
  - Bitrate override (0 = inherit)
  - Resolution override (empty = inherit)
  - FPS override (0 = inherit)
  - Video Codec (empty = inherit)
  - Preset (empty = inherit)

---

## Status Badges

Consistent badge styles used across the UI:

| State | Color | Label |
|-------|-------|-------|
| Live / Active | Green | `LIVE` |
| Offline | Gray | `OFFLINE` |
| Scheduled | Blue | `SCHEDULED` |
| Error | Red | `ERROR` |
| Healthy | Green | `HEALTHY` |
| Warning | Yellow/Amber | `WARNING` |
| Degraded | Red | `DEGRADED` |

---

## SSE Consumption Pattern

All SSE-driven UI components follow this pattern:

```js
const source = new EventSource('/api/streams/<id>/health-stream');
source.onmessage = (event) => {
  const data = JSON.parse(event.data);
  updateUI(data);
};
source.onerror = () => {
  // show reconnecting indicator, EventSource auto-reconnects
};
// On component teardown:
source.close();
```

Always call `source.close()` when the consuming component is removed from the DOM (modal close, page navigation) to release the HTTP connection.

---

## Mobile

- Bottom nav replaces sidebar on small screens (CSS breakpoint-based).
- All modals are full-screen on mobile.
- Health stats bar on stream cards stacks vertically on narrow viewports.
- PWA manifest enables "Add to Home Screen" on iOS and Android.

---

## Accessibility

- All interactive elements must have visible focus states.
- Status badges use both color and text label (not color alone) for colorblind users.
- Form inputs have associated `<label>` elements.
- ARIA attributes on SSE-driven live regions: `aria-live="polite"` for non-critical updates.
