# DrawSaver — Technical Architecture Document

> **Version:** 1.0  
> **Last Updated:** 2026-05-27  

---

## 1. High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENTS                                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │ Browser  │  │ Browser  │  │ Tablet   │  │ Mobile   │       │
│  │ (React)  │  │ (React)  │  │ (React)  │  │ (Future) │       │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘       │
│       │              │              │              │             │
│       └──────────────┴──────┬───────┴──────────────┘             │
└─────────────────────────────┼───────────────────────────────────┘
                              │
                    ┌─────────▼──────────┐
                    │   CDN / Reverse    │
                    │   Proxy (Nginx)    │
                    └─────────┬──────────┘
                              │
              ┌───────────────┼───────────────┐
              │               │               │
     ┌────────▼─────┐ ┌──────▼───────┐ ┌─────▼──────┐
     │  REST API    │ │  WebSocket   │ │  Static    │
     │  Server      │ │  Server      │ │  Assets    │
     │  (Express)   │ │  (Socket.IO) │ │  (CDN)     │
     └────────┬─────┘ └──────┬───────┘ └────────────┘
              │               │
              └───────┬───────┘
                      │
         ┌────────────┼────────────┐
         │            │            │
    ┌────▼────┐ ┌─────▼─────┐ ┌───▼─────┐
    │ MongoDB │ │   Redis   │ │   S3    │
    │ (Data)  │ │  (Cache/  │ │ (Files) │
    │         │ │  PubSub)  │ │         │
    └─────────┘ └───────────┘ └─────────┘
```

---

## 2. Frontend Architecture

### Technology
- **Framework:** React 18+ with Vite (fast build, HMR)
- **Canvas Library:** Fabric.js (primary) — rich object model, event system, serialization
- **State Management:** Zustand — lightweight, no boilerplate, perfect for canvas state
- **Routing:** React Router v6
- **Real-time:** Socket.IO Client
- **Styling:** Tailwind CSS + custom design tokens
- **HTTP Client:** Axios with interceptors for auth token refresh

### Component Architecture

```
App
├── AuthProvider (context — JWT management)
├── SocketProvider (context — Socket.IO connection lifecycle)
│
├── Pages/
│   ├── LandingPage
│   ├── LoginPage
│   ├── RegisterPage
│   ├── DashboardPage
│   │   ├── DrawingCard (thumbnail, metadata, actions)
│   │   └── CreateRoomModal
│   ├── DrawingWorkspace
│   │   ├── CanvasContainer (Fabric.js instance)
│   │   ├── Toolbar (tool selection, brush settings)
│   │   ├── ColorPanel (picker + recent + opacity)
│   │   ├── LayersPanel (list, visibility, reorder)
│   │   ├── UsersPanel (presence, cursors)
│   │   ├── HistoryPanel (undo/redo, version list)
│   │   └── ExportModal
│   └── ProfilePage
│
├── Hooks/
│   ├── useCanvas() — Fabric.js lifecycle
│   ├── useSocket() — Socket.IO events
│   ├── useAuth() — JWT token management
│   ├── useDrawingTools() — tool state machine
│   ├── useAutoSave() — debounced save logic
│   └── usePresence() — cursor/user tracking
│
└── Store/ (Zustand)
    ├── canvasStore — active tool, color, stroke, zoom
    ├── roomStore — room info, participants
    └── authStore — user, tokens, auth state
```

### Canvas State Management Strategy

```
┌──────────────────────────────────────────────────┐
│                 Zustand Store                     │
│  ┌─────────────┐  ┌──────────┐  ┌────────────┐  │
│  │ activeTool  │  │ color    │  │ brushSize  │  │
│  │ activeLayer │  │ opacity  │  │ zoom       │  │
│  └──────┬──────┘  └────┬─────┘  └─────┬──────┘  │
│         │              │              │          │
│         └──────────────┼──────────────┘          │
│                        │                         │
│              ┌─────────▼──────────┐              │
│              │  Fabric.js Canvas  │              │
│              │  (source of truth  │              │
│              │   for objects)     │              │
│              └─────────┬──────────┘              │
│                        │                         │
│         ┌──────────────┼──────────────┐          │
│         │              │              │          │
│   ┌─────▼─────┐ ┌─────▼─────┐ ┌─────▼─────┐   │
│   │  Local     │ │  Socket   │ │  Auto     │   │
│   │  Undo/Redo │ │  Emit     │ │  Save     │   │
│   └───────────┘ └───────────┘ └───────────┘   │
└──────────────────────────────────────────────────┘
```

**Key Principle:** Fabric.js canvas is the source of truth for drawing objects. Zustand manages UI state (active tool, colors, zoom). Socket.IO syncs canvas operations between clients.

---

## 3. Backend Architecture

### Technology
- **Runtime:** Node.js 20 LTS
- **Framework:** Express.js
- **WebSocket:** Socket.IO (integrated with Express via shared HTTP server)
- **ORM/ODM:** Mongoose (MongoDB)
- **Auth:** Passport.js (OAuth strategies) + jsonwebtoken + bcryptjs
- **Validation:** Zod (schema validation)
- **File Upload:** Multer → S3 (via AWS SDK v3)

### Service Layer Architecture

```
┌─────────────────────────────────────────────┐
│               Express App                    │
│                                             │
│  ┌──────────────────────────────────────┐   │
│  │           Middleware Stack            │   │
│  │  cors → helmet → rateLimit →         │   │
│  │  morgan → authGuard → zodValidate    │   │
│  └──────────────┬───────────────────────┘   │
│                 │                            │
│  ┌──────────────▼───────────────────────┐   │
│  │             Routes                    │   │
│  │  /api/v1/auth/*                      │   │
│  │  /api/v1/rooms/*                     │   │
│  │  /api/v1/drawings/*                  │   │
│  │  /api/v1/users/*                     │   │
│  └──────────────┬───────────────────────┘   │
│                 │                            │
│  ┌──────────────▼───────────────────────┐   │
│  │           Controllers                 │   │
│  │  Parse req → call service → send res  │   │
│  └──────────────┬───────────────────────┘   │
│                 │                            │
│  ┌──────────────▼───────────────────────┐   │
│  │            Services                   │   │
│  │  Business logic, orchestration        │   │
│  └──────────────┬───────────────────────┘   │
│                 │                            │
│  ┌──────────────▼───────────────────────┐   │
│  │          Data Access Layer            │   │
│  │  Mongoose Models, Redis Client,       │   │
│  │  S3 Client                            │   │
│  └──────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

### API Versioning
All REST endpoints are prefixed with `/api/v1/`. This allows non-breaking API evolution.

---

## 4. WebSocket Architecture

### Connection Lifecycle

```
Client                          Server
  │                                │
  ├── connect(token) ──────────►  │  Verify JWT
  │                                │  Join user to socket pool
  │  ◄────── connected ──────────  │  
  │                                │
  ├── join-room(roomId) ────────►  │  Validate room access
  │                                │  socket.join(roomId)
  │  ◄──── room-state ──────────  │  Send current canvas state
  │  ◄──── user-joined ─────────  │  Broadcast to room
  │                                │
  │  === Drawing Loop ===          │
  ├── draw-action(delta) ───────►  │  Validate, timestamp
  │  ◄──── draw-action ─────────  │  Broadcast to room (except sender)
  │                                │
  ├── cursor-move(pos) ─────────►  │
  │  ◄──── cursor-update ───────  │  Broadcast to room
  │                                │
  │  === Disconnect ===            │
  ├── disconnect ───────────────►  │  Remove from room
  │  ◄──── user-left ───────────  │  Broadcast to room
  │                                │
```

### Room Management in Socket.IO

```javascript
// Server-side room management pseudocode
io.on('connection', (socket) => {
  const user = verifyToken(socket.handshake.auth.token);
  
  socket.on('join-room', async (roomId) => {
    // Validate room exists and user has access
    const room = await RoomService.getRoom(roomId);
    if (!room) return socket.emit('error', 'Room not found');
    
    socket.join(roomId);
    
    // Send current canvas state to joining user
    const canvasState = await DrawingService.getCanvasState(roomId);
    socket.emit('room-state', { canvasState, participants: getParticipants(roomId) });
    
    // Notify others
    socket.to(roomId).emit('user-joined', { userId: user.id, username: user.username });
  });
  
  socket.on('draw-action', (data) => {
    // Broadcast to everyone in room except sender
    socket.to(data.roomId).emit('draw-action', {
      ...data,
      userId: user.id,
      timestamp: Date.now()
    });
  });
});
```

### Scaling WebSockets with Redis Adapter

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  WS Server 1 │    │  WS Server 2 │    │  WS Server 3 │
│  (Socket.IO)  │    │  (Socket.IO)  │    │  (Socket.IO)  │
└──────┬───────┘    └──────┬───────┘    └──────┬───────┘
       │                   │                   │
       └───────────────────┼───────────────────┘
                           │
                   ┌───────▼───────┐
                   │  Redis PubSub │
                   │  (Adapter)    │
                   └───────────────┘
```

When scaling to multiple Socket.IO instances, use `@socket.io/redis-adapter` so events are broadcast across all server instances. Sticky sessions via Nginx ensure clients reconnect to the same server for stateful connections.

---

## 5. Database Architecture

### Primary Database: MongoDB

**Why MongoDB:**
- Flexible schema for drawing data (arbitrary JSON structures)
- Native JSON storage matches Fabric.js canvas serialization
- Good horizontal scaling (replica sets, sharding)
- Rich query support for metadata

### Caching Layer: Redis

**Purpose:**
- Session storage (active room state, user sessions)
- Socket.IO adapter for multi-server pub/sub
- Rate limiting counters
- Temporary canvas state (working buffer before persist)
- User presence tracking

### Data Flow

```
┌─────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│ Canvas   │────►│  Redis   │────►│  MongoDB │────►│    S3    │
│ Actions  │     │ (Buffer) │     │ (Persist)│     │ (Assets) │
│          │     │ ~30s TTL │     │          │     │          │
└──────────┘     └──────────┘     └──────────┘     └──────────┘
     │                                                   ▲
     │               Auto-save timer                     │
     └─────── every 30s, flush Redis → MongoDB ──────────┘
              on export, canvas → S3 as image
```

---

## 6. Cloud & Storage Architecture

### File Storage: AWS S3 (or compatible — MinIO for local dev)

| Asset Type | Bucket/Path | Format | Retention |
|------------|-------------|--------|-----------|
| User avatars | `drawsaver-assets/avatars/{userId}` | JPEG/PNG, max 2MB | Permanent |
| Drawing thumbnails | `drawsaver-assets/thumbnails/{drawingId}` | JPEG, 400x300 | Permanent |
| Drawing exports | `drawsaver-assets/exports/{drawingId}/{version}` | PNG/JPG/SVG/PDF | 30 days |
| Drawing snapshots | `drawsaver-data/snapshots/{drawingId}/{version}` | JSON (canvas state) | Permanent |

### CDN Strategy
- Serve static frontend assets via **CloudFront** (or Vercel Edge)
- Serve user-uploaded images via **CloudFront** with S3 origin
- Cache headers: `max-age=31536000` for hashed static assets, `max-age=3600` for dynamic images

---

## 7. Scalability Strategy

### Phase 1: Single-Server MVP
```
Single Server (4 vCPU, 8GB RAM)
├── Express API + Socket.IO (same process)
├── MongoDB Atlas (M10 cluster)
├── Redis Cloud (free tier)
└── S3 bucket
```
**Capacity:** ~100 concurrent users, ~20 concurrent rooms

### Phase 2: Separated Services
```
├── API Server ×2 (behind load balancer)
├── WebSocket Server ×2 (sticky sessions)
├── MongoDB Atlas (M30 replica set)
├── Redis (dedicated instance, pub/sub adapter)
└── S3 + CloudFront CDN
```
**Capacity:** ~1,000 concurrent users, ~200 rooms

### Phase 3: Full Scale
```
├── API Server ×N (auto-scaling group)
├── WebSocket Server ×N (auto-scaling, Redis adapter)
├── MongoDB Sharded Cluster
├── Redis Cluster (HA)
├── S3 + CloudFront
├── Message Queue (Bull/BullMQ for async jobs)
└── Monitoring (Datadog/Grafana)
```
**Capacity:** 10,000+ concurrent users

### Key Scaling Decisions

| Concern | Strategy |
|---------|----------|
| WebSocket scaling | Redis adapter + sticky sessions via Nginx `ip_hash` |
| Database scaling | MongoDB replica set → sharding by `roomId` |
| File storage | S3 with CDN, presigned URLs for uploads |
| Background jobs | BullMQ (Redis-backed) for exports, thumbnail generation |
| API rate limiting | Redis-based sliding window (express-rate-limit + rate-limit-redis) |
| Canvas data size | Delta compression, periodic snapshot compaction |
