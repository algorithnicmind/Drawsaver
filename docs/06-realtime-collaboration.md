# DrawSaver — Real-Time Collaboration System

> **Version:** 1.0  
> **Last Updated:** 2026-05-27  

---

## 1. Architecture Overview

```
┌────────────┐      ┌────────────┐      ┌────────────┐
│  Client A  │      │  Client B  │      │  Client C  │
│  (Browser) │      │  (Browser) │      │  (Browser) │
└─────┬──────┘      └─────┬──────┘      └─────┬──────┘
      │                   │                   │
      │    WebSocket      │    WebSocket      │    WebSocket
      │                   │                   │
┌─────▼───────────────────▼───────────────────▼──────┐
│                   Socket.IO Server                  │
│  ┌─────────────────────────────────────────────┐   │
│  │              Room: "room-abc123"             │   │
│  │                                             │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐    │   │
│  │  │Socket A │  │Socket B │  │Socket C │    │   │
│  │  └─────────┘  └─────────┘  └─────────┘    │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  Event Flow:                                        │
│  Client A draws → Server validates → Broadcast      │
│  to B and C (NOT back to A)                         │
└─────────────────────┬───────────────────────────────┘
                      │
              ┌───────▼───────┐
              │  Redis Buffer │ → Periodic flush to MongoDB
              └───────────────┘
```

---

## 2. Socket Communication Flow

### 2.1 Room Join Flow

```
Client                              Server
  │                                    │
  │── connect(auth: {token}) ────────► │ 1. Verify JWT
  │                                    │ 2. Map socket.id → userId
  │◄── connected ────────────────────  │
  │                                    │
  │── join-room({roomId}) ───────────► │ 3. Validate room exists
  │                                    │ 4. Check user has access
  │                                    │ 5. socket.join(roomId)
  │                                    │ 6. Add to presence map
  │                                    │
  │◄── room-state ───────────────────  │ 7. Send current canvas + participants
  │                                    │
  │    (broadcast to others in room)   │
  │                                    │── user-joined ──► Other clients
  │                                    │
```

### 2.2 Drawing Sync Flow

```
Client A draws a rectangle:

Client A                           Server                          Client B
  │                                   │                                │
  │ 1. User draws rect on canvas      │                                │
  │ 2. Fabric.js fires object:added   │                                │
  │ 3. Serialize object to JSON       │                                │
  │                                   │                                │
  │── draw-action ──────────────────► │                                │
  │   {                               │ 4. Validate action             │
  │     action: "object:added",       │ 5. Add server timestamp        │
  │     objectData: {                 │ 6. Buffer in Redis              │
  │       id: "obj_a1b2c3",          │                                │
  │       type: "rect",              │── draw-action ────────────────► │
  │       properties: {...}           │   (broadcast, exclude sender)   │
  │     }                             │                                │ 7. Deserialize object
  │   }                               │                                │ 8. Add to local canvas
  │                                   │                                │ 9. Render
  │                                   │                                │
```

### 2.3 Disconnect & Reconnect Flow

```
Client A loses connection:

Client A                           Server                          Client B
  │                                   │                                │
  │── disconnect (network loss) ────► │                                │
  │                                   │ 1. Start 30s grace period      │
  │                                   │    (don't remove from room)    │
  │                                   │                                │
  │                                   │── presence-update ────────────►│
  │                                   │   { userId: A, status: "away" }│
  │                                   │                                │
  │   (within 30 seconds)            │                                │
  │── reconnect ─────────────────── ►│                                │
  │── join-room (same roomId) ──────►│ 2. Cancel grace timer          │
  │                                   │ 3. Send current state          │
  │◄── room-state (full sync) ──────  │                                │
  │                                   │── presence-update ────────────►│
  │                                   │   { userId: A, status: "active"}│
  │                                   │                                │
  │   (after 30 seconds, no reconnect)│                               │
  │                                   │ 4. Remove from room            │
  │                                   │── user-left ──────────────────►│
```

---

## 3. Multiplayer Synchronization

### 3.1 Sync Strategy: Operation Broadcasting

DrawSaver uses **operation-based synchronization** for the MVP:

1. Each client maintains a local Fabric.js canvas (source of truth for that client)
2. When a user performs an action, it is applied locally first, then emitted to the server
3. The server broadcasts the action to all other clients in the room
4. Other clients apply the action to their local canvas

```
Local-first → Emit → Server Broadcast → Remote Apply
```

### 3.2 Object Identity

Every canvas object gets a unique ID assigned on creation:

```javascript
// On object creation, assign a unique ID
canvas.on('object:added', (e) => {
  const obj = e.target;
  if (!obj.customId) {
    obj.customId = `obj_${userId}_${Date.now()}_${Math.random().toString(36).substr(2, 6)}`;
  }
});
```

This ID is used to identify the same object across all clients for modify/delete operations.

### 3.3 Action Types and Payloads

```typescript
// Object Added — send full serialized object
{
  action: 'object:added',
  objectData: {
    id: 'obj_user1_1716840000_abc123',
    type: 'rect',
    properties: canvas.getActiveObject().toJSON(['customId'])
  }
}

// Object Modified — send only changed properties (delta)
{
  action: 'object:modified',
  objectData: {
    id: 'obj_user1_1716840000_abc123',
    type: 'rect',
    properties: {
      left: 150,
      top: 200,
      scaleX: 1.5,
      angle: 45
    }
  }
}

// Object Removed
{
  action: 'object:removed',
  objectData: {
    id: 'obj_user1_1716840000_abc123'
  }
}
```

### 3.4 Batching

For rapid drawing (freehand paths), events are batched:

```javascript
// Client-side batching for path drawing
let actionBatch = [];
let batchTimer = null;

function queueAction(action) {
  actionBatch.push(action);
  
  if (!batchTimer) {
    batchTimer = setTimeout(() => {
      socket.emit('draw-actions-batch', { roomId, actions: actionBatch });
      actionBatch = [];
      batchTimer = null;
    }, 50); // Flush every 50ms
  }
}
```

---

## 4. Conflict Handling

### 4.1 MVP Strategy: Last-Write-Wins (LWW)

For the MVP, conflicts are resolved using **last-write-wins** based on server timestamps:

```
User A modifies rect.left = 100  (server timestamp: T1)
User B modifies rect.left = 200  (server timestamp: T2, where T2 > T1)

Result: rect.left = 200 (User B's change wins)
```

**Why LWW for MVP:**
- Simple to implement
- Acceptable UX for small teams (2-5 users)
- Conflicts are rare when users work on different areas of the canvas
- No external library dependencies

### 4.2 Conflict Mitigation Strategies

Even with LWW, conflicts can be reduced:

1. **Visual Object Locking:** When a user selects an object, broadcast a `object:locked` event. Other users see a visual indicator (colored border) that the object is being edited. This doesn't prevent editing but discourages it.

```javascript
// Client emits lock when selecting an object
canvas.on('selection:created', (e) => {
  socket.emit('object:lock', { 
    roomId, 
    objectId: e.selected[0].customId,
    userId 
  });
});

canvas.on('selection:cleared', () => {
  socket.emit('object:unlock', { roomId, objectId, userId });
});
```

2. **Spatial Partitioning Awareness:** Show other users' cursor positions so users naturally avoid editing in the same area.

3. **Operation Ordering:** Server assigns monotonically increasing sequence numbers to all operations. Clients apply operations in sequence order.

### 4.3 Future: CRDT-Based Sync (v2+)

For large-scale collaboration (10+ users), migrate to a CRDT (Conflict-free Replicated Data Type) approach using **Yjs**:

```
┌──────────┐        ┌──────────┐        ┌──────────┐
│ Client A │        │ Server   │        │ Client B │
│ Yjs Doc  │◄──────►│ Yjs Doc  │◄──────►│ Yjs Doc  │
│          │  sync  │ (y-socket│  sync  │          │
│          │        │  -io)    │        │          │
└──────────┘        └──────────┘        └──────────┘
```

Benefits:
- Automatic conflict resolution
- Offline editing support
- No data loss on concurrent edits
- Built-in undo/redo per user

---

## 5. Presence System

### 5.1 User States

```
┌──────────┐      ┌──────────┐      ┌──────────┐
│  Active  │─30s──│   Idle   │──DC──│  Offline │
│ (drawing │ no   │ (in room │      │ (left    │
│  or move)│ input│  but AFK)│      │  room)   │
└──────────┘      └──────────┘      └──────────┘
```

| State | Condition | Visual |
|-------|-----------|--------|
| `active` | Mouse/touch activity within last 30s | Green dot |
| `idle` | No activity for 30s+ | Yellow dot |
| `offline` | Disconnected / left room | Grey dot (then removed) |

### 5.2 Server-Side Presence Tracking

```javascript
// Redis-based presence tracking
const PRESENCE_KEY = (roomId) => `presence:${roomId}`;
const PRESENCE_TTL = 45; // seconds

// On heartbeat (client sends every 30s)
socket.on('heartbeat', async () => {
  await redis.hSet(PRESENCE_KEY(roomId), userId, JSON.stringify({
    status: 'active',
    lastSeen: Date.now(),
    socketId: socket.id
  }));
  await redis.expire(PRESENCE_KEY(roomId), PRESENCE_TTL);
});

// Get room presence
async function getRoomPresence(roomId) {
  const presence = await redis.hGetAll(PRESENCE_KEY(roomId));
  return Object.entries(presence).map(([userId, data]) => ({
    userId,
    ...JSON.parse(data)
  }));
}
```

### 5.3 Presence UI Data

Each client receives a participant list on room join and updates via events:

```typescript
interface Participant {
  userId: string;
  username: string;
  avatar: string | null;
  role: 'owner' | 'editor' | 'viewer';
  status: 'active' | 'idle' | 'offline';
  cursorColor: string;  // Assigned unique color per user
}
```

---

## 6. Cursor Tracking

### 6.1 Implementation

Each user's cursor position is broadcast to other users in the room, rendered as a labeled pointer on the canvas.

```javascript
// === CLIENT SIDE ===

// Emit cursor position (throttled to 50ms)
const CURSOR_THROTTLE_MS = 50;
let lastCursorEmit = 0;

canvas.on('mouse:move', (e) => {
  const now = Date.now();
  if (now - lastCursorEmit < CURSOR_THROTTLE_MS) return;
  lastCursorEmit = now;
  
  const pointer = canvas.getPointer(e.e);
  socket.emit('cursor-move', {
    roomId,
    x: pointer.x,
    y: pointer.y
  });
});

// Receive other users' cursors
socket.on('cursor-update', ({ userId, username, x, y, color }) => {
  renderRemoteCursor(userId, username, x, y, color);
});

// Render remote cursor on canvas overlay
function renderRemoteCursor(userId, username, x, y, color) {
  let cursor = remoteCursors.get(userId);
  
  if (!cursor) {
    // Create cursor element (HTML overlay, not canvas object)
    cursor = document.createElement('div');
    cursor.className = 'remote-cursor';
    cursor.innerHTML = `
      <svg width="16" height="16" viewBox="0 0 16 16">
        <path d="M0 0 L12 5 L5 12 Z" fill="${color}"/>
      </svg>
      <span class="cursor-label" style="background: ${color}">${username}</span>
    `;
    canvasOverlay.appendChild(cursor);
    remoteCursors.set(userId, cursor);
  }
  
  // Smooth animation with CSS transform
  cursor.style.transform = `translate(${x}px, ${y}px)`;
}
```

### 6.2 Cursor Color Assignment

Each user in a room is assigned a unique cursor color:

```javascript
const CURSOR_COLORS = [
  '#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4',
  '#FFEAA7', '#DDA0DD', '#98D8C8', '#F7DC6F',
  '#BB8FCE', '#85C1E9', '#F0B27A', '#82E0AA'
];

function assignCursorColor(userIndex) {
  return CURSOR_COLORS[userIndex % CURSOR_COLORS.length];
}
```

### 6.3 Cursor Cleanup

When a user disconnects or goes idle, their cursor fades out and is removed:

```javascript
socket.on('user-left', ({ userId }) => {
  const cursor = remoteCursors.get(userId);
  if (cursor) {
    cursor.classList.add('cursor-fade-out'); // CSS animation
    setTimeout(() => {
      cursor.remove();
      remoteCursors.delete(userId);
    }, 500);
  }
});
```

```css
.remote-cursor {
  position: absolute;
  pointer-events: none;
  transition: transform 0.1s ease-out;
  z-index: 1000;
}

.cursor-label {
  font-size: 11px;
  color: white;
  padding: 1px 6px;
  border-radius: 3px;
  margin-left: 12px;
  white-space: nowrap;
}

.cursor-fade-out {
  animation: fadeOut 0.5s forwards;
}

@keyframes fadeOut {
  to { opacity: 0; transform: scale(0.5); }
}
```

---

## 7. Performance Optimization

### 7.1 Event Throttling Summary

| Event | Throttle | Reason |
|-------|----------|--------|
| `cursor-move` | 50ms | Smooth enough for visual tracking |
| `draw-action` (path) | 50ms batch | Freehand drawing generates many points |
| `draw-action` (shape) | Immediate | Discrete events, low frequency |
| `presence heartbeat` | 30s | Only needed for idle detection |

### 7.2 Bandwidth Estimation

| Event | Size | Frequency | Bandwidth (per user) |
|-------|------|-----------|---------------------|
| Cursor move | ~100 bytes | 20/sec | ~2 KB/s |
| Draw action | ~500 bytes | 5/sec avg | ~2.5 KB/s |
| Presence | ~50 bytes | 1/30sec | Negligible |
| **Total** | | | **~5 KB/s** |

For a room with 10 users, each client receives: ~45 KB/s (cursor + draw actions from 9 other users). This is well within reasonable bandwidth.

### 7.3 Server-Side Optimization

1. **Skip self-broadcast:** Use `socket.to(roomId)` not `io.to(roomId)` to avoid sending events back to the originator
2. **Binary encoding:** For large canvas syncs, use `msgpack` instead of JSON
3. **Room cleanup:** Periodically check for empty rooms and free resources
4. **Connection limits:** Max 50 connections per room, max 5 rooms per user simultaneously
