# AI Agent Coding Rules
**Version:** 3.0.3  
**Last Updated:** 2026-07-17  

## 1. Tech Stack Preferences
- Gunakan Go standard library + Echo framework untuk API.
- Hindari ORM berat (seperti GORM) kecuali sangat diperlukan; raw SQL / `database/sql` lebih diutamakan untuk kontrol performa.
- Vanilla JavaScript di frontend, tanpa framework (React/Vue) untuk menjaga ukuran bundel tetap minimal.

## 2. UI/UX Rules
- Tidak boleh ada page flicker. Selalu perbarui elemen DOM secara spesifik (via `textContent` atau modifikasi class list) alih-alih `innerHTML`.
- Pertahankan dukungan mobile (flex-col, padding responsif).
- Gunakan class utility Tailwind secara konsisten.

## 3. Operasional Agent
- Wajib memperbarui `CHANGELOG.md` dan dokumentasi terkait segera setelah fitur diimplementasikan.
- Jangan commit ke Git kecuali diperintahkan secara spesifik.

