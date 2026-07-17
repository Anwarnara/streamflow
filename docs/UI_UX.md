# UI/UX Design System
**Version:** 3.1.0  
**Last Updated:** 2026-07-17  

## 1. Design Philosophy
- **Dark Theme Default**: Mengurangi eye strain untuk monitoring berjam-jam.
- **No-Flicker Updates**: Update parsial via `textContent` (contoh: SSE Chat dan Hardware Stats).

## 2. Components
- **3-Dot Menus**: Menggunakan absolute positioning `bottom-8 z-50` agar membuka ke atas.
- **Chat Box**: Auto-scroll dengan animasi `.animate-fade-in` pada setiap chat baru yang masuk via SSE.
- **Progress Bars**: CSS `transition-all duration-300`.
- **Select Video Dropdown**: Render aman menggunakan DOM API native (bukan innerHTML string literal) untuk mencegah XSS/Syntax Error dari nama file, dilengkapi thumbnail dan file size.

