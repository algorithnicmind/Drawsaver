# DrawSaver — API Documentation

> **Version:** 1.0  
> **Last Updated:** 2026-05-27  
> **Base URL:** `https://api.drawsaver.app/api/v1`  

---

## Authentication

All protected endpoints require a JWT access token in the `Authorization` header:
```
Authorization: Bearer <access_token>
```

Tokens expire after 15 minutes. Use the refresh endpoint to obtain new tokens.

---

## 1. Authentication APIs

### POST `/auth/register`

Register a new user with email and password.

**Request:**
```json
{
  "username": "johndoe",
  "email": "john@example.com",
  "password": "SecurePass123!",
  "displayName": "John Doe"
}
```

**Validation Rules:**
- `username`: 3-30 chars, alphanumeric + `_` `-`, unique
- `email`: Valid email format, unique
- `password`: Min 8 chars, at least 1 uppercase, 1 lowercase, 1 number

**Response `201`:**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "665abc123def456",
      "username": "johndoe",
      "email": "john@example.com",
      "displayName": "John Doe",
      "avatar": null,
      "emailVerified": false
    },
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "expiresIn": 900
  },
  "message": "Registration successful. Please verify your email."
}
```
Set-Cookie: `refreshToken=<token>; HttpOnly; Secure; SameSite=Strict; Max-Age=604800`

**Errors:**
| Status | Code | Description |
|--------|------|-------------|
| 400 | `VALIDATION_ERROR` | Invalid input fields |
| 409 | `EMAIL_EXISTS` | Email already registered |
| 409 | `USERNAME_EXISTS` | Username already taken |

---

### POST `/auth/login`

**Request:**
```json
{
  "email": "john@example.com",
  "password": "SecurePass123!"
}
```

**Response `200`:**
```json
{
  "success": true,
  "data": {
    "user": { "id": "...", "username": "...", "email": "...", "avatar": "..." },
    "accessToken": "eyJ...",
    "expiresIn": 900
  }
}
```

**Errors:**
| Status | Code | Description |
|--------|------|-------------|
| 401 | `INVALID_CREDENTIALS` | Wrong email or password |
| 403 | `EMAIL_NOT_VERIFIED` | Email not yet verified |

---

### POST `/auth/refresh`

Exchange refresh token (from cookie) for new access + refresh tokens.

**Request:** No body. Refresh token read from `HttpOnly` cookie.

**Response `200`:**
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJ...",
    "expiresIn": 900
  }
}
```
Set-Cookie: New `refreshToken` cookie (token rotation).

---

### POST `/auth/logout`

Invalidate refresh token.

**Response `200`:**
```json
{ "success": true, "message": "Logged out successfully" }
```
Set-Cookie: Clear `refreshToken` cookie.

---

### GET `/auth/google`
Redirect to Google OAuth consent screen. Returns `302` redirect.

### GET `/auth/google/callback`
OAuth callback. On success, redirects to frontend with tokens:
```
https://drawsaver.app/auth/callback?token=<accessToken>
```

### GET `/auth/github`
Same pattern as Google OAuth.

### GET `/auth/github/callback`
Same pattern as Google OAuth callback.

---

### POST `/auth/forgot-password`

**Request:**
```json
{ "email": "john@example.com" }
```

**Response `200`:** (always returns 200 to prevent email enumeration)
```json
{ "success": true, "message": "If the email exists, a reset link has been sent." }
```

---

### POST `/auth/reset-password`

**Request:**
```json
{
  "token": "reset-token-from-email-link",
  "newPassword": "NewSecurePass456!"
}
```

**Response `200`:**
```json
{ "success": true, "message": "Password reset successful." }
```

---

## 2. User APIs

### GET `/users/me` 🔒

Get current authenticated user's profile.

**Response `200`:**
```json
{
  "success": true,
  "data": {
    "id": "665abc123def456",
    "username": "johndoe",
    "email": "john@example.com",
    "displayName": "John Doe",
    "avatar": "https://s3.../avatars/665abc123.jpg",
    "emailVerified": true,
    "createdAt": "2026-05-20T10:30:00Z"
  }
}
```

### PATCH `/users/me` 🔒

Update profile. Supports `multipart/form-data` for avatar upload.

**Request (JSON):**
```json
{
  "displayName": "Johnny",
  "username": "johnny_d"
}
```

**Request (multipart):**
```
Content-Type: multipart/form-data
- avatar: <file> (max 2MB, JPEG/PNG)
- displayName: "Johnny"
```

---

## 3. Room APIs

### POST `/rooms` 🔒

Create a new drawing room.

**Request:**
```json
{
  "name": "Design Sprint #3",
  "visibility": "private",
  "maxParticipants": 10,
  "settings": {
    "backgroundColor": "#1a1a2e",
    "canvasWidth": 1920,
    "canvasHeight": 1080
  }
}
```

**Response `201`:**
```json
{
  "success": true,
  "data": {
    "id": "665room789abc",
    "name": "Design Sprint #3",
    "slug": "design-sprint-3-a1b2c3",
    "owner": "665abc123def456",
    "inviteCode": "XK7M9P2Q",
    "inviteUrl": "https://drawsaver.app/join/XK7M9P2Q",
    "visibility": "private",
    "maxParticipants": 10,
    "settings": { ... },
    "createdAt": "2026-05-27T12:00:00Z"
  }
}
```

---

### GET `/rooms` 🔒

List rooms the user owns or is a member of.

**Query Parameters:**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `page` | number | 1 | Pagination page |
| `limit` | number | 20 | Items per page (max 50) |
| `sort` | string | `lastActivityAt` | Sort field |
| `filter` | string | `all` | `all`, `owned`, `shared` |

**Response `200`:**
```json
{
  "success": true,
  "data": {
    "rooms": [
      {
        "id": "...",
        "name": "Design Sprint #3",
        "thumbnail": "https://s3.../thumbnails/...",
        "participantCount": 3,
        "lastActivityAt": "2026-05-27T12:00:00Z",
        "myRole": "owner"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 45,
      "totalPages": 3
    }
  }
}
```

---

### GET `/rooms/:id` 🔒

Get room details including members.

### DELETE `/rooms/:id` 🔒 (Owner only)

Soft-delete a room. Sets `isActive: false`.

### POST `/rooms/join` 🔒

Join a room by invite code.

**Request:**
```json
{ "inviteCode": "XK7M9P2Q" }
```

### DELETE `/rooms/:id/members/:userId` 🔒 (Owner only)

Remove a user from a room.

---

## 4. Drawing APIs

### GET `/drawings/room/:roomId` 🔒

Get the current drawing state for a room.

**Response `200`:**
```json
{
  "success": true,
  "data": {
    "id": "665draw123",
    "title": "Sprint Whiteboard",
    "canvasData": { "version": "6.0", "objects": [...] },
    "currentVersion": 12,
    "objectCount": 47,
    "updatedAt": "2026-05-27T12:30:00Z"
  }
}
```

---

### PUT `/drawings/:id` 🔒

Save/update the drawing (manual save).

**Request:**
```json
{
  "title": "Sprint Whiteboard - Final",
  "canvasData": { "version": "6.0", "objects": [...] }
}
```

**Response `200`:**
```json
{
  "success": true,
  "data": {
    "id": "665draw123",
    "currentVersion": 13,
    "updatedAt": "2026-05-27T12:35:00Z"
  },
  "message": "Drawing saved. Version 13 created."
}
```

---

### POST `/drawings/:id/auto-save` 🔒

Auto-save endpoint (called by client every 30s). Does not create a version snapshot unless significant changes detected.

**Request:**
```json
{
  "canvasData": { ... },
  "objectCount": 48
}
```

**Response `200`:**
```json
{
  "success": true,
  "data": { "saved": true, "versionCreated": false }
}
```

---

### GET `/drawings/:id/versions` 🔒

List version history.

**Response `200`:**
```json
{
  "success": true,
  "data": {
    "versions": [
      { "version": 13, "thumbnail": "...", "createdBy": "johndoe", "createdAt": "...", "changeDescription": "Added 5 objects" },
      { "version": 12, "thumbnail": "...", "createdBy": "jane", "createdAt": "...", "changeDescription": "Modified 2 objects" }
    ]
  }
}
```

### POST `/drawings/:id/versions/:version/restore` 🔒

Restore a specific version (creates a new version with the old state).

---

### POST `/drawings/:id/export` 🔒

Export drawing as image/document.

**Request:**
```json
{
  "format": "png",
  "quality": 0.92,
  "backgroundColor": "transparent",
  "scale": 2
}
```

**Response `200`:**
```json
{
  "success": true,
  "data": {
    "downloadUrl": "https://s3.../exports/665draw123/export-v13.png",
    "expiresIn": 3600
  }
}
```

---

## 5. WebSocket Events

### Connection

```javascript
const socket = io('wss://api.drawsaver.app', {
  auth: { token: '<accessToken>' },
  transports: ['websocket']
});
```

### Client → Server Events

| Event | Payload | Description |
|-------|---------|-------------|
| `join-room` | `{ roomId: string }` | Join a drawing room |
| `leave-room` | `{ roomId: string }` | Leave a room |
| `draw-action` | `{ roomId, action, objectData, timestamp }` | Drawing operation (see below) |
| `cursor-move` | `{ roomId, x, y }` | Cursor position update |
| `undo` | `{ roomId }` | Undo last action |
| `redo` | `{ roomId }` | Redo last undone action |
| `canvas-clear` | `{ roomId }` | Clear entire canvas |

### Server → Client Events

| Event | Payload | Description |
|-------|---------|-------------|
| `room-state` | `{ canvasData, participants: [{ userId, username, avatar, role }] }` | Full state on room join |
| `user-joined` | `{ userId, username, avatar }` | New user entered |
| `user-left` | `{ userId }` | User disconnected |
| `draw-action` | `{ userId, action, objectData, timestamp }` | Another user's drawing operation |
| `cursor-update` | `{ userId, username, x, y, color }` | Other user's cursor position |
| `presence-update` | `{ userId, status }` | User status change (active/idle) |
| `error` | `{ code, message }` | Error notification |
| `room-closed` | `{ reason }` | Room has been closed/deleted |

### Draw Action Types

```typescript
type DrawAction = {
  roomId: string;
  action: 'object:added' | 'object:modified' | 'object:removed';
  objectData: {
    id: string;        // Unique object ID on canvas
    type: string;      // 'path', 'rect', 'circle', 'textbox', etc.
    properties: object; // Fabric.js serialized object properties
  };
  timestamp: number;   // Client timestamp (server also adds its own)
};
```

### Cursor Move (Throttled)

Cursor events are **throttled to 50ms** on the client to avoid flooding:

```javascript
// Client-side: throttle cursor emission
const throttledCursorMove = throttle((x, y) => {
  socket.emit('cursor-move', { roomId, x, y });
}, 50);

canvas.on('mouse:move', (e) => {
  throttledCursorMove(e.pointer.x, e.pointer.y);
});
```

---

## 6. Error Response Format

All errors follow a consistent format:

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Username must be between 3 and 30 characters",
    "details": [
      { "field": "username", "message": "Too short" }
    ]
  }
}
```

### Standard Error Codes

| HTTP | Code | Description |
|------|------|-------------|
| 400 | `VALIDATION_ERROR` | Request body failed Zod validation |
| 401 | `UNAUTHORIZED` | Missing or expired token |
| 401 | `INVALID_CREDENTIALS` | Wrong email/password |
| 403 | `FORBIDDEN` | Insufficient permissions |
| 404 | `NOT_FOUND` | Resource not found |
| 409 | `CONFLICT` | Duplicate resource (email, username) |
| 429 | `RATE_LIMITED` | Too many requests |
| 500 | `INTERNAL_ERROR` | Unexpected server error |

---

## 7. Rate Limiting

| Endpoint Group | Limit | Window |
|---------------|-------|--------|
| Auth (login, register) | 10 requests | 15 minutes |
| Auth (forgot-password) | 3 requests | 1 hour |
| General API | 100 requests | 1 minute |
| Drawing save | 30 requests | 1 minute |
| Export | 10 requests | 5 minutes |
| WebSocket events | 60 events/sec | Per connection |

Rate limit headers are included in all responses:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1716840000
```
