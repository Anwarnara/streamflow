# Permissions & Access Control

**Version:** 3.3.0
**Last Updated:** 2026-07-19

## Overview

StreamFlow uses a simple role-based access control model. In the current single-tenant deployment, all registered users carry the `admin` role by default. The `user_role` column in the `users` table is the basis for future role expansion.

---

## Current Roles

| Role | Description |
|------|-------------|
| `admin` | Full access to all features: create/edit/delete streams, videos, users, playlists, and destinations. |

---

## Authentication

- Login via `POST /api/auth/login` returns a session token.
- Tokens are stored in the `user_sessions` table with a 72-hour expiry.
- Every authenticated API request must carry the session token in the `Authorization` header or as a cookie (depending on frontend implementation).
- Sessions are invalidated on logout (`POST /api/auth/logout`) or automatic expiry.

---

## Resource Ownership

Even within the `admin` role, resources have a `user_id` FK linking them to their creator. Future multi-user or multi-tenant expansion would use this column to scope access.

| Resource | Owner Column |
|----------|-------------|
| `videos` | `user_id` |
| `streams` | `user_id` |
| `playlists` | `user_id` |
| `media_folders` | `user_id` |

---

## RTMP Ingest Authorization

- When a client connects to Nginx-RTMP on port `:1935`, Nginx calls `POST /api/rtmp/auth` as an `on_publish` callback.
- The backend validates the `stream_key` against the `streams` table.
- If the key is not found or the stream is not in an expected state, the auth endpoint returns a non-200 status, and Nginx drops the connection.

---

## Sensitive Data Access Rules

| Data | Rule |
|------|------|
| Passwords | Stored as bcrypt hash. Never returned in any API response. |
| YouTube OAuth tokens | Never returned in list/get user API responses. Used server-side only. |
| RTMP stream keys | Not logged at INFO level. Truncated in UI display. |
| Session tokens | Stored in DB; never logged. |

---

## Planned Future Roles

If multi-tenant support is added (see `IDEAS.md`), the following role hierarchy is proposed:

| Role | Permissions |
|------|------------|
| `viewer` | Read-only: watch streams, view stream health |
| `operator` | Create and manage own streams/videos; cannot manage users |
| `admin` | Full platform access including user management |
| `superadmin` | Cross-organization access (multi-tenant only) |
