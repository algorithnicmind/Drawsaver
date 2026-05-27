# DrawSaver — Product Requirement Document (PRD)

> **Version:** 1.0  
> **Last Updated:** 2026-05-27  
> **Status:** Active  
> **Owner:** DrawSaver Engineering

---

## 1. Vision

DrawSaver is a real-time collaborative drawing platform that enables individuals and teams to create, share, and persist digital drawings on a shared canvas. It aims to be the **"Google Docs of visual collaboration"** — instant, intuitive, and multiplayer-first.

**Core Value Proposition:** Zero-friction collaborative drawing with persistent history, real-time sync, and future AI-powered creative assistance.

---

## 2. Goals

| Goal | Metric | Target |
|------|--------|--------|
| Real-time collaboration | Sync latency | < 100ms P95 |
| User adoption | MAU after 6 months | 10,000+ |
| Drawing persistence | Data durability | 99.99% |
| Performance | Canvas FPS | 60fps with 10+ concurrent users |
| Availability | Uptime SLA | 99.9% |
| Time to MVP | Delivery | 8–10 weeks |

---

## 3. Target Users

| Persona | Use Case | Priority |
|---------|----------|----------|
| **Students** | Collaborative study diagrams, brainstorming | High |
| **Designers** | Quick sketching, wireframing with teammates | High |
| **Remote Teams** | Whiteboard-style meetings, visual planning | High |
| **Educators** | Live teaching on shared canvas | Medium |
| **Artists** | Casual collaborative art | Medium |
| **Content Creators** | Thumbnail sketching, storyboarding | Low |

---

## 4. User Stories

### Authentication
| ID | Story | Priority |
|----|-------|----------|
| US-01 | As a user, I can register with email/password so I can create an account | P0 |
| US-02 | As a user, I can login with Google/GitHub OAuth so I can skip manual registration | P0 |
| US-03 | As a user, I can reset my password via email link | P1 |
| US-04 | As a user, I can edit my profile (name, avatar) | P2 |

### Drawing
| ID | Story | Priority |
|----|-------|----------|
| US-10 | As a user, I can draw freehand on a canvas using brush/pencil tools | P0 |
| US-11 | As a user, I can select brush size and color | P0 |
| US-12 | As a user, I can use an eraser tool | P0 |
| US-13 | As a user, I can draw shapes (rectangle, circle, line, arrow) | P0 |
| US-14 | As a user, I can add text to the canvas | P1 |
| US-15 | As a user, I can undo/redo my actions | P0 |
| US-16 | As a user, I can zoom and pan on the canvas | P1 |
| US-17 | As a user, I can work with multiple layers | P2 |
| US-18 | As a user, I can use fill colors and gradients | P2 |

### Collaboration
| ID | Story | Priority |
|----|-------|----------|
| US-20 | As a user, I can create a new drawing room | P0 |
| US-21 | As a user, I can join a room via invite link | P0 |
| US-22 | As a user, I can see other users' cursors in real-time | P0 |
| US-23 | As a user, I can see who is currently in the room (presence) | P0 |
| US-24 | As a user, I can draw simultaneously with others on the same canvas | P0 |
| US-25 | As a room owner, I can set the room as public or private | P1 |
| US-26 | As a room owner, I can kick users from the room | P2 |

### Persistence & Export
| ID | Story | Priority |
|----|-------|----------|
| US-30 | As a user, I can save my drawing to the cloud | P0 |
| US-31 | As a user, I can view my saved drawings on the dashboard | P0 |
| US-32 | As a user, I can download my drawing as PNG/JPG | P0 |
| US-33 | As a user, I can export as SVG/PDF | P1 |
| US-34 | As a user, I can browse version history and restore previous versions | P1 |
| US-35 | As a user, my drawing auto-saves periodically | P0 |

---

## 5. Functional Requirements

### FR-1: Authentication System
- Email/password registration with email verification
- OAuth 2.0 via Google and GitHub
- JWT-based session management (access + refresh tokens)
- Password reset via email with time-limited tokens
- Profile management (username, avatar)

### FR-2: Drawing Engine
- HTML5 Canvas-based rendering via Fabric.js/Konva.js
- Tools: pencil, brush, eraser, shapes (rect, circle, ellipse, line, arrow), text
- Color picker with recent colors, opacity control
- Stroke width slider (1–50px)
- Undo/redo stack (minimum 50 states)
- Zoom (25%–400%) and pan with scroll/drag
- Layer management: add, delete, reorder, toggle visibility, rename
- Canvas clear with confirmation dialog

### FR-3: Room Management
- Create room → generates unique room ID and shareable URL
- Join room via URL or room code
- Room settings: name, visibility (public/private), max participants (default: 20)
- Room owner can manage participants (kick, promote)
- Room lobby showing current participants

### FR-4: Real-Time Collaboration
- WebSocket connection per room using Socket.IO
- Broadcast drawing operations as delta events (not full canvas snapshots)
- Cursor position sharing with user labels
- User presence indicators (online/idle/drawing)
- Automatic reconnection with state recovery on disconnect
- Conflict resolution via operation ordering (last-write-wins for MVP, CRDT for v2)

### FR-5: Persistence
- Auto-save every 30 seconds and on significant canvas changes
- Manual save trigger
- Version history with snapshots (max 50 versions per drawing)
- Drawing metadata storage (title, thumbnail, created/updated timestamps)
- Cloud image storage for exported assets

### FR-6: Export
- Export canvas as PNG (transparent or white background)
- Export as JPG (configurable quality)
- Export as SVG (vector format)
- Export as PDF (A4/letter sizing)

---

## 6. Non-Functional Requirements

| Category | Requirement | Target |
|----------|-------------|--------|
| **Performance** | Canvas render loop | 60fps |
| **Performance** | WebSocket event latency | < 100ms P95 |
| **Performance** | API response time | < 200ms P95 |
| **Performance** | Initial page load (LCP) | < 2.5s |
| **Scalability** | Concurrent users per room | 20 |
| **Scalability** | Total concurrent rooms | 1,000+ |
| **Scalability** | Drawing data per canvas | Up to 50MB |
| **Security** | Authentication | JWT + OAuth 2.0 |
| **Security** | Data in transit | TLS 1.3 |
| **Security** | Data at rest | AES-256 encryption |
| **Security** | Rate limiting | 100 req/min per user |
| **Reliability** | Uptime | 99.9% |
| **Reliability** | Data durability | 99.99% |
| **Reliability** | Auto-save reliability | No data loss on crash |
| **Compatibility** | Browsers | Chrome, Firefox, Safari, Edge (latest 2 versions) |
| **Compatibility** | Devices | Desktop-first, responsive tablet support |

---

## 7. MVP Scope (Phase 1)

### In Scope ✅
- Email/password + Google OAuth authentication
- Single-canvas drawing with core tools (pencil, eraser, shapes, color, stroke width)
- Undo/redo
- Create and join rooms
- Real-time multiplayer drawing via WebSocket
- Cursor sharing and user presence
- Manual save + auto-save
- Export as PNG
- Dashboard with saved drawings
- Basic responsive layout

### Out of Scope for MVP ❌
- Layers system
- SVG/PDF export
- Version history with restore
- AI features
- Voice/video collaboration
- Mobile native apps
- Admin dashboard
- Payment/subscription system
- Public gallery/community features

---

## 8. Success Criteria

| Milestone | Criteria |
|-----------|----------|
| MVP Launch | Auth + drawing + rooms + real-time sync + save/export working end-to-end |
| Beta | 100+ active users, < 5 critical bugs, < 100ms sync latency |
| v1.0 | Full feature set (layers, versioning, all exports), 1000+ MAU |
| v2.0 | AI features, mobile apps, enterprise tier |
