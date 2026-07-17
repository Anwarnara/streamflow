# Frontend Design & Architecture
## StreamFlow: Multi-Platform Live Streaming Application

**Version:** 3.0.2  
**Last Updated:** 2026-07-17  
**Status:** Production Ready (Go Migration Complete)

---

## Frontend Technology Stack

### Core Framework
- **Template Engine**: Go HTML Templates (html/template)
- **Rendering Mode**: Server-Side Rendering (SSR)
- **JavaScript**: Vanilla ES6+ (No frontend framework)
- **CSS Framework**: Inline CSS with dark theme + Tabler Icons
- **AJAX Library**: Native Fetch API
- **Real-time Updates**: 1-second polling (setTimeout + fetch)

### Key Libraries
```javascript
// Client-side JavaScript dependencies (loaded via CDN)
- None (Pure vanilla JavaScript implementation)

// Server-side rendering
- EJS v3.x (Template rendering)
```

---

## Architecture Pattern

### Server-Side Rendering (SSR) Flow

```
User Request
    ↓
Express Router
    ↓
Controller Logic (app.js)
    ↓
Data Fetching (Models)
    ↓
EJS Template Rendering
    ↓
HTML Response to Browser
    ↓
Client-side JavaScript Hydration
```

### Page Load Lifecycle

1. **Initial Request**: Browser requests page URL
2. **Server Processing**: Express routes request to controller
3. **Data Preparation**: Controller fetches data from SQLite
4. **Template Rendering**: EJS compiles template with data
5. **HTML Delivery**: Fully-rendered HTML sent to browser
6. **JavaScript Execution**: Client-side JS attaches event listeners
7. **AJAX Updates**: Subsequent interactions via Fetch API

---

## Component Hierarchy

```
Application Root (app.js)
│
├── Public Pages (No Authentication)
│   ├── Login Page (views/login.ejs)
│   │   └── Client: public/js/auth.js (login form handling)
│   └── Register Page (views/register.ejs)
│       └── Client: public/js/auth.js (registration form handling)
│
└── Protected Pages (Requires Authentication)
    ├── Dashboard (views/dashboard.ejs)
    │   ├── Stream Statistics Cards
    │   ├── Active Streams List
    │   └── Client: public/js/dashboard.js
    │       ├── Real-time stream status polling
    │       ├── Quick action buttons (Start/Stop)
    │       └── Stream log viewer
    │
    ├── Video Management (views/videos.ejs)
    │   ├── Video Gallery Grid
    │   ├── Upload Form (Local & Google Drive)
    │   ├── Video Preview Modal
    │   └── Client: public/js/videos.js
    │       ├── File upload with progress
    │       ├── Google Drive import handler
    │       ├── Video delete confirmation
    │       └── Metadata display (duration, codec, resolution)
    │
    ├── Playlist Management (views/playlists.ejs)
    │   ├── Playlist List
    │   ├── Playlist Creator Form
    │   ├── Video/Audio Selector
    │   └── Client: public/js/playlists.js
    │       ├── Drag-and-drop video ordering
    │       ├── Shuffle toggle
    │       └── Loop configuration
    │
    ├── Stream Management (views/streams.ejs)
    │   ├── Stream Configuration Form
    │   │   ├── Video/Playlist Selector
    │   │   ├── Platform Selector (YouTube/Facebook/Custom)
    │   │   ├── RTMP Configuration
    │   │   ├── Simple/Advanced Mode Toggle
    │   │   └── Schedule Settings
    │   ├── Active Streams Monitor
    │   └── Client: public/js/streams.js
    │       ├── Form validation
    │       ├── Real-time log streaming (WebSocket alternative: polling)
    │       ├── Stream control (Start/Stop/Delete)
    │       └── Scheduled stream countdown
    │
    └── Stream History (views/history.ejs)
        ├── Historical Stream Table
        ├── Date Range Filter
        ├── Platform Filter
        └── Client: public/js/history.js
            ├── Pagination
            ├── Filter interactions
            └── Export to CSV (future)
```

---

## State Management

### Session State (Server-side)
```javascript
// express-session storage
req.session = {
  userId: <user_id>,
  username: <username>,
  isAuthenticated: true
}
```

### Client-side State Management

**No Global State Library** — Each page manages its own state via:

1. **DOM State**: Data stored in HTML attributes
```html
<div class="stream-card" 
     data-stream-id="abc-123" 
     data-status="live" 
     data-platform="youtube">
```

2. **In-Memory JavaScript Objects**
```javascript
// videos.js
const videoCache = new Map();

// streams.js
let activeStreamsPolling = null;
let streamLogs = {};
```

3. **LocalStorage** (Minimal usage)
```javascript
// User preferences (optional future feature)
localStorage.setItem('preferredPlatform', 'youtube');
```

---

## Routing Strategy

### Server-Side Routes (Express)

```javascript
// Public routes
GET  /               → Redirect to /login or /dashboard
GET  /login          → Login page (views/login.ejs)
POST /login          → Authentication handler
GET  /register       → Registration page
POST /register       → User creation handler

// Protected routes (auth middleware)
GET  /dashboard      → Main dashboard (views/dashboard.ejs)
GET  /videos         → Video management page
POST /videos/upload  → Video upload endpoint
GET  /videos/:id     → Video details API
DELETE /videos/:id   → Video deletion

GET  /playlists      → Playlist management page
POST /playlists      → Create playlist
PUT  /playlists/:id  → Update playlist
DELETE /playlists/:id → Delete playlist

GET  /streams        → Stream management page
POST /streams        → Create stream configuration
POST /streams/:id/start → Start stream
POST /streams/:id/stop  → Stop stream
GET  /streams/:id/logs  → Get stream logs (AJAX)
DELETE /streams/:id     → Delete stream

GET  /history        → Stream history page
GET  /api/history    → Historical data API (JSON)

GET  /logout         → Session destruction
```

### Client-side Navigation
- **Traditional Page Loads**: No SPA, full page refresh on navigation
- **AJAX Interactions**: Form submissions and data updates via Fetch API
- **No Client-side Router**: All routing handled by Express

---

## Data Fetching Strategy

### Initial Page Load (SSR)
```javascript
// Server-side data injection
app.get('/dashboard', requireAuth, async (req, res) => {
  const streams = await Stream.findAll(req.session.userId);
  const videos = await Video.findAll(req.session.userId);
  
  res.render('dashboard', {
    user: req.session.username,
    streams: streams,
    videos: videos
  });
});
```

### Dynamic Updates (Client-side AJAX)
```javascript
// public/js/dashboard.js
async function pollStreamStatus() {
  const response = await fetch('/api/streams/status');
  const data = await response.json();
  
  updateStreamCards(data.streams);
}

setInterval(pollStreamStatus, 5000); // Poll every 5 seconds
```

### Error Handling Pattern
```javascript
async function deleteVideo(videoId) {
  try {
    const response = await fetch(`/videos/${videoId}`, {
      method: 'DELETE',
      headers: { 'Content-Type': 'application/json' }
    });
    
    if (!response.ok) {
      const error = await response.json();
      showErrorNotification(error.message);
      return;
    }
    
    showSuccessNotification('Video deleted successfully');
    removeVideoFromDOM(videoId);
  } catch (err) {
    showErrorNotification('Network error. Please try again.');
  }
}
```

### Loading States
```javascript
// Generic loading indicator
function showLoading(elementId) {
  const el = document.getElementById(elementId);
  el.innerHTML = '<div class="spinner"></div>';
  el.classList.add('loading');
}

function hideLoading(elementId) {
  const el = document.getElementById(elementId);
  el.classList.remove('loading');
}
```

---

## Form Handling

### Standard Form Submission Pattern

```html
<!-- views/streams.ejs -->
<form id="stream-form" onsubmit="handleStreamSubmit(event)">
  <input type="text" name="title" required>
  <select name="video_id" required>
    <% videos.forEach(video => { %>
      <option value="<%= video.id %>"><%= video.title %></option>
    <% }); %>
  </select>
  <button type="submit">Create Stream</button>
</form>
```

```javascript
// public/js/streams.js
async function handleStreamSubmit(event) {
  event.preventDefault();
  
  const formData = new FormData(event.target);
  const data = Object.fromEntries(formData);
  
  showLoading('stream-form');
  
  try {
    const response = await fetch('/streams', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    
    if (!response.ok) throw new Error('Failed to create stream');
    
    showSuccessNotification('Stream created successfully');
    window.location.href = '/dashboard';
  } catch (err) {
    showErrorNotification(err.message);
  } finally {
    hideLoading('stream-form');
  }
}
```

### File Upload with Progress

```javascript
// public/js/videos.js
async function uploadVideo(file) {
  const formData = new FormData();
  formData.append('video', file);
  
  const xhr = new XMLHttpRequest();
  
  xhr.upload.addEventListener('progress', (e) => {
    if (e.lengthComputable) {
      const percent = (e.loaded / e.total) * 100;
      updateProgressBar(percent);
    }
  });
  
  xhr.addEventListener('load', () => {
    if (xhr.status === 200) {
      showSuccessNotification('Upload complete');
      refreshVideoGallery();
    } else {
      showErrorNotification('Upload failed');
    }
  });
  
  xhr.open('POST', '/videos/upload');
  xhr.send(formData);
}
```

---

## Responsive Design Strategy

### Breakpoints
```css
/* public/css/style.css */

/* Mobile-first approach */
.container {
  padding: 1rem;
}

/* Tablet (≥ 768px) */
@media (min-width: 768px) {
  .container {
    max-width: 720px;
    margin: 0 auto;
  }
  
  .stream-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* Desktop (≥ 1024px) */
@media (min-width: 1024px) {
  .container {
    max-width: 960px;
  }
  
  .stream-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

/* Large Desktop (≥ 1440px) */
@media (min-width: 1440px) {
  .container {
    max-width: 1200px;
  }
  
  .stream-grid {
    grid-template-columns: repeat(4, 1fr);
  }
}
```

### Mobile Considerations
- Touch-friendly buttons (min 44px tap target)
- Swipe gestures for video gallery (future enhancement)
- Simplified forms on mobile (reduced fields)
- Collapsible sidebar navigation

---

## Performance Optimization

### Initial Load Optimization
1. **Minimize CSS**: Single `style.css` file (< 50KB)
2. **Defer JavaScript**: Scripts loaded at end of `<body>`
3. **Image Optimization**: Video thumbnails generated server-side
4. **Gzip Compression**: Express compression middleware

### Runtime Optimization
1. **Polling Optimization**: Adaptive polling interval based on activity
```javascript
let pollInterval = isActive ? 5000 : 30000; // 5s active, 30s idle
```

2. **DOM Manipulation**: Batch updates to minimize reflows
```javascript
const fragment = document.createDocumentFragment();
streams.forEach(stream => {
  const card = createStreamCard(stream);
  fragment.appendChild(card);
});
container.appendChild(fragment);
```

3. **Event Delegation**: Single listener for dynamic elements
```javascript
document.getElementById('stream-list').addEventListener('click', (e) => {
  if (e.target.matches('.delete-btn')) {
    handleDeleteStream(e.target.dataset.streamId);
  }
});
```

---

## User Experience Features

### Real-time Feedback
- Toast notifications (success/error/info)
- Progress indicators for long operations
- Optimistic UI updates (update DOM before server confirmation)
- Keyboard shortcuts (ESC to close modals)

### Accessibility
- Semantic HTML5 elements
- ARIA labels for screen readers
- Keyboard navigation support
- High contrast mode compatibility (future)

### Error Recovery
- Auto-retry failed requests (max 3 attempts)
- Graceful degradation when features unavailable
- Clear error messages with actionable steps
- Fallback content for missing data
