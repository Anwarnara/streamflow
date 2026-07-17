# AI Agent Coding Rules
**Version:** 3.1.0  
**Last Updated:** 2026-07-17  

## 1. Go Best Practices
- Backend API sepenuhnya mengendalikan FFmpeg via `os/exec`. Jangan gunakan lib CGo FFmpeg untuk menghindari segfault memory leak.
- Semua API Endpoint yang meng-update DOM secara masif (seperti chat atau stats) wajib menggunakan sistem parsial (`textContent`, SSE).

## 2. Documentation
- Setiap fitur baru wajib direfleksikan ke 20 file dokumentasi ini.

