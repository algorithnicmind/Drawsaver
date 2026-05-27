# DrawSaver — UI/UX Documentation

> **Version:** 1.0  
> **Last Updated:** 2026-05-27  

---

## 1. Design System

### Color Palette

| Token | Value | Usage |
|-------|-------|-------|
| `--bg-primary` | `#0f0f23` | Main background (dark mode) |
| `--bg-secondary` | `#1a1a2e` | Card/panel background |
| `--bg-tertiary` | `#16213e` | Hover states, input fields |
| `--accent-primary` | `#6c5ce7` | Primary buttons, active states |
| `--accent-secondary` | `#00cec9` | Secondary actions, links |
| `--accent-success` | `#00b894` | Success states, online indicators |
| `--accent-warning` | `#fdcb6e` | Warning states |
| `--accent-danger` | `#ff7675` | Error states, destructive actions |
| `--text-primary` | `#f8f9fa` | Primary text |
| `--text-secondary` | `#a0a0b0` | Secondary/muted text |
| `--border` | `#2d2d44` | Borders, dividers |

### Typography

| Level | Font | Size | Weight | Usage |
|-------|------|------|--------|-------|
| H1 | Inter | 32px | 700 | Page titles |
| H2 | Inter | 24px | 600 | Section headers |
| H3 | Inter | 18px | 600 | Card titles |
| Body | Inter | 14px | 400 | General text |
| Small | Inter | 12px | 400 | Labels, captions |
| Mono | JetBrains Mono | 13px | 400 | Code, room IDs |

### Spacing Scale

`4px → 8px → 12px → 16px → 24px → 32px → 48px → 64px`

### Border Radius

| Token | Value | Usage |
|-------|-------|-------|
| `--radius-sm` | 4px | Small elements (tags) |
| `--radius-md` | 8px | Buttons, inputs |
| `--radius-lg` | 12px | Cards, panels |
| `--radius-xl` | 16px | Modals |
| `--radius-full` | 9999px | Avatars, pills |

---

## 2. Main Pages

### 2.1 Landing Page

```
┌─────────────────────────────────────────────────────────────┐
│  🎨 DrawSaver          Features  Pricing  [Login] [Sign Up] │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│     Create. Collaborate.                                    │
│     Save Forever.                                           │
│                                                             │
│     Real-time collaborative drawing                         │
│     for teams and creators.                                 │
│                                                             │
│     [Get Started Free]  [Watch Demo ▶]                      │
│                                                             │
│     ┌─────────────────────────────────────┐                 │
│     │  Animated canvas preview / demo     │                 │
│     │  (shows collaborative drawing)      │                 │
│     └─────────────────────────────────────┘                 │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  ⚡ Real-Time     🎨 Rich Tools    💾 Auto-Save             │
│  Draw together    Brushes, shapes,  Never lose               │
│  with zero lag    text, layers      your work                │
│                                                             │
│  🔗 Easy Sharing  🔒 Secure       📱 Responsive             │
│  Share via link   End-to-end       Works on any              │
│  or room code     protection       device                    │
├─────────────────────────────────────────────────────────────┤
│  Footer: Links | Social | © 2026 DrawSaver                  │
└─────────────────────────────────────────────────────────────┘
```

**Key Elements:**
- Dark gradient hero with animated canvas background
- CTA buttons with hover glow effect
- Feature cards with icons and micro-animations on scroll
- Glassmorphism card effect for premium feel

---

### 2.2 Authentication Pages

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  ┌──────────────────────────┐                               │
│  │  🎨 DrawSaver            │                               │
│  │                          │                               │
│  │  Welcome back            │  ┌────────────────────────┐  │
│  │                          │  │                        │  │
│  │  [Continue with Google]  │  │  Animated canvas       │  │
│  │  [Continue with GitHub]  │  │  illustration or       │  │
│  │                          │  │  pattern background    │  │
│  │  ──── or ────            │  │                        │  │
│  │                          │  │                        │  │
│  │  Email: [____________]   │  │                        │  │
│  │  Password: [__________]  │  │                        │  │
│  │                          │  └────────────────────────┘  │
│  │  [Forgot password?]      │                               │
│  │                          │                               │
│  │  [Log In]                │                               │
│  │                          │                               │
│  │  Don't have an account?  │                               │
│  │  Sign up                 │                               │
│  └──────────────────────────┘                               │
└─────────────────────────────────────────────────────────────┘
```

**Split layout:** Form on left, visual canvas art on right. Mobile: form only, full-width.

---

### 2.3 Dashboard

```
┌─────────────────────────────────────────────────────────────┐
│  🎨 DrawSaver    [Search...]       🔔  👤 John ▼           │
├────────┬────────────────────────────────────────────────────┤
│        │                                                    │
│  📁    │  My Drawings                   [+ New Room]       │
│  Home  │                                                    │
│        │  ┌─────────┐ ┌─────────┐ ┌─────────┐             │
│  🎨    │  │ thumb   │ │ thumb   │ │ thumb   │             │
│ Recent │  │         │ │         │ │         │             │
│        │  │ Sprint  │ │ Wireframe│ │ Sketch  │             │
│  👥    │  │ ✏️ 3 min │ │ ✏️ 2 hrs │ │ ✏️ 1 day │             │
│ Shared │  │ 👥 5    │ │ 👥 2    │ │ 👥 1    │             │
│        │  └─────────┘ └─────────┘ └─────────┘             │
│  ⭐    │                                                    │
│ Starred│  Shared With Me                                    │
│        │                                                    │
│  🗑️    │  ┌─────────┐ ┌─────────┐                         │
│ Trash  │  │ thumb   │ │ thumb   │                         │
│        │  │ Team    │ │ Client  │                         │
│        │  │ Board   │ │ Review  │                         │
│        │  └─────────┘ └─────────┘                         │
├────────┴────────────────────────────────────────────────────┤
```

**Key Elements:**
- Collapsible sidebar with icon-only mode on mobile
- Drawing cards with thumbnail previews, last edited time, participant count
- Grid/list view toggle
- Context menu on cards (Open, Rename, Share, Export, Delete)
- "Join Room" input for entering invite codes

---

### 2.4 Drawing Workspace (Core UI)

```
┌─────────────────────────────────────────────────────────────────────┐
│  ← Back   "Sprint Whiteboard"  ✏️                 💾 Auto-saved    │
│           [Share] [Export ▼]              👥 5 online    ● ● ● ● ● │
├──────┬──────────────────────────────────────────────┬───────────────┤
│      │                                              │               │
│  ✏️  │                                              │  Layers       │
│  🖌️  │                                              │  ┌─────────┐ │
│  ⬜  │                                              │  │ 👁️ Layer 3│ │
│  ⭕  │              CANVAS                          │  │ 👁️ Layer 2│ │
│  📐  │                                              │  │ 👁️ Layer 1│ │
│  🔤  │           (Fabric.js)                        │  └─────────┘ │
│  🧽  │                                              │  [+ Layer]   │
│  ↩️  │                                              │               │
│  ↪️  │           ┌──────────────────┐               │  Users        │
│  🔍  │           │   User cursors    │               │  ● John (you)│
│      │           │   visible here    │               │  ● Jane 🎨   │
│ ──── │           └──────────────────┘               │  ● Bob        │
│      │                                              │               │
│  🎨  │                                              │               │
│ Color│                                              │               │
│ [██] │                                              │               │
│      │                                              │               │
│ Size │                                              │               │
│ ──●──│                                              │               │
│      │                                              │               │
│ Opac │                                              │               │
│ ──●──│                                              │               │
├──────┴──────────────────────────────────────────────┴───────────────┤
│  Zoom: [−] 100% [+]  │  Position: x:450 y:320  │  Objects: 47     │
└─────────────────────────────────────────────────────────────────────┘
```

**Layout Breakdown:**

| Zone | Position | Content |
|------|----------|---------|
| **Top Bar** | Top | Room name (editable), share/export buttons, auto-save indicator, participant avatars |
| **Left Toolbar** | Left, vertical | Drawing tools, separated by dividers. Active tool highlighted |
| **Left Panel (lower)** | Left, below tools | Color picker, brush size slider, opacity slider |
| **Canvas Area** | Center | Fabric.js canvas, fills remaining space. Shows remote cursors as overlay |
| **Right Panel** | Right, collapsible | Layers list (reorder via drag), Users list with status indicators |
| **Status Bar** | Bottom | Zoom controls, cursor coordinates, object count |

---

## 3. User Flows

### 3.1 New User → First Drawing

```
Landing Page → [Sign Up] → Register Form → Email Verification
     ↓
Dashboard (empty state: "Create your first drawing!")
     ↓
[+ New Room] → Room Name Modal → Drawing Workspace
     ↓
Start drawing with default tools
     ↓
Auto-save triggers → ✅ Saved
     ↓
[Share] → Copy invite link → Send to collaborator
     ↓
Collaborator joins → See their cursor → Draw together
```

### 3.2 Returning User → Resume Drawing

```
Landing Page → [Login] → Dashboard
     ↓
Click on drawing card → Drawing Workspace (loads saved state)
     ↓
Continue drawing → Auto-save
```

### 3.3 Join via Invite

```
Click invite link (https://drawsaver.app/join/XK7M9P2Q)
     ↓
If authenticated → Join room → Drawing Workspace
If not authenticated → Login/Register → Redirect to room
```

### 3.4 Version Restore

```
Drawing Workspace → History Panel → Browse versions
     ↓
Click version thumbnail → Preview overlay
     ↓
[Restore This Version] → Confirmation dialog → Canvas updates
```

---

## 4. Canvas Workspace Detailed Design

### 4.1 Toolbar Specifications

| Tool | Icon | Shortcut | Behavior |
|------|------|----------|----------|
| Select/Move | ↖️ cursor | `V` | Select, move, resize, rotate objects |
| Pencil | ✏️ | `P` | Freehand drawing (creates Path object) |
| Brush | 🖌️ | `B` | Thicker freehand with pressure sensitivity |
| Rectangle | ⬜ | `R` | Click-drag to create rectangle |
| Circle/Ellipse | ⭕ | `C` | Click-drag to create ellipse |
| Line/Arrow | 📐 | `L` | Click-drag for line. Shift+click for arrow |
| Text | 🔤 | `T` | Click to place text box, type to edit |
| Eraser | 🧽 | `E` | Click on objects to delete them |
| Undo | ↩️ | `Ctrl+Z` | Undo last action |
| Redo | ↪️ | `Ctrl+Shift+Z` | Redo last undone action |
| Zoom | 🔍 | `Ctrl+Scroll` | Zoom in/out. Also: `Ctrl++` and `Ctrl+-` |

### 4.2 Color Picker

```
┌──────────────────────┐
│  Current: ██ #6c5ce7 │
│                      │
│  ┌────────────────┐  │
│  │  Saturation/   │  │
│  │  Brightness    │  │
│  │  Picker Area   │  │
│  │  (256×256)     │  │
│  └────────────────┘  │
│  Hue: ═══════●═════  │
│  Opacity: ════●════  │
│                      │
│  Hex: [#6c5ce7]     │
│                      │
│  Recent:             │
│  ██ ██ ██ ██ ██ ██   │
│  ██ ██ ██ ██ ██ ██   │
└──────────────────────┘
```

### 4.3 Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+Z` | Undo |
| `Ctrl+Shift+Z` / `Ctrl+Y` | Redo |
| `Ctrl+S` | Manual save |
| `Ctrl+A` | Select all |
| `Delete` / `Backspace` | Delete selected |
| `Ctrl+D` | Duplicate selected |
| `Ctrl+G` | Group selected |
| `Ctrl+Shift+G` | Ungroup |
| `Ctrl+C` / `Ctrl+V` | Copy / Paste |
| `Space + Drag` | Pan canvas |
| `Ctrl+0` | Reset zoom to 100% |
| `Ctrl+Shift+E` | Export dialog |
| `Escape` | Deselect / Cancel |

---

## 5. Responsive Behavior

### Breakpoints

| Breakpoint | Width | Behavior |
|------------|-------|----------|
| **Desktop** | ≥ 1024px | Full layout: sidebar + canvas + right panel |
| **Tablet** | 768–1023px | Collapsible panels, floating toolbar |
| **Mobile** | < 768px | Bottom toolbar, panels as slide-up sheets |

### Desktop Layout
Full 3-column layout as shown in the wireframe.

### Tablet Layout
```
┌────────────────────────────────────────────┐
│  Top Bar (same as desktop)                 │
├────────────────────────────────────────────┤
│                                            │
│  ┌──┐                                     │
│  │🔧│  Floating toolbar (left edge)        │
│  │  │                                     │
│  └──┘        CANVAS                       │
│              (full width)                  │
│                                            │
│                        ┌──┐               │
│                        │👥│ Users FAB      │
│                        └──┘               │
├────────────────────────────────────────────┤
│  Status bar                                │
└────────────────────────────────────────────┘
```

### Mobile Layout
```
┌────────────────────────────────────────────┐
│  ← │ Room Name │ 💾 │ 👥3 │ ⋮             │
├────────────────────────────────────────────┤
│                                            │
│              CANVAS                        │
│              (full viewport)               │
│                                            │
│                                            │
├────────────────────────────────────────────┤
│  ✏️  🖌️  ⬜  ⭕  🔤  🧽  │ 🎨  │ ↩️ ↪️    │
│  (scrollable bottom toolbar)               │
└────────────────────────────────────────────┘
```

**Mobile Interactions:**
- Touch to draw (no hover states)
- Pinch to zoom
- Two-finger pan
- Long-press for context menu
- Panels open as bottom sheets with drag handle

---

## 6. Empty & Loading States

### Dashboard Empty State
```
┌─────────────────────────────────────┐
│                                     │
│          🎨                         │
│   No drawings yet!                  │
│                                     │
│   Create your first drawing         │
│   and start collaborating.          │
│                                     │
│   [+ Create New Room]               │
│                                     │
└─────────────────────────────────────┘
```

### Canvas Loading State
Skeleton pulse animation on canvas area + "Loading canvas..." text with spinner.

### Connection Lost State
Toast notification: "Connection lost. Reconnecting..." with animated dots. Canvas remains interactive (offline mode), changes queue for sync on reconnect.

---

## 7. Animations & Transitions

| Element | Animation | Duration | Easing |
|---------|-----------|----------|--------|
| Page transitions | Fade + slide up | 200ms | ease-out |
| Modal open/close | Scale 0.95→1 + fade | 250ms | cubic-bezier(0.16, 1, 0.3, 1) |
| Sidebar collapse | Width slide | 200ms | ease-in-out |
| Drawing cards hover | Scale 1.02 + shadow elevation | 150ms | ease-out |
| Tool selection | Background color + scale pulse | 100ms | ease-out |
| Remote cursor movement | CSS transform | 100ms | ease-out (linear interpolation) |
| Toast notifications | Slide in from top-right + fade | 300ms | spring-like |
| Button hover | Background brightness shift | 100ms | ease |
| Skeleton loading | Pulse opacity 0.4→1 | 1.5s | ease-in-out, infinite |
