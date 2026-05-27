# DrawSaver — Project Folder Structure

> **Version:** 1.0  
> **Last Updated:** 2026-05-27  

---

## Monorepo Structure

```
DrawSaver/
│
├── docs/                              # Project documentation
│   ├── 01-product-requirements.md
│   ├── 02-technical-architecture.md
│   ├── 03-tech-stack.md
│   ├── 04-database-design.md
│   ├── 05-api-documentation.md
│   ├── 06-realtime-collaboration.md
│   ├── 07-authentication-security.md
│   ├── 08-ui-ux-documentation.md
│   ├── 09-project-folder-structure.md
│   ├── 10-development-roadmap.md
│   ├── 11-devops-deployment.md
│   ├── 12-ai-integration-plan.md
│   ├── 13-project-tracking.md
│   └── assets/                        # Diagrams, screenshots for docs
│
├── frontend/                          # React + Vite SPA
│   ├── public/
│   │   ├── favicon.ico
│   │   └── robots.txt
│   │
│   ├── src/
│   │   ├── assets/                    # Static images, SVGs, fonts
│   │   │   ├── images/
│   │   │   └── icons/
│   │   │
│   │   ├── components/                # Reusable UI components
│   │   │   ├── ui/                    # Generic UI primitives
│   │   │   │   ├── Button.jsx
│   │   │   │   ├── Modal.jsx
│   │   │   │   ├── Input.jsx
│   │   │   │   ├── Avatar.jsx
│   │   │   │   ├── Tooltip.jsx
│   │   │   │   ├── Spinner.jsx
│   │   │   │   ├── Toast.jsx
│   │   │   │   └── DropdownMenu.jsx
│   │   │   │
│   │   │   ├── canvas/                # Canvas-specific components
│   │   │   │   ├── CanvasContainer.jsx
│   │   │   │   ├── Toolbar.jsx
│   │   │   │   ├── ToolButton.jsx
│   │   │   │   ├── ColorPicker.jsx
│   │   │   │   ├── BrushSizeSlider.jsx
│   │   │   │   ├── LayersPanel.jsx
│   │   │   │   ├── LayerItem.jsx
│   │   │   │   └── ZoomControls.jsx
│   │   │   │
│   │   │   ├── collaboration/         # Real-time collaboration UI
│   │   │   │   ├── RemoteCursor.jsx
│   │   │   │   ├── UsersPanel.jsx
│   │   │   │   ├── UserAvatar.jsx
│   │   │   │   └── PresenceIndicator.jsx
│   │   │   │
│   │   │   ├── room/                  # Room management
│   │   │   │   ├── CreateRoomModal.jsx
│   │   │   │   ├── JoinRoomModal.jsx
│   │   │   │   ├── RoomSettings.jsx
│   │   │   │   └── ShareDialog.jsx
│   │   │   │
│   │   │   ├── drawing/               # Drawing cards, history
│   │   │   │   ├── DrawingCard.jsx
│   │   │   │   ├── DrawingGrid.jsx
│   │   │   │   ├── VersionHistory.jsx
│   │   │   │   └── ExportModal.jsx
│   │   │   │
│   │   │   └── layout/                # Layout components
│   │   │       ├── Navbar.jsx
│   │   │       ├── Sidebar.jsx
│   │   │       ├── Footer.jsx
│   │   │       └── PageLayout.jsx
│   │   │
│   │   ├── pages/                     # Route-level page components
│   │   │   ├── LandingPage.jsx
│   │   │   ├── LoginPage.jsx
│   │   │   ├── RegisterPage.jsx
│   │   │   ├── ForgotPasswordPage.jsx
│   │   │   ├── DashboardPage.jsx
│   │   │   ├── DrawingWorkspace.jsx
│   │   │   ├── ProfilePage.jsx
│   │   │   ├── JoinRoomPage.jsx
│   │   │   └── NotFoundPage.jsx
│   │   │
│   │   ├── hooks/                     # Custom React hooks
│   │   │   ├── useCanvas.js           # Fabric.js canvas lifecycle
│   │   │   ├── useSocket.js           # Socket.IO connection & events
│   │   │   ├── useAuth.js             # Auth state, login/logout
│   │   │   ├── useDrawingTools.js     # Tool state machine
│   │   │   ├── useAutoSave.js         # Debounced auto-save logic
│   │   │   ├── usePresence.js         # User presence tracking
│   │   │   ├── useKeyboardShortcuts.js
│   │   │   └── useUndoRedo.js         # Undo/redo stack
│   │   │
│   │   ├── store/                     # Zustand state stores
│   │   │   ├── canvasStore.js         # Active tool, color, brush, zoom
│   │   │   ├── roomStore.js           # Room info, participants
│   │   │   ├── authStore.js           # User, tokens, auth state
│   │   │   └── uiStore.js             # Panel visibility, modals
│   │   │
│   │   ├── services/                  # API client layer
│   │   │   ├── api.js                 # Axios instance with interceptors
│   │   │   ├── authService.js         # /auth/* API calls
│   │   │   ├── roomService.js         # /rooms/* API calls
│   │   │   ├── drawingService.js      # /drawings/* API calls
│   │   │   └── userService.js         # /users/* API calls
│   │   │
│   │   ├── socket/                    # Socket.IO client logic
│   │   │   ├── socketClient.js        # Socket.IO instance & config
│   │   │   ├── socketEvents.js        # Event name constants
│   │   │   └── socketHandlers.js      # Event handler registration
│   │   │
│   │   ├── utils/                     # Utility functions
│   │   │   ├── canvasHelpers.js       # Canvas serialization, export
│   │   │   ├── colorUtils.js          # Color conversion helpers
│   │   │   ├── dateUtils.js           # Date formatting
│   │   │   ├── throttle.js            # Throttle/debounce functions
│   │   │   ├── generateId.js          # Unique ID generation
│   │   │   └── constants.js           # App-wide constants
│   │   │
│   │   ├── styles/                    # Global styles
│   │   │   ├── index.css              # Global CSS + design tokens
│   │   │   ├── canvas.css             # Canvas-specific styles
│   │   │   └── animations.css         # Keyframe animations
│   │   │
│   │   ├── context/                   # React contexts (minimal use)
│   │   │   ├── AuthContext.jsx
│   │   │   └── SocketContext.jsx
│   │   │
│   │   ├── App.jsx                    # Root component with routing
│   │   ├── main.jsx                   # Entry point
│   │   └── router.jsx                 # React Router configuration
│   │
│   ├── .env.example                   # Environment variable template
│   ├── index.html
│   ├── vite.config.js
│   ├── tailwind.config.js
│   ├── postcss.config.js
│   ├── package.json
│   └── tsconfig.json                  # If using TypeScript
│
├── backend/                           # Node.js + Express API
│   ├── src/
│   │   ├── config/                    # Configuration & connections
│   │   │   ├── database.js            # MongoDB connection
│   │   │   ├── redis.js               # Redis client
│   │   │   ├── s3.js                  # AWS S3 client
│   │   │   ├── passport.js            # OAuth strategies
│   │   │   └── env.js                 # Environment validation (Zod)
│   │   │
│   │   ├── models/                    # Mongoose models
│   │   │   ├── User.js
│   │   │   ├── Room.js
│   │   │   ├── Drawing.js
│   │   │   ├── DrawingVersion.js
│   │   │   └── DrawingAction.js
│   │   │
│   │   ├── routes/                    # Express route definitions
│   │   │   ├── index.js               # Route aggregator
│   │   │   ├── auth.routes.js
│   │   │   ├── room.routes.js
│   │   │   ├── drawing.routes.js
│   │   │   └── user.routes.js
│   │   │
│   │   ├── controllers/               # Request handlers
│   │   │   ├── auth.controller.js
│   │   │   ├── room.controller.js
│   │   │   ├── drawing.controller.js
│   │   │   └── user.controller.js
│   │   │
│   │   ├── services/                  # Business logic
│   │   │   ├── auth.service.js
│   │   │   ├── room.service.js
│   │   │   ├── drawing.service.js
│   │   │   ├── user.service.js
│   │   │   ├── email.service.js       # Email sending (verification, reset)
│   │   │   └── storage.service.js     # S3 upload/download
│   │   │
│   │   ├── middleware/                # Express middleware
│   │   │   ├── authGuard.js           # JWT verification
│   │   │   ├── roleCheck.js           # Role-based access control
│   │   │   ├── validate.js            # Zod validation middleware
│   │   │   ├── rateLimiter.js         # Rate limiting config
│   │   │   ├── errorHandler.js        # Global error handler
│   │   │   └── upload.js              # Multer file upload config
│   │   │
│   │   ├── socket/                    # Socket.IO server logic
│   │   │   ├── socketServer.js        # Socket.IO init & auth middleware
│   │   │   ├── roomHandler.js         # Room join/leave events
│   │   │   ├── drawingHandler.js      # Drawing sync events
│   │   │   ├── cursorHandler.js       # Cursor tracking events
│   │   │   └── presenceHandler.js     # User presence management
│   │   │
│   │   ├── validators/                # Zod validation schemas
│   │   │   ├── auth.schema.js
│   │   │   ├── room.schema.js
│   │   │   ├── drawing.schema.js
│   │   │   └── user.schema.js
│   │   │
│   │   ├── utils/                     # Utility functions
│   │   │   ├── logger.js              # Pino logger config
│   │   │   ├── generateToken.js       # JWT generation
│   │   │   ├── generateSlug.js        # URL slug generation
│   │   │   ├── generateInviteCode.js  # Random invite codes
│   │   │   ├── asyncHandler.js        # Try-catch wrapper for controllers
│   │   │   └── AppError.js            # Custom error class
│   │   │
│   │   ├── jobs/                      # Background jobs (BullMQ)
│   │   │   ├── queue.js               # Queue configuration
│   │   │   ├── thumbnailJob.js        # Generate drawing thumbnails
│   │   │   └── cleanupJob.js          # Clean expired data
│   │   │
│   │   └── app.js                     # Express app setup
│   │
│   ├── server.js                      # HTTP server entry point
│   ├── .env.example
│   ├── package.json
│   ├── Dockerfile
│   └── tsconfig.json
│
├── shared/                            # Shared code between frontend & backend
│   ├── constants/
│   │   ├── socketEvents.js            # Socket event name constants
│   │   └── roles.js                   # Role enums
│   ├── types/                         # Shared TypeScript types (if using TS)
│   │   ├── user.types.js
│   │   ├── room.types.js
│   │   └── drawing.types.js
│   └── package.json
│
├── .github/                           # GitHub configuration
│   ├── workflows/
│   │   ├── ci.yml                     # CI pipeline (lint, test, build)
│   │   └── deploy.yml                 # CD pipeline (deploy on merge)
│   ├── PULL_REQUEST_TEMPLATE.md
│   └── ISSUE_TEMPLATE/
│       ├── bug_report.md
│       └── feature_request.md
│
├── docker-compose.yml                 # Local dev: MongoDB + Redis + MinIO
├── docker-compose.prod.yml            # Production compose
├── .gitignore
├── .prettierrc
├── .eslintrc.js
├── LICENSE
└── README.md
```

---

## Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| **Files (components)** | PascalCase | `DrawingCard.jsx` |
| **Files (hooks)** | camelCase with `use` prefix | `useCanvas.js` |
| **Files (utils/services)** | camelCase | `authService.js` |
| **Files (routes/controllers)** | kebab-case with suffix | `auth.routes.js` |
| **Files (models)** | PascalCase | `User.js` |
| **Directories** | camelCase or kebab-case | `components/`, `socket/` |
| **React components** | PascalCase | `<CanvasContainer />` |
| **Functions** | camelCase | `generateInviteCode()` |
| **Constants** | SCREAMING_SNAKE_CASE | `MAX_PARTICIPANTS` |
| **CSS classes** | kebab-case | `.drawing-card` |
| **Zustand stores** | camelCase with `Store` suffix | `canvasStore` |
| **Environment vars** | SCREAMING_SNAKE_CASE | `MONGODB_URI` |
| **API routes** | kebab-case, plural nouns | `/api/v1/drawings/:id` |
| **Socket events** | kebab-case | `draw-action`, `cursor-move` |
| **Database collections** | camelCase, plural | `users`, `drawingVersions` |

---

## Key Architectural Rules

1. **No circular imports:** Services can import models, but models never import services
2. **Controllers are thin:** Only parse request, call service, send response
3. **Services contain business logic:** All validation, orchestration, and data transformations
4. **Hooks encapsulate side effects:** Canvas, socket, and API interactions are in hooks, not components
5. **Stores are UI state only:** Don't store server data in Zustand — use hooks to fetch and cache
6. **Socket events are constants:** Always import from `shared/constants/socketEvents.js`, never use string literals
7. **One component per file:** No multi-component files except tightly coupled internal components
