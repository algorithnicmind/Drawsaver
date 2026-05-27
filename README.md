# 🎨 DrawSaver

**Create. Collaborate. Save Forever.**

DrawSaver is a real-time collaborative drawing platform where users can create, share, and persist digital drawings on a shared canvas. Think of it as the **Google Docs of visual collaboration** — instant, intuitive, and multiplayer-first.

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)

---

## ✨ Features

### Core Drawing
- ✏️ **Drawing Tools** — Pencil, brush, eraser, shapes (rectangle, circle, line, arrow), text
- 🎨 **Color System** — Full color picker, opacity control, recent colors
- ↩️ **Undo/Redo** — Full action history with keyboard shortcuts
- 🔍 **Zoom & Pan** — Navigate large canvases with smooth zoom (25%–400%)
- 📐 **Layers** — Multiple layers with visibility, reorder, and rename

### Real-Time Collaboration
- 👥 **Multiplayer Drawing** — Multiple users draw on the same canvas simultaneously
- 🖱️ **Live Cursors** — See other users' cursor positions with name labels
- 🟢 **Presence System** — Know who's active, idle, or offline
- 🔗 **Room Sharing** — Create rooms and invite via link or code

### Persistence & Export
- 💾 **Auto-Save** — Drawings save automatically every 30 seconds
- 📸 **Version History** — Browse and restore previous versions
- 📥 **Export** — Download as PNG, JPG, SVG, or PDF
- ☁️ **Cloud Storage** — All drawings stored securely in the cloud

### Authentication
- 🔐 **Secure Auth** — Email/password registration with JWT
- 🌐 **OAuth** — Sign in with Google or GitHub
- 🛡️ **Security** — Rate limiting, input validation, HTTPS

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 18 + Vite |
| Canvas Engine | Fabric.js |
| State Management | Zustand |
| Styling | Tailwind CSS |
| Backend | Node.js + Express |
| WebSocket | Socket.IO |
| Database | MongoDB (Mongoose) |
| Cache/PubSub | Redis |
| File Storage | AWS S3 |
| Auth | Passport.js + JWT |
| Testing | Vitest + Playwright |
| CI/CD | GitHub Actions |
| Deployment | Vercel (frontend) + Render (backend) |

---

## 📁 Project Structure

```
DrawSaver/
├── docs/               # Project documentation (14 documents)
├── frontend/           # React + Vite SPA
│   ├── src/
│   │   ├── components/ # UI components (canvas, collaboration, layout)
│   │   ├── pages/      # Route-level pages
│   │   ├── hooks/      # Custom hooks (useCanvas, useSocket, useAuth)
│   │   ├── store/      # Zustand state stores
│   │   ├── services/   # API client layer (Axios)
│   │   ├── socket/     # Socket.IO client
│   │   └── utils/      # Helpers and constants
│   └── ...
├── backend/            # Node.js + Express API
│   ├── src/
│   │   ├── config/     # DB, Redis, S3, Passport config
│   │   ├── models/     # Mongoose models
│   │   ├── routes/     # Express routes
│   │   ├── controllers/# Request handlers
│   │   ├── services/   # Business logic
│   │   ├── middleware/  # Auth, validation, rate limiting
│   │   ├── socket/     # Socket.IO event handlers
│   │   └── utils/      # Logger, error classes
│   └── ...
├── shared/             # Shared constants and types
├── docker-compose.yml  # Local dev environment
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites

- **Node.js** 20+ ([download](https://nodejs.org/))
- **Docker** & Docker Compose ([download](https://www.docker.com/))
- **Git** ([download](https://git-scm.com/))

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/DrawSaver.git
cd DrawSaver
```

### 2. Start Infrastructure (MongoDB, Redis, MinIO)

```bash
docker compose up -d
```

This starts:
- MongoDB on `localhost:27017`
- Redis on `localhost:6379`
- MinIO (S3-compatible) on `localhost:9000` (Console: `localhost:9001`)

### 3. Setup Backend

```bash
cd backend
cp .env.example .env    # Configure environment variables
npm install
npm run dev             # Starts on http://localhost:5000
```

### 4. Setup Frontend

```bash
cd frontend
cp .env.example .env    # Configure API URL
npm install
npm run dev             # Starts on http://localhost:5173
```

### 5. Open the App

Navigate to **http://localhost:5173** in your browser.

---

## ⚙️ Environment Variables

### Backend (`.env`)

```env
NODE_ENV=development
PORT=5000
MONGODB_URI=mongodb://admin:devpassword@localhost:27017/drawsaver?authSource=admin
REDIS_URL=redis://localhost:6379
JWT_ACCESS_SECRET=your-access-secret-min-32-chars
JWT_REFRESH_SECRET=your-refresh-secret-min-32-chars
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
GITHUB_CLIENT_ID=your-github-client-id
GITHUB_CLIENT_SECRET=your-github-client-secret
S3_ENDPOINT=http://localhost:9000
S3_ACCESS_KEY=minioadmin
S3_SECRET_KEY=minioadmin
S3_BUCKET=drawsaver-assets
FRONTEND_URL=http://localhost:5173
```

### Frontend (`.env`)

```env
VITE_API_URL=http://localhost:5000/api/v1
VITE_WS_URL=http://localhost:5000
VITE_GOOGLE_CLIENT_ID=your-google-client-id
```

---

## 📋 Development Commands

### Backend

| Command | Description |
|---------|-------------|
| `npm run dev` | Start dev server with hot reload |
| `npm run start` | Start production server |
| `npm run test` | Run unit tests |
| `npm run lint` | Run ESLint |
| `npm run lint:fix` | Auto-fix lint issues |

### Frontend

| Command | Description |
|---------|-------------|
| `npm run dev` | Start Vite dev server |
| `npm run build` | Production build |
| `npm run preview` | Preview production build |
| `npm run test` | Run Vitest unit tests |
| `npm run test:e2e` | Run Playwright E2E tests |
| `npm run lint` | Run ESLint |

### Docker

| Command | Description |
|---------|-------------|
| `docker compose up -d` | Start all services |
| `docker compose down` | Stop all services |
| `docker compose down -v` | Stop + remove volumes (full reset) |
| `docker compose logs -f` | Tail all logs |
| `docker compose ps` | Check service status |

---

## 🚢 Deployment

### Frontend → Vercel

1. Connect GitHub repo to [Vercel](https://vercel.com)
2. Set root directory to `frontend/`
3. Set build command: `npm run build`
4. Set output directory: `dist`
5. Add environment variables in Vercel dashboard

### Backend → Render

1. Create a new Web Service on [Render](https://render.com)
2. Connect GitHub repo
3. Set root directory to `backend/`
4. Set build command: `npm install`
5. Set start command: `node server.js`
6. Add environment variables
7. Enable Docker deployment (uses `Dockerfile`)

### Database → MongoDB Atlas

1. Create cluster at [MongoDB Atlas](https://cloud.mongodb.com)
2. Create database user
3. Whitelist IP addresses (or allow all for Render)
4. Get connection string → set as `MONGODB_URI`

---

## 📚 Documentation

All project documentation lives in the [`docs/`](docs/) directory:

| # | Document | Description |
|---|----------|-------------|
| 01 | [Product Requirements](docs/01-product-requirements.md) | Vision, goals, user stories, MVP scope |
| 02 | [Technical Architecture](docs/02-technical-architecture.md) | System design, component diagrams |
| 03 | [Tech Stack](docs/03-tech-stack.md) | Technology choices with rationale |
| 04 | [Database Design](docs/04-database-design.md) | Mongoose schemas, indexes, optimization |
| 05 | [API Documentation](docs/05-api-documentation.md) | REST endpoints, WebSocket events |
| 06 | [Real-Time Collaboration](docs/06-realtime-collaboration.md) | Sync strategy, cursors, presence |
| 07 | [Auth & Security](docs/07-authentication-security.md) | JWT, OAuth, RBAC, protections |
| 08 | [UI/UX Documentation](docs/08-ui-ux-documentation.md) | Wireframes, design system, user flows |
| 09 | [Folder Structure](docs/09-project-folder-structure.md) | Project organization, naming conventions |
| 10 | [Development Roadmap](docs/10-development-roadmap.md) | Phase-wise timeline with estimates |
| 11 | [DevOps & Deployment](docs/11-devops-deployment.md) | Docker, CI/CD, hosting, monitoring |
| 12 | [AI Integration Plan](docs/12-ai-integration-plan.md) | Future AI features with implementation |
| 13 | [Project Tracking](docs/13-project-tracking.md) | Sprint planning, git workflow, team process |

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/DS-XXX-description`
3. Commit with conventional commits: `git commit -m "feat: add pencil tool"`
4. Push to your fork: `git push origin feature/DS-XXX-description`
5. Open a Pull Request

See [Project Tracking](docs/13-project-tracking.md) for full contribution guidelines.

---

## 📄 License

This project is licensed under the Apache License 2.0 — see the [LICENSE](LICENSE) file for details.

---

## 🗺️ Roadmap

- [x] Project documentation
- [ ] Phase 1: MVP (Auth + Canvas + Save)
- [ ] Phase 2: Real-time collaboration
- [ ] Phase 3: Advanced features (layers, versioning, exports)
- [ ] Phase 4: Production deployment
- [ ] Phase 5: AI-powered features

---

**Built with ❤️ by the DrawSaver team**
