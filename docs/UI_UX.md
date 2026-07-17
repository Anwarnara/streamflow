# UI/UX Design System
**Version:** 3.0.3  
**Last Updated:** 2026-07-17  

## 1. Design Philosophy
- **Dark Theme Default**: Mengurangi eye strain untuk monitoring berjam-jam.
- **No-Flicker Updates**: Update elemen UI secara parsial via `textContent` daripada membangun ulang DOM (menghindari layout shift).

## 2. Color Palette
- **Background**: `bg-gray-900`
- **Cards/Containers**: `bg-dark-700`, `bg-dark-800`
- **Accents**: `text-primary` (brand color), `text-red-400` (destructive actions)

## 3. Components
- **3-Dot Menus**: Menggunakan absolute positioning `bottom-8 z-50` agar membuka ke atas dan tidak tertutup footer.
- **Progress Bars**: Menggunakan CSS `transition-all duration-300` agar smooth.
- **Icons**: Menggunakan Tabler Icons.

