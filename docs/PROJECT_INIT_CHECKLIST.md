# Project Initialization Checklist

**Version:** 3.3.0
**Last Updated:** 2026-07-19

## Purpose

Use this checklist when setting up a new StreamFlow deployment from scratch, or when onboarding to an existing deployment.

---

## 1. System Prerequisites

- [ ] Linux host (Ubuntu 22.04 LTS or equivalent)
- [ ] Go 1.25 installed (`go version`)
- [ ] PostgreSQL 14 installed and running
- [ ] Nginx installed with RTMP module (`nginx -V 2>&1 | grep rtmp`)
- [ ] FFmpeg custom AVX-512 build placed at `/usr/local/bin/ffmpeg` (`ffmpeg -version`)
- [ ] systemd available

---

## 2. Repository & Build

- [ ] Source code at `/var/www/streaming/streaming-go/`
- [ ] `go mod tidy` to fetch dependencies
- [ ] Build: `go build -o bin/backend ./cmd/backend/`
- [ ] Binary present: `/var/www/streaming/streaming-go/bin/backend`

---

## 3. Database Setup

- [ ] PostgreSQL database `streamflow` created
- [ ] Database user with read/write privileges configured
- [ ] All 14 tables present (verify via `\dt` in `psql streamflow`):
  - [ ] `users`
  - [ ] `videos`
  - [ ] `streams`
  - [ ] `stream_destinations`
  - [ ] `media_folders`
  - [ ] `playlists`
  - [ ] `playlist_items`
  - [ ] `playlist_videos`
  - [ ] `playlist_audios`
  - [ ] `stream_analytics`
  - [ ] `stream_chats`
  - [ ] `stream_history`
  - [ ] `stream_playlist_items`
  - [ ] `user_sessions`
- [ ] v3.3.0 migration applied: `stream_destinations` has `is_active`, `bitrate`, `resolution`, `fps`, `video_codec`, `preset` columns

---

## 4. Nginx Configuration

- [ ] Nginx reverse proxy configured to forward `:7575` → `:8100`
- [ ] Nginx-RTMP configured to listen on `:1935`
- [ ] `on_publish` auth callback set to `http://127.0.0.1:8100/api/rtmp/auth`
- [ ] Nginx reloaded: `nginx -s reload`

---

## 5. systemd Service

- [ ] `streamflow-backend.service` unit file installed (typically in `/etc/systemd/system/`)
- [ ] `systemctl daemon-reload`
- [ ] `systemctl enable streamflow-backend.service`
- [ ] `systemctl start streamflow-backend.service`
- [ ] `systemctl status streamflow-backend.service` shows `active (running)`

---

## 6. Application Verification

- [ ] Web UI accessible at `http://<host>:7575`
- [ ] API responds: `curl http://localhost:8100/api/auth/me`
- [ ] At least one admin user created
- [ ] Test RTMP stream key auth: connect with OBS to `rtmp://<host>:1935/live/<key>`
- [ ] Upload a test video and verify thumbnail generation
- [ ] Create a test stream, start it, verify health metrics appear on `/stream-health`

---

## 7. Documentation

- [ ] All docs in `/var/www/streaming/docs/` are up to date (check `**Version:** 3.3.0`)
- [ ] `WORKLOG.md` has a deployment entry
- [ ] `HANDOFF.md` reviewed and current

---

## 8. Post-Deployment

- [ ] Log monitoring set up: `journalctl -u streamflow-backend.service -f`
- [ ] Disk space checked for video upload storage
- [ ] Backups configured for PostgreSQL `streamflow` database
- [ ] Firewall rules: `:7575` (HTTP), `:1935` (RTMP) open; `:8100` restricted to localhost/proxy only
