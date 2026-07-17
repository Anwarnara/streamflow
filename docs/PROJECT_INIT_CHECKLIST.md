# Project Initialization Checklist
**Version:** 3.0.3  
**Last Updated:** 2026-07-17  

## Langkah Instalasi Server Baru
- [ ] Install Go 1.25.0
- [ ] Install PostgreSQL 14
- [ ] Install Nginx
- [ ] Compile/Install FFmpeg 6.1.1 (AVX-512 optimized)
- [ ] Konfigurasi `pg_hba.conf` untuk koneksi lokal
- [ ] Buat user dan database PostgreSQL (`streamflow_user`)
- [ ] Clone repository
- [ ] Build binary: `go build -o bin/backend cmd/backend/main.go`
- [ ] Setup `streamflow-backend.service` di systemd
- [ ] Setup Nginx proxy_pass config
- [ ] Setup SSL Let's Encrypt

