# StreamFlow - System Architecture
## Technical Architecture Overview

**Version:** 3.0.2  
**Last Updated:** 2026-07-17  
**Architecture Type:** Monolithic with Service-Oriented Components  

---

## 1. High-Level Architecture

### 1.1 System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   Browser   │  │   Mobile    │  │     PWA     │         │
│  │  (Desktop)  │  │  (Safari)   │  │ (Installed) │         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘         │
│         │                 │                 │                 │
│         └─────────────────┴─────────────────┘                │
│                           │                                   │
│                           │ HTTPS (TLS 1.3)                  │
└───────────────────────────┼───────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    REVERSE PROXY LAYER                       │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Nginx 1.18.0                                        │  │
│  │  - Port 7575 (Public)                                │  │
│  │  - SSL/TLS (Let's Encrypt)                           │  │
│  │  - Request buffering: OFF (for uploads)              │  │
│  │  - Client max body: 0 (unlimited)                    │  │
│  │  - Proxy to :8100                                    │  │
│  └──────────────────┬───────────────────────────────────┘  │
└─────────────────────┼───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                   APPLICATION LAYER (Go)                     │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Echo Framework v4.12.0 (Port 8100)                  │  │
│  │                                                        │  │
│  │  ┌─────────────────────────────────────────────┐    │  │
│  │  │  Middleware Stack                            │    │  │
│  │  │  - Logger                                    │    │  │
│  │  │  - Recover (panic handler)                   │    │  │
│  │  │  - Session Auth                              │    │  │
│  │  │  - CORS (if needed)                          │    │  │
│  │  └─────────────────────────────────────────────┘    │  │
│  │                                                        │  │
│  │  ┌─────────────────────────────────────────────┐    │  │
│  │  │  Route Handlers (85 endpoints)               │    │  │
│  │  │  - Auth (login, logout, signup)              │    │  │
│  │  │  - Users (CRUD + disk usage)                 │    │  │
│  │  │  - Videos (upload, list, delete, rename)     │    │  │
│  │  │  - Chunked Upload (6 endpoints)              │    │  │
│  │  │  - Streams (CRUD + start/stop)               │    │  │
│  │  │  - Gallery (folders, organization)           │    │  │
│  │  │  - Playlists (CRUD + video management)       │    │  │
│  │  │  - Rotations (scheduler)                     │    │  │
│  │  │  - Settings (logs, integrations)             │    │  │
│  │  │  - Forms (profile, password, gdrive)         │    │  │
│  │  │  - OAuth (YouTube)                           │    │  │
│  │  │  - PWA (service worker, manifest)            │    │  │
│  │  │  - System Stats (real-time monitoring)       │    │  │
│  │  └─────────────────────────────────────────────┘    │  │
│  │                                                        │  │
│  │  ┌─────────────────────────────────────────────┐    │  │
│  │  │  Business Logic Layer                        │    │  │
│  │  │  - Service functions                         │    │  │
│  │  │  - Validation logic                          │    │  │
│  │  │  - Business rules                            │    │  │
│  │  └─────────────────────────────────────────────┘    │  │
│  │                                                        │  │
│  │  ┌─────────────────────────────────────────────┐    │  │
│  │  │  Repository Layer                            │    │  │
│  │  │  - User Repository                           │    │  │
│  │  │  - Video Repository                          │    │  │
│  │  │  - Stream Repository                         │    │  │
│  │  │  - Folder Repository                         │    │  │
│  │  └─────────────────────────────────────────────┘    │  │
│  └──────────────────┬───────────────────────────────────┘  │
└─────────────────────┼───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                     DATABASE LAYER                           │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  PostgreSQL 14.23 (Port 5432)                        │  │
│  │  - Database: streamflow                              │  │
│  │  - User: streamflow_user                             │  │
│  │  - Connection Pool: 25 max connections               │  │
│  │  - SSL Mode: disabled (local)                        │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                   STORAGE LAYER                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Local Filesystem                                    │  │
│  │  - /var/www/streaming/uploads/                       │  │
│  │  - /var/www/streaming/uploads/chunks/                │  │
│  │  - /var/www/streaming/static/                        │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Component Architecture

### 2.1 Backend Structure (Go)

```
/var/www/streaming-go/
├── cmd/
│   └── backend/
│       └── main.go              # Application entry point
│
├── internal/
│   ├── auth/                    # Authentication module
│   │   ├── handler.go           # Login, logout, signup
│   │   ├── middleware.go        # Auth middleware
│   │   └── session.go           # Session management
│   │
│   ├── handlers/                # API handlers (7 files)
│   │   ├── users.go             # User management (8 endpoints)
│   │   ├── chunked_upload.go    # Chunked upload (6 endpoints)
│   │   ├── gallery.go           # Gallery/folders (5 endpoints)
│   │   ├── history_settings.go  # History + Settings (8 endpoints)
│   │   ├── playlist_audio.go    # Playlists + Audio (7 endpoints)
│   │   ├── rotations_misc.go    # Rotations + Misc (10 endpoints)
│   │   └── forms_oauth_pwa.go   # Forms + OAuth + PWA (8 endpoints)
│   │
│   ├── media/                   # Media handling
│   │   └── handler.go           # Video upload, list, delete
│   │
│   ├── streaming/               # Streaming module
│   │   └── handler.go           # Stream CRUD, start, stop
│   │
│   ├── models/                  # Data models
│   │   ├── user.go
│   │   ├── video.go
│   │   ├── stream.go
│   │   └── folder.go
│   │
│   └── repository/              # Database layer
│       ├── user.go              # User CRUD
│       ├── video.go             # Video CRUD
│       ├── stream.go            # Stream CRUD
│       └── folder.go            # Folder CRUD
│
├── templates/                   # HTML templates (7 pages)
│   ├── login.html
│   ├── signup.html
│   ├── dashboard.html
│   ├── gallery.html
│   ├── history.html
│   ├── settings.html
│   └── users.html
│
├── static/                      # Static assets
│   ├── css/
│   ├── js/
│   └── images/
│
└── bin/
    └── backend                  # Compiled binary
```

### 2.2 Frontend Architecture

**Template Engine:** Go HTML Templates  
**Rendering:** Server-side (Go templates)  
**Client-side:** Vanilla JavaScript (no framework)  
**Real-time Updates:** setTimeout + fetch API (1s polling)

**Page Structure:**
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{{ .Title }} - StreamFlow</title>
  <!-- Tabler Icons CDN -->
  <!-- Inline CSS (dark theme) -->
</head>
<body class="bg-gray-900 text-gray-100">
  <!-- Sidebar Navigation -->
  <aside class="fixed left-0 top-0 h-screen w-64">
    <!-- Nav items -->
  </aside>
  
  <!-- Main Content -->
  <main class="ml-64 p-6">
    <!-- Page-specific content -->
  </main>
  
  <!-- Inline JavaScript -->
  <script>
    // API calls via fetch
    // Real-time updates via setTimeout
    // Event handlers
  </script>
</body>
</html>
```

---

## 3. Infrastructure & Deployment

### 3.1 Server Configuration

**Operating System:** Ubuntu 20.04 LTS  
**VPS Provider:** OrangeVPS  
**Public IP:** 104.234.26.223  
**Public Port:** 7575 (HTTPS)  
**Internal Port:** 8100 (Backend)  
**Database Port:** 5432 (PostgreSQL)  

### 3.2 Service Management

**Backend Service:** `streamflow-backend.service`
```ini
[Unit]
Description=StreamFlow Backend (Go)
After=network.target postgresql.service

[Service]
Type=simple
User=root
WorkingDirectory=/var/www/streaming-go
ExecStart=/var/www/streaming-go/bin/backend
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

**Commands:**
```bash
systemctl start streamflow-backend
systemctl stop streamflow-backend
systemctl restart streamflow-backend
systemctl status streamflow-backend
journalctl -u streamflow-backend -f  # View logs
```

### 3.3 Nginx Configuration

**Config File:** `/etc/nginx/sites-available/streaming`

```nginx
server {
    listen 7575 ssl;
    server_name 104.234.26.223;
    
    # SSL Configuration
    ssl_certificate /etc/letsencrypt/live/domain/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/domain/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    
    # Upload Configuration
    client_max_body_size 0;  # Unlimited upload
    proxy_request_buffering off;  # Stream uploads
    
    # Proxy to Go backend
    location / {
        proxy_pass http://localhost:8100;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        
        # Timeouts for long uploads
        proxy_connect_timeout 600s;
        proxy_send_timeout 600s;
        proxy_read_timeout 600s;
    }
}
```

### 3.4 Database Configuration

**PostgreSQL 14.23**

```yaml
Host: localhost
Port: 5432
Database: streamflow
User: streamflow_user
Password: [from .env]
Max Connections: 25
SSL Mode: disable (local)
```

**Connection String:**
```
postgresql://streamflow_user:password@localhost:5432/streamflow?sslmode=disable
```

---

## 4. Data Flow Diagrams

### 4.1 Authentication Flow

```
User → Browser
  ↓ POST /api/auth/login
  ↓ {username, password}
Backend (auth.Handler)
  ↓ Verify credentials
Database (users table)
  ↓ User found? ✓
Backend
  ↓ Create session
  ↓ Set secure cookie
  ↓ Return {success: true, user: {...}}
Browser
  ↓ Store cookie
  ↓ Redirect to /dashboard
User → Dashboard (authenticated)
```

### 4.2 Video Upload Flow (Chunked)

```
User → Browser
  ↓ Select file (e.g., 5GB video)
  ↓ POST /api/videos/chunk/init
  ↓ {fileName, totalSize, chunkSize}
Backend (chunkedHandler.InitUpload)
  ↓ Generate uploadId
  ↓ Create session {uploadId, totalChunks, uploadedChunks: {}}
  ↓ Return {uploadId, totalChunks}
Browser
  ↓ Split file into chunks (1MB each)
  ↓ Loop: for each chunk
  ↓   POST /api/videos/chunk/upload?uploadId=X&chunkIndex=N
  ↓   Body: binary chunk data
Backend (chunkedHandler.UploadChunk)
  ↓ Read chunk via io.ReadAll(c.Request().Body)
  ↓ Write to disk: uploads/chunks/{uploadId}/chunk_{N}
  ↓ Mark chunk as uploaded: session.UploadedChunks[N] = true
  ↓ Return {success: true}
Browser
  ↓ Check: all chunks uploaded?
  ↓ POST /api/videos/chunk/complete?uploadId=X
Backend (chunkedHandler.CompleteUpload)
  ↓ Verify all chunks present
  ↓ Merge chunks into final file
  ↓ Save metadata to database (videos table)
  ↓ Delete chunk files
  ↓ Return {success: true, videoId}
Browser
  ↓ Show success
  ↓ Redirect to /gallery
```

### 4.3 Real-time Dashboard Updates

```
Browser (dashboard.html)
  ↓ Page load
  ↓ Call loadDashboard()
  ↓
  ↓ GET /api/system/stats
Backend (SystemStatsHandler)
  ↓ Read CPU usage (gopsutil)
  ↓ Read memory stats
  ↓ Read disk stats (syscall.Statfs)
  ↓ Read network stats (/proc/net/dev)
  ↓ Return {cpu, memory_used, memory_total, disk_used, disk_total, upload_speed, download_speed}
Browser
  ↓ Parse JSON response
  ↓ Update DOM elements:
  ↓   - document.getElementById('cpu-usage').textContent = cpu
  ↓   - document.getElementById('memory-usage').textContent = memUsedMB
  ↓   - document.getElementById('disk-used').textContent = diskUsedGB
  ↓   - Update progress bars (smooth CSS transitions)
  ↓
  ↓ setInterval(loadSystemStats, 1000)  # Micro-cached 900ms server-side to prevent flickering
  ↓ ↻ Loop continues
```

---

## 5. Security Architecture

### 5.1 Authentication & Authorization

**Session Management:**
- Secure cookie-based sessions
- HTTPOnly flag enabled (prevents XSS)
- SameSite: Lax (CSRF protection)
- Expiration: 3 days (72 hours)

**Password Security:**
- bcrypt hashing (cost factor 10)
- Salted automatically
- Never stored in plaintext

**Role-Based Access Control:**
```go
Roles:
  - admin: Full access (user management, system settings)
  - member: Standard access (own content only)

Middleware:
  - RequireAuth(): Verify session exists
  - RequireAdmin(): Verify user.Role == "admin"
```

### 5.2 API Security

**Input Validation:**
- Request body size limits (configurable)
- Content-Type validation
- Parameter type checking
- SQL injection prevention (parameterized queries)

**Output Sanitization:**
- HTML template escaping (automatic)
- JSON encoding (automatic)
- No raw SQL queries exposed

**Rate Limiting:**
- (To be implemented in v3.1)
- Nginx level: connections/second limit
- Application level: per-user API quotas

### 5.3 Network Security

**SSL/TLS:**
- Let's Encrypt certificates
- TLS 1.2 minimum (1.3 preferred)
- Strong cipher suites
- HSTS enabled (planned)

**Firewall:**
- UFW enabled
- Open ports: 22 (SSH), 7575 (HTTPS)
- Internal services: 5432 (PostgreSQL), 8100 (Backend)

---

## 6. Performance Optimization

### 6.1 Go Backend Advantages

**vs Node.js:**
- ✅ **5-10x faster API response** (compiled vs interpreted)
- ✅ **60-70% CPU reduction** (efficient runtime)
- ✅ **40% lower memory** (garbage collection)
- ✅ **Native concurrency** (goroutines vs event loop)

**Echo Framework Benefits:**
- High-performance router (radix tree)
- Built-in middleware
- Zero-allocation JSON encoding
- HTTP/2 support

### 6.2 Database Optimization

**Connection Pooling:**
```go
MaxIdleConns: 10
MaxOpenConns: 25
ConnMaxLifetime: 5 minutes
```

**Query Optimization:**
- Indexed columns (user_id, video_id, stream_id)
- Prepared statements
- Batch inserts where applicable

### 6.3 Caching Strategy

**Current (v3.0):**
- No application-level caching (not needed yet)
- Browser cache for static assets (images, CSS, JS)
- PostgreSQL query cache

**Future (v3.1+):**
- Redis for session storage
- In-memory cache for frequent queries
- CDN for static assets

---

## 7. Monitoring & Observability

### 7.1 Real-time Metrics

**Dashboard Stats (1-second refresh):**
- CPU usage percentage
- Memory used/total (MB)
- Disk used/total (GB)
- Network upload/download (Mbps)

**Implementation:**
```go
// CPU
cpu, _ := cpu.Percent(time.Second, false)

// Memory
vmem, _ := mem.VirtualMemory()
used := vmem.Used
total := vmem.Total

// Disk
var stat syscall.Statfs_t
syscall.Statfs("/", &stat)
diskUsed := stat.Blocks - stat.Bfree
diskTotal := stat.Blocks

// Network
// Parse /proc/net/dev for interface stats
```

### 7.2 Logging

**Log Levels:**
- INFO: Normal operations
- WARN: Non-critical issues (replaced console.error)
- ERROR: Critical failures

**Log Destinations:**
- SystemD journal (`journalctl -u streamflow-backend`)
- Echo framework logger (stdout)

**Log Format:**
```json
{
  "time": "2026-07-16T03:14:52Z",
  "level": "info",
  "method": "GET",
  "uri": "/api/system/stats",
  "status": 200,
  "latency": "2.345ms"
}
```

### 7.3 Health Checks

**Endpoints:**
- `/api/server-time` - Basic health (no auth)
- `/api/system/stats` - Detailed health (auth required)

**Systemd Monitoring:**
- Automatic restart on failure
- 5-second restart delay
- Failure alerting (via journalctl)

---

## 8. Scalability Considerations

### 8.1 Current Architecture (Monolithic)

**Pros:**
- Simple deployment
- Low latency (no network hops)
- Easy debugging
- Cost-effective

**Cons:**
- Single point of failure
- Vertical scaling only
- Resource contention possible

### 8.2 Future Scaling Path

**Horizontal Scaling (v4.0+):**
```
Load Balancer (Nginx)
  ├── Backend Instance 1 (Go)
  ├── Backend Instance 2 (Go)
  └── Backend Instance 3 (Go)
        ↓
  Shared PostgreSQL (Primary)
        ↓
  PostgreSQL Replicas (Read-only)
        ↓
  Shared Storage (NFS / S3)
```

**Database Scaling:**
- PostgreSQL replication (streaming replication)
- Read replicas for queries
- PgBouncer for connection pooling
- Partitioning for large tables

**Storage Scaling:**
- Move to object storage (S3, MinIO)
- CDN for media delivery
- Distributed filesystem (GlusterFS, Ceph)

---

## 9. Disaster Recovery

### 9.1 Backup Strategy

**Database Backups:**
- Automated daily backups (pg_dump)
- Retention: 7 days
- Storage: Local disk + remote backup

**File Backups:**
- Weekly full backup (uploads/)
- Incremental daily backups
- Off-site storage (rsync to backup server)

### 9.2 Recovery Procedures

**Database Recovery:**
```bash
# Restore from backup
psql streamflow < backup_2026-07-16.sql

# Verify data integrity
psql -d streamflow -c "SELECT COUNT(*) FROM users;"
```

**Application Recovery:**
```bash
# Stop service
systemctl stop streamflow-backend

# Restore binary
cp /backup/backend /var/www/streaming-go/bin/

# Restart service
systemctl start streamflow-backend
```

**RTO (Recovery Time Objective):** < 15 minutes  
**RPO (Recovery Point Objective):** < 24 hours  

---

## 10. Technology Stack Summary

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| **Backend Language** | Go | 1.25.0 | Application logic |
| **Web Framework** | Echo | 4.12.0 | HTTP routing |
| **Database** | PostgreSQL | 14.23 | Data persistence |
| **Reverse Proxy** | Nginx | 1.18.0 | SSL termination, routing |
| **Media Processing** | FFmpeg (AVX-512) | 6.1.1 | Custom compiled binary at `/usr/local/bin/ffmpeg` |
| **Template Engine** | Go HTML | stdlib | Server-side rendering |
| **Icon Library** | Tabler Icons | 2.x | UI icons |
| **Operating System** | Ubuntu | 20.04 LTS | Server OS |
| **Service Manager** | systemd | 245 | Process management |
| **SSL Provider** | Let's Encrypt | - | Free SSL certificates |

---

**Document Version:** 3.0.1  
**Last Updated:** 2026-07-16  
**Next Review:** 2026-08-16  
**Status:** ✅ PRODUCTION READY
