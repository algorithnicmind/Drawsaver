# DrawSaver — Authentication & Security Document

> **Version:** 1.0  
> **Last Updated:** 2026-05-27  

---

## 1. Authentication Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                      Authentication Flow                      │
│                                                              │
│  ┌────────┐   ┌────────────┐   ┌──────────┐   ┌─────────┐  │
│  │ Client │──►│ /auth/login│──►│ Passport │──►│ MongoDB │  │
│  │        │   │            │   │ Strategy │   │  (User) │  │
│  │        │◄──│ JWT Tokens │◄──│ Validate │◄──│  Lookup │  │
│  └────────┘   └────────────┘   └──────────┘   └─────────┘  │
│       │                                                      │
│       │  Access Token (15min) → Authorization: Bearer <token>│
│       │  Refresh Token (7d)   → HttpOnly Cookie              │
│       │                                                      │
│  ┌────▼────┐   ┌──────────────┐                              │
│  │ API     │──►│ Auth Guard   │  Verify token on every       │
│  │ Request │   │ Middleware   │  protected request            │
│  └─────────┘   └──────────────┘                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. JWT Strategy

### Token Structure

**Access Token (15 minutes):**
```json
{
  "header": { "alg": "HS256", "typ": "JWT" },
  "payload": {
    "sub": "665abc123def456",
    "username": "johndoe",
    "email": "john@example.com",
    "iat": 1716840000,
    "exp": 1716840900
  }
}
```

**Refresh Token (7 days):**
```json
{
  "payload": {
    "sub": "665abc123def456",
    "tokenFamily": "fam_abc123",
    "iat": 1716840000,
    "exp": 1717444800
  }
}
```

### Token Flow Implementation

```javascript
// === Token Generation ===
const generateTokens = (user) => {
  const accessToken = jwt.sign(
    { sub: user._id, username: user.username, email: user.email },
    process.env.JWT_ACCESS_SECRET,
    { expiresIn: '15m' }
  );
  
  const refreshToken = jwt.sign(
    { sub: user._id, tokenFamily: crypto.randomUUID() },
    process.env.JWT_REFRESH_SECRET,
    { expiresIn: '7d' }
  );
  
  return { accessToken, refreshToken };
};

// === Auth Guard Middleware ===
const authGuard = async (req, res, next) => {
  const authHeader = req.headers.authorization;
  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ success: false, error: { code: 'UNAUTHORIZED' } });
  }
  
  try {
    const token = authHeader.split(' ')[1];
    const decoded = jwt.verify(token, process.env.JWT_ACCESS_SECRET);
    req.user = decoded;
    next();
  } catch (err) {
    if (err.name === 'TokenExpiredError') {
      return res.status(401).json({ success: false, error: { code: 'TOKEN_EXPIRED' } });
    }
    return res.status(401).json({ success: false, error: { code: 'INVALID_TOKEN' } });
  }
};

// === Refresh Token Rotation ===
const refreshTokenHandler = async (req, res) => {
  const oldRefreshToken = req.cookies.refreshToken;
  if (!oldRefreshToken) return res.status(401).json({ error: 'No refresh token' });
  
  try {
    const decoded = jwt.verify(oldRefreshToken, process.env.JWT_REFRESH_SECRET);
    const user = await User.findById(decoded.sub).select('+refreshTokenHash');
    
    // Verify token hash matches stored hash (rotation detection)
    const isValid = await bcrypt.compare(oldRefreshToken, user.refreshTokenHash);
    if (!isValid) {
      // Token reuse detected! Possible theft. Invalidate all sessions.
      user.refreshTokenHash = null;
      await user.save();
      return res.status(401).json({ error: 'Token reuse detected. All sessions invalidated.' });
    }
    
    // Generate new token pair
    const { accessToken, refreshToken } = generateTokens(user);
    
    // Store hash of new refresh token
    user.refreshTokenHash = await bcrypt.hash(refreshToken, 12);
    await user.save();
    
    // Set new refresh token cookie
    res.cookie('refreshToken', refreshToken, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict',
      maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days
    });
    
    res.json({ success: true, data: { accessToken, expiresIn: 900 } });
  } catch (err) {
    res.status(401).json({ error: 'Invalid refresh token' });
  }
};
```

### WebSocket Authentication

```javascript
// Socket.IO middleware for JWT verification
io.use(async (socket, next) => {
  const token = socket.handshake.auth?.token;
  if (!token) return next(new Error('Authentication required'));
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_ACCESS_SECRET);
    socket.userId = decoded.sub;
    socket.username = decoded.username;
    next();
  } catch (err) {
    next(new Error('Invalid or expired token'));
  }
});
```

---

## 3. OAuth 2.0 Integration

### Google OAuth

```javascript
// Passport Google Strategy
passport.use(new GoogleStrategy({
  clientID: process.env.GOOGLE_CLIENT_ID,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET,
  callbackURL: '/api/v1/auth/google/callback',
  scope: ['profile', 'email']
}, async (accessToken, refreshToken, profile, done) => {
  try {
    let user = await User.findOne({ 
      authProvider: 'google', 
      authProviderId: profile.id 
    });
    
    if (!user) {
      // Check if email already exists (link accounts)
      user = await User.findOne({ email: profile.emails[0].value });
      
      if (user) {
        // Link Google to existing account
        user.authProvider = 'google';
        user.authProviderId = profile.id;
        user.emailVerified = true;
      } else {
        // Create new user
        user = new User({
          username: generateUniqueUsername(profile.displayName),
          email: profile.emails[0].value,
          displayName: profile.displayName,
          avatar: profile.photos[0]?.value,
          authProvider: 'google',
          authProviderId: profile.id,
          emailVerified: true
        });
      }
      await user.save();
    }
    
    done(null, user);
  } catch (err) {
    done(err);
  }
}));
```

### GitHub OAuth — Same pattern, different strategy fields.

---

## 4. Access Control

### Role-Based Permissions

| Action | `owner` | `editor` | `viewer` | Unauthenticated |
|--------|---------|----------|----------|-----------------|
| View room canvas | ✅ | ✅ | ✅ | ✅ (public rooms only) |
| Draw on canvas | ✅ | ✅ | ❌ | ❌ |
| Edit room settings | ✅ | ❌ | ❌ | ❌ |
| Kick members | ✅ | ❌ | ❌ | ❌ |
| Delete room | ✅ | ❌ | ❌ | ❌ |
| Invite members | ✅ | ✅ | ❌ | ❌ |
| Export drawing | ✅ | ✅ | ✅ | ❌ |
| Save drawing | ✅ | ✅ | ❌ | ❌ |
| View version history | ✅ | ✅ | ✅ | ❌ |
| Restore version | ✅ | ❌ | ❌ | ❌ |

### Middleware Implementation

```javascript
// Role-check middleware
const requireRole = (...allowedRoles) => {
  return async (req, res, next) => {
    const { roomId } = req.params;
    const userId = req.user.sub;
    
    const room = await Room.findById(roomId);
    if (!room) return res.status(404).json({ error: 'Room not found' });
    
    // Check if user is owner
    if (room.owner.toString() === userId) {
      req.userRole = 'owner';
      return next();
    }
    
    // Check membership
    const member = room.members.find(m => m.user.toString() === userId);
    if (!member) return res.status(403).json({ error: 'Not a member of this room' });
    
    if (!allowedRoles.includes(member.role)) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    
    req.userRole = member.role;
    next();
  };
};

// Usage in routes
router.delete('/rooms/:roomId', authGuard, requireRole('owner'), deleteRoom);
router.put('/drawings/:id', authGuard, requireRole('owner', 'editor'), saveDrawing);
```

---

## 5. Security Protections

### 5.1 Password Security

```javascript
// Hashing (on registration / password change)
const SALT_ROUNDS = 12;
const hashedPassword = await bcrypt.hash(plainPassword, SALT_ROUNDS);

// Validation requirements
const passwordSchema = z.string()
  .min(8, 'Password must be at least 8 characters')
  .regex(/[A-Z]/, 'Must contain at least one uppercase letter')
  .regex(/[a-z]/, 'Must contain at least one lowercase letter')
  .regex(/[0-9]/, 'Must contain at least one number')
  .regex(/[^A-Za-z0-9]/, 'Must contain at least one special character');
```

### 5.2 HTTP Security Headers (Helmet)

```javascript
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'", "https://fonts.googleapis.com"],
      imgSrc: ["'self'", "data:", "https://*.amazonaws.com", "https://*.googleusercontent.com"],
      connectSrc: ["'self'", "wss://*.drawsaver.app", "https://accounts.google.com"],
      fontSrc: ["'self'", "https://fonts.gstatic.com"],
      objectSrc: ["'none'"],
      upgradeInsecureRequests: []
    }
  },
  crossOriginEmbedderPolicy: false, // Allow loading external images
}));
```

### 5.3 CORS Configuration

```javascript
app.use(cors({
  origin: process.env.FRONTEND_URL, // e.g., 'https://drawsaver.app'
  credentials: true,                // Allow cookies
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  maxAge: 86400 // Cache preflight for 24h
}));
```

### 5.4 Rate Limiting

```javascript
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';
import { redisClient } from './config/redis.js';

// General API rate limit
const apiLimiter = rateLimit({
  store: new RedisStore({ sendCommand: (...args) => redisClient.sendCommand(args) }),
  windowMs: 60 * 1000,     // 1 minute
  max: 100,                 // 100 requests per minute
  standardHeaders: true,
  legacyHeaders: false,
  message: { success: false, error: { code: 'RATE_LIMITED', message: 'Too many requests' } }
});

// Strict auth rate limit
const authLimiter = rateLimit({
  store: new RedisStore({ sendCommand: (...args) => redisClient.sendCommand(args) }),
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 10,                   // 10 attempts
  skipSuccessfulRequests: true
});

app.use('/api/v1/', apiLimiter);
app.use('/api/v1/auth/login', authLimiter);
app.use('/api/v1/auth/register', authLimiter);
```

### 5.5 Input Validation with Zod

```javascript
import { z } from 'zod';

// Validation middleware factory
const validate = (schema) => (req, res, next) => {
  const result = schema.safeParse(req.body);
  if (!result.success) {
    return res.status(400).json({
      success: false,
      error: {
        code: 'VALIDATION_ERROR',
        message: 'Invalid request data',
        details: result.error.issues.map(i => ({
          field: i.path.join('.'),
          message: i.message
        }))
      }
    });
  }
  req.validatedBody = result.data;
  next();
};

// Schema examples
const registerSchema = z.object({
  username: z.string().min(3).max(30).regex(/^[a-zA-Z0-9_-]+$/),
  email: z.string().email(),
  password: z.string().min(8)
    .regex(/[A-Z]/).regex(/[a-z]/).regex(/[0-9]/),
  displayName: z.string().max(50).optional()
});

const createRoomSchema = z.object({
  name: z.string().min(1).max(100).trim(),
  visibility: z.enum(['public', 'private']).default('private'),
  maxParticipants: z.number().int().min(2).max(50).default(20)
});

// Usage
router.post('/auth/register', validate(registerSchema), registerController);
```

### 5.6 XSS Protection

- All user input sanitized via Zod schema validation
- Helmet CSP headers prevent inline script execution
- React auto-escapes JSX output by default
- Canvas text rendering is via Fabric.js (does not execute HTML)
- `sanitize-html` library for any rich text fields (future: chat)

### 5.7 CSRF Protection

- API is stateless (JWT in Authorization header) — not vulnerable to CSRF
- Refresh token in `HttpOnly; SameSite=Strict` cookie — protected by browser SameSite policy
- No cookie-based authentication for state-changing operations

### 5.8 MongoDB Injection Prevention

Mongoose automatically sanitizes queries, but additional protection:

```javascript
import mongoSanitize from 'express-mongo-sanitize';

app.use(mongoSanitize({
  replaceWith: '_',
  onSanitize: ({ req, key }) => {
    console.warn(`Sanitized key: ${key} in request to ${req.originalUrl}`);
  }
}));
```

---

## 6. Security Checklist

| Area | Control | Status |
|------|---------|--------|
| Authentication | JWT with short-lived access tokens | MVP |
| Authentication | Refresh token rotation with reuse detection | MVP |
| Authentication | OAuth 2.0 (Google, GitHub) | MVP |
| Password | bcrypt with 12 salt rounds | MVP |
| Password | Strength requirements enforced | MVP |
| Transport | HTTPS/TLS everywhere | MVP |
| Transport | WSS for WebSocket connections | MVP |
| Headers | Helmet security headers + CSP | MVP |
| Input | Zod schema validation on all endpoints | MVP |
| Input | MongoDB query sanitization | MVP |
| Rate Limiting | Per-endpoint rate limits via Redis | MVP |
| CORS | Strict origin whitelist | MVP |
| Cookies | HttpOnly, Secure, SameSite=Strict | MVP |
| Data | Sensitive fields excluded from queries (`select: false`) | MVP |
| 2FA | TOTP-based two-factor auth | Phase 3 |
| Audit | Security audit logging | Phase 3 |
| Monitoring | Anomaly detection (brute force, unusual patterns) | Phase 4 |

---

## 7. Environment Variables

```env
# === Authentication ===
JWT_ACCESS_SECRET=<random-64-char-string>
JWT_REFRESH_SECRET=<different-random-64-char-string>
JWT_ACCESS_EXPIRY=15m
JWT_REFRESH_EXPIRY=7d

# === OAuth ===
GOOGLE_CLIENT_ID=xxx.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=xxx
GITHUB_CLIENT_ID=xxx
GITHUB_CLIENT_SECRET=xxx

# === Security ===
BCRYPT_SALT_ROUNDS=12
RATE_LIMIT_WINDOW_MS=60000
RATE_LIMIT_MAX=100

# === CORS ===
FRONTEND_URL=https://drawsaver.app

# === Session ===
COOKIE_DOMAIN=.drawsaver.app
```

> **Critical:** Never commit `.env` files. Use `.env.example` with placeholder values as a template.
