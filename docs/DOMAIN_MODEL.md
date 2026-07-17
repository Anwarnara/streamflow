# Domain Model & Business Rules
**Version:** 3.0.3  
**Last Updated:** 2026-07-17  

## 1. Entitas Utama
- **User**: Pengguna sistem (Admin, Member).
- **Video**: Aset media fisik yang diunggah.
- **Folder**: Struktur organisasi logis untuk Video.
- **Stream**: Konfigurasi transmisi ke platform eksternal (memiliki Stream Key).
- **Playlist**: Kumpulan urutan Video.
- **Rotation**: Jadwal otomatisasi penyiaran Video/Playlist.

## 2. Business Rules
- **Ownership**: Member hanya dapat melihat, mengedit, dan menghapus Video/Stream miliknya sendiri.
- **Admin Privileges**: Admin dapat melihat seluruh user dan mengelola pengaturan sistem.
- **File Upload**: File besar dipecah menjadi chunks 50MB di klien, digabung di server.
- **Session**: Sesi login valid selama 72 jam (3 hari) via cookie HTTPOnly.

