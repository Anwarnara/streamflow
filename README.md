# StreamFlow

**Professional Video Streaming Platform**

A high-performance video streaming platform built with **Go microservices**, PostgreSQL, and FFmpeg. Stream to multiple platforms simultaneously (YouTube, Twitch, TikTok, Facebook) with real-time monitoring and analytics.

[![Version](https://img.shields.io/badge/version-3.0.0-blue.svg)](https://github.com/yourusername/streamflow)
[![Backend](https://img.shields.io/badge/backend-Go%201.22-00ADD8.svg)](https://golang.org/)
[![Database](https://img.shields.io/badge/database-PostgreSQL%2014-336791.svg)](https://www.postgresql.org/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE.md)

---

## ✨ Features

### Core Functionality
- 🎥 **Multi-Platform Streaming** - YouTube, Twitch, TikTok, Facebook, Instagram
- 📹 **Video Management** - Upload, organize, and manage video library
- 🔄 **Loop Streaming** - 24/7 continuous streaming with video loops
- 📊 **Real-Time Analytics** - Monitor stream health, bitrate, CPU usage
- 🎛️ **FFmpeg Control** - Full control over encoding parameters
- 📱 **Responsive Design** - Works on desktop, tablet, and mobile

### Technical Features
- ⚡ **High Performance** - Go backend with 5-10x faster response times
- 🏗️ **Microservices** - 5 independent services for scalability
- 💾 **PostgreSQL** - ACID-compliant database with advanced indexing
- 🔐 **Secure Auth** - Bcrypt password hashing, session management
- 🚀 **Production Ready** - Systemd services with auto-restart
- 📈 **Scalable** - Handle 30-60 concurrent streams on 4-core VPS

---

## 🚀 Quick Start

### Prerequisites
- Ubuntu 22.04 (or similar Linux)
- Go 1.22.5+
- PostgreSQL 14+
- FFmpeg 6.1+

### Installation

```bash
# 1. Clone repository
git clone https://github.com/yourusername/streamflow.git
cd streamflow

# 2. Install dependencies
cd streaming-go
go mod download

# 3. Setup PostgreSQL
sudo -u postgres psql
CREATE DATABASE streamflow_db;
CREATE USER streamflow_user WITH ENCRYPTED PASSWORD 'your_password';
GRANT ALL PRIVILEGES ON DATABASE streamflow_db TO streamflow_user;
\q

# 4. Configure environment
cp .env.example .env
# Edit .env with your database credentials

# 5. Build services
./scripts/build-all.sh

# 6. Install systemd services
sudo ./scripts/install-services.sh

# 7. Start services
sudo systemctl start streamflow-backend health-monitor analytics upload-queue api-gateway

# 8. Verify
curl http://localhost:8100/api/health
```

### Access

Open your browser and navigate to:
- **Application:** http://localhost:8100
- **Login:** http://localhost:8100/login
- **Dashboard:** http://localhost:8100/dashboard

Default credentials (first run):
- Username: `admin`
- Password: `admin123` (change immediately!)

---

## 📖 Architecture

### System Overview

```
┌─────────────────────────────────────────┐
│         Web Browser (Client)            │
└─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────┐
│     Go Backend (Port 8100)              │
│  Auth • Media • Streaming • Templates   │
└─────────────────────────────────────────┘
        │           │           │
        ▼           ▼           ▼
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Health   │  │Analytics │  │  Upload  │
│ Monitor  │  │ Service  │  │  Queue   │
│  :9001   │  │  :9002   │  │  :9003   │
└──────────┘  └──────────┘  └──────────┘
        │           │           │
        └───────────┼───────────┘
                    ▼
          ┌──────────────────┐
          │   PostgreSQL     │
          │   Port 5432      │
          └──────────────────┘
```

### Components

- **Main Backend** (8100) - HTTP server, auth, media, streaming control
- **Health Monitor** (9001) - FFmpeg process monitoring, metrics
- **Analytics** (9002) - Performance tracking, reporting
- **Upload Queue** (9003) - Video processing, thumbnail generation
- **API Gateway** (8080) - Service routing, load balancing
- **PostgreSQL** (5432) - Persistent data storage

See [TECH_System_Architecture.md](docs/TECH_System_Architecture.md) for details.

---

## 💻 Technology Stack

### Backend
- **Language:** Go 1.22.5
- **Framework:** Echo v4.12.0
- **Database:** PostgreSQL 14.23
- **Templates:** html/template

### Media Processing
- **FFmpeg:** 6.1.1 with AVX-512
- **Formats:** MP4, FLV, RTMP
- **Codecs:** H.264, AAC

### Frontend
- **HTML/CSS:** Tailwind CSS
- **JavaScript:** Vanilla JS
- **PWA:** Service Worker enabled

### Infrastructure
- **Deployment:** Systemd services
- **OS:** Ubuntu 22.04
- **Logging:** journalctl

---

## 📚 Documentation

### User Guides
- [Getting Started](docs/USER_GUIDE.md) - First-time setup
- [Creating Streams](docs/STREAMING_GUIDE.md) - How to stream
- [Managing Videos](docs/MEDIA_GUIDE.md) - Upload and organize

### Technical Docs
- [System Architecture](docs/TECH_System_Architecture.md) - Architecture overview
- [API Reference](docs/API_REFERENCE.md) - API documentation
- [Database Schema](docs/TECH_Database_Design.md) - Database design
- [Migration Guide](docs/MIGRATION_COMPLETE.md) - Go migration details

### Operations
- [Deployment Guide](docs/DEPLOYMENT.md) - Production deployment
- [Monitoring](docs/MONITORING.md) - Health checks & metrics
- [Troubleshooting](docs/TROUBLESHOOTING.md) - Common issues

---

## 🔧 Configuration

### Environment Variables

Create `.env` file in project root:

```bash
# Database
DATABASE_URL=postgresql://streamflow_user:password@localhost:5432/streamflow_db

# Server
PORT=8100
ENVIRONMENT=production

# Session
SESSION_SECRET=your-random-secret-key
SESSION_EXPIRY=24h

# FFmpeg
FFMPEG_PATH=/usr/bin/ffmpeg
FFMPEG_THREADS=4

# Upload
MAX_UPLOAD_SIZE=2GB
UPLOAD_DIR=/var/www/streaming/uploads
```

### Systemd Services

Services are configured in `/etc/systemd/system/`:
- `streamflow-backend.service`
- `health-monitor.service`
- `analytics.service`
- `upload-queue.service`
- `api-gateway.service`

All services auto-start on boot and auto-restart on failure.

---

## 🎯 Usage Examples

### Start a Stream

```bash
# Via API
curl -X POST http://localhost:8100/api/streams \
  -H "Content-Type: application/json" \
  -d '{
    "title": "My Live Stream",
    "video_id": "video-uuid",
    "rtmp_url": "rtmp://a.rtmp.youtube.com/live2",
    "stream_key": "your-stream-key",
    "bitrate": 2500,
    "fps": 30
  }'

# Start streaming
curl -X POST http://localhost:8100/api/streams/{stream-id}/start
```

### Upload Video

```bash
curl -X POST http://localhost:8100/api/videos/upload \
  -F "title=My Video" \
  -F "video=@/path/to/video.mp4"
```

### Check Health

```bash
# Backend
curl http://localhost:8100/api/health

# Microservices
curl http://localhost:9001/ping
curl http://localhost:9002/ping
curl http://localhost:9003/ping
```

---

## 🔍 Monitoring

### Service Status

```bash
# Check all services
systemctl status streamflow-backend health-monitor analytics upload-queue api-gateway

# View logs
journalctl -u streamflow-backend -f
```

### Health Checks

All services expose health endpoints:
- Backend: `http://localhost:8100/api/health`
- Health Monitor: `http://localhost:9001/ping`
- Analytics: `http://localhost:9002/ping`
- Upload Queue: `http://localhost:9003/ping`
- API Gateway: `http://localhost:8080/ping`

---

## 🛠️ Development

### Build from Source

```bash
cd /var/www/streaming-go

# Build main backend
go build -o bin/backend ./cmd/backend

# Build microservices
go build -o bin/health-monitor ./cmd/health-monitor
go build -o bin/analytics ./cmd/analytics
go build -o bin/upload-queue ./cmd/upload-queue
go build -o bin/api-gateway ./cmd/api-gateway

# Run locally
PORT=8100 ./bin/backend
```

### Run Tests

```bash
# Unit tests
go test ./...

# Integration tests
./scripts/integration-test.sh

# Load tests
./scripts/load-test.sh
```

### Hot Reload (Development)

```bash
# Install air
go install github.com/cosmtrek/air@latest

# Run with hot reload
air
```

---

## 📊 Performance

### Benchmarks (vs Node.js v2.2.2)

| Metric | Node.js | Go | Improvement |
|--------|---------|-----|-------------|
| API Latency | 80-150ms | 13-21ms | **5-10x faster** |
| CPU Usage (10 streams) | 65% | 22% | **66% reduction** |
| Memory Usage | 380MB | 180MB | **53% reduction** |
| Concurrent Streams | 15-20 | 30-60 | **2-3x capacity** |

### Capacity

- **4-core VPS:** 30-60 concurrent streams
- **8-core Server:** 100+ concurrent streams
- **Database:** 1000+ concurrent connections
- **API Throughput:** 5000+ req/sec

---

## 🔐 Security

### Authentication
- Bcrypt password hashing (cost 10)
- HTTP-only session cookies
- CSRF protection (planned)
- Rate limiting (planned)

### Authorization
- Role-based access control
- User ownership verification
- Admin-only endpoints

### Database
- Parameterized queries (no SQL injection)
- Connection pooling with limits
- UUID primary keys (no enumeration)

### Best Practices
- Keep `.env` file secure (never commit)
- Change default passwords immediately
- Use HTTPS in production (Nginx reverse proxy)
- Regular security updates

---

## 🤝 Contributing

Contributions are welcome! Please follow these guidelines:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

---

## 📜 Changelog

See [CHANGELOG.md](docs/CHANGELOG.md) for version history.

### Current Version: 3.0.0 (July 16, 2026)
- ✅ Complete Go backend rewrite
- ✅ Microservices architecture
- ✅ PostgreSQL database
- ✅ 5-10x performance improvement
- ✅ Node.js fully removed

---

## 📄 License

This project is licensed under the MIT License - see [LICENSE.md](LICENSE.md) for details.

---

## 🙏 Acknowledgments

- **Go Team** - Amazing language and ecosystem
- **Echo Framework** - Fast and elegant web framework
- **PostgreSQL** - Robust and scalable database
- **FFmpeg** - Powerful media processing
- **Tailwind CSS** - Beautiful and responsive design

---

## 📞 Support

### Issues & Bugs
- Report on [GitHub Issues](https://github.com/yourusername/streamflow/issues)
- Check [Troubleshooting Guide](docs/TROUBLESHOOTING.md)

### Questions
- Ask on [Discussions](https://github.com/yourusername/streamflow/discussions)
- Email: support@streamflow.example

### Community
- Discord: [Join our server](https://discord.gg/streamflow)
- Twitter: [@streamflow](https://twitter.com/streamflow)

---

## 🗺️ Roadmap

### v3.1.0 (Q3 2026)
- [ ] Prometheus metrics export
- [ ] Grafana dashboards
- [ ] Real-time viewer count
- [ ] Multi-destination streaming

### v3.2.0 (Q4 2026)
- [ ] Cloud storage (S3, GCS)
- [ ] Advanced scheduling
- [ ] Video transcoding
- [ ] Mobile app (React Native)

### v4.0.0 (2027)
- [ ] Kubernetes deployment
- [ ] Distributed architecture
- [ ] WebRTC support
- [ ] Live chat integration

---

## ⭐ Star History

If you find StreamFlow useful, please consider giving it a star! ⭐

---

**Built with ❤️ using Go**

**Version:** 3.0.0  
**Last Updated:** July 16, 2026  
**Status:** Production & Stable
