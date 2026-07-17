# Architecture Principles
**Version:** 3.1.0  
**Last Updated:** 2026-07-17  

## 1. Bypassing Middlemen
Sistem dibangun untuk melewarkan batasan proxy Layer 7.
- Upload VOD besar menggunakan Chunked processing.
- Input streaming menggunakan Nginx-RTMP TCP langsung (Port 1935).

## 2. Zero-CPU Copy Muxing
Jika tidak ada watermark yang di-request user, operasi Simulcast FFmpeg harus dijalankan murni menggunakan codec `copy`, sehingga server hanya bertindak sebagai Network Router, bukan transcoder.

