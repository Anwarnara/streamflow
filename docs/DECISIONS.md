# Architecture Decision Records (ADR)
**Version:** 3.0.3  
**Last Updated:** 2026-07-17  

## ADR-001: Migrasi dari Node.js ke Go
**Konteks**: Konsumsi RAM Node.js terlalu tinggi untuk VPS berkapasitas kecil saat melayani stream.
**Keputusan**: Rewrite backend menggunakan Go (Echo framework).
**Hasil**: Penggunaan CPU turun 60%, RAM turun 40%, API latency turun drastis.

## ADR-002: Chunked Upload vs Standard Upload
**Konteks**: Nginx dan Cloudflare memiliki batasan request body size (Cloudflare free limit: 100MB).
**Keputusan**: Implementasi upload per-potongan (chunk) sebesar 50MB di frontend JavaScript, dirakit kembali oleh Go.
**Hasil**: Bisa mengunggah file video berukuran GB tanpa terblokir WAF.

## ADR-003: UI Dropdown Positioning
**Konteks**: Menu 3-titik di galeri terpotong oleh bar bawah di layar kecil.
**Keputusan**: Mengubah absolute positioning dari `top-8` ke `bottom-8` dan menaikkan z-index ke `z-50`.

