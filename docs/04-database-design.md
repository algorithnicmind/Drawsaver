# DrawSaver — Database Design Document

> **Version:** 1.0  
> **Last Updated:** 2026-05-27  
> **Database:** MongoDB (via Mongoose ODM)  

---

## Entity Relationship Overview

```
┌──────────┐       ┌──────────┐       ┌──────────────┐
│  Users   │──1:N──│  Rooms   │──1:N──│  Drawings    │
│          │       │          │       │  (Versions)   │
└──────────┘       └──────┬───┘       └──────┬───────┘
     │                    │                   │
     │               N:M (members)            │
     │                    │              ┌────▼────────┐
     └────────────────────┘              │  History     │
                                         │  (Changes)   │
                                         └─────────────┘
```

---

## Collections

### 1. `users`

Stores user accounts and profile information.

```javascript
const UserSchema = new mongoose.Schema({
  username: {
    type: String,
    required: true,
    unique: true,
    trim: true,
    minlength: 3,
    maxlength: 30,
    match: /^[a-zA-Z0-9_-]+$/
  },
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
    trim: true
  },
  password: {
    type: String,
    minlength: 8,
    select: false  // Never returned in queries by default
  },
  authProvider: {
    type: String,
    enum: ['local', 'google', 'github'],
    default: 'local'
  },
  authProviderId: {
    type: String,  // OAuth provider user ID
    sparse: true
  },
  avatar: {
    type: String,  // S3 URL or OAuth avatar URL
    default: null
  },
  displayName: {
    type: String,
    maxlength: 50
  },
  emailVerified: {
    type: Boolean,
    default: false
  },
  refreshTokenHash: {
    type: String,  // Hashed refresh token for rotation
    select: false
  },
  lastLoginAt: Date,
  passwordResetToken: { type: String, select: false },
  passwordResetExpires: { type: Date, select: false }
}, {
  timestamps: true  // createdAt, updatedAt
});

// Indexes
UserSchema.index({ email: 1 });
UserSchema.index({ authProvider: 1, authProviderId: 1 });
```

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `_id` | ObjectId | Auto | Primary key |
| `username` | String | Yes | Unique, 3-30 chars, alphanumeric |
| `email` | String | Yes | Unique, lowercase |
| `password` | String | Conditional | Required for `local` auth, hashed with bcrypt |
| `authProvider` | String | Yes | `local`, `google`, or `github` |
| `authProviderId` | String | No | External OAuth ID |
| `avatar` | String | No | URL to profile image |
| `displayName` | String | No | Shown in UI |
| `emailVerified` | Boolean | Yes | Default `false` |
| `refreshTokenHash` | String | No | For token rotation |
| `lastLoginAt` | Date | No | Tracks last login |
| `createdAt` | Date | Auto | Mongoose timestamp |
| `updatedAt` | Date | Auto | Mongoose timestamp |

---

### 2. `rooms`

Stores drawing room metadata and membership.

```javascript
const RoomSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    trim: true,
    maxlength: 100
  },
  slug: {
    type: String,
    unique: true,
    lowercase: true  // URL-friendly room identifier
  },
  owner: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  members: [{
    user: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
    role: { type: String, enum: ['owner', 'editor', 'viewer'], default: 'editor' },
    joinedAt: { type: Date, default: Date.now }
  }],
  visibility: {
    type: String,
    enum: ['public', 'private'],
    default: 'private'
  },
  maxParticipants: {
    type: Number,
    default: 20,
    min: 2,
    max: 50
  },
  inviteCode: {
    type: String,
    unique: true  // 8-char alphanumeric code for sharing
  },
  isActive: {
    type: Boolean,
    default: true
  },
  lastActivityAt: {
    type: Date,
    default: Date.now
  },
  settings: {
    backgroundColor: { type: String, default: '#ffffff' },
    canvasWidth: { type: Number, default: 1920 },
    canvasHeight: { type: Number, default: 1080 },
    allowAnonymous: { type: Boolean, default: false }
  }
}, {
  timestamps: true
});

// Indexes
RoomSchema.index({ owner: 1 });
RoomSchema.index({ 'members.user': 1 });
RoomSchema.index({ inviteCode: 1 });
RoomSchema.index({ visibility: 1, isActive: 1 });
RoomSchema.index({ lastActivityAt: -1 });
```

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `_id` | ObjectId | Auto | Primary key |
| `name` | String | Yes | Room display name |
| `slug` | String | Yes | URL-friendly unique identifier |
| `owner` | ObjectId → User | Yes | Room creator |
| `members` | Array | No | List of `{ user, role, joinedAt }` |
| `visibility` | String | Yes | `public` or `private` |
| `maxParticipants` | Number | Yes | Default 20 |
| `inviteCode` | String | Yes | Shareable 8-char code |
| `isActive` | Boolean | Yes | Soft delete flag |
| `lastActivityAt` | Date | Yes | For cleanup/sorting |
| `settings` | Object | Yes | Canvas configuration |

---

### 3. `drawings`

Stores the current state of each canvas. One drawing per room (current version).

```javascript
const DrawingSchema = new mongoose.Schema({
  room: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Room',
    required: true,
    unique: true  // One active drawing per room
  },
  canvasData: {
    type: mongoose.Schema.Types.Mixed,
    required: true
    // Fabric.js canvas.toJSON() output:
    // { version: "6.x", objects: [...], background: "..." }
  },
  thumbnail: {
    type: String  // S3 URL of auto-generated thumbnail
  },
  title: {
    type: String,
    default: 'Untitled Drawing',
    maxlength: 200
  },
  currentVersion: {
    type: Number,
    default: 1
  },
  objectCount: {
    type: Number,
    default: 0  // Denormalized for quick display
  },
  dataSizeBytes: {
    type: Number,
    default: 0  // Track data size for quota management
  },
  lastEditedBy: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }
}, {
  timestamps: true
});

// Indexes
DrawingSchema.index({ room: 1 });
DrawingSchema.index({ lastEditedBy: 1 });
DrawingSchema.index({ updatedAt: -1 });
```

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `_id` | ObjectId | Auto | Primary key |
| `room` | ObjectId → Room | Yes | Unique — one drawing per room |
| `canvasData` | Mixed (JSON) | Yes | Fabric.js serialized canvas state |
| `thumbnail` | String | No | S3 URL of preview image |
| `title` | String | No | Drawing title |
| `currentVersion` | Number | Yes | Increments on each save |
| `objectCount` | Number | No | Number of canvas objects |
| `dataSizeBytes` | Number | No | Size tracking |
| `lastEditedBy` | ObjectId → User | No | Last user to modify |

---

### 4. `drawingVersions`

Stores historical snapshots for version history and restore.

```javascript
const DrawingVersionSchema = new mongoose.Schema({
  drawing: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Drawing',
    required: true
  },
  version: {
    type: Number,
    required: true
  },
  canvasData: {
    type: mongoose.Schema.Types.Mixed,
    required: true  // Full snapshot of canvas at this version
  },
  thumbnail: String,
  createdBy: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  },
  changeDescription: {
    type: String  // Auto-generated: "Added 5 objects", "Cleared canvas"
  },
  dataSizeBytes: Number
}, {
  timestamps: { createdAt: true, updatedAt: false }  // Only createdAt
});

// Compound unique index
DrawingVersionSchema.index({ drawing: 1, version: -1 }, { unique: true });
DrawingVersionSchema.index({ drawing: 1, createdAt: -1 });

// Cap at 50 versions per drawing — handled in application logic
```

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `_id` | ObjectId | Auto | Primary key |
| `drawing` | ObjectId → Drawing | Yes | Parent drawing |
| `version` | Number | Yes | Version number |
| `canvasData` | Mixed | Yes | Full canvas snapshot at this version |
| `thumbnail` | String | No | Version thumbnail |
| `createdBy` | ObjectId → User | No | Who triggered the save |
| `changeDescription` | String | No | Auto-generated change summary |

---

### 5. `drawingActions` (Optional — for real-time replay)

Stores granular drawing operations for timelapse replay feature (Phase 3+).

```javascript
const DrawingActionSchema = new mongoose.Schema({
  room: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Room',
    required: true
  },
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  actionType: {
    type: String,
    enum: ['object:added', 'object:modified', 'object:removed', 'canvas:cleared'],
    required: true
  },
  objectData: mongoose.Schema.Types.Mixed,
  timestamp: {
    type: Date,
    default: Date.now,
    index: true
  }
}, {
  timestamps: false  // Use manual timestamp for precision
});

// TTL index: auto-delete actions older than 30 days
DrawingActionSchema.index({ timestamp: 1 }, { expireAfterSeconds: 2592000 });
DrawingActionSchema.index({ room: 1, timestamp: 1 });
```

---

## Optimization Strategy

### Indexes Summary

| Collection | Index | Purpose |
|-----------|-------|---------|
| `users` | `{ email: 1 }` | Login lookup |
| `users` | `{ authProvider: 1, authProviderId: 1 }` | OAuth lookup |
| `rooms` | `{ owner: 1 }` | "My rooms" query |
| `rooms` | `{ 'members.user': 1 }` | "Rooms I'm in" query |
| `rooms` | `{ inviteCode: 1 }` | Join by code |
| `rooms` | `{ visibility: 1, isActive: 1 }` | Public room listing |
| `drawings` | `{ room: 1 }` (unique) | Room → drawing lookup |
| `drawings` | `{ updatedAt: -1 }` | Recent drawings |
| `drawingVersions` | `{ drawing: 1, version: -1 }` (unique) | Version listing |
| `drawingActions` | `{ room: 1, timestamp: 1 }` | Replay queries |
| `drawingActions` | `{ timestamp: 1 }` (TTL) | Auto-cleanup |

### Data Size Estimates

| Collection | Avg Document Size | Growth Rate | Notes |
|-----------|------------------|-------------|-------|
| `users` | ~1 KB | Linear | Slow growth |
| `rooms` | ~2 KB | Linear | One per collaboration session |
| `drawings` | 50 KB – 5 MB | Per room | Depends on object count |
| `drawingVersions` | 50 KB – 5 MB | Per save | Capped at 50 per drawing |
| `drawingActions` | ~500 bytes | High | TTL auto-deletes after 30 days |

### Performance Guidelines

1. **Canvas Data Compression:** Before storing `canvasData`, compress using `zlib.gzip()` for documents > 100KB. Store as `Buffer` and decompress on read.

2. **Version Cleanup:** Application-level logic to keep only the last 50 versions. On new version create, delete oldest if count > 50.

3. **Thumbnail Generation:** Run asynchronously via BullMQ job after save. Don't block the save operation.

4. **Read Optimization:** Use `.lean()` in Mongoose for read-only queries (skips hydration, 5-10x faster).

5. **Projection:** Always use `.select()` to return only needed fields, especially for `canvasData` which can be large.

```javascript
// ❌ Bad: Returns full canvasData for every drawing
const drawings = await Drawing.find({ lastEditedBy: userId });

// ✅ Good: Only return metadata for listing
const drawings = await Drawing.find({ lastEditedBy: userId })
  .select('title thumbnail objectCount updatedAt')
  .sort({ updatedAt: -1 })
  .limit(20)
  .lean();
```

6. **Connection Pooling:** Mongoose default pool size is 5. For production, set to 10-20:

```javascript
mongoose.connect(MONGO_URI, {
  maxPoolSize: 15,
  minPoolSize: 5,
  serverSelectionTimeoutMS: 5000,
  socketTimeoutMS: 45000,
});
```
