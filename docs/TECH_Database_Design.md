# Database Design
## StreamFlow: Multi-Platform Live Streaming Application

**Version:** 3.0.2  
**Last Updated:** 2026-07-17

---

## Database Technology

### SQLite3
- **Version**: 3.x
- **File Location**: `/var/www/streaming/database.sqlite`
- **Driver**: `sqlite3` npm package
- **Connection Mode**: Single-threaded, synchronous writes

### Rationale
- **Simplicity**: Zero configuration, file-based database
- **Portability**: Single file backup/restore
- **Performance**: Sufficient for single-server deployment (< 100 concurrent users)
- **ACID Compliance**: Full transaction support

### Limitations
- **Horizontal Scaling**: NOT supported (single file lock)
- **Concurrent Writes**: Limited (queue-based)
- **Max Database Size**: ~281 TB (theoretical, practical limit ~1TB)

---

## Entity Relationship Diagram (ERD)

```
┌─────────────────┐
│     users       │
│─────────────────│
│ id (PK)         │──┐
│ username (UQ)   │  │
│ password        │  │
│ created_at      │  │
└─────────────────┘  │
                     │ 1:N
                     ├─────────────────────────────────┐
                     │                                 │
                     │                                 │
         ┌───────────▼──────────┐         ┌───────────▼──────────┐
         │       videos         │         │      streams         │
         │──────────────────────│         │──────────────────────│
         │ id (PK)              │──┐      │ id (PK)              │
         │ title                │  │      │ title                │
         │ filepath             │  │      │ video_id (FK)        │───┐
         │ duration             │  │      │ user_id (FK)         │   │
         │ size                 │  │      │ platform             │   │
         │ resolution           │  │      │ rtmp_url             │   │
         │ codec                │  │      │ stream_key           │   │
         │ fps                  │  │      │ status               │   │
         │ bitrate              │  │      │ use_advanced_settings│   │
         │ user_id (FK)         │  │      │ resolution           │   │
         │ created_at           │  │      │ bitrate              │   │
         └──────────────────────┘  │      │ fps                  │   │
                                   │      │ orientation          │   │
         ┌─────────────────────┐   │      │ loop_video           │   │
         │     playlists       │   │      │ start_time           │   │
         │─────────────────────│   │      │ end_time             │   │
         │ id (PK)             │   │      │ is_youtube_api       │   │
         │ title               │   │      │ youtube_broadcast_id │   │
         │ is_shuffle          │   │      │ created_at           │   │
         │ user_id (FK)        │   │      │ updated_at           │   │
         │ created_at          │   │      └──────────────────────┘   │
         └─────────────────────┘   │                                 │
                  │                │                                 │
                  │ 1:N            │ N:M                             │
                  │                │ (via playlist_videos)           │
                  ▼                │                                 │
         ┌─────────────────────┐   │                                 │
         │  playlist_videos    │◄──┘                                 │
         │─────────────────────│                                     │
         │ id (PK)             │                                     │
         │ playlist_id (FK)    │                                     │
         │ video_id (FK)       │                                     │
         │ position            │                                     │
         └─────────────────────┘                                     │
                                                                     │
         ┌─────────────────────┐                                     │
         │  playlist_audios    │                                     │
         │─────────────────────│                                     │
         │ id (PK)             │                                     │
         │ playlist_id (FK)    │                                     │
         │ audio_id (FK)       │                                     │
         │ position            │                                     │
         └─────────────────────┘                                     │
                                                                     │
         ┌─────────────────────┐                                     │
         │   stream_history    │◄────────────────────────────────────┘
         │─────────────────────│
         │ id (PK)             │
         │ stream_id (FK)      │
         │ title               │
         │ platform            │
         │ video_id            │
         │ video_title         │
         │ start_time          │
         │ end_time            │
         │ duration (seconds)  │
         │ resolution          │
         │ bitrate             │
         │ fps                 │
         │ use_advanced_settings│
         │ user_id (FK)        │
         └─────────────────────┘
```

### Relationships
- **users ↔ videos**: One-to-Many (1 user → N videos)
- **users ↔ streams**: One-to-Many (1 user → N streams)
- **users ↔ playlists**: One-to-Many (1 user → N playlists)
- **videos ↔ streams**: One-to-Many (1 video → N streams)
- **playlists ↔ videos**: Many-to-Many (via `playlist_videos`)
- **playlists ↔ audios**: Many-to-Many (via `playlist_audios`)
- **streams ↔ stream_history**: One-to-Many (1 stream → N history records)

---

## Table Schemas

### 1. users

**Description**: User authentication and profile data

```sql
CREATE TABLE users (
  id TEXT PRIMARY KEY,
  username TEXT UNIQUE NOT NULL,
  password TEXT NOT NULL,
  created_at TEXT DEFAULT (datetime('now'))
);

CREATE UNIQUE INDEX idx_users_username ON users(username);
```

| Column     | Type | Constraints         | Description                    |
|------------|------|---------------------|--------------------------------|
| id         | TEXT | PRIMARY KEY         | UUID v4                        |
| username   | TEXT | UNIQUE, NOT NULL    | Login username                 |
| password   | TEXT | NOT NULL            | bcrypt hashed password (cost 10)|
| created_at | TEXT | DEFAULT now()       | ISO 8601 timestamp             |

**Sample Data**:
```sql
INSERT INTO users VALUES (
  'usr-abc-123-def',
  'streamer123',
  '$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy', -- "password"
  '2026-07-15T10:00:00.000Z'
);
```

---

### 2. videos

**Description**: Uploaded video metadata and file references

```sql
CREATE TABLE videos (
  id TEXT PRIMARY KEY,
  title TEXT NOT NULL,
  filepath TEXT NOT NULL,
  duration REAL DEFAULT 0,
  size INTEGER DEFAULT 0,
  resolution TEXT,
  codec TEXT,
  fps INTEGER,
  bitrate INTEGER,
  user_id TEXT NOT NULL,
  created_at TEXT DEFAULT (datetime('now')),
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

CREATE INDEX idx_videos_user_id ON videos(user_id);
CREATE INDEX idx_videos_created_at ON videos(created_at DESC);
```

| Column      | Type    | Constraints        | Description                           |
|-------------|---------|--------------------|---------------------------------------|
| id          | TEXT    | PRIMARY KEY        | UUID v4                               |
| title       | TEXT    | NOT NULL           | Display name                          |
| filepath    | TEXT    | NOT NULL           | Relative path: `uploads/videos/...`   |
| duration    | REAL    | DEFAULT 0          | Duration in seconds (e.g., 3600.5)    |
| size        | INTEGER | DEFAULT 0          | File size in bytes                    |
| resolution  | TEXT    |                    | Format: `1920x1080`                   |
| codec       | TEXT    |                    | Video codec (e.g., `h264`, `vp9`)     |
| fps         | INTEGER |                    | Frames per second                     |
| bitrate     | INTEGER |                    | Bitrate in kbps                       |
| user_id     | TEXT    | NOT NULL, FK       | Owner user ID                         |
| created_at  | TEXT    | DEFAULT now()      | Upload timestamp                      |

**Sample Data**:
```sql
INSERT INTO videos VALUES (
  'vid-123-abc',
  'My Gaming Stream',
  'uploads/videos/1721083200000-stream.mp4',
  3600.0,
  524288000,
  '1920x1080',
  'h264',
  30,
  5000,
  'usr-abc-123-def',
  '2026-07-15T12:00:00.000Z'
);
```

---

### 3. streams

**Description**: Stream configuration and status tracking

```sql
CREATE TABLE streams (
  id TEXT PRIMARY KEY,
  title TEXT NOT NULL,
  video_id TEXT NOT NULL,
  user_id TEXT NOT NULL,
  platform TEXT DEFAULT 'Custom',
  platform_icon TEXT,
  rtmp_url TEXT NOT NULL,
  stream_key TEXT NOT NULL,
  status TEXT DEFAULT 'offline',
  use_advanced_settings INTEGER DEFAULT 0,
  resolution TEXT DEFAULT '1280x720',
  bitrate INTEGER DEFAULT 2500,
  fps INTEGER DEFAULT 30,
  orientation TEXT DEFAULT 'landscape',
  loop_video INTEGER DEFAULT 0,
  start_time TEXT,
  end_time TEXT,
  is_youtube_api INTEGER DEFAULT 0,
  youtube_broadcast_id TEXT,
  created_at TEXT DEFAULT (datetime('now')),
  updated_at TEXT DEFAULT (datetime('now')),
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
  FOREIGN KEY (video_id) REFERENCES videos(id) ON DELETE CASCADE
);

CREATE INDEX idx_streams_user_id ON streams(user_id);
CREATE INDEX idx_streams_status ON streams(status);
CREATE INDEX idx_streams_start_time ON streams(start_time);
```

| Column                | Type    | Constraints      | Description                                  |
|-----------------------|---------|------------------|----------------------------------------------|
| id                    | TEXT    | PRIMARY KEY      | UUID v4                                      |
| title                 | TEXT    | NOT NULL         | Stream display name                          |
| video_id              | TEXT    | NOT NULL, FK     | Source video or playlist ID                  |
| user_id               | TEXT    | NOT NULL, FK     | Owner user ID                                |
| platform              | TEXT    | DEFAULT 'Custom' | Platform name (YouTube, Facebook, Custom)    |
| platform_icon         | TEXT    |                  | Icon URL or emoji                            |
| rtmp_url              | TEXT    | NOT NULL         | RTMP server URL                              |
| stream_key            | TEXT    | NOT NULL         | RTMP stream key (sensitive)                  |
| status                | TEXT    | DEFAULT 'offline'| `offline`, `scheduled`, `live`               |
| use_advanced_settings | INTEGER | DEFAULT 0        | Boolean: 0=Simple Mode, 1=Advanced Mode      |
| resolution            | TEXT    | DEFAULT '1280x720'| Target resolution (Advanced Mode)           |
| bitrate               | INTEGER | DEFAULT 2500     | Video bitrate in kbps (Advanced Mode)        |
| fps                   | INTEGER | DEFAULT 30       | Target FPS (Advanced Mode)                   |
| orientation           | TEXT    | DEFAULT 'landscape'| `landscape` or `portrait`                  |
| loop_video            | INTEGER | DEFAULT 0        | Boolean: 0=No loop, 1=Loop forever           |
| start_time            | TEXT    |                  | Scheduled start time (ISO 8601)              |
| end_time              | TEXT    |                  | Scheduled end time (ISO 8601)                |
| is_youtube_api        | INTEGER | DEFAULT 0        | Boolean: YouTube Data API integration        |
| youtube_broadcast_id  | TEXT    |                  | YouTube LiveBroadcast ID                     |
| created_at            | TEXT    | DEFAULT now()    | Creation timestamp                           |
| updated_at            | TEXT    | DEFAULT now()    | Last update timestamp                        |

**Sample Data**:
```sql
INSERT INTO streams VALUES (
  'stream-abc-123',
  'Daily Gaming Stream',
  'vid-123-abc',
  'usr-abc-123-def',
  'YouTube',
  '📺',
  'rtmp://a.rtmp.youtube.com/live2',
  'xxxx-xxxx-xxxx-xxxx',
  'scheduled',
  0, -- Simple mode (codec copy)
  '1920x1080',
  2500,
  30,
  'landscape',
  1, -- Loop enabled
  '2026-07-16T10:00:00.000Z',
  '2026-07-16T12:00:00.000Z',
  0,
  NULL,
  '2026-07-15T21:00:00.000Z',
  '2026-07-15T21:00:00.000Z'
);
```

---

### 4. playlists

**Description**: Playlist metadata for multi-video streams

```sql
CREATE TABLE playlists (
  id TEXT PRIMARY KEY,
  title TEXT NOT NULL,
  is_shuffle INTEGER DEFAULT 0,
  user_id TEXT NOT NULL,
  created_at TEXT DEFAULT (datetime('now')),
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

CREATE INDEX idx_playlists_user_id ON playlists(user_id);
```

| Column     | Type    | Constraints    | Description                         |
|------------|---------|----------------|-------------------------------------|
| id         | TEXT    | PRIMARY KEY    | UUID v4                             |
| title      | TEXT    | NOT NULL       | Playlist display name               |
| is_shuffle | INTEGER | DEFAULT 0      | Boolean: Randomize playback order   |
| user_id    | TEXT    | NOT NULL, FK   | Owner user ID                       |
| created_at | TEXT    | DEFAULT now()  | Creation timestamp                  |

---

### 5. playlist_videos

**Description**: Many-to-many relationship: playlists ↔ videos

```sql
CREATE TABLE playlist_videos (
  id TEXT PRIMARY KEY,
  playlist_id TEXT NOT NULL,
  video_id TEXT NOT NULL,
  position INTEGER NOT NULL,
  FOREIGN KEY (playlist_id) REFERENCES playlists(id) ON DELETE CASCADE,
  FOREIGN KEY (video_id) REFERENCES videos(id) ON DELETE CASCADE
);

CREATE INDEX idx_playlist_videos_playlist_id ON playlist_videos(playlist_id);
CREATE UNIQUE INDEX idx_playlist_videos_unique ON playlist_videos(playlist_id, video_id);
```

| Column      | Type    | Constraints         | Description                      |
|-------------|---------|---------------------|----------------------------------|
| id          | TEXT    | PRIMARY KEY         | UUID v4                          |
| playlist_id | TEXT    | NOT NULL, FK        | Parent playlist ID               |
| video_id    | TEXT    | NOT NULL, FK        | Video ID                         |
| position    | INTEGER | NOT NULL            | Sort order (1, 2, 3...)          |

---

### 6. playlist_audios

**Description**: Many-to-many relationship: playlists ↔ audio tracks

```sql
CREATE TABLE playlist_audios (
  id TEXT PRIMARY KEY,
  playlist_id TEXT NOT NULL,
  audio_id TEXT NOT NULL,
  position INTEGER NOT NULL,
  FOREIGN KEY (playlist_id) REFERENCES playlists(id) ON DELETE CASCADE,
  FOREIGN KEY (audio_id) REFERENCES videos(id) ON DELETE CASCADE
);

CREATE INDEX idx_playlist_audios_playlist_id ON playlist_audios(playlist_id);
```

| Column      | Type    | Constraints    | Description                       |
|-------------|---------|----------------|-----------------------------------|
| id          | TEXT    | PRIMARY KEY    | UUID v4                           |
| playlist_id | TEXT    | NOT NULL, FK   | Parent playlist ID                |
| audio_id    | TEXT    | NOT NULL, FK   | Audio file ID (references videos) |
| position    | INTEGER | NOT NULL       | Sort order                        |

---

### 7. stream_history

**Description**: Historical stream records for analytics

```sql
CREATE TABLE stream_history (
  id TEXT PRIMARY KEY,
  stream_id TEXT NOT NULL,
  title TEXT NOT NULL,
  platform TEXT,
  platform_icon TEXT,
  video_id TEXT,
  video_title TEXT,
  resolution TEXT,
  bitrate INTEGER,
  fps INTEGER,
  start_time TEXT NOT NULL,
  end_time TEXT NOT NULL,
  duration INTEGER NOT NULL,
  use_advanced_settings INTEGER DEFAULT 0,
  user_id TEXT NOT NULL,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

CREATE INDEX idx_stream_history_user_id ON stream_history(user_id);
CREATE INDEX idx_stream_history_start_time ON stream_history(start_time DESC);
CREATE INDEX idx_stream_history_platform ON stream_history(platform);
```

| Column                | Type    | Constraints      | Description                              |
|-----------------------|---------|------------------|------------------------------------------|
| id                    | TEXT    | PRIMARY KEY      | UUID v4                                  |
| stream_id             | TEXT    | NOT NULL         | Original stream ID (not FK, soft ref)    |
| title                 | TEXT    | NOT NULL         | Stream title at time of execution        |
| platform              | TEXT    |                  | Platform name                            |
| platform_icon         | TEXT    |                  | Platform icon                            |
| video_id              | TEXT    |                  | Source video ID (soft reference)         |
| video_title           | TEXT    |                  | Video title at time of stream            |
| resolution            | TEXT    |                  | Actual stream resolution                 |
| bitrate               | INTEGER |                  | Actual stream bitrate (kbps)             |
| fps                   | INTEGER |                  | Actual stream FPS                        |
| start_time            | TEXT    | NOT NULL         | Actual start timestamp (ISO 8601)        |
| end_time              | TEXT    | NOT NULL         | Actual end timestamp (ISO 8601)          |
| duration              | INTEGER | NOT NULL         | Total duration in seconds                |
| use_advanced_settings | INTEGER | DEFAULT 0        | Boolean: Advanced mode used              |
| user_id               | TEXT    | NOT NULL, FK     | Stream owner user ID                     |

**Sample Data**:
```sql
INSERT INTO stream_history VALUES (
  'hist-abc-123',
  'stream-abc-123',
  'Daily Gaming Stream',
  'YouTube',
  '📺',
  'vid-123-abc',
  'My Gaming Stream',
  '1920x1080',
  2500,
  30,
  '2026-07-15T10:00:00.000Z',
  '2026-07-15T12:00:00.000Z',
  7200, -- 2 hours
  0,
  'usr-abc-123-def'
);
```

---

## Data Integrity Rules

### Foreign Key Constraints

```sql
-- Cascade delete: When user is deleted, all related data is removed
FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE

-- Cascade delete: When video is deleted, streams using it are removed
FOREIGN KEY (video_id) REFERENCES videos(id) ON DELETE CASCADE
```

### Unique Constraints

```sql
-- Username must be unique
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- Video can only appear once per playlist
CREATE UNIQUE INDEX idx_playlist_videos_unique 
  ON playlist_videos(playlist_id, video_id);
```

### Check Constraints (Application-level)

```javascript
// Status must be one of: offline, scheduled, live
const VALID_STATUSES = ['offline', 'scheduled', 'live'];

// Resolution format validation
const RESOLUTION_REGEX = /^\d{3,4}x\d{3,4}$/; // e.g., 1920x1080
```

---

## Indexing Strategy

### Performance Indexes

```sql
-- User-scoped queries (most common)
CREATE INDEX idx_videos_user_id ON videos(user_id);
CREATE INDEX idx_streams_user_id ON streams(user_id);
CREATE INDEX idx_playlists_user_id ON playlists(user_id);
CREATE INDEX idx_stream_history_user_id ON stream_history(user_id);

-- Status filtering (dashboard)
CREATE INDEX idx_streams_status ON streams(status);

-- Time-based queries (scheduler, history)
CREATE INDEX idx_streams_start_time ON streams(start_time);
CREATE INDEX idx_stream_history_start_time ON stream_history(start_time DESC);

-- History filtering
CREATE INDEX idx_stream_history_platform ON stream_history(platform);
```

### Index Cardinality Analysis

| Index                          | Estimated Cardinality | Query Benefit                     |
|--------------------------------|-----------------------|-----------------------------------|
| idx_users_username             | High (unique)         | Login authentication              |
| idx_videos_user_id             | Medium (1:100)        | Video gallery loading             |
| idx_streams_user_id            | Medium (1:50)         | Dashboard stream list             |
| idx_streams_status             | Low (3 values)        | Active stream filtering           |
| idx_stream_history_start_time  | High (unique)         | Historical analytics, pagination  |

---

## Transaction Management

### Write Operations

```javascript
// Example: Create stream with transaction
db.serialize(() => {
  db.run('BEGIN TRANSACTION');
  
  try {
    // 1. Insert stream record
    db.run(
      'INSERT INTO streams (id, title, video_id, user_id, ...) VALUES (?, ?, ?, ?, ...)',
      [id, title, videoId, userId, ...]
    );
    
    // 2. Schedule cron job (external service)
    schedulerService.scheduleStream(streamId, startTime);
    
    db.run('COMMIT');
  } catch (err) {
    db.run('ROLLBACK');
    throw err;
  }
});
```

### Read Operations (No transaction needed)

```javascript
// Reads are always consistent (SQLite ACID guarantee)
db.all('SELECT * FROM streams WHERE user_id = ?', [userId], (err, rows) => {
  // Data is consistent snapshot at query time
});
```

---

## Backup & Recovery Strategy

### Daily Backup

```bash
#!/bin/bash
# /var/www/streaming/scripts/backup-db.sh

BACKUP_DIR="/var/backups/streaming"
DATE=$(date +%Y%m%d_%H%M%S)
DB_PATH="/var/www/streaming/database.sqlite"

mkdir -p "$BACKUP_DIR"

# SQLite backup (consistent snapshot)
sqlite3 "$DB_PATH" ".backup '$BACKUP_DIR/database_$DATE.sqlite'"

# Compress
gzip "$BACKUP_DIR/database_$DATE.sqlite"

# Keep last 30 days
find "$BACKUP_DIR" -name "database_*.sqlite.gz" -mtime +30 -delete
```

### Point-in-Time Recovery

```bash
# Restore from backup
gunzip -c /var/backups/streaming/database_20260715_210000.sqlite.gz > /var/www/streaming/database.sqlite

# Restart application
pm2 restart streamflow
```

---

## Migration Strategy

### Schema Versioning

```sql
-- migrations table (future implementation)
CREATE TABLE schema_migrations (
  version INTEGER PRIMARY KEY,
  applied_at TEXT DEFAULT (datetime('now'))
);
```

### Example Migration: Add column

```sql
-- Migration: Add 'thumbnail' column to videos
ALTER TABLE videos ADD COLUMN thumbnail TEXT;

-- Backfill existing records (optional)
UPDATE videos SET thumbnail = 'uploads/thumbnails/default.jpg';

-- Record migration
INSERT INTO schema_migrations (version) VALUES (2);
```
