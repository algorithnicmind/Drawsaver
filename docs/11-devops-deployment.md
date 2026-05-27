# DrawSaver — DevOps & Deployment Document

> **Version:** 1.0  
> **Last Updated:** 2026-05-27  

---

## 1. Infrastructure Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        PRODUCTION                                │
│                                                                 │
│  ┌──────────┐    ┌──────────────────────────────────────────┐   │
│  │  Vercel   │    │            Render / Railway              │   │
│  │ (Frontend)│    │  ┌────────────┐  ┌────────────────┐     │   │
│  │  CDN Edge │    │  │ API Server │  │ WebSocket Server│     │   │
│  │  React    │    │  │ (Docker)   │  │ (Docker)        │     │   │
│  └──────────┘    │  └─────┬──────┘  └───────┬─────────┘     │   │
│                  │        │                  │               │   │
│                  └────────┼──────────────────┼───────────────┘   │
│                           │                  │                   │
│            ┌──────────────┼──────────────────┼──────────────┐   │
│            │              │                  │              │   │
│       ┌────▼─────┐  ┌────▼─────┐  ┌────────▼──────┐       │   │
│       │ MongoDB  │  │  Redis   │  │   AWS S3      │       │   │
│       │  Atlas   │  │  Cloud   │  │ (File Store)  │       │   │
│       │ (M10+)   │  │          │  │ + CloudFront  │       │   │
│       └──────────┘  └──────────┘  └───────────────┘       │   │
│                                                             │   │
│  ┌──────────┐  ┌──────────┐                                 │   │
│  │  Sentry  │  │ Grafana/ │  Monitoring & Observability     │   │
│  │ (Errors) │  │ Datadog  │                                 │   │
│  └──────────┘  └──────────┘                                 │   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Hosting Strategy

| Service | Provider | Tier | Monthly Cost (est.) |
|---------|----------|------|---------------------|
| **Frontend** | Vercel | Free → Pro ($20) | $0–20 |
| **Backend API** | Render | Starter ($7) → Standard ($25) | $7–25 |
| **WebSocket Server** | Render | Same instance as API (MVP) → separate (scale) | Included |
| **MongoDB** | MongoDB Atlas | M0 Free → M10 ($57) | $0–57 |
| **Redis** | Redis Cloud | Free (30MB) → Essentials ($5) | $0–5 |
| **File Storage** | AWS S3 | Pay-as-you-go | ~$1–5 |
| **CDN** | CloudFront | Pay-as-you-go | ~$1–5 |
| **Domain** | Cloudflare | Free DNS | $10/yr |
| **SSL** | Auto (Vercel/Render) | Free | $0 |
| **Error Tracking** | Sentry | Free (5K events/mo) | $0 |
| **Total MVP** | | | **~$10–30/mo** |

---

## 3. Docker Setup

### Backend Dockerfile

```dockerfile
# === Build Stage ===
FROM node:20-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .

# === Production Stage ===
FROM node:20-alpine

WORKDIR /app

# Security: run as non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S drawsaver -u 1001

COPY --from=builder --chown=drawsaver:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=drawsaver:nodejs /app/src ./src
COPY --from=builder --chown=drawsaver:nodejs /app/server.js ./
COPY --from=builder --chown=drawsaver:nodejs /app/package.json ./

USER drawsaver

EXPOSE 5000

HEALTHCHECK --interval=30s --timeout=3s --start-period=10s \
  CMD wget --no-verbose --tries=1 --spider http://localhost:5000/api/v1/health || exit 1

CMD ["node", "server.js"]
```

### docker-compose.yml (Local Development)

```yaml
version: '3.8'

services:
  # === MongoDB ===
  mongodb:
    image: mongo:7
    container_name: drawsaver-mongo
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: devpassword
    restart: unless-stopped

  # === Redis ===
  redis:
    image: redis:7-alpine
    container_name: drawsaver-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    restart: unless-stopped

  # === MinIO (S3-compatible local storage) ===
  minio:
    image: minio/minio
    container_name: drawsaver-minio
    ports:
      - "9000:9000"
      - "9001:9001"  # Console
    volumes:
      - minio_data:/data
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    command: server /data --console-address ":9001"
    restart: unless-stopped

  # === Backend API ===
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: drawsaver-api
    ports:
      - "5000:5000"
    depends_on:
      - mongodb
      - redis
      - minio
    environment:
      NODE_ENV: development
      PORT: 5000
      MONGODB_URI: mongodb://admin:devpassword@mongodb:27017/drawsaver?authSource=admin
      REDIS_URL: redis://redis:6379
      S3_ENDPOINT: http://minio:9000
      S3_ACCESS_KEY: minioadmin
      S3_SECRET_KEY: minioadmin
      S3_BUCKET: drawsaver-assets
      JWT_ACCESS_SECRET: dev-access-secret-change-in-production
      JWT_REFRESH_SECRET: dev-refresh-secret-change-in-production
      FRONTEND_URL: http://localhost:5173
    volumes:
      - ./backend/src:/app/src  # Hot reload in dev
    restart: unless-stopped

volumes:
  mongo_data:
  redis_data:
  minio_data:
```

### Start Local Dev Environment

```bash
# Start all services
docker compose up -d

# Verify services are running
docker compose ps

# View logs
docker compose logs -f backend

# Stop all services
docker compose down

# Stop and remove volumes (full reset)
docker compose down -v
```

---

## 4. CI/CD Pipeline

### GitHub Actions — CI Pipeline

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  # === Lint & Type Check ===
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
          cache-dependency-path: |
            frontend/package-lock.json
            backend/package-lock.json

      - name: Install frontend deps
        run: cd frontend && npm ci

      - name: Lint frontend
        run: cd frontend && npm run lint

      - name: Install backend deps
        run: cd backend && npm ci

      - name: Lint backend
        run: cd backend && npm run lint

  # === Backend Tests ===
  test-backend:
    runs-on: ubuntu-latest
    services:
      mongodb:
        image: mongo:7
        ports: ['27017:27017']
      redis:
        image: redis:7-alpine
        ports: ['6379:6379']

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install deps
        run: cd backend && npm ci

      - name: Run tests
        run: cd backend && npm test
        env:
          NODE_ENV: test
          MONGODB_URI: mongodb://localhost:27017/drawsaver-test
          REDIS_URL: redis://localhost:6379
          JWT_ACCESS_SECRET: test-secret
          JWT_REFRESH_SECRET: test-refresh-secret

  # === Frontend Tests ===
  test-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install deps
        run: cd frontend && npm ci

      - name: Run tests
        run: cd frontend && npm test

      - name: Build
        run: cd frontend && npm run build

  # === E2E Tests ===
  e2e:
    runs-on: ubuntu-latest
    needs: [test-backend, test-frontend]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install Playwright
        run: cd frontend && npm ci && npx playwright install --with-deps

      - name: Start services
        run: docker compose up -d

      - name: Wait for services
        run: sleep 10

      - name: Run E2E tests
        run: cd frontend && npx playwright test

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: frontend/playwright-report/
```

### GitHub Actions — CD Pipeline

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy-backend:
    runs-on: ubuntu-latest
    needs: [ci]  # Wait for CI to pass
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to Render
        env:
          RENDER_API_KEY: ${{ secrets.RENDER_API_KEY }}
          RENDER_SERVICE_ID: ${{ secrets.RENDER_SERVICE_ID }}
        run: |
          curl -X POST "https://api.render.com/v1/services/$RENDER_SERVICE_ID/deploys" \
            -H "Authorization: Bearer $RENDER_API_KEY" \
            -H "Content-Type: application/json"

  # Frontend deploys automatically via Vercel Git integration
```

---

## 5. Environment Configuration

### Environment Files

| File | Purpose | Git tracked? |
|------|---------|-------------|
| `.env.example` | Template with placeholder values | ✅ Yes |
| `.env` | Local development values | ❌ No (.gitignore) |
| `.env.test` | Test environment values | ❌ No |
| `.env.production` | Production values (set in hosting platform) | ❌ No |

### Backend `.env.example`

```env
# === Server ===
NODE_ENV=development
PORT=5000

# === MongoDB ===
MONGODB_URI=mongodb://admin:devpassword@localhost:27017/drawsaver?authSource=admin

# === Redis ===
REDIS_URL=redis://localhost:6379

# === JWT ===
JWT_ACCESS_SECRET=your-access-secret-min-32-chars
JWT_REFRESH_SECRET=your-refresh-secret-min-32-chars

# === OAuth ===
GOOGLE_CLIENT_ID=xxx.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=xxx
GITHUB_CLIENT_ID=xxx
GITHUB_CLIENT_SECRET=xxx

# === S3 / MinIO ===
S3_ENDPOINT=http://localhost:9000
S3_REGION=us-east-1
S3_ACCESS_KEY=minioadmin
S3_SECRET_KEY=minioadmin
S3_BUCKET=drawsaver-assets

# === Frontend ===
FRONTEND_URL=http://localhost:5173

# === Email (optional, for verification) ===
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password
EMAIL_FROM=noreply@drawsaver.app

# === Sentry ===
SENTRY_DSN=https://xxx@sentry.io/xxx
```

### Frontend `.env.example`

```env
VITE_API_URL=http://localhost:5000/api/v1
VITE_WS_URL=http://localhost:5000
VITE_GOOGLE_CLIENT_ID=xxx.apps.googleusercontent.com
VITE_SENTRY_DSN=https://xxx@sentry.io/xxx
```

### Environment Validation (Backend)

```javascript
// backend/src/config/env.js
import { z } from 'zod';

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'test', 'production']).default('development'),
  PORT: z.coerce.number().default(5000),
  MONGODB_URI: z.string().url(),
  REDIS_URL: z.string().url(),
  JWT_ACCESS_SECRET: z.string().min(32),
  JWT_REFRESH_SECRET: z.string().min(32),
  FRONTEND_URL: z.string().url(),
  S3_ENDPOINT: z.string().optional(),
  S3_BUCKET: z.string().default('drawsaver-assets'),
});

const parsed = envSchema.safeParse(process.env);

if (!parsed.success) {
  console.error('❌ Invalid environment variables:', parsed.error.format());
  process.exit(1);
}

export const env = parsed.data;
```

---

## 6. Monitoring & Logging

### Structured Logging (Pino)

```javascript
// backend/src/utils/logger.js
import pino from 'pino';

export const logger = pino({
  level: process.env.NODE_ENV === 'production' ? 'info' : 'debug',
  transport: process.env.NODE_ENV !== 'production' 
    ? { target: 'pino-pretty', options: { colorize: true } }
    : undefined,
  serializers: {
    req: (req) => ({
      method: req.method,
      url: req.url,
      userId: req.user?.sub,
    }),
    err: pino.stdSerializers.err,
  },
});

// Usage in middleware
app.use((req, res, next) => {
  req.log = logger.child({ requestId: crypto.randomUUID() });
  const start = Date.now();
  
  res.on('finish', () => {
    req.log.info({
      method: req.method,
      url: req.originalUrl,
      status: res.statusCode,
      duration: Date.now() - start,
    }, 'request completed');
  });
  
  next();
});
```

### Health Check Endpoint

```javascript
// GET /api/v1/health
router.get('/health', async (req, res) => {
  const checks = {
    server: 'ok',
    mongodb: 'checking',
    redis: 'checking',
    uptime: process.uptime(),
    timestamp: new Date().toISOString(),
  };
  
  try {
    await mongoose.connection.db.admin().ping();
    checks.mongodb = 'ok';
  } catch { checks.mongodb = 'error'; }
  
  try {
    await redisClient.ping();
    checks.redis = 'ok';
  } catch { checks.redis = 'error'; }
  
  const status = (checks.mongodb === 'ok' && checks.redis === 'ok') ? 200 : 503;
  res.status(status).json(checks);
});
```

### Monitoring Checklist

| Metric | Tool | Alert Threshold |
|--------|------|----------------|
| Error rate | Sentry | > 1% of requests |
| API response time P95 | Render metrics / Datadog | > 500ms |
| WebSocket connections | Custom metric + Redis | > 80% of max |
| MongoDB connection pool | Atlas monitoring | > 90% utilized |
| Memory usage | Render/Docker stats | > 80% of container limit |
| CPU usage | Render/Docker stats | > 70% sustained |
| Disk usage (S3) | AWS CloudWatch | > 80% of budget |
| Failed login attempts | Application logs | > 50/hour from same IP |

---

## 7. Deployment Checklist

### Pre-Launch

- [ ] All environment variables set in production
- [ ] MongoDB Atlas IP whitelist configured
- [ ] S3 bucket created with correct CORS policy
- [ ] Domain DNS pointing to correct services
- [ ] SSL certificates active
- [ ] Rate limiting configured
- [ ] CORS origin restricted to production domain
- [ ] Error tracking (Sentry) connected
- [ ] Health check endpoint responding
- [ ] Backup schedule configured (MongoDB Atlas)
- [ ] Log retention policy set

### Post-Launch

- [ ] Smoke test all critical flows (register, login, draw, save, collab)
- [ ] Verify WebSocket connections work through CDN/proxy
- [ ] Monitor error rates for first 24 hours
- [ ] Verify auto-save is persisting correctly
- [ ] Load test with expected concurrent users
- [ ] Verify OAuth redirects work with production URLs
