# Error Handling & States
**Version:** 3.0.3  
**Last Updated:** 2026-07-17  

## 1. Backend Errors
API mengembalikan HTTP Status codes standar:
- `400 Bad Request`: Input tidak valid (missing fields, format salah).
- `401 Unauthorized`: Session tidak ada atau kadaluarsa.
- `403 Forbidden`: Akses ditolak (bukan pemilik resource).
- `500 Internal Server Error`: Kesalahan DB atau sistem file.

Format standar: `{"error": "Deskripsi pesan error"}`

## 2. Frontend Error UI
- **Toast Notifications**: Error sementara (Gagal rename, gagal delete).
- **Fallback Icons**: Jika thumbnail gagal diload/generate, tampilkan ikon `<i class="ti ti-video">`.
- **System Stats Zero-Flicker**: Backend menggunakan Mutex cache (900ms) agar API tidak merespons `0` saat data sedang dibaca.

