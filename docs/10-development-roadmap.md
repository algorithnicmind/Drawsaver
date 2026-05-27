# DrawSaver — Development Roadmap

> **Version:** 1.0  
> **Last Updated:** 2026-05-27  

---

## Timeline Overview

| Phase | Duration | Focus | Target |
|-------|----------|-------|--------|
| **Phase 1 — MVP** | Weeks 1–4 | Auth + Canvas + Basic Save | Working solo drawing app |
| **Phase 2 — Collaboration** | Weeks 5–7 | WebSocket + Rooms + Real-time | Multi-user drawing |
| **Phase 3 — Polish** | Weeks 8–10 | History, Export, Layers, UX | Feature-complete beta |
| **Phase 4 — Production** | Weeks 11–12 | DevOps, Testing, Optimization | Production deploy |
| **Phase 5 — AI & Scale** | Months 4–6+ | AI features, Mobile, Enterprise | Growth phase |

---

## Phase 1 — MVP Foundation (Weeks 1–4)

### Week 1: Project Setup & Auth

| Task | Priority | Est. Hours |
|------|----------|------------|
| Initialize Vite + React frontend project | P0 | 2h |
| Initialize Express + Node.js backend project | P0 | 2h |
| Set up MongoDB Atlas + Mongoose connection | P0 | 2h |
| Set up Redis connection | P0 | 1h |
| Docker Compose for local dev (MongoDB, Redis, MinIO) | P1 | 3h |
| Implement User model + registration endpoint | P0 | 3h |
| Implement login endpoint + JWT generation | P0 | 3h |
| Implement refresh token rotation | P0 | 3h |
| Implement Google OAuth (Passport.js) | P0 | 4h |
| Frontend: Login + Register pages | P0 | 4h |
| Frontend: Auth context + token management | P0 | 3h |
| Frontend: Protected route wrapper | P0 | 1h |
| **Week 1 Deliverable:** User can register, login, and access protected dashboard | | **~31h** |

### Week 2: Canvas Engine

| Task | Priority | Est. Hours |
|------|----------|------------|
| Integrate Fabric.js canvas in React | P0 | 4h |
| Implement pencil/brush tool | P0 | 3h |
| Implement eraser tool | P0 | 2h |
| Implement shape tools (rect, circle, line) | P0 | 4h |
| Implement text tool | P1 | 3h |
| Implement color picker component | P0 | 3h |
| Implement brush size + opacity sliders | P0 | 2h |
| Implement undo/redo stack | P0 | 4h |
| Implement zoom + pan controls | P1 | 3h |
| Implement keyboard shortcuts | P1 | 2h |
| Build toolbar UI component | P0 | 3h |
| Canvas responsive sizing | P0 | 2h |
| **Week 2 Deliverable:** Fully functional single-user canvas with all core tools | | **~35h** |

### Week 3: Rooms & Persistence

| Task | Priority | Est. Hours |
|------|----------|------------|
| Room model + CRUD API endpoints | P0 | 4h |
| Drawing model + save/load API endpoints | P0 | 4h |
| Auto-save hook (debounced, every 30s) | P0 | 3h |
| Manual save trigger (Ctrl+S) | P0 | 1h |
| Create Room modal (frontend) | P0 | 3h |
| Dashboard page: list user's drawings | P0 | 4h |
| Drawing card component with thumbnails | P0 | 3h |
| Load saved drawing into canvas on room open | P0 | 3h |
| S3 setup for thumbnail storage | P1 | 3h |
| Room invite code generation + join flow | P0 | 3h |
| **Week 3 Deliverable:** Users can create rooms, save/load drawings, see dashboard | | **~31h** |

### Week 4: MVP Polish

| Task | Priority | Est. Hours |
|------|----------|------------|
| Landing page design + implementation | P0 | 6h |
| Profile page (view/edit) | P1 | 3h |
| Avatar upload to S3 | P2 | 2h |
| Toast notifications system | P0 | 2h |
| Loading states + skeleton components | P0 | 3h |
| Error boundary + 404 page | P0 | 2h |
| Export as PNG (client-side Fabric.js) | P0 | 2h |
| Mobile responsive adjustments | P1 | 4h |
| Bug fixes + QA pass | P0 | 6h |
| **Week 4 Deliverable:** Polished MVP with auth, canvas, save, export, responsive UI | | **~30h** |

### 🏁 MVP Milestone Checklist
- [ ] User registration + login (email + Google OAuth)
- [ ] Canvas with pencil, brush, eraser, shapes, text, color, undo/redo
- [ ] Create/join rooms
- [ ] Save and load drawings
- [ ] Auto-save
- [ ] Export as PNG
- [ ] Dashboard with drawing cards
- [ ] Landing page
- [ ] Basic responsive layout

---

## Phase 2 — Real-Time Collaboration (Weeks 5–7)

### Week 5: WebSocket Foundation

| Task | Priority | Est. Hours |
|------|----------|------------|
| Socket.IO server setup + JWT auth middleware | P0 | 4h |
| Socket.IO client setup + connection lifecycle | P0 | 3h |
| Room join/leave events | P0 | 3h |
| Send full canvas state to joining user | P0 | 3h |
| Broadcast draw-action events (object:added/modified/removed) | P0 | 6h |
| Receive and apply remote draw actions on canvas | P0 | 6h |
| Object ID system for cross-client identification | P0 | 3h |
| **Week 5 Deliverable:** Two users can draw on the same canvas in real-time | | **~28h** |

### Week 6: Presence & Cursors

| Task | Priority | Est. Hours |
|------|----------|------------|
| Cursor position broadcast (throttled 50ms) | P0 | 3h |
| Remote cursor rendering (SVG pointer + label) | P0 | 4h |
| Cursor color assignment per user | P0 | 1h |
| Cursor cleanup on disconnect | P0 | 1h |
| Presence system (active/idle/offline) | P0 | 4h |
| Users panel component (shows participants) | P0 | 3h |
| Reconnection handling with state recovery | P0 | 5h |
| Visual object locking indicators | P1 | 3h |
| Concurrent edit stress testing (5+ users) | P0 | 4h |
| **Week 6 Deliverable:** Cursors, presence, and stable multi-user experience | | **~28h** |

### Week 7: Collaboration Polish

| Task | Priority | Est. Hours |
|------|----------|------------|
| Redis adapter for Socket.IO (multi-server prep) | P1 | 3h |
| Event batching for freehand drawing | P0 | 3h |
| Bandwidth optimization (delta updates) | P0 | 4h |
| Connection status UI (connected/reconnecting/offline) | P0 | 2h |
| Offline queuing (buffer actions during disconnect) | P1 | 4h |
| Room settings UI (visibility, max participants) | P1 | 3h |
| Kick user functionality (owner only) | P2 | 2h |
| Collaboration integration testing | P0 | 5h |
| **Week 7 Deliverable:** Production-quality real-time collaboration | | **~26h** |

---

## Phase 3 — Advanced Features (Weeks 8–10)

### Week 8: Version History & Layers

| Task | Priority | Est. Hours |
|------|----------|------------|
| DrawingVersion model + API endpoints | P1 | 4h |
| Create version snapshot on manual save | P1 | 3h |
| Version history panel UI | P1 | 4h |
| Version preview + restore functionality | P1 | 4h |
| Layer system implementation in Fabric.js | P2 | 6h |
| Layers panel UI (add, delete, reorder, visibility) | P2 | 5h |
| Layer sync across collaborators | P2 | 4h |
| **Week 8 Deliverable:** Version history + layers system | | **~30h** |

### Week 9: Export & Sharing

| Task | Priority | Est. Hours |
|------|----------|------------|
| Export as JPG with quality settings | P1 | 2h |
| Export as SVG | P1 | 3h |
| Export as PDF (with page sizing) | P1 | 4h |
| Export modal UI with format/quality options | P1 | 3h |
| Server-side export via BullMQ job | P1 | 4h |
| Public room listing page | P2 | 3h |
| Social share links (copy URL, QR code) | P2 | 3h |
| Email invitation system | P2 | 4h |
| **Week 9 Deliverable:** Full export suite + sharing options | | **~26h** |

### Week 10: UX Polish & Performance

| Task | Priority | Est. Hours |
|------|----------|------------|
| Animation polish (page transitions, micro-interactions) | P1 | 4h |
| Dark/light theme toggle | P2 | 3h |
| Canvas performance optimization (large object counts) | P0 | 5h |
| Lazy loading for dashboard images | P1 | 2h |
| Accessibility audit (keyboard nav, ARIA labels) | P1 | 4h |
| Cross-browser testing (Chrome, Firefox, Safari, Edge) | P0 | 4h |
| Mobile touch interaction refinement | P1 | 4h |
| Error tracking setup (Sentry) | P1 | 2h |
| **Week 10 Deliverable:** Polished, performant, cross-browser UX | | **~28h** |

---

## Phase 4 — Production Readiness (Weeks 11–12)

### Week 11: Testing & Security

| Task | Priority | Est. Hours |
|------|----------|------------|
| Unit tests: auth service, room service | P0 | 6h |
| Unit tests: drawing service, validation | P0 | 4h |
| Integration tests: API endpoints | P0 | 6h |
| E2E tests: auth flow, drawing flow (Playwright) | P0 | 6h |
| WebSocket stress testing | P0 | 3h |
| Security audit (OWASP top 10 checklist) | P0 | 4h |
| Penetration testing (auth, injection) | P1 | 3h |
| **Week 11 Deliverable:** Comprehensive test coverage + security validation | | **~32h** |

### Week 12: DevOps & Launch

| Task | Priority | Est. Hours |
|------|----------|------------|
| Dockerfile for backend | P0 | 2h |
| CI/CD pipeline (GitHub Actions) | P0 | 4h |
| Staging environment deployment | P0 | 3h |
| Production environment deployment | P0 | 4h |
| Environment variable management | P0 | 2h |
| Monitoring + alerting setup | P0 | 3h |
| Logging configuration (structured JSON) | P0 | 2h |
| Domain + SSL setup | P0 | 2h |
| README documentation | P0 | 3h |
| Launch checklist verification | P0 | 3h |
| **Week 12 Deliverable:** Production deployed + monitored | | **~28h** |

---

## Phase 5 — AI & Growth (Months 4–6+)

| Feature | Priority | Est. Effort | Dependencies |
|---------|----------|-------------|-------------|
| AI shape auto-correction | P2 | 2 weeks | TensorFlow.js or cloud ML API |
| AI sketch-to-clean-art | P3 | 3 weeks | Stable Diffusion API |
| AI smart color suggestions | P3 | 1 week | OpenAI API |
| Voice/video chat in rooms | P3 | 3 weeks | WebRTC |
| Mobile app (React Native) | P2 | 6 weeks | Shared API |
| Admin dashboard | P2 | 2 weeks | New frontend route + analytics DB |
| Public gallery + community | P3 | 3 weeks | Social features |
| Payment integration (Stripe) | P2 | 2 weeks | Pricing model |
| Enterprise SSO (SAML) | P3 | 2 weeks | Enterprise sales |

---

## Feature Priority Matrix

```
                    High Impact
                        │
         ┌──────────────┼──────────────┐
         │              │              │
         │  P0: DO NOW  │  P1: NEXT    │
         │  ─────────── │  ──────────  │
    Low  │  Auth        │  Layers      │  High
   Effort│  Canvas tools│  SVG export  │  Effort
         │  Basic rooms │  Version     │
         │  Auto-save   │  history     │
         │  PNG export  │  OAuth       │
         │              │  GitHub      │
         ├──────────────┼──────────────┤
         │              │              │
         │  P2: LATER   │  P3: MAYBE   │
         │  ─────────── │  ──────────  │
         │  Dark mode   │  AI features │
         │  QR sharing  │  Mobile app  │
         │  Admin panel │  Video chat  │
         │              │  Community   │
         │              │              │
         └──────────────┼──────────────┘
                        │
                    Low Impact
```
