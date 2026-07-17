# Architecture Principles
**Version:** 3.0.3  
**Last Updated:** 2026-07-17  

## 1. Monolith First
Tetap pertahankan arsitektur monolith untuk kesederhanaan deployment dan operasional. Pecah menjadi microservices hanya jika resource scaling menjadi bottleneck absolut.

## 2. Stateless Backend
Go API harus sepenuhnya stateless. Semua status sesi disimpan di klien (via Secure Cookies) atau di Database, memudahkan scaling horizontal.

## 3. Asynchronous Media Processing
Proses berat seperti ekstrak thumbnail atau konversi video tidak boleh memblokir thread HTTP utama.

## 4. Bypassing Middlemen
Arsitektur dioptimalkan untuk melewati proxy L7 (seperti Cloudflare) untuk media upload/streaming menggunakan Nginx stream langsung dan chunked processing.

