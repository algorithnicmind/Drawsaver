# DrawSaver — Tech Stack Documentation

> **Version:** 1.0  
> **Last Updated:** 2026-05-27  

---

## Stack Summary

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Frontend** | React 18 + Vite | UI framework + build tool |
| **Canvas** | Fabric.js | Drawing engine with object model |
| **State** | Zustand | Lightweight client state management |
| **Styling** | Tailwind CSS | Utility-first CSS |
| **Backend** | Node.js + Express | REST API server |
| **WebSocket** | Socket.IO | Real-time bidirectional communication |
| **Database** | MongoDB + Mongoose | Primary data store |
| **Cache** | Redis | Caching, sessions, pub/sub, rate limiting |
| **Auth** | Passport.js + JWT | Authentication and authorization |
| **Storage** | AWS S3 | File/image storage |
| **Deployment** | Docker + Render/Railway | Containerized deployment |
| **CI/CD** | GitHub Actions | Automated testing and deployment |
| **Monitoring** | Sentry + Pino | Error tracking + structured logging |

---

## Detailed Technology Decisions

### 1. Frontend Framework — React 18 + Vite

**Why React:**
- Largest ecosystem, best talent availability
- Component model fits canvas workspace layout
- Hooks API for clean canvas lifecycle management
- Excellent tooling (DevTools, testing libraries)

**Why Vite (not CRA or Next.js):**
- ⚡ Sub-second HMR — critical for canvas-heavy UI iteration
- No SSR needed (canvas is purely client-side rendered)
- Smaller bundle, faster builds than CRA
- Native ES modules in development

**Why not Next.js:**
- Canvas drawing is 100% client-side — no SSR benefit
- Added complexity (server components, file-system routing) not needed for SPA
- Socket.IO integration is cleaner with a standalone backend

| Criteria | Vite + React | Next.js | CRA |
|----------|-------------|---------|-----|
| Build speed | ⭐⭐⭐ | ⭐⭐ | ⭐ |
| SSR needed? | No | Yes (overkill) | No |
| Complexity | Low | Medium | Low |
| Bundle size | Small | Medium | Large |
| Canvas fit | ⭐⭐⭐ | ⭐⭐ | ⭐⭐ |

---

### 2. Canvas Library — Fabric.js

**Why Fabric.js:**
- Full object model (each shape is a manipulable object)
- Built-in serialization (`canvas.toJSON()` / `canvas.loadFromJSON()`) — perfect for save/restore
- Rich event system for drawing interactions
- Built-in selection, grouping, transformation handles
- Active maintenance, large community

**Why not Konva.js:**
- Konva uses React-Konva which re-renders the canvas tree on every state change — performance issues with many objects
- Fabric.js manages its own render loop, more efficient for real-time drawing
- Fabric.js has better serialization out-of-the-box

**Why not raw Canvas API:**
- No object model — need to rebuild hit detection, selection, serialization
- Significant engineering effort for features Fabric.js provides free

| Criteria | Fabric.js | Konva.js | Raw Canvas |
|----------|----------|---------|------------|
| Object model | ⭐⭐⭐ | ⭐⭐⭐ | None |
| Serialization | Built-in | Manual | Manual |
| React integration | Good | Native | Manual |
| Performance (many objects) | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ |
| Learning curve | Medium | Medium | High |
| Community | Large | Medium | N/A |

---

### 3. State Management — Zustand

**Why Zustand:**
- Minimal boilerplate (no reducers, actions, providers)
- Direct mutation-style API (`set({ tool: 'brush' })`)
- Built-in middleware: `devtools`, `persist`, `subscribeWithSelector`
- Works perfectly outside React (can be used in Fabric.js event handlers)
- Tiny bundle (~1KB)

**Why not Redux Toolkit:**
- Overkill for this use case — canvas state is mostly ephemeral
- Boilerplate overhead for slices, reducers
- Redux DevTools still available via Zustand middleware

**Why not Context API:**
- Re-renders entire tree on any context change
- Not suitable for high-frequency canvas state updates (tool changes, color picks)

```javascript
// Example: Canvas store with Zustand
import { create } from 'zustand';

export const useCanvasStore = create((set) => ({
  activeTool: 'pencil',
  brushColor: '#000000',
  brushSize: 3,
  opacity: 1,
  zoom: 1,
  
  setTool: (tool) => set({ activeTool: tool }),
  setColor: (color) => set({ brushColor: color }),
  setBrushSize: (size) => set({ brushSize: size }),
}));
```

---

### 4. Backend — Node.js + Express

**Why Node.js:**
- Same language (JavaScript/TypeScript) as frontend — shared types, validation schemas
- Non-blocking I/O ideal for WebSocket-heavy workloads
- Excellent Socket.IO support (same team)
- Large package ecosystem (npm)

**Why Express (not Fastify/Hono):**
- Most mature, widest middleware ecosystem
- Best Socket.IO integration docs
- Passport.js and most auth libraries are Express-first
- Team familiarity, lowest onboarding friction

**Future consideration:** Migrate to Fastify if throughput becomes a bottleneck (2-3x faster than Express in benchmarks).

---

### 5. WebSocket — Socket.IO

**Why Socket.IO (not raw WebSocket or ws):**
- Automatic reconnection with exponential backoff
- Room abstraction (`socket.join('room-123')`, `io.to('room-123').emit(...)`)
- Fallback to HTTP long-polling if WebSocket is blocked (corporate firewalls)
- Built-in acknowledgement callbacks
- Redis adapter for horizontal scaling
- Binary data support (for canvas snapshots)

**Why not raw `ws` library:**
- No rooms, no auto-reconnection, no fallback
- Need to build all of these manually

| Criteria | Socket.IO | ws | Ably/Pusher |
|----------|----------|----|----|
| Rooms | Built-in | Manual | Built-in |
| Auto-reconnect | ✅ | Manual | ✅ |
| Scaling | Redis adapter | Manual | Managed |
| Cost | Free | Free | $$ |
| Fallback | Long-polling | None | Yes |
| Binary support | ✅ | ✅ | Limited |

---

### 6. Database — MongoDB + Mongoose

**Why MongoDB:**
- Drawing data is deeply nested JSON (Fabric.js `canvas.toJSON()`)
- Schema flexibility — drawing objects vary by type (rect vs path vs text)
- Good performance for write-heavy workloads (drawing saves)
- MongoDB Atlas provides managed hosting, backups, monitoring
- Change Streams for real-time notifications (future use)

**Why not PostgreSQL:**
- JSONB in Postgres works but adds complexity for deeply nested canvas data
- Drawing data doesn't need relational joins
- Schema migrations add friction during rapid iteration

**Why Mongoose:**
- Schema validation at application level
- Middleware hooks (pre-save, post-remove)
- Population (join-like) for user → drawings queries
- TypeScript support via `InferSchemaType`

---

### 7. Cache & Pub/Sub — Redis

**Purpose in DrawSaver:**

| Use Case | How |
|----------|-----|
| Socket.IO scaling | `@socket.io/redis-adapter` for cross-server event broadcast |
| Rate limiting | Sliding window counter via `rate-limit-redis` |
| Session cache | Store active room state (participants, last activity) |
| Canvas buffer | Buffer drawing actions before MongoDB flush (30s interval) |
| User presence | Track online/idle status with key expiry |

---

### 8. Authentication — Passport.js + JWT + bcryptjs

**Strategy:**
- **Access Token:** JWT, 15-minute expiry, sent in `Authorization: Bearer` header
- **Refresh Token:** JWT, 7-day expiry, stored in HTTP-only cookie
- **OAuth:** Passport.js strategies for Google and GitHub
- **Password hashing:** bcryptjs with 12 salt rounds

**Why not NextAuth/Clerk/Auth0:**
- Vendor lock-in for a core feature
- Custom JWT gives full control over token claims
- OAuth via Passport.js is straightforward
- No monthly cost

---

### 9. File Storage — AWS S3

**Why S3:**
- Industry standard, 99.999999999% durability
- Presigned URLs for direct client uploads (no server bottleneck)
- Lifecycle policies for automatic cleanup of old exports
- CloudFront CDN integration
- S3-compatible alternatives (MinIO, Cloudflare R2) for cost optimization later

**For local development:** Use MinIO (S3-compatible, runs in Docker).

---

### 10. Deployment — Docker + Render/Railway

**MVP Deployment:**
- **Frontend:** Vercel (free tier, instant deploys from Git)
- **Backend:** Render (free → $7/mo, Docker support, managed TLS)
- **Database:** MongoDB Atlas (M0 free tier → M10 for production)
- **Redis:** Redis Cloud (free 30MB tier)
- **S3:** AWS S3 (or Cloudflare R2 for cost savings)

**Why Docker:**
- Consistent development/production environments
- Easy horizontal scaling
- Required for Render/Railway deployment
- Simplifies CI/CD pipeline

---

### 11. Developer Experience

| Tool | Purpose |
|------|---------|
| **TypeScript** | Type safety across frontend + backend (shared types) |
| **ESLint + Prettier** | Code quality and formatting |
| **Husky + lint-staged** | Pre-commit hooks |
| **Vitest** | Fast unit testing (Vite-native) |
| **Playwright** | E2E testing for critical flows |
| **Pino** | Structured JSON logging (backend) |
| **Sentry** | Error tracking and performance monitoring |
| **dotenv** | Environment variable management |

---

## Package List (Estimated)

### Frontend (`package.json`)
```json
{
  "dependencies": {
    "react": "^18.3",
    "react-dom": "^18.3",
    "react-router-dom": "^6.x",
    "fabric": "^6.x",
    "zustand": "^4.x",
    "socket.io-client": "^4.x",
    "axios": "^1.x",
    "react-hot-toast": "^2.x",
    "lucide-react": "^0.x"
  },
  "devDependencies": {
    "vite": "^5.x",
    "@vitejs/plugin-react": "^4.x",
    "tailwindcss": "^3.x",
    "typescript": "^5.x",
    "vitest": "^1.x",
    "@playwright/test": "^1.x"
  }
}
```

### Backend (`package.json`)
```json
{
  "dependencies": {
    "express": "^4.x",
    "socket.io": "^4.x",
    "@socket.io/redis-adapter": "^8.x",
    "mongoose": "^8.x",
    "redis": "^4.x",
    "passport": "^0.7",
    "passport-google-oauth20": "^2.x",
    "passport-github2": "^0.1",
    "jsonwebtoken": "^9.x",
    "bcryptjs": "^2.x",
    "zod": "^3.x",
    "@aws-sdk/client-s3": "^3.x",
    "multer": "^1.x",
    "helmet": "^7.x",
    "cors": "^2.x",
    "express-rate-limit": "^7.x",
    "rate-limit-redis": "^4.x",
    "pino": "^8.x",
    "pino-pretty": "^10.x",
    "dotenv": "^16.x"
  },
  "devDependencies": {
    "typescript": "^5.x",
    "tsx": "^4.x",
    "vitest": "^1.x",
    "nodemon": "^3.x"
  }
}
```
