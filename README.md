<p align="center">
  <img src="https://img.shields.io/badge/🎨-DrawSaver-6c5ce7?style=for-the-badge&labelColor=0f0f23" alt="DrawSaver" />
</p>

<h1 align="center">DrawSaver</h1>

<p align="center">
  <strong>Create. Collaborate. Save Forever.</strong>
</p>

<p align="center">
  A real-time collaborative drawing platform — the <b>Google Docs of visual collaboration</b>.
  <br />
  Instant. Intuitive. Multiplayer-first.
</p>

<p align="center">
  <a href="#-getting-started"><img src="https://img.shields.io/badge/Get_Started-6c5ce7?style=for-the-badge&logo=rocket&logoColor=white" alt="Get Started" /></a>
  <a href="#-documentation"><img src="https://img.shields.io/badge/Docs-00cec9?style=for-the-badge&logo=readthedocs&logoColor=white" alt="Documentation" /></a>
  <a href="#-roadmap"><img src="https://img.shields.io/badge/Roadmap-fdcb6e?style=for-the-badge&logo=target&logoColor=black" alt="Roadmap" /></a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/License-Apache%202.0-blue?style=flat-square" alt="License" />
  <img src="https://img.shields.io/badge/PRs-Welcome-brightgreen?style=flat-square" alt="PRs Welcome" />
  <img src="https://img.shields.io/badge/Node.js-20+-339933?style=flat-square&logo=node.js&logoColor=white" alt="Node.js" />
  <img src="https://img.shields.io/badge/React-18-61DAFB?style=flat-square&logo=react&logoColor=black" alt="React" />
  <img src="https://img.shields.io/badge/Socket.IO-4-010101?style=flat-square&logo=socket.io&logoColor=white" alt="Socket.IO" />
  <img src="https://img.shields.io/badge/MongoDB-7-47A248?style=flat-square&logo=mongodb&logoColor=white" alt="MongoDB" />
</p>

---

## 📖 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [System Architecture](#-system-architecture)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Getting Started](#-getting-started)
- [Environment Variables](#-environment-variables)
- [Development Commands](#-development-commands)
- [How Real-Time Collaboration Works](#-how-real-time-collaboration-works)
- [Deployment](#-deployment)
- [Roadmap](#-roadmap)
- [Documentation](#-documentation)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🌟 Overview

DrawSaver empowers individuals and teams to **draw, brainstorm, and collaborate visually** — all in real-time on a shared canvas. Whether you're a student sketching diagrams, a designer wireframing, or a remote team whiteboarding, DrawSaver gives you a seamless multiplayer creative workspace.

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│    🎯 Problem         →    💡 Solution                       │
│                                                              │
│    Fragmented visual   →    One platform for drawing,        │
│    collaboration            collaborating, and saving        │
│    tools with no            — instantly, with anyone,        │
│    real-time sync           anywhere.                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## ✨ Features

<table>
<tr>
<td width="50%">

### 🖌️ Drawing Engine
- **Pencil & Brush** — Freehand with pressure sensitivity
- **Shapes** — Rectangle, Circle, Line, Arrow
- **Text Tool** — Add and edit text on canvas
- **Eraser** — Remove objects with a click
- **Color Picker** — Full picker + opacity + recent colors
- **Undo / Redo** — 50+ state history stack
- **Zoom & Pan** — 25%–400% with smooth scroll
- **Layers** — Add, reorder, toggle visibility

</td>
<td width="50%">

### 👥 Real-Time Collaboration
- **Live Multi-User Drawing** — Instant sync < 100ms
- **Remote Cursors** — See labeled cursors of others
- **Presence Indicators** — Active / Idle / Offline status
- **Room System** — Create, join via link or code
- **Visual Locking** — See when someone is editing an object
- **Auto-Reconnect** — Seamless recovery on disconnect
- **Conflict Handling** — Last-write-wins (CRDT in v2)

</td>
</tr>
<tr>
<td width="50%">

### 💾 Persistence & Export
- **Auto-Save** — Every 30 seconds + on major changes
- **Version History** — Browse & restore up to 50 versions
- **Export Formats** — PNG, JPG, SVG, PDF
- **Cloud Storage** — Drawings persist in the cloud
- **Thumbnails** — Auto-generated previews on dashboard

</td>
<td width="50%">

### 🔐 Authentication & Security
- **JWT Auth** — Access + Refresh token rotation
- **OAuth 2.0** — Google & GitHub sign-in
- **Rate Limiting** — Redis-backed sliding window
- **Input Validation** — Zod schemas on every endpoint
- **Security Headers** — Helmet CSP + CORS
- **RBAC** — Owner / Editor / Viewer roles per room

</td>
</tr>
</table>

---

## 🏗️ System Architecture

```mermaid
graph TB
    subgraph Clients["🖥️ Clients"]
        B1["Browser A<br/>(React + Fabric.js)"]
        B2["Browser B<br/>(React + Fabric.js)"]
        B3["Browser C<br/>(React + Fabric.js)"]
    end

    subgraph Edge["🌐 Edge Layer"]
        CDN["CDN / Vercel Edge<br/>Static Assets"]
    end

    subgraph Backend["⚙️ Backend Services"]
        API["REST API Server<br/>(Express.js)"]
        WS["WebSocket Server<br/>(Socket.IO)"]
    end

    subgraph Data["🗄️ Data Layer"]
        DB["MongoDB Atlas<br/>(Primary Store)"]
        CACHE["Redis<br/>(Cache / PubSub)"]
        S3["AWS S3<br/>(File Storage)"]
    end

    B1 & B2 & B3 -->|"HTTPS"| CDN
    B1 & B2 & B3 -->|"REST API"| API
    B1 & B2 & B3 <-->|"WebSocket"| WS

    API --> DB
    API --> CACHE
    API --> S3
    WS --> CACHE
    WS --> DB

    style Clients fill:#1a1a2e,stroke:#6c5ce7,color:#f8f9fa
    style Edge fill:#16213e,stroke:#00cec9,color:#f8f9fa
    style Backend fill:#1a1a2e,stroke:#6c5ce7,color:#f8f9fa
    style Data fill:#16213e,stroke:#00cec9,color:#f8f9fa
```

### Data Flow — Drawing Sync

```mermaid
sequenceDiagram
    participant A as 👤 User A
    participant S as 🖥️ Server
    participant R as 📦 Redis
    participant B as 👤 User B

    A->>S: draw-action (rect added)
    S->>S: Validate + timestamp
    S->>R: Buffer action
    S->>B: Broadcast draw-action
    B->>B: Apply to local canvas
    
    Note over R,S: Every 30s
    S->>S: Flush buffer → MongoDB
```

---

## 🛠️ Tech Stack

<table>
<tr>
<th align="left">Layer</th>
<th align="left">Technology</th>
<th align="left">Why</th>
</tr>
<tr><td>⚛️ <b>Frontend</b></td><td>React 18 + Vite</td><td>Fast HMR, component model, largest ecosystem</td></tr>
<tr><td>🎨 <b>Canvas</b></td><td>Fabric.js</td><td>Object model, built-in serialization, rich events</td></tr>
<tr><td>📦 <b>State</b></td><td>Zustand</td><td>Minimal boilerplate, works outside React (canvas events)</td></tr>
<tr><td>💅 <b>Styling</b></td><td>Tailwind CSS</td><td>Utility-first, rapid UI development</td></tr>
<tr><td>🟢 <b>Runtime</b></td><td>Node.js 20 LTS</td><td>Same language as frontend, non-blocking I/O</td></tr>
<tr><td>🚂 <b>API</b></td><td>Express.js</td><td>Mature, widest middleware ecosystem</td></tr>
<tr><td>🔌 <b>Real-Time</b></td><td>Socket.IO</td><td>Rooms, auto-reconnect, fallback, Redis adapter</td></tr>
<tr><td>🍃 <b>Database</b></td><td>MongoDB + Mongoose</td><td>Flexible schema fits canvas JSON, horizontal scaling</td></tr>
<tr><td>⚡ <b>Cache</b></td><td>Redis</td><td>PubSub for Socket.IO scaling, rate limiting, sessions</td></tr>
<tr><td>☁️ <b>Storage</b></td><td>AWS S3</td><td>99.999999999% durability, presigned uploads, CDN-ready</td></tr>
<tr><td>🔐 <b>Auth</b></td><td>Passport.js + JWT</td><td>OAuth strategies, full control over token flow</td></tr>
<tr><td>✅ <b>Testing</b></td><td>Vitest + Playwright</td><td>Fast unit tests + reliable E2E browser tests</td></tr>
<tr><td>🚀 <b>CI/CD</b></td><td>GitHub Actions</td><td>Native GitHub integration, matrix builds</td></tr>
<tr><td>📡 <b>Deploy</b></td><td>Vercel + Render</td><td>Zero-config frontend CDN + Docker backend hosting</td></tr>
</table>

---

## 📁 Project Structure

```
DrawSaver/
│
├── 📂 docs/                    # 13 production-grade documentation files
│
├── 📂 frontend/                # React + Vite SPA
│   └── src/
│       ├── components/         # UI: canvas/, collaboration/, layout/, ui/
│       ├── pages/              # LandingPage, Dashboard, DrawingWorkspace, ...
│       ├── hooks/              # useCanvas, useSocket, useAuth, useAutoSave
│       ├── store/              # Zustand: canvasStore, roomStore, authStore
│       ├── services/           # API clients (Axios + interceptors)
│       ├── socket/             # Socket.IO client + event handlers
│       └── utils/              # Helpers, constants, throttle
│
├── 📂 backend/                 # Node.js + Express API
│   └── src/
│       ├── config/             # MongoDB, Redis, S3, Passport, env validation
│       ├── models/             # User, Room, Drawing, DrawingVersion
│       ├── routes/             # /auth, /rooms, /drawings, /users
│       ├── controllers/        # Thin request handlers
│       ├── services/           # Business logic layer
│       ├── middleware/         # authGuard, validate, rateLimiter, errorHandler
│       ├── socket/             # Room, drawing, cursor, presence handlers
│       ├── validators/         # Zod schemas
│       └── utils/              # Logger, error class, token helpers
│
├── 📂 shared/                  # Shared constants + types
├── 📂 .github/workflows/      # CI/CD pipelines
├── 🐳 docker-compose.yml      # Local dev: MongoDB + Redis + MinIO
└── 📄 README.md
```

---

## 🚀 Getting Started

### Prerequisites

| Tool | Version | Download |
|------|---------|----------|
| Node.js | 20+ | [nodejs.org](https://nodejs.org/) |
| Docker | Latest | [docker.com](https://www.docker.com/) |
| Git | Latest | [git-scm.com](https://git-scm.com/) |

### Quick Start

```mermaid
graph LR
    A["1️⃣ Clone Repo"] --> B["2️⃣ Start Docker"]
    B --> C["3️⃣ Setup Backend"]
    C --> D["4️⃣ Setup Frontend"]
    D --> E["5️⃣ Open App 🎉"]

    style A fill:#6c5ce7,stroke:#6c5ce7,color:#fff
    style B fill:#00cec9,stroke:#00cec9,color:#fff
    style C fill:#6c5ce7,stroke:#6c5ce7,color:#fff
    style D fill:#00cec9,stroke:#00cec9,color:#fff
    style E fill:#00b894,stroke:#00b894,color:#fff
```

#### Step 1 — Clone the repository

```bash
git clone https://github.com/algorithnicmind/DrawSaver.git
cd DrawSaver
```

#### Step 2 — Start infrastructure (MongoDB, Redis, MinIO)

```bash
docker compose up -d
```

> This starts MongoDB (`localhost:27017`), Redis (`localhost:6379`), and MinIO S3 (`localhost:9000`).

#### Step 3 — Setup & start the backend

```bash
cd backend
cp .env.example .env       # Configure your environment
npm install
npm run dev                # → http://localhost:5000
```

#### Step 4 — Setup & start the frontend

```bash
cd frontend
cp .env.example .env       # Set API URL
npm install
npm run dev                # → http://localhost:5173
```

#### Step 5 — Open the app

Navigate to **[http://localhost:5173](http://localhost:5173)** and start drawing! 🎨

---

## ⚙️ Environment Variables

<details>
<summary><b>🔧 Backend <code>.env</code></b> (click to expand)</summary>

```env
# Server
NODE_ENV=development
PORT=5000

# Database
MONGODB_URI=mongodb://admin:devpassword@localhost:27017/drawsaver?authSource=admin
REDIS_URL=redis://localhost:6379

# Authentication
JWT_ACCESS_SECRET=your-access-secret-min-32-chars
JWT_REFRESH_SECRET=your-refresh-secret-min-32-chars

# OAuth
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
GITHUB_CLIENT_ID=your-github-client-id
GITHUB_CLIENT_SECRET=your-github-client-secret

# Storage (S3/MinIO)
S3_ENDPOINT=http://localhost:9000
S3_ACCESS_KEY=minioadmin
S3_SECRET_KEY=minioadmin
S3_BUCKET=drawsaver-assets

# CORS
FRONTEND_URL=http://localhost:5173
```

</details>

<details>
<summary><b>⚛️ Frontend <code>.env</code></b> (click to expand)</summary>

```env
VITE_API_URL=http://localhost:5000/api/v1
VITE_WS_URL=http://localhost:5000
VITE_GOOGLE_CLIENT_ID=your-google-client-id
```

</details>

---

## 📋 Development Commands

<details>
<summary><b>Backend Commands</b></summary>

| Command | Description |
|:--------|:------------|
| `npm run dev` | Start dev server with hot reload |
| `npm run start` | Start production server |
| `npm run test` | Run unit tests (Vitest) |
| `npm run lint` | Run ESLint |
| `npm run lint:fix` | Auto-fix lint issues |

</details>

<details>
<summary><b>Frontend Commands</b></summary>

| Command | Description |
|:--------|:------------|
| `npm run dev` | Start Vite dev server (HMR) |
| `npm run build` | Production build |
| `npm run preview` | Preview production build |
| `npm run test` | Run Vitest unit tests |
| `npm run test:e2e` | Run Playwright E2E tests |
| `npm run lint` | Run ESLint |

</details>

<details>
<summary><b>Docker Commands</b></summary>

| Command | Description |
|:--------|:------------|
| `docker compose up -d` | Start all services |
| `docker compose down` | Stop all services |
| `docker compose down -v` | Stop + remove volumes (full reset) |
| `docker compose logs -f` | Tail all service logs |
| `docker compose ps` | Check service status |

</details>

---

## 🔄 How Real-Time Collaboration Works

```mermaid
graph TD
    subgraph UserA["👤 User A"]
        A1["Draws rectangle<br/>on local canvas"]
        A2["Fabric.js fires<br/>object:added"]
        A3["Serialize object<br/>to JSON"]
        A4["Emit draw-action<br/>via Socket.IO"]
    end

    subgraph Server["🖥️ Socket.IO Server"]
        S1["Receive &<br/>validate action"]
        S2["Add server<br/>timestamp"]
        S3["Buffer in<br/>Redis"]
        S4["Broadcast to<br/>room (excl. sender)"]
    end

    subgraph UserB["👤 User B"]
        B1["Receive<br/>draw-action"]
        B2["Deserialize<br/>object"]
        B3["Add to local<br/>canvas"]
        B4["Render ✅"]
    end

    A1 --> A2 --> A3 --> A4
    A4 -->|WebSocket| S1
    S1 --> S2 --> S3 --> S4
    S4 -->|WebSocket| B1
    B1 --> B2 --> B3 --> B4

    style UserA fill:#1a1a2e,stroke:#6c5ce7,color:#f8f9fa
    style Server fill:#16213e,stroke:#00cec9,color:#f8f9fa
    style UserB fill:#1a1a2e,stroke:#6c5ce7,color:#f8f9fa
```

### Key Design Decisions

| Aspect | Strategy | Rationale |
|:-------|:---------|:----------|
| **Sync Model** | Operation-based (not snapshots) | Lower bandwidth, instant updates |
| **Conflict Resolution** | Last-write-wins (MVP) → CRDT (v2) | Simple first, scale later |
| **Cursor Tracking** | Throttled to 50ms, HTML overlay | Smooth rendering without canvas overhead |
| **Presence** | Redis key expiry + heartbeats | Stateless, auto-cleanup on crash |
| **Reconnection** | 30s grace period, full state resync | No data loss on network blips |

---

## 🚢 Deployment

```mermaid
graph LR
    subgraph Production
        V["Vercel<br/>Frontend CDN"]
        R["Render<br/>Backend Docker"]
        MA["MongoDB Atlas<br/>Database"]
        RC["Redis Cloud<br/>Cache"]
        S3["AWS S3<br/>Files"]
    end

    GH["GitHub Push"] -->|"auto-deploy"| V
    GH -->|"CI/CD"| R
    R --> MA
    R --> RC
    R --> S3

    style Production fill:#0f0f23,stroke:#6c5ce7,color:#f8f9fa
    style GH fill:#00b894,stroke:#00b894,color:#fff
```

| Service | Platform | Free Tier → Production |
|:--------|:---------|:----------------------|
| **Frontend** | Vercel | Free → Pro ($20/mo) |
| **Backend** | Render | $7/mo → $25/mo |
| **Database** | MongoDB Atlas | M0 Free → M10 ($57/mo) |
| **Cache** | Redis Cloud | Free 30MB → $5/mo |
| **Storage** | AWS S3 | Pay-as-you-go (~$1–5/mo) |
| **Errors** | Sentry | Free 5K events/mo |
| **📊 Total MVP** | | **~$10–30/mo** |

<details>
<summary><b>Step-by-step deployment instructions</b></summary>

**Frontend → Vercel**
1. Connect GitHub repo to [Vercel](https://vercel.com)
2. Set root directory: `frontend/`
3. Build command: `npm run build`
4. Output directory: `dist`
5. Add environment variables in dashboard

**Backend → Render**
1. Create Web Service on [Render](https://render.com)
2. Connect GitHub repo
3. Root directory: `backend/`
4. Start command: `node server.js`
5. Add environment variables
6. Enable Docker deployment

**Database → MongoDB Atlas**
1. Create cluster at [MongoDB Atlas](https://cloud.mongodb.com)
2. Create database user
3. Whitelist IPs (allow all for Render)
4. Copy connection string → set as `MONGODB_URI`

</details>

---

## 🗺️ Roadmap

```mermaid
gantt
    title DrawSaver Development Roadmap
    dateFormat  YYYY-MM-DD
    axisFormat  %b %d

    section Phase 1 — MVP
    Project Setup & Auth        :done, p1a, 2026-06-01, 7d
    Canvas Engine               :active, p1b, after p1a, 7d
    Rooms & Persistence         :p1c, after p1b, 7d
    MVP Polish & Launch         :p1d, after p1c, 7d

    section Phase 2 — Collaboration
    WebSocket Foundation        :p2a, after p1d, 7d
    Presence & Cursors          :p2b, after p2a, 7d
    Collaboration Polish        :p2c, after p2b, 7d

    section Phase 3 — Advanced
    Version History & Layers    :p3a, after p2c, 7d
    Export & Sharing            :p3b, after p3a, 7d
    UX Polish & Performance     :p3c, after p3b, 7d

    section Phase 4 — Production
    Testing & Security          :p4a, after p3c, 7d
    DevOps & Launch             :p4b, after p4a, 7d

    section Phase 5 — AI & Scale
    AI Features                 :p5, after p4b, 30d
```

### Progress Tracker

| Phase | Status | Focus |
|:------|:-------|:------|
| ✅ Phase 0 | **Complete** | Project documentation (14 docs) |
| 🔄 Phase 1 | **In Progress** | Auth + Canvas + Save (MVP) |
| ⬜ Phase 2 | Planned | WebSocket + Rooms + Real-time |
| ⬜ Phase 3 | Planned | History, Export, Layers, UX |
| ⬜ Phase 4 | Planned | Testing, DevOps, Production deploy |
| ⬜ Phase 5 | Future | AI drawing assist, Mobile, Enterprise |

---

## 📚 Documentation

All project documentation lives in the [`docs/`](docs/) directory — 13 production-grade documents covering every aspect of the system:

<table>
<tr>
<td>

| # | Document |
|:--|:---------|
| 01 | [Product Requirements](docs/01-product-requirements.md) |
| 02 | [Technical Architecture](docs/02-technical-architecture.md) |
| 03 | [Tech Stack](docs/03-tech-stack.md) |
| 04 | [Database Design](docs/04-database-design.md) |
| 05 | [API Documentation](docs/05-api-documentation.md) |
| 06 | [Real-Time Collaboration](docs/06-realtime-collaboration.md) |
| 07 | [Auth & Security](docs/07-authentication-security.md) |

</td>
<td>

| # | Document |
|:--|:---------|
| 08 | [UI/UX Documentation](docs/08-ui-ux-documentation.md) |
| 09 | [Folder Structure](docs/09-project-folder-structure.md) |
| 10 | [Development Roadmap](docs/10-development-roadmap.md) |
| 11 | [DevOps & Deployment](docs/11-devops-deployment.md) |
| 12 | [AI Integration Plan](docs/12-ai-integration-plan.md) |
| 13 | [Project Tracking](docs/13-project-tracking.md) |

</td>
</tr>
</table>

---

## 🤝 Contributing

We welcome contributions! Here's the workflow:

```mermaid
graph LR
    A["🍴 Fork"] --> B["🌿 Branch"]
    B --> C["💻 Code"]
    C --> D["✅ Test"]
    D --> E["📤 Push"]
    E --> F["🔀 PR"]

    style A fill:#6c5ce7,stroke:#6c5ce7,color:#fff
    style B fill:#00cec9,stroke:#00cec9,color:#fff
    style C fill:#6c5ce7,stroke:#6c5ce7,color:#fff
    style D fill:#00b894,stroke:#00b894,color:#fff
    style E fill:#00cec9,stroke:#00cec9,color:#fff
    style F fill:#6c5ce7,stroke:#6c5ce7,color:#fff
```

```bash
# 1. Fork the repo and clone your fork
git clone https://github.com/your-username/DrawSaver.git

# 2. Create a feature branch
git checkout -b feature/DS-XXX-your-feature

# 3. Make your changes and commit (conventional commits)
git commit -m "feat: add pencil pressure sensitivity"

# 4. Push and open a PR
git push origin feature/DS-XXX-your-feature
```

> See [Project Tracking](docs/13-project-tracking.md) for full guidelines, PR template, and code review checklist.

---

## 📄 License

This project is licensed under the **Apache License 2.0** — see the [LICENSE](LICENSE) file for details.

---

<p align="center">
  <b>Built with ❤️ by <a href="https://github.com/algorithnicmind">@algorithnicmind</a></b>
  <br />
  <sub>If you find this project useful, consider giving it a ⭐</sub>
</p>
