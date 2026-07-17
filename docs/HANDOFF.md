# Project Handoff Guide
**Version:** 3.0.3  
**Last Updated:** 2026-07-17  

## 1. Repository
- GitHub: `Anwarnara/streamflow`
- Branch Utama: `main`

## 2. Infrastructure Deployment
- Server: OrangeVPS (104.234.26.223)
- Service: systemd (`streamflow-backend.service`)
- Proxy: Nginx (port 7575)

## 3. Local Development
1. Clone repo.
2. Install Go 1.25.0+.
3. Setup PostgreSQL lokal.
4. Salin `.env.example` ke `.env`.
5. Jalankan `go run cmd/backend/main.go`.

*(Catatan: Jangan lampirkan file .env produksi di dokumen ini)*

