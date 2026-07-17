# RBAC & Permissions Matrix
**Version:** 3.0.3  
**Last Updated:** 2026-07-17  

## 1. Role Definitions
- **Admin**: Akses penuh ke seluruh fitur dan data sistem.
- **Member**: Akses terbatas pada entitas yang dimiliki (user_id match).

## 2. Permission Matrix
| Fitur / Endpoint | Admin | Member |
|------------------|-------|--------|
| Login/Signup     | Yes   | Yes    |
| Upload Video     | Yes   | Yes    |
| Edit Any Video   | Yes   | No     |
| Edit Own Video   | Yes   | Yes    |
| User Management  | Yes   | No     |
| System Stats     | Yes   | Yes    |

## 3. Implementation
Diimplementasikan via middleware `RequireAuth` dan pengecekan tingkat handler (`if video.UserID != userID`).

